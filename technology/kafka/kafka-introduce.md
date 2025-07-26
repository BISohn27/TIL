# 카프카의 기본 구성

- 카프카는 데이터를 받아서 전달하는 **데이터 버스(Data Bus)** 역할
- 클라이언트
  - **Producer**: 데이터(메세지)를 만듬
  - **Consumer**: 데이터(메세지)를 소비
- **Zookeeper**: 카프카의 동작을 위한 메타데이터 관리 및 브로커의 상태를 점검하는 **Coordinator**
- **Broker**: 카프카 애플리케이션이 설치 된 서버 또는 노드

## 리플리케이션

- 각 메세지들을 여러 개로 복제하여 카프카 클러스터 내 브로커들에 분산
- `replication-fator`설정을 통해 몇 개의 리플리케이션을 유지할지 설정
- 카프카에서는 토픽 내 파티션이 리플리케이션
- 리플리케이션 팩터 수가 커지면 안정성은 높아지지만 브로커 리소스를 많이 사용하는 트레이드 오프 발생
  - 테스트나 개발 환경: 리플리케이션 팩터 수 1
  - 운영 환경(로그성 메세지로서 약간의 유실 허용): 리플리케이션 팩터 수 2
  - 운영 환경(유실 허용하지 않음): 리플리케이션 팩터 수 3

## 파티션

- 하나의 토픽이 한번에 처리 될 수 있는 한계를 높이기 위해 토픽 하나를 여러개로 나눠 병렬 처리
- 나뉜 파티션 수 만큼 컨슈머를 연결 가능
- 파티션 번호는 0부터 시작하여 늘어날 때마다 1씩 증가
- 적절한 파티션을 선정하는 것은 모호한 측면이 존재
  - 각 메세지 크기나 초당 메세지 건수 등에 따라 달라지기 때문에 정확하게 예측하기 힘듬
- 파티션 수는 초기 생성 후 늘리는 것은 가능하지만 한번 늘어난 파티션 수는 줄일 수 없음
  - 초기 토픽 생성 시 파티션 수를 작게 설정(2 또는 4)하고, 메세지 처리량이나 컨슈머 LAG 등을 모니터링하면서 조금씩 증가 시키는 것이 좋음
- **LAG**: `프로듀서가 보낸 메세지 수 - 컨슈머가 가져간 메세지 수`
  - **프로듀서가 보낸 메세지 수**: `latest offset`, 파티션에 지금까지 쓰여진 가장 마지막 오프셋 + 1
    -> 메세지가 들어올 경우 쓰여질 다음 위치
  - **컨슈머가 가져간 메세지 수**: `committed offset`, 컨슈머 그룹이 마지막으로 커밋한 오프셋 번호
    -> 다음 컨슈머 요청 시 읽을 위치

## 세그먼트

- 프로듀서에 의해 전송된 메세지는 토픽의 파티션에 저장
- 각 메세지는 **세그먼트**라는 로그 파일 형태로 브로커의 로컬 디스크에 저장
  -> 메세지는 파티션 내 여러 세그먼트로 나뉘어서 저장

## 분산 시스템

- 네트워크 상에서 연결된 컴퓨터들의 그룹
- 단일 시스템이 처리하지 못하는 높은 성능 목표
- 하나의 서버 또는 노드 등에 장애가 발생해도 다른 서버 또는 노드가 대신 처리하여 장애 대응 용이
- 부하가 높은 경우 시스템 확장 용이
- 카프카는 브로커를 추가하는 방식으로 시스템 확장 가능

## 페이지 캐시

> Linear reads and writes provide a predictable usage pattern, and are heavily optimized by the operating system. A modern operating system provides read-ahead and write-behind techniques that prefetch data in large block multiples and group smaller logical writes into large physical writes for these linear operations. To learn more, read the ACM Queue article . Of note, the article authors find that sequential disk access can in some cases be faster than random memory access.

- 인메모리 캐시를 활용하는 다른 일반적인 애플리케이션과 달리 카프카는 메세지를 파일에 저장하는 방식을 사용
  - OS는 선형 읽기 및 쓰기 같은 예측 가능한 사용 패턴에 대해 많은 최적화를 진행
    - 큰 블록 단위로 데이터를 미리 읽어오기 (read-ahead)
    - 작은 논리적 쓰기들을 모아서 큰 물리적 쓰기로 묶기 (write-behind)

> Because of the performance divergence between throughput and disk seeks, modern operating systems have become increasingly aggressive in their use of main memory for disk caching. A modern OS will divert all free memory to disk caching with little performance penalty when the memory is reclaimed.

