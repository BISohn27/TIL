# IoC, DIP, DI, IoC Container

- Design Principle : Inversion of Control(IoC), Dependency Inversion Principle(DIP)

- Design Pattern : Dependency Injection(DI)

- Framework : IoC Container

  ![img](https://www.tutorialsteacher.com/Content/images/ioc/principles-and-patterns.png)

## IoC

- 애플리케이션 클래스들 간의 느슨한 관계를 만들기 위하여 객체 지향 프로그램을 설계할 때 다른 종류의 제어 권한을 다른 주체에게 위임하는 것을 의미
- 여기서 컨트롤이란 한 클래스가 가지고 있는 주요한 권한 외에 애플리케이션의 흐름, 해당 클래스와 dependent 관계를 맺고 있는 객체의 생성과 바인딩 같은 것을 말한다.



## Dependency Inversion Principle

- DIP 또한 애플리케이션 클래스들 간의 느슨한 관계를 형성하기 위한 원칙이다.
- DIP 와 IoC는 느슨한 관계를 형성하기 위해 함께 사용하는 것이 권장된다.
- DIP는 high level 모듈이 low level 모듈에 의존해서는 안된다는 원칙이며, 둘은 abstraction(추상적인 클래스)에 의존해야 한다.



##  Dependency Injection

- DI는 IoC 원칙을 실제 적용하기 위한 디자인 패턴으로, 특히 의존 관계에 있는 객체의 생성을 위임하는 것을 말한다.



## IoC Container

- IoC Container는 의존 관계에 있는 객체들의 생성을 자동적으로 처리하여 IoC를 구현하기 위한 프로그래머의 노력을 줄여주기 위하여 사용되는 프레임워크이다.
- 

