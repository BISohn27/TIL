# Spring & Spring Boot

# Spring

- 스프링 프레임워크의 가장 큰 특징은 dependency injection과 inversion of control을 통해 어플리케이션의 loosely coupling을 가능하게 해준다는 것이다.

- dependency injection이 없는 상황에서 한 클래스의 메소드가 다른 클래스의 메소드를 사용한다면, 해당 클래스의 인스턴스를 클래스 내에 생성하여 가지고 있어야만 한다.

- 스프링은 어노테이션이나 xml을 통해 인스턴스를 대신 생성하고, 인스턴스가 필요한 클래스에 인스턴스를 넣어주는 dependency injection으로 tight coupling을 해결해준다.

  - @component 어노테이션을 통해 팩토리 패턴을 적용하여 객체를 생성하고
  - @autowired 어노테이션으로 클래스에서 필요한 객체를 찾아 매칭시켜준다.
  - spring framework will create a bean for dependeny and autowire it into dependent

- 그 외에 스프링 프레임 워크는 20가지 모듈을 제공한다. (아래에는 대표적인 모듈들)

  - spring jdbc
  - spring mvc
  - spring aop
  - spring orm
  - spring jms
  - spring test
  - spring expression language (spel)

  

  ### Aspect Oriented Programming(AOP)

- 객체지향 프로그래밍에서 class가 key unit이라면 스프링 프레임워크의 또다른 특징인 aop는 관점(aspect)가 key unit이다.
- 관점 지향 프로그래밍에서 로그인 등 프로젝트에 보안을 유지해야할 필요가 있을 때 비지니스 로직에서 보안 유지를 위한 장치들을 걷어내고, aop에 보안을 위임할 수 있다.
- aop를 통해 메소드가 호출 된 이후, 메소드가 호출 되기 전, 메소드가 반환된 이후, 예외가 발생했을 때 등의 상황에서 어떤 action이라도 실행할 수 있다.
- 스프링 프레임 워크는 모듈 간 확실한 분리를 제공하는 방식으로 프로그래머가 프로그래밍 원칙을 지키는데 도움을 준다.



- 