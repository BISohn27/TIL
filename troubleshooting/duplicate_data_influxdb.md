지금 진행하고 있는 프로젝트에서는 인플럭스 DB를 사용하여 통계 데이터를 저장하고 있다. 주로 대시보드와 상세 통계 화면에서 이력 통계를 표시하기 위해 사용하고 있는데, 부하 테스트를 하면서 저장된 데이터를 확인해보니 이력을 넣은 Postgres에서 조회한 데이터 수와 통계를 저장하는 인플럭스 DB  데이터 수가 심각하게 맞지 않는 상황이 발생하고 있는 것이 발견되었다. 

인플럭스 DB쪽 데이터 수가 적게 조회되는 상황이라 처음에는 인플럭스 쪽으로 저장할 때 오류가 발생하는 것으로 판단되어 로그를 분석해보았는데 로그 상에서는 아무런 문제가 없었다. 데이터 저장 시 오류가 발생한 것이 아니라면 대체 무슨 문제일까 이상해서 찾아보니 인플럭스 DB에서 데이터 포인트의 고유성을 판단할 때 사용되는 조건이 문제인 것으로 밝혀졌다.

> InfluxDB identifies unique data points by their measurement, tag set, and timestamp (each a part of [**Line protocol**](https://docs.influxdata.com/influxdb/v2/reference/syntax/line-protocol) used to write data to InfluxDB).
> 

위에는 인플럭스 DB 공식 홈페이지에서 나온 것으로 인플럭스 DB에서는 데이터 포인트의 고유성을 아래의 3개 조합으로 판단하고 있다.

```
web,host=host2,region=us_west firstByte=15.0 1559260800000000000
--- -------------------------                -------------------
 |               |                                    |
Measurement   Tag set                             Timestamp
```

- measurement
- tag set
- timestamp

현재 개발하고 있는 프로젝트에서 부하 테스트 시 tag 조건이 동일한 상황으로 테스트를 하고 있기 때문에 높은 TPS를 테스트하기 위해 부하를 강하게 줄 경우, timestamp까지 겹치는 상황이 빈번하게 발생하면서 중복으로 인식되는 데이터 수가 많아진 것이다. 

이 문제는 TPS가 높아질 경우 데이터 정합성이 더 심하게 맞지 않는 상황이 발생하여 중복을 피하는 방법을 찾을 필요가 있었다.

공식 레퍼런스에서 추천하는 방법은 크게 두가지로 볼 수 있다

- tag에 고유 값 추가

> [**Add an arbitrary tag**](https://docs.influxdata.com/influxdb/v2/write-data/best-practices/duplicate-points/?dl=oss&code_lang=txt&code_lines=5&code_type=code&first_line=web%252Chost%253Dhost2%252Cregion%253Dus_west%2520firstByte%253D15.0%25201559260800000000000#add-an-arbitrary-tag)
> 
> 
> Add an arbitrary tag with unique values so InfluxDB reads the duplicate points as unique.
> 

- timestamp에 나노초 추가

> [**Increment the timestamp**](https://docs.influxdata.com/influxdb/v2/write-data/best-practices/duplicate-points/?dl=oss&code_lang=txt&code_lines=5&code_type=code&first_line=web%252Chost%253Dhost2%252Cregion%253Dus_west%2520firstByte%253D15.0%25201559260800000000000#increment-the-timestamp)
> 
> 
> Increment the timestamp by a nanosecond to enforce the uniqueness of each point.
> 

처음에는 카프카로 이력을 발행하는 쪽에서 나노초를 추가하거나 카프카 이력 메세지를 소비하여 데이터를 저장하는 쪽에서 고유값 UUID를 넣는 방향으로 간단하게 해결하려고 했으나 아래와 같은 문제가 발생하여 다른 방법을 찾아봐야만 했다.

- tag에 UUID와 같은 고유 값을 추가할 경우 tag 조합으로 인한 cardinality 문제
    - tag는 인플럭스에서 인덱스로 사용되기 때문에 모든 tag의 조합 폭발 문제로 [**high series cardinality**](https://docs.influxdata.com/influxdb/v2/write-data/best-practices/resolve-high-cardinality/?utm_source=chatgpt.com#learn-the-causes-of-high-series-cardinality)를 유발하여 메모리 사용 폭증이 발생
    
    > Each unique set of indexed data elements forms a [**series key**](https://docs.influxdata.com/influxdb/v2/reference/glossary/#series-key). [**Tags**](https://docs.influxdata.com/influxdb/v2/reference/glossary/#tag) containing highly variable information like unique IDs, hashes, and random strings lead to a large number of [**series**](https://docs.influxdata.com/influxdb/v2/reference/glossary/#series), also known as high [**series cardinality**](https://docs.influxdata.com/influxdb/v2/reference/glossary/#series-cardinality). High series cardinality is a primary driver of high memory usage for many database workloads.
    > 
    
- timestamp에 나노초를 넣어 모든 이력을 데이터 포인트로 만들 경우 데이터 저장량이 급증하는 문제
    - API Gateway를 개발하는 프로젝트이기 때문에 요청량을 TPS 1,000으로 가정하고 계산을 할 경우
      
        1시간에 3,600,000건 씩 데이터를 저장
        
    - 현재 대시보드에서는 24시간 API 소요 시간에 대한 95 percentile과 99 percentile을 보여주고 있기 때문에 1시간에 3,600,000건 쌓이는 데이터에서 조회 시 정렬해서(인플럭스에서 값에 해당하는 field는 사전 정렬이 안된다고 함) 95 백분율 수와 99 백분율 수를 찾는건 주어진 인플럭스 연결 시간 내에 찾는건 불가능

결국 위의 문제들로 결국 기존 저장 로직을 사전 집계 후 사전 집계 된 데이터를 저장하는 방식으로 변경하기로 결정되었다.