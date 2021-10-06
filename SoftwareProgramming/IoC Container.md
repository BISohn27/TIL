# IoC Container

- DI 디자인 패턴을 자동적으로 구현하기 위한 프레임워크
- 객체의 생성과 생명 주기, 클래스에 dependencies를 주입하는 것을 관리하기 위한 목적
- 필요로 하는 객체를 생성하여 런타임 때에 생성자 주입, 프로퍼티 주입, 메소드 주입 3가지 모두 가능하다.
- DI Lifecycle을 따라야 한다.
  - Register: IoC Container에 type-mapping이 되어 있어, 특정 유형에 맞는 dependency들이 regist되어야 한다.
  - Resolve:  client가 필요로 하는 dependency를 선별하여 만들어 주입해 줄 수 있어야 한다.
  - Dispose: 필요하지 않는 dependency를 처리할 수 있어야 한다.

