## PostgreSQL Count(\*) 성능 저하 트러블슈팅 리포트

## 1. 배경 및 문제 정의

- 대용량 트랜잭션 로그 테이블(`api_logging`)에서 `count(*)`, 페이징 등 aggregate 쿼리 성능 저하 발생
- TPS 1,000 이상의 실시간 대용량 insert 환경에서,
  - 동일 쿼리가 때때로 수십 초~수 분 이상 소요됨
- 설계/운영 구조 상 원인 파악 및 근본적 해결 방안이 필요

---

## 2. 주요 증상 및 쿼리

- **쿼리 예시**

  ```sql
  SELECT count(log.obid)
  FROM api_logging log
  WHERE log.start_date BETWEEN '2025-07-11 00:00:00' AND '2025-07-11 23:59:59'
  ```

- EXPLAIN ANALYZE (부하 테스트 (TPS 1000) 시)

  ```sql
  Finalize Aggregate (cost=1997083.38..1997083.39 rows=1 width=8) (actual time=15847.157..15847.953 rows=1 loops=1)
   -> Gather (cost=1997083.16..1997083.37 rows=2 width=8) (actual time=15847.103..15847.946 rows=3 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   -> Partial Aggregate (cost=1996083.16..1996083.17 rows=1 width=8) (actual time=15842.544..15842.545 rows=1 loops=3)
   -> Parallel Index Scan using idx_api_logging2 on api_logging log (cost=0.56..1977210.70 rows=7548986 width=37) (actual time=0.043..15230.803 rows=6448696 loops=3)
   Index Cond: ((start_date >= '2025-07-11 00:00:00'::timestamp without time zone) AND (start_date <= '2025-07-11 23:59:59'::timestamp without time zone))
  Planning Time: 0.152 ms
  Execution Time: 15848.017 ms
  ```

- **현상 요약**:

  - 쿼리 실행시간이 15~40초 이상 소요

  - 인덱스 스캔이지만 Index Only Scan으로 최적화되지 않음

## 3. 문제 가설 및 원인 후보

- ① **MVCC + Visibility Map 미적용:**

  - 실시간 대량 insert 환경에서 autovacuum이 즉시 동작하지 않아 visibility map이 all-visible 상태가 아님

    → 인덱스만 보고 집계할 수 없으므로 heap까지 row를 검증, 추가 I/O 소모

- ② **PK(UUID) 구조 영향:**

  - UUID 랜덤 값 사용 시, row가 테이블 전체에 고르게 분산

  - → heap fetch가 전역적으로 발생, Index Only Scan 효율 저하 가능성

## 4. 실험 설계 및 수행

### 1) **부하 중단 후 vacuum/쿼리 실행 실험**

1. TPS 중단 (INSERT/UPDATE/DELETE 등 write 부하 제거)

2. `VACUUM (ANALYZE, VERBOSE) api_logging` 즉시 실행

3. 동일 count 쿼리/실행계획 분석

- **실험 결과:**

  - `Heap Fetches: 0`

  - `Parallel Index Only Scan`으로 집계 완료

  - Execution Time: 1.8초 (기존 대비 10배 이상 개선)

  - → **Visibility Map이 최신 상태면 heap fetch가 완전히 제거됨 확인**

### 2) **VACUUM 로그 기록**

```sql
vacuuming "public.api_logging"
index "api_logging_pkey" now contains 31755826 row versions in 297656 pages
...
"api_logging": found 0 removable, 31755826 nonremovable row versions in 2206889 out of 2206889 pages
...
analyzing "public.api_logging"
"api_logging": scanned 30000 of 2206889 pages, containing 431240 live rows and 0 dead rows; 30000 rows in sample, 31723294 estimated total rows
```

- 부하 중단 및 VACUUM 실행 이후,  
  전체 2,206,889개 페이지가 모두 “dead row 없음”, "all-visible" 상태로 정리됨

- 이 상태에서 실행한 count 쿼리의 실행계획 및 성능 변화:

