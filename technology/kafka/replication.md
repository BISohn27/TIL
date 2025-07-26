# 리플리케이션

- 분산 시스템에서 애플리케이션의 고가용성을 위해 **리플리케이션**이 중요하지만 구현 시 해결해야 할 문제 존재
  - 리플리케이션의 동작 구현 어려움
  - 애플리케이션의 성능 저하 유발
- `replication-factor`: 파티션이 브로커에 나눠서 저장되는 수

## 리더와 팔로워

- **리더**는 리플리케이션 중 하나가 선정
  - 모든 읽기와 쓰기는 리더를 통해서만 가능  
    → 프로듀서는 리더에게만 메세지를 전송  
    → 컨슈머는 리더로부터 메세지를 가져옴
- **팔로워**는 리더에 문제가 발생하거나 이슈가 있을 경우를 대비
  - 파티션 리더에 새로운 메세지를 받았는지 확인
  - 새로운 메세지가 있으면 해당 메세지를 리더로부터 복제
- **리더**와 **팔로워**는 **ISR(In-Sync-Replica)** 논리 그룹에 속함
  - 해당 그룹에 속한 팔로워들만 새로운 리더가 될 자격을 가짐
  - ISR 그룹에 속하지 못한 팔로워는 리더가 될 자격 없음
  - 지속적으로 리더와의 데이터 일치시킨 팔로워만 ISR 그룹 멤버로 계속 유지되어 새로운 리더가 될 자격 생김
  - 파티션 그룹 리더는 ISR 그룹에 속한 팔로워가 데이터 불일치한 부분이 없는지 계속 감시
    - 팔로워가 특정 주기의 시간만큼 복제 요청을 하고 있는지 감시  
      → 복제 요청이 없으면, 해당 팔로워에 문제가 생겼다고 간주하고 ISR 그룹에서 제외
- ISR 그룹 내 모든 팔로워의 복제가 끝나면, 리더는 내부적으로 커밋되었다는 표시
- **하이워터마크**: 마지막 커밋 오프셋 위치
- 커밋되었다는 것은 리플리케이션 팩터 수만큼의 리플리케이션이 완료되었다는 의미
- 데이터 일관성을 위해 커밋된 메세지만 컨슈머는 소비
- 커밋되지 않은 메세지를 읽을 때의 문제:
  1. 리플리케이션이 동작하기 전에 리더가 죽음
  2. 새로운 리더가 선출
  3. 새로운 리더는 구 리더의 커밋되지 않은 메세지가 없을 수 있음  
     -> 리더가 죽기 전 메세지를 읽어간 컨슈머와 리더가 죽은 후 메세지를 읽어간 컨슈머 간 데이터 정합성 문제 발생
- **replication-offset-checkpoint** 파일

  - 모든 브로커가 재시작될 때, 마지막 커밋 오프셋 위치를 저장

    | Topic       | Partition | Replicated Offset |
    | ----------- | --------- | ----------------- |
    | user-events | 0         | 1050012           |
    | user-events | 1         | 1039987           |
    | logs        | 0         | 250000            |
    | logs        | 1         | 249980            |

  - 특정 토픽이나 파티션에 복제가 되지 않는 등의 문제가 발생 -> 모든 브로커의 체크포인트 파일을 확인하여 판단

