# Tomcat NIO Connector

## Tomcat NIO Connector는 과연 Non-Blocking일까?

Spring MVC를 사용할 때, 흔히 “톰캣은 블로킹 서버”라는 이야기를 자주 듣는다.  
하지만 분명 톰캣 8부터는 NIO Connector가 기본 설정이고, 톰캣 9에서는 기존 BIO(Blocking I/O) 커넥터가 deprecated되었다.  
그렇다면 톰캣은 이제 논블로킹 방식으로 동작하는 게 맞는 것 아닐까?  
이 질문에서 출발해 톰캣과 Spring MVC의 요청 처리 방식을 정리해보았다.

## BIO Connector – 완전한 블로킹 모델

BIO(Blocking I/O) 커넥터는 전통적인 요청 처리 모델이다.

- 하나의 커넥션마다 하나의 스레드가 할당된다.
- 예를 들어, 100명의 사용자가 동시에 접속하면 100개의 스레드가 필요하다.
- 커넥션은 유지되지만 실제 요청이 적거나 느릴 경우, 스레드는 유휴 상태로 리소스를 낭비하게 된다.
- 요청이 타임아웃 되기 전까지 스레드는 계속 점유된다.
- 이러한 비효율성 때문에 Tomcat 9부터는 BIO가 deprecated되었다.

## NIO Connector – 네트워크 I/O는 논블로킹

Tomcat의 NIO 커넥터는 BIO의 한계를 개선한 구조로, 내부적으로 Java NIO 기반의 Selector 모델을 사용한다.

- 여러 개의 Poller Thread가 다수의 연결을 감시한다.
- 클라이언트로부터 요청 데이터가 들어오면, 논블로킹 방식으로 패킷을 수신하고, 완전한 HTTP 요청이 조립되었을 때 처리 대상으로 넘긴다.
- 실제 비즈니스 로직은 Worker Thread가 맡는다. 이때부터는 전통적인 스레드-퍼-요청 방식으로 처리된다.

즉, Tomcat NIO는 소켓 연결과 데이터 수신까지는 논블로킹으로 처리하지만, 요청이 완전히 수신되어 서블릿으로 넘어가는 시점부터는 블로킹 처리 방식으로 전환된다.

논블로킹인 부분

- 소켓 수신 및 읽기 처리 단계: NIO 기반 **Poller**가 비동기적으로 다수의 연결을 처리

블로킹인 부분

- 요청 처리 단계 (서블릿 이후): 완전한 요청이 도착한 후, **Worker Thread**가 요청 하나를 처리하면서 스레드를 전부 점유

따라서 “톰캣이 논블로킹이다"라는 말은 소켓 I/O 수준에서만 유효하다. 비즈니스 로직 실행 단계는 여전히 전통적인 블로킹 모델으로 볼 수 있다.
이는 Connector 설정에서 maxThreads(**Worker Thread** 최대 개수) 설정에 대한 공식 레퍼런스에서도 확인 할 수 있다.

> Tomcat 공식 문서:  
> Each incoming, non-asynchronous request requires a thread for the duration of that request. If more simultaneous requests are received than can be handled by the currently available request processing threads, additional threads will be created up to the configured maximum (the value of the maxThreads attribute).

## 블로킹 프레임워크 Spring MVC

Spring MVC는 Servlet API 기반의 프레임워크다.  
서블릿 컨테이너에서 요청을 받을 때, 하나의 요청마다 하나의 스레드가 할당되며, 컨트롤러부터 서비스, DB 호출까지 해당 스레드가 점유된 상태(블록킹)로 전체 흐름을 처리한다.

> Spring 공식 문서:  
> “Spring MVC relies on Servlet blocking I/O and lets applications use the Servlet API directly if they need to.”

여기서 blocking I/O는 BIO 커넥터를 의미하는 것이 아니라,
Servlet API 자체가 요청 하나당 스레드 하나를 점유하는 동기적(blocking) 처리 모델임을 의미한다.

> A traditional way of Servlet-based approach is used, resulting in threads being blocked while executing I/O operations. i.e., Blocking I/O model.

출처
https://dzone.com/articles/understanding-tomcat-nio  
https://docs.spring.io/spring-framework/reference/web/webflux/new-framework.html  
https://tomcat.apache.org/tomcat-9.0-doc/config/http.html  
https://www.geeksforgeeks.org/springboot/difference-between-spring-mvc-and-spring-webflux/
