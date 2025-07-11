# [설계 검토/검증 문서] API 로그 페이징 구조 개선

> 본 문서는 `PostgreSQL Count(*) 성능 저하 트러블슈팅 리포트`와 연계됩니다.  
> 상세 분석 내용은 `API-로그-count-트러블슈팅.md` 문서를 참고하세요.

## 1. 배경 및 문제 정의

- 기존 시스템에서는 `count(*)` 기반으로 전체 페이지 수를 계산해 페이징 UI에 표시하고 있었음

- 하루 1,000만 건 이상 쌓이는 `api_logging` 테이블에서 `count(*)`의 수행 시간이 수십 초~수 분에 달해 성능 문제가 발생함

## 2. 기존 방식의 문제점

- 실시간 insert가 많은 구조에서 MVCC 특성상 `count(*)` 수행 비용이 매우 큼
- offset이 커질수록 성능 저하 우려

## 3. 요구사항

- 전체 count 없이도 자연스러운 페이징 제공
- 사용자 입장에서 UX 이질감 최소화
- 시스템 부하 최소화 및 확장성 확보

## 4. 고려한 대안

| 대안                    | 설명                      | 장점                  | 단점                     |
| --------------------- | ----------------------- | ------------------- | ---------------------- |
| A. 기존 count 유지        | 기존과 동일                  | 사용자에게 익숙            | 성능 병목, 확장 불가           |
| B. summary 테이블        | 사전 집계 테이블 활용            | 조회 성능 우수            | 동적 필터 적용 불가            |
| C. 커서 기반 페이징          | `start_date < ?` 기준 페이징 | 성능 안정적              | UX상 페이지 번호 표현 어려움      |
| **D. hasNext 기반 페이징** | **OFFSET + LIMIT + 1**  | **UX 유지 가능, 구현 간단** | OFFSET 커질 경우 성능 저하 가능성 |

## 5. 선택한 방식: D. hasNext 기반 페이징

### 쿼리 구조

```sql
SELECT 1
FROM api_logging
WHERE start_date BETWEEN :start AND :end
ORDER BY start_date DESC
OFFSET :page_size * :page_group
LIMIT 1
```

### UI 설계

- 페이징은 10개 단위 묶음으로 표시

- `다음 10페이지` 버튼만 제공, 전체 페이지 수는 표시하지 않음

- 마지막 페이지 점프 기능 제거

## 6. 트레이드오프 수용

- 전체 페이지 수 노출은 제거함  
  → 전체 카운트를 위한 쿼리 성능 비용이 **사용자 경험의 이점을 상쇄**한다고 판단됨

- 응답 시간이 페이징 위치에 따라 점진적으로 증가할 수 있음  
  → 장기적으로는 커서 기반 페이징으로의 전환 가능성을 열어둠

- 트레이드오프를 고려하여 **대규모 write 환경에서도 UX 저하 없이 동작 가능한 설계로 변경**함

## 7. 결론

> 본 쿼리는 단순 카운팅 용도로 사용하기엔 성능 비용이 크며,  
> **페이징 목적 등에서 사용 시 전체 구조 변경 또는 우회 전략이 필요**함.  
> 따라서 `count(*)`를 직접 사용하는 구조는 대용량 테이블에서는 제거하는 방향이 바람직하다고 판단됨.

---

## [부록] 실측 기반 성능 실험

### 1. 실험 환경 (기존 전체 조회 쿼리)

- 테이블: `api_logging` (TPS 1,000 기준으로 하루 1,500만 건 이상 삽입)

- 필터 조건: `start_date BETWEEN '2025-07-11 00:00:00' AND '2025-07-11 23:59:59'`

- 인덱스 구성: `(start_date, obid)` 복합 인덱스

- 테스트 쿼리:
  
  ```sql
  SELECT count(log.obid)
  FROM api_logging log
  WHERE log.start_date BETWEEN '2025-07-11 00:00:00' AND '2025-07-11 23:59:59';
  ```

### 측정 결과

> Finalize Aggregate  (cost=1561426.13..1561426.14 rows=1 width=8)
>   ->  Gather  (cost=1561425.92..1561426.13 rows=2 width=8)
>         Workers Planned: 2
>         Workers Launched: 2
>         ->  Partial Aggregate  (cost=1560425.92..1560425.93 rows=1 width=8)
>               ->  Parallel Index Scan using idx_api_logging2 on api_logging
>                     Index Cond: ((start_date >= '2025-07-11 00:00:00') AND (start_date <= '2025-07-11 23:59:59'))
> Planning Time: 8.448 ms
> Execution Time: 41496.809 ms

| 항목      | 측정값                                    |
| ------- | -------------------------------------- |
| 총 대상 건수 | 약 1,000만 건                             |
| 총 실행 시간 | 41.5초                                  |
| 병렬 수행   | 2 workers 사용 (partial aggregate)       |
| 사용 인덱스  | `(start_date, obid)` 복합 인덱스            |
| 쿼리 특성   | Heap Fetch 발생으로 Index Only Scan 완전 실패  |
| 병목 원인   | 대량 row visibility 확인 → heap scan 비용 과다 |

### 2. 실험 환경 (트레이드 오프 반영한 쿼리)

- 데이터 수: 하루 약 10,000,000건 이상

- 필터: `start_date BETWEEN '2025-07-11 00:00:00' AND '2025-07-11 23:59:59'`

- 인덱스: `(start_date, obid)` 복합 인덱스 사용

- 테스트 기준 쿼리:
  
  ```sql
  SELECT 1
  FROM api_logging
  WHERE start_date BETWEEN :start AND :end
  ORDER BY start_date DESC
  OFFSET :offset
  LIMIT 1;
  ```

### 측정 결과

> Limit  (cost=1169868.33..1169868.45 rows=1 width=12) (actual time=8203.287..8203.288 rows=1 loops=1)
>   ->  Index Only Scan using idx_api_logging2 on api_logging  (cost=0.56..1785709.18 rows=15264192 width=12) (actual time=0.075..7875.851 rows=10000001 loops=1)
>         Index Cond: ((start_date >= '2025-07-11 00:00:00'::timestamp without time zone) AND (start_date <= '2025-07-11 23:59:59'::timestamp without time zone))
>         Heap Fetches: 10000104
> Planning Time: 0.219 ms
> Execution Time: 8203.321 ms

| OFFSET 값   | 목적               | Execution Time | Heap Fetches | 판단                                          |
| ---------- | ---------------- | -------------- | ------------ | ------------------------------------------- |
| 1,000,000  | 100,000번째 페이지 진입 | 686ms          | 1,000,111    | 양호                                          |
| 4,000,000  | 극단적인 케이스         | 2,842ms        | 4,000,001    | 수용 가능                                       |
| 10,000,000 | 극단적인 케이스         | 8,203ms        | 10,000,104   | 실제 사용에서는 드물지만, 테스트 결과 성능적으로 수용 가능한 수준으로 확인됨 |