- 리더와 팔로워의 리플리케이션 동작
  - 모든 읽기와 쓰기를 담당하는 리더  
    -> 리플리케이션을 위해 팔로워와 많은 통신을 추가로 주고 받아야 하면 성능 문제 발생
  - 카프카는 리더와 팔로워 간 통신을 최소화하는 방식으로 리더의 추가적인 부하를 줄임
    1. 프로듀서가 리더에게 새로운 메세지 전송
    2. 팔로워는 리더에게 오프셋 메세지 fetch 요청을 보내 새로운 메세지가 전송되었는지 확인  
       (리더는 팔로워의 fetch 요청을 통해 팔로워 상태 인지 가능)
    3. 리더는 팔로워에게 새로운 받은 메세지를 전달했다는 사실만 인지  
       -> 실제 리플리케이션 성공 여부 판단하지 않음 (RabbitMQ는 ack 메세지를 통해 성공 여부 알림)
    4. 팔로워의 메세지 fetch 요청을 주기적으로 받으며, 팔로워의 리플리케이션 상태를 확인  
       4-1. 모든 팔로워가 요청하는 오프셋(리플리케이션 완료된 오프셋 +1)이 **현재 하이워터마크**보다 크면 요청 오프셋 중 가장 작은 값에 해당하는 메세지의 오프셋을 커밋 표시  
       4-2. **하이워터마크**를 증가 시킴  
       4-3. 리플리케이션 요청에 대한 응답으로 커밋된 이전 메세지 오프셋을 전달
       4-4. 팔로워는 리더의 응답 메세지로 이전 메세지가 커밋되었음을 표시  
       (팔로워 요청 오프셋이 새로 들어온 메세지들의 오프셋보다 작으면 해당 팔로워의 리플리케이션이 실패했다고 인지)
  - 카프카는 리플리케이션 ack 메세지를 제거함으로써 성능을 높임
  - 팔로워가 리플리케이션을 pull 함으로써 추가적인 리더의 부하를 줄임

## 리더에포크

- 카프카의 파티션들이 복구 동작을 할 때 메세지의 일관성을 유지하기 위한 용도
- 컨트롤러에 의해 관리되는 32비트 숫자
- 리더에포크 정보는 리플리케이션 프로토콜에 의해 전파
- 새로운 리더가 변경된 후 변경된 리더에 대한 정보가 팔로워에게 전달
- 복구 동작 시 하이워터마크를 대체하는 수단

- 리더가 팔로워에게 하이워터마크 증가 응답 메세지를 주기 전에 죽는 경우

  - 메세지 1을 받아 오프셋 1이 됨
    | type | 오프셋 | 하이워터마크 | message |
    |-------------|-----------|-------------------|------|
    | 리더 | 1 | 1 | message1 |
    | 팔로워 | 1 | 1 | message1 |
  - 메세지 2를 받아 오프셋 2가 되고, 팔로워로부터 요청을 받아 하이워터마크 증가
    | type | 오프셋 | 하이워터마크 | message |
    |-------------|-----------|-------------------|------|
    | 리더 | 2 | 2 | message2 |
    | 팔로워 | 2 | 1 | message2 |
  - 리더에포크 없는 경우
    - 리더가 요청을 응답하는 과정에서 다운될 경우 팔로워는 오프셋 1번 메세지를 받지 못한 상태에서 리더로 승격
    - 팔로워는 자신이 가지고 있는 메세지 중 리더한테 받은 하이워터마크보다 오프셋이 높은 메세지들은 신뢰할 수 없다고 판단하고 지움
    - 새로운 리더는 오프셋 0번 메세지만 가지고 있기 때문에 오프셋 1번 메세지는 유실
  - 리더에포크 사용하는 경우
    1. 팔로워는 복구 과정에서 리더에게 리더에포크 요청
    2. 리더는 리더에포크 응답으로 '1번 오프셋까지 유효'한 것을 알림
    3. 팔로워는 자신의 하이워터마크보다 높은 1번 오프셋의 메세지를 삭제하지 않고, 하이워터마크를 증가시켜 리더와 일치 시킴
    4. 이 과정에서 리더가 죽더라도 팔로워는 모든 메세지를 보존한 채 새로운 리더로 승격