```sql
Finalize Aggregate  (cost=965232.44..965232.45 rows=1 width=8) (actual time=1882.309..1882.820 rows=1 loops=1)
  ->  Gather  (cost=965232.23..965232.44 rows=2 width=8) (actual time=1882.026..1882.801 rows=3 loops=1)
        Workers Planned: 2
        Workers Launched: 2
        ->  Partial Aggregate  (cost=964232.23..964232.24 rows=1 width=8) (actual time=1876.448..1876.449 rows=1 loops=3)
              ->  Parallel Index Only Scan using idx_api_logging3 on api_logging log  (cost=0.56..943548.41 rows=8273529 width=37) (actual time=0.080..1518.817 rows=6605954 loops=3)
                    Index Cond: ((start_date >= '2025-07-11 00:00:00'::timestamp without time zone) AND (start_date <= '2025-07-11 23:59:59'::timestamp without time zone))
                    Heap Fetches: 0
Planning Time: 0.497 ms
Execution Time: 1882.882 ms
```

#### **해석**

- **Execution Time: 1.8초**  
  → VACUUM 전(15~40초)과 비교해 **최대 20배 이상 성능 개선**  
  → Index Only Scan이 이론적으로 최대의 성능을 내는 상태

- 실행계획상 `Parallel Index Scan using idx_api_logging2` -> `Parallel Index Only Scan` 변경
  → 실제로 인덱스만 접근해서 집계가 이뤄졌음을 확인할 수 있고, visibility map이 큰 문제라는 것이 검증

## 5. 추가 가설: PK(UUID) vs Sequence

- 랜덤 UUID는 Vacuum/Visibility Map에 불리한 구조일 수 있음

- 실험 결과 visibility map만 잘 형성되면 UUID 사용도 성능상 큰 문제 없음

- 참고 문서:

  - [PostgreSQL Visibility Map 공식 문서](https://www.postgresql.org/docs/current/storage-vm.html)

## 6. Auto Vacuum 동작하지 않는 원인

### vacuum 실행 이력 확인

```sql
-- vacuum 관련 통계 쿼리
SELECT
  relname,
  autovacuum_count,
  vacuum_count,
  analyze_count,
  autoanalyze_count
FROM
  pg_stat_all_tables
WHERE
  relname = 'api_tx_logging';
```

---

| 테이블명    | autovacuum_count | vacuum_count | analyze_count | autoanalyze_count |
| ----------- | ---------------- | ------------ | ------------- | ----------------- |
| api_logging | 0                | 1            | 1             | 137               |

---

→ autovacuum_count = 0 → 자동 vacuum이 전혀 수행되지 않았음을 확인

### AutoVacuum 조건 분석

- `vacuum threshold = autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * number of Tuples`

- 기본값: autovacuum_vacuum_threshold = 50, scale_factor = 0.2 (20%)

- INSERT-only 테이블은 n_dead_tup이 거의 없어 trigger를 만족시키지 못함
- 그 결과 visibility map이 누락되고 Index Only Scan이 불가능

## 7. 결론 및 개선 방안

### **주요 원인**

- insert-only 테이블에서는 PostgreSQL의 autovacuum 트리거 조건을 충족하지 않아 visibility map이 생성되지 않음
- 이로 인해 Index Only Scan이 동작하지 않고, 불필요한 heap fetch가 발생
- 성능 문제를 해결하려면:
  - (A) autovacuum 설정을 조정하거나 주기적 manual vacuum을 고려
  - (B) 설계를 변경하여 `count(*)`나 페이지 번호 기반 쿼리 의존도를 줄이는 방향 모색

### **개선 방향**

- 대규모 write 환경에서 count(\*)를 제거하거나 페이징 구조를 변경

- 주기적으로 manual VACUUM을 수행하거나 autovacuum 설정을 조정  
   → insert-only 테이블의 경우, 튜플 변경(UPDATE/DELETE)이 없어 기본 autovacuum 조건으로는 VACUUM이 트리거되지 않음  
   → 이를 해결하기 위해 다음과 같이 설정을 조정할 수 있음:
  - autovacuum_vacuum_scale_factor를 0으로 설정  
    → 튜플 변경 비율 조건을 제거하고, threshold 기준만 적용되도록 함
  - autovacuum_vacuum_threshold를 원하는 트리거 기준 row 수로 설정  
    → 예: 1,000,000으로 설정하면 INSERT 100만 건마다 autovacuum 실행
  - 출처 : https://techblog.woowahan.com/9478/
- 페이징 개선 설계는 별도 문서 참고 → API-로그-페이징-구조-개선.md
