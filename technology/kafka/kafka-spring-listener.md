# 스프링 Kafka 리스너

현재 프로젝트에서는 Spring Kafka를 활용해 메시지를 수신하고 있습니다. 그러나 @KafkaListener를 통한 간접적인 소비 방식 때문에 내부적으로 Kafka Consumer가 어떻게 작동하는지 명확히 파악하기 어렵습니다. 특히 리스너 내부의 비즈니스 로직을 구현할 때 **좋은 설계 기준을 잡기 어려움**을 느껴, Kafka 리스너의 동작 원리를 정리해보았습니다.

## **Kafka 리스너란?**

Kafka 리스너는 Kafka 토픽을 구독하고, 수신한 메시지를 처리하는 **Spring Bean**입니다.

Spring Kafka는 @KafkaListener 어노테이션을 통해 리스너를 등록하고 자동 실행합니다.

## **@KafkaListener 어노테이션**

Kafka 리스너를 선언하는 데 사용되는 핵심 어노테이션입니다. 다양한 설정을 통해 리스너 동작을 제어할 수 있습니다.

### 주요 항목

| 속성               | 설명                                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| `topics`           | 구독할 Kafka **토픽 이름 또는 이름 배열 -** 예: `"my-topic"`, `{ "t1", "t2" }`                         |
| `topicPattern`     | 정규표현식을 사용해 동적으로 **토픽 매칭 -** 예: `"^topic-prefix-.*"`                                  |
| `groupId`          | 이 리스너가 속할 **컨슈머 그룹 ID**(생략하면 `ConsumerFactory`의 기본값 사용)                          |
| `containerFactory` | 사용할 `KafkaListenerContainerFactory` **빈 이름 -** 예: `"kafkaListenerContainerFactory"`             |
| `id`               | 리스너의 **고유 ID** (기본은 메서드 이름 기반으로 생성) 여러 컨테이너를 구분하거나 수동 제어할 때 사용 |
| `concurrency`      | 병렬 컨슈머 **스레드 수** (단, 컨테이너 팩토리에서 이 설정을 오버라이드할 수도 있음)                   |
| `autoStartup`      | 애플리케이션 시작 시 자동으로 리스너를 시작할지 여부 - 기본값은 `true`                                 |
| `properties`       | Consumer 설정을 덮어쓰는 **문자열 맵 -** 예: `{ "max.poll.records=500", "fetch.min.bytes=1024" }`      |
| `errorHandler`     | 리스너 단위로 지정하는 **ErrorHandler 빈 이름** (전역 에러핸들러가 아닌, 이 리스너 전용 에러 처리)     |

**리스너 메서드 시그니처에 따른 MessageListener 분기**

- 단일 메시지 처리

```java
MessageListener<K, V>  // 자동 커밋
void onMessage(ConsumerRecord<K, V> data)
```

```java
AcknowledgingMessageListener<K, V>  // 수동 커밋
void onMessage(ConsumerRecord<K, V> data, Acknowledgment ack)
```

```java
ConsumerAwareMessageListener<K, V>  // Consumer 접근 가능
void onMessage(ConsumerRecord<K, V> data, Consumer<?, ?> consumer)
```

```java
AcknowledgingConsumerAwareMessageListener<K, V> // Consumer 접근 가능, 수동 처리
void onMessage(ConsumerRecord<K, V> data, Acknowledgment ack, Consumer<?, ?> consumer)
```

- 배치 메세지 처리

```java
BatchMessageListener<K, V> // 배치 자동 커밋
void onMessage(List<ConsumerRecord<K, V>> data)
```

```java
BatchAcknowledgingMessageListener<K, V> // 배치 수동 커밋
void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment ack)
```

```java
BatchConsumerAwareMessageListener<K, V> // 배치 Consumer 접근 가능
void onMessage(List<ConsumerRecord<K, V>> data, Consumer<?, ?> consumer)
```

```java
BatchAcknowledgingConsumerAwareMessageListener<K, V> // 배치 수동 커밋, Consumer 접근 가능
void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment ack, Consumer<?, ?> consumer)
```

## **Kafka 리스너 생성 흐름**

Kafka 리스너는 **Spring 컨텍스트 초기화 시 동적으로 생성**됩니다.

### **주요 컴포넌트**

- KafkaListenerAnnotationBeanPostProcessor
- KafkaListenerEndpointRegistrar
- KafkaListenerEndpointRegistry

### **1.** KafkaListenerAnnotationBeanPostProcessor

- BeanPostProcessor, SmartInitializingSingleton 구현체
  - `*BeanPostProcessor` 인터페이스\*
    - `postProcessBeforeInitialization` : 스프링 빈 객체가 만들어지고, 초기화 작업 전 호출
    - `postProcessAfterInitialization` : 스프링 빈 객체가 초기화 된 이후 호출
  - `SmartInitializingSingleton` 인터페이스
    - `afterSingletonsInstantiated` : 일반적인 스프링 빈들이 모두 생성 된 이후에 호출
- `postProcessAfterInitialization` 호출
  → 모든 빈 초기화 후 @KafkaListener 어노테이션이 붙은 메서드를 탐색
- MethodKafkaListenerEndpoint 객체 생성 후 KafkaListenerEndpointRegistrar에 등록

```java
processKafkaListener(listener, method, bean, beanName)
```

- 클래스 수준에서는 @KafkaListener + @KafkaHandler를 조합하여 멀티 메서드 리스너 생성

### 2. KafkaListenerEndpointRegistrar

- 리스너 정의(KafkaListenerEndpoint)를 수집하여 보관
- 스프링 컨테이너가 완전히 초기화된 후 afterPropertiesSet() 호출 시 KafkaListenerEndpointRegistry에 리스너 등록

```java
public void registerEndpoint(KafkaListenerEndpoint endpoint, KafkaListenerContainerFactory<?> factory)
```

### **3. KafkaListenerEndpointRegistry**

- MessageListenerContainer 인스턴스를 생성하고 실행
- 실제 Kafka와 연결되는 컨슈머 스레드를 관리

```java
registerListenerContainer(endpoint, containerFactory)
```

- SmartLifecycle 구현체로, 스프링 시작/중지 시점에 자동으로 컨테이너를 시작/종료

### 전체 리스너 생성 흐름 요약

- 리스너 정의 생성
  - KafkaListenerAnnotationBeanPostProcessor
    → KafkaListenerEndpointRegistrar
- 리스너 컨테이너 생성
  - KafkaListenerAnnotationBeanPostProcessor
    → KafkaListenerEndpointRegistrar
    → KafkaListenerEndpointRegistry
- 리스너 컨테이너 시작

  - KafkaListenerEndpointRegistry

- 흐름
  1. @KafkaListener 어노테이션이 붙은 메서드를 스캔
  2. MethodKafkaListenerEndpoint 객체 생성
  3. KafkaListenerEndpointRegistrar에 등록
  4. afterSingletonsInstantiated() 이후 KafkaListenerEndpointRegistry에 리스너 등록
  5. MessageListenerContainer 생성
     - KafkaListenerContainerFactory를 통해 생성
       - ConcurrentMessageListenerContainer의 경우 ConcurrentKafkaListenerContainerFactory에 의해서 concurrency 설정 만큼 생성
  6. 스프링 라이프 사이클에 의해 MessageListenerContainer 시작