- 새로 들어온 메세지를 리플리케이션 하지 못한 상태에서 리더와 팔로워가 모두 죽음, 그 이후 팔로워만 복구 (리더에포크 없음)

  - 메세지 1을 받아 오프셋 1이 됨
    | type | 오프셋 | 하이워터마크 | message |
    |-------------|-----------|-------------------|------|
    | 리더 | 2 | 2 | message2 |
    | 팔로워 | 1 | 1 | message1 |
  - 팔로워 복구 후 새로운 리더가 되어 메세지를 신규로 받음
    | type | 오프셋 | 하이워터마크 | message |
    |-------------|-----------|-------------------|------|
    | 리더(팔로워) | 2 | 2 | message2 |
    | 팔로워(새로운 리더) | 2 | 2 | message3 |
  - 기존 리더가 복구 후 새로운 리더 하이워터마크와 비교 시 동일한 오프셋을 가지고 있기 때문에 메세지 삭제 없이 리플리케이션 시작
  - 새로운 메세지 message4가 들어올 경우, 새로운 리더와 구 리더 모두 오프셋 3, 하이워터마크 3으로 변경
  - 하지만 오프셋 2번 자리에 메세지는 새로운 리더는 message3, 구 리더는 message2으로 데이터 정합성이 맞지 않는 상태
  - 리더에포크에는 자신이 팔로워일 때의 하이워터마크와 뉴리더일 때의 하이워터마크를 알고 있음
    - 구 리더가 장애에서 복구되는 경우, 새로운 리더의 리더에포크를 받아 자신의 오프셋 1번이 정합성 문제가 있다는 것을 인지하고, 오프셋 1번 메세지를 삭제

- 리더에포크는 새로운 리더가 선출될 때 변경된 정보가 업데이트

## Replication Protocol