> - Performs linear writes at about 600 MBps.
> - Performs random writes at about 100 Kbps, over 6000 times slower than linear writes

- OS는 디스크 탐색(랜덤 접근)과 순차 처리 성능 간 차이를 극복하기 위해 메인 메모리를 디스크 캐싱을 위해 적극적으로 사용

> All disk reads and writes will go through this unified cache. This feature is hard to turn off without using direct I/O, so even if a process maintains an in-process cache of the data, this data will likely be duplicated in the OS’s page cache, effectively storing everything twice.

- 디스크를 읽고 쓰는 모든 작업은 OS가 사용하는 **통합 캐시(페이지 캐시)** 거쳐서 진행
- 프로세스가 인메모리 캐시를 사용하더라도, 해당 데이터는 OS 페이지 캐시에도 중복 저장되어 있을 가능성이 큼

> Kafka is built on top of the JVM, and Java memory usage has the following characteristics:
>
> - The memory overhead of objects is very high, often doubling the size of the data stored (or worse).
> - Java garbage collection becomes increasingly slow as the in-heap data increases.

> As a result, using the file system and relying on the kernel page cache is better than an in-memory cache or other structure. The page cache provides at least double the available cache by providing automatic access to all free memory, and likely double again by storing a compact byte structure rather than objects.

- 자바는 인메모리로 객체를 저장할 경우 사이즈가 2배로 늘어나는 문제 발생
- 힙 데이터 크기가 늘어날수록 GC 속도도 느려짐
- 위의 이유로, 인메모리 캐시를 사용하는 것보다 파일 시스템을 사용하여 커널 페이지 캐시를 사용하는 것이 유리

> With Kafka, instead of maintaining as much as possible and flushing it to the file system when memory space runs out, all data is immediately written to a persistent log on the file system without flushing to disk. This means the data is essentially transferred into the kernel’s page cache. All logic for maintaining coherency between the cache and file system is in the OS, which tends to do so more efficiently and more correctly than one-off in-process attempts.

- 카프카는 별도의 인메모리 캐시를 활용하지 않고 바로 파일 시스템에 메세지 로그를 쓰는 방법을 사용
- OS는 카프카가 쓴 메세지를 페이지 캐시에 저장
- 캐시와 파일 시스템 간 데이터 정합성은 OS에서 담당하기 때문에 프로세스에서 직접 구현한 캐시보다 효과적이고 정확

→ 카프카는 파일 시스템을 활용하여 OS 페이지 캐시를 적극적으로 활용하는 방식으로 메세지를 효율적으로 저장하고 읽음

## 배치 전송 처리

- 여러 메세지 전송을 하나로 묶어 처리함으로써 네트워크 오버헤드를 줄임

## 압축 전송

- gzip, snappy, lz4, zstd 등 지원
- 압축을 통해 네트워크 대역폭이나 회선 비용을 줄임
- 배치 전송과 결합할 경우 더욱 높은 효과 (압축 파일 개수가 많을 수록 압축 효율이 올라감)
- 높은 압축률: gzip, zstd
- 빠른 응답 속도: lz4, snappy
- 메세지의 형식이나 크기에 따라 결과가 다르기 때문에 테스트 필요

## 토픽, 파티션, 오프셋

- **토픽**: 데이터는 토픽 단위로 저장(메세지 분류 체계)
- **파티션**: 병렬 처리를 위해 토픽은 파티션으로 나뉨
- **오프셋**: 파티션에 메세지가 저장되는 위치 (순차적으로 증가하는 64비트 정수)
- 오프셋은 파티션 마다 관리되며, 파티션 내에서 오프셋 숫자는 고유
- 오프셋을 통해 메세지의 순서를 보장, 컨슈머에서는 마지막까지 읽은 위치 파악

## 고가용성

- 카프카는 **리플리케이션**을 통해 고가용성 보장
- 카프카에서는 **리더**와 **팔로워**로 나뉨

## 주키퍼 의존성

- 분산 애플리케이션에서 코디네이터 역할
- 여러 대의 서버를 앙상블(클러스터)로 구성
- 살아 있는 노드 수가 과반수 이상을 유지한다면 지속적인 서비스 가능
- 과반 투표가 나올 수 있도록 반드시 홀수로 구성
- **znode**를 이용해 카프카의 메타 정보가 주키퍼에 기록
- **znode**를 이용해 브로커의 노드 관리, 토픽 관리, 컨트롤러 관리
- 주키퍼 성능 한계로 인해 주키퍼 의존성을 벗어나고자 **kraft** 개발