- Leader Epoch
  ![](https://images.ctfassets.net/gt6dp23g0g38/1rHc9oqwn8DD94JIQckrXL/a36399e2acc2437810ddaa8a568307ca/Kafka_Internals_031.png)

  - 특정 Replica가 리더일 때 어떤 일이 일어났는지 추적하기 위해 사용
  - 새로운 리더가 선출될 때마다 1씩 증가
  - **log reconciliation**에 중요한 역할

- Fetch Request
  ![](https://images.ctfassets.net/gt6dp23g0g38/QMNcHw9rAoiFGXj4DnP9I/51ef0ec74b91b1f01a88f8fa3934b2f0/Kafka_Internals_032.png)

  - 리더가 로컬 로그 세그먼트에 새로운 메세지를 추가
  - 팔로워는 Fetch Request에 fetch 받아야할 offset을 포함

- Follower Fetch Response
  ![](https://images.ctfassets.net/gt6dp23g0g38/7kr6K36N4VF4D5F3gpY71h/10329b5e4b700afdb1aa28a46432fd44/Kafka_Internals_033.png)

  - 리더는 Fetch 요청에 대한 응답을 반환
  - 응답에는 팔로워 요청에 포함되었던 offset 이후 레코드들이 포함
  - 각 레코드들의 offset과 현재 리더의 Leader epoch도 포함

- Committing Partition Offsets
  ![](https://images.ctfassets.net/gt6dp23g0g38/7ADIKF2poAYD0iE1p1hJNF/eff71842eed8637f50d888e27f962343/Kafka_Internals_034.png)

  - ISR 그룹에 속한 모든 팔로워들이 특정 offset 레코드를 전부 fetch할 경우, 해당 offset 레코드는 commit되어 컨슈머가 소비할 수 있는 상태로 변경
  - **하이워터마크**: 커밋되어 컨슈머가 소비할 수 있는 상태의 메세지 오프셋
  - 리더는 팔로워의 Fetch Request에 포함된 offset을 통해 해당 팔로워가 어디까지 리플리케이션 했는지 판단  
    예) 팔로워가 offset 3을 요청으로 보낼 경우, 해당 팔로워는 offset 3 이전 레코드는 전부 리플리케이션 되었다고 판단 가능
  - 모든 팔로워가 특정 offset을 요청에 포함해서 보낼 경우 해당 offset으로 **하이워터마크**을 변경

- Advancing the Follower High Watermark
  ![](https://images.ctfassets.net/gt6dp23g0g38/2GtWQTnR5GwuDaUltAxEHM/50dd8e261231d98af4dcae5fc57bc41e/Kafka_Internals_035.png)

  - 리더는 Fetch Response에 **하이워터마크**를 함께 보내 현재 메세지 커밋 상태를 팔로워에게 알림
  - 리더의 **하이워터마크**는 Fetch Response에 의해서만 팔로워에게 전달
  - 실제 리더의 **하이워터마크**는 팔로워가 알고 있는 **하이워터마크**보다 클 가능성이 항상 존재 (팔로워는 Fetch Response를 받을 시점의 값만 비동기적으로 확인 가능하기 때문)

- Handling Leader Failure
  ![](https://images.ctfassets.net/gt6dp23g0g38/4gmUY2HRzEEgtWX4aYO5RK/92c1cd987a80a083e0903ab21bb7a6e6/Kafka_Internals_036.png)

  - 리더가 장애로 리더 역할을 하지 못할 경우 새로운 리더 선출
  - 리더 선출 및 선출된 리더를 팔로워들에게 알리는 역할은 control plane에서 담당
  - data plane에서는 기존 리더 장애로 새로운 리더 선출 과정에서 발생하는 데이터 유실이 가장 큰 문제
  - 이를 위해 리더는 ISR 그룹에서 선출
    → ISR 내 팔로워는 가장 마지막에 커밋된 offset 메세지를 보관하고 있는 것을 보장

- Temporary Decreased High Watermark
  ![](https://images.ctfassets.net/gt6dp23g0g38/Kr5GipOTKKo9KojdSmREF/a230a16ec35d8887e91cff2ddeb29e00/Kafka_Internals_037.png)

  - **하이워터마크**는 비동기적으로 Fetch Request을 통해서만 팔로워에게 업데이트되기 때문에 팔로워의 업데이트 요청 상태에 따라 팔로워 마다 다를 수 있음
    - 기존 리더로부터 최신 **하이워터마크**를 전달 받은 팔로워 존재
    - 기존 리더로부터 최신 **하이워터마크**를 전달 받지 못한 팔로워 존재
  - 새로운 리더는 장애가 발생한 구 리더의 최신 **하이워터마크**를 업데이트 받지 못했을 가능성이 존재
    - ISR 그룹은 가장 마지막으로 **커밋된 오프셋 메세지**를 모든 팔로워가 들고 있음을 보장
    - 구 리더의 마지막 **하이워터마크**가 모든 팔로워에게 업데이트 되었음을 보장하지는 않음 (**하이워터마크**는 eventual consistency)
  - 구 리더의 마지막 **하이워터마크** 값을 가진 팔로워의 요청에 새로운 리더는 응답 불가
  - `새로운 리더 하이워터마크 ~ 최신 하이워터마크`의 요청에 대해서는 `OFFSET_NOT_AVAILABLE` 에러 발생
  - 새로운 리더가 **하이워터마크**를 가장 마지막으로 커밋된 오프셋으로 업데이트할 때까지 팔로워는 계속 fetch 시도

- Partition Replica Reconciliation
  ![](https://images.ctfassets.net/gt6dp23g0g38/1MCX2GxiBgktyO7kPgSBGu/f0e8800cd5c4d66ec23243795b5597f5/Kafka_Internals_038.png)

  - 리더로 선출된 직후, 특정 팔로워는 구 리더의 커밋되지 않은 레코드를 가지고 있을 수 있음 (팔로워 마다 Fetch Request를 보낸 정도가 다름)
    → 새로운 리더의 **하이워터마크**가 다른 팔로워보다 작은 이유
  - **하이워터마크** 불일치 문제가 발생할 경우, 새로운 리더는 기존 리더에게서 리플리케이션 된 offset을 모든 팔로워들로부터 요청을 통해 받아야만 **하이워터마크**를 갱신 가능
  - 싱크가 맞지 않는 offset 요청을 받을 경우 **replica reconciliation** 프로세스를 실행하여 동기화 진행

- Fetch Response Informs Follower of Divergence
  ![](https://images.ctfassets.net/gt6dp23g0g38/1MCX2GxiBgktyO7kPgSBGu/1087c6850f64d0e30a86f97f8ac6f77a/Kafka_Internals_039.png)

  - 현재 리더(epoch)의 offset과 일치하지 않는 요청을 받으면 현재 리더의 로그 데이터와 비교하여 요청한 offset이 유효하지 않은지 판단
  - 만약 유효하지 않다고 판단되면, 유효한 가장 마지막 offset의 값을 반환하여 팔로워가 스스로 클린업을 진행하도록 함

- Follower Truncates Log to Match Leader Log
  ![](https://images.ctfassets.net/gt6dp23g0g38/2SLCk6ccSIlkvPjrKq2YCi/492556be995fa402bd06e20cc24a6964/Kafka_Internals_040.png)

  - 팔로워는 응답 결과를 토대로 새로운 리더와 싱크를 맞추기 위해 일치하지 않는 데이터를 truncate

- Subsequent Fetch with Updated Offset and Epoch
  ![](https://images.ctfassets.net/gt6dp23g0g38/aYcacWtuT1gnS6RCQRxaa/5b2760f264a6fd99b29c16a03d030b03/Kafka_Internals_041.png)

  - 팔로워는 새로운 offset으로 Fetch Request를 보냄으로써 불일치 문제 해결

- Follower 102 Reconciled & Follower 102 Acknowledges New Records
  ![](https://images.ctfassets.net/gt6dp23g0g38/5rtqIUVT29SGB5vharEdcE/ed09090d7e99d57752e001dfc8758d56/Kafka_Internals_042.png)
  ![](https://images.ctfassets.net/gt6dp23g0g38/10rKaHrv3sJxZ9CxmYwL9w/b3a8d28f7b99a4b8cee79fe9a1b7697e/Kafka_Internals_043.png)

  - 팔로워는 새로운 레코드를 현재 리더의 epoch로 저장
  - 팔로워의 Fetch Request로 **하이워터마크**를 증가시키면서 정상 동작

- Follower 101 Rejoins the Cluster
  ![](https://images.ctfassets.net/gt6dp23g0g38/slWIzdKdFUuiEwK1BAG4E/cd88e602396e4bcdacad99487ea4387f/Kafka_Internals_044.png)

  - 기존 리더가 다시 복구될 경우, 기존 리더는 **replica reconciliation** 과정을 통해 새로운 리더를 리플리케이션하고, 모든 레코드가 일치 할 때 ISR에 복귀
    - 자신의 리더에포크와 새로운 리더의 리더에포크 비교
    - 일치하지 않은 데이터가 있을 경우 일치하는 시점까지 truncate
    - 새로운 리더의 레코드 복제

- Handling Failed or Slow Followers
  ![](https://images.ctfassets.net/gt6dp23g0g38/1T8pQOyW8YcUj64phgAYW5/c550c246504f246faab3f172f2f073a0/Kafka_Internals_045.png)
  - 팔로워의 리플리케이션의 속도가 느려 설정된 임계 값 시간 이상 팔로워한테 리플리케이션 요청이 오지 않을 경우, 해당 팔로워를 ISR 그룹에서 제거
  - 리플리케이션 속도가 느린 팔로워를 제거함으로써 리더는 리플리케이션을 정상적으로 진행하는 팔로워들에게 메세지를 계속 복제 후 **하이워터마크**를 증가시키며 컨슈머가 메세지를 소비할 수 있도록 함
  - 팔로워가 다시 정상 동작하여, 리더의 레코드를 전부 리플리케이션으로 따라왔을 때 다시 ISR 그룹에 합류시킴
