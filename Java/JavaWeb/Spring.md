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

  

  ---

  

  ### Aspect Oriented Programming(AOP)

- 객체지향 프로그래밍에서 class가 key unit이라면 스프링 프레임워크의 또다른 특징인 aop는 관점(aspect)가 key unit이다.

- 관점 지향 프로그래밍에서 로그인 등 프로젝트에 보안을 유지해야할 필요가 있을 때 비지니스 로직에서 보안 유지를 위한 장치들을 걷어내고, aop에 보안을 위임할 수 있다.

- aop를 통해 메소드가 호출 된 이후, 메소드가 호출 되기 전, 메소드가 반환된 이후, 예외가 발생했을 때 등의 상황에서 어떤 action이라도 실행할 수 있다.

- 스프링 프레임 워크는 모듈 간 확실한 분리를 제공하는 방식으로 프로그래머가 프로그래밍 원칙을 지키는데 도움을 준다.

  

  ---

  

  ### xml을 통한 spring사용

- 스프링에 설정된 xml 형식에 따라 작성하면 원하는 객체를 서버 구동과 동시에 만들 수 있다.

  ​	클래스 내 정의된 setter 메소드를 통해 기본 자료형을 멤버 변수로 가진 bean 객체 생성

  ```xml
  <beans>
  	<bean id="personService" class="com.spring.ex01.PersonServiceImpl">
      	<property name="name">
          	<value>홍길동</value>
          </property>
      </bean>
  </beans>
  ```

  - bean태그를 통해 웹 어플리케이션에서 사용할 객체들을 만들 수 있다
    - id 속성 값에 xml 내에서와 java 클래스에서 해당 bean 객체를 부를 때 사용할 이름을 명시한다.
    - class 속성 값을 통해 사용될 실제 클래스가 어떤 클래스인지를 지정한다.
  - property 태그는 사용할 클래스에 들어 있는 멤버 변수(property)에 값을 넣어준다.
    - name 속성 값은 멤버 변수의 이름이다.
    - value는 primitive(기본형) 자료형 값을 넣을 줄 때 사용한다.



​		클래스 내 정의된 setter 메소드를 통해 객체를 멤버 변수로 가진 bean 객체 생성(DI)

```xml
<beans>
	<bean id="memberService" class="com.spring.ex03.MemberServiceImpl">
    	<property name="memberDao" ref="memberDAO"/>
    </bean>
    
    <bean id="memberDAO" class ="com.spring.ex03.MemberDAOImpl"/>
</beans>		
```

- bean태그를 통해 웹 어플리케이션에서 사용할 객체들을 만들 수 있다
  - id 속성 값에 xml 내에서와 java 클래스에서 해당 bean 객체를 부를 때 사용할 이름을 명시한다.
  - class 속성 값을 통해 사용될 실제 클래스가 어떤 클래스인지를 지정한다.
- property 태그는 사용할 클래스에 들어 있는 멤버 변수(property)에 값을 넣어준다.
  - name 속성 값은 멤버 변수의 이름이다. 여기서는 멤버 변수가 객체로 depenency이다.
  - ref 속성 값은 멤버 참조 변수가 어떤 객체를 참조하고 있는지를 넣어주는 곳으로 xml 상에서 참조하고자 하는 객체의 정의된 id 값을 넣어줘야 한다.



​		setter 메소드가 아닌 생성자를 통한 멤버 변수 할당

```xml
<beans>
    <bean id="personService1" class="com.spring.ex02.PersonServiceImpl">
        <constructor-arg value="이순신"/>
    </bean>

    <bean id="personService2" class="com.spring.ex02.PersonServiceImpl">
        <constructor-arg value="이순신"/>
        <constructor-arg value="100"/>
    </bean>
</beans>
```

- constructor-arg 태그에 value 속성값을 넣어 생성자 argument들을 넣어 bean 객체를 생성한다.



---



### 		Spring MVC

![img](Spring%20&%20Spring%20Boot.assets/img.png)

- Spring MVC의 구조
  1. 클라이언트 측으로부터 넘어온 요청은 우선 Front Controller인 DispatcherServlet으로 들어가게 된다.
  2. HandlerMapping에서는 Dispatcher로 넘어온 요청의 url을 분석한다. 이때의 url은 미리 mapping된 정보로 클라이언트가 요청을 하게 되고, HandlerMapping에서는 mapping된 url을 실제 해당 요청을 처리하는 Page Controller가 어떤 컨트롤러인지 찾아주는 역할을 한다.
  3. HandlerMapping으로부터 지정된 Page Controller는 Data Access Object를 사용하여 데이터베이스로부터 클라이언트에게 필요한 데이터를 가져오고, 가져온 데이터와 클라이언트 브라우저에 해당 데이터를 보여줄 뷰 정보를 ModelAndView에 담아 Dispatcher로 보낸다.
  4. ViewResolver에서 컨트롤러로부터 넘어온 데이터에서 뷰에 대한 정보를 분석하여 실제 사용해야할 뷰를 지정한다.



- SimpleUrlController

  - 이 컨트롤러는 클라이언트 요청에 대한 데이터 처리를 각각 개별 객체를 통해 처리하는 방식이다.

  ```xml
  <bean id="simpleUrlController" class="com.spring.ex01.SimpleController"/>
  
  <bean id="urlMapping"
    class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
      <props>
        <prop key="/test/index.do">simpleUrlController</prop>
      </props>
     </property>
  </bean>
  ```

  - id가 urlMapping인 객체를 통해 mappings라는 이름을 가진 멤버 변수에 key 값으로 사용자가 요청한 url 값을 할당하고, key의 value 값으로 실제로 불러야할 컨트롤러 객체의 xml id 값을 넣어준다.



- MultiActionController

  - 사용자가 요청한 정보를 개별 객체로 처리하는 SimpleUrlController와 달리 MultiActionController는 하나의 객체 안에 처리해야할 각각의 요청을 Method로 대응하도록 만든 것이다.

  ```xml
  <bean id="viewResolver"    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  	<property name="viewClass"
                value="org.springframework.web.servlet.view.JstlView" />
      <property name="prefix" value="/test/"/>
      <property name="suffix" value=".jsp"/>
  </bean>
  
  <bean id="userUrlMapping"      class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
  	<property name="mappings">
      	<props>
          	<prop key="/test/*.do">userController</prop>
          </props>	
      </property>
  </bean>
  
  <bean id="userController" class="com.spring.ex02.UserController">
  	<property name="methodNameResolver">
      	<ref local="userMethodNameResolver"/>
      </property>
  </bean>
  
  <bean id="userMethodNameResolver"
  class="org.springframework.web.servlet.mvc.multiaction.PropertiesMethodNameResolver">
  	<property name="mappings">
      	<props>
          	<prop key="/test/login.do">login</prop>
          </props>
      </property>
  </bean>
  ```

  - /test/*.do 형태로 들어온 모든 url은 xml id가 userurlmapping인 simpleurlhandlermapping객체로 전부 usercontroller로 보내지게 된다.

  - usercontroller의 methodnameresolver 멤버 변수에 전달받은 url로부터 호출해야할 메소드 이름을 매핑시켜 놓은 bean 객체를 할당해준다.

  - userMethodNameResolver xml id를 가진 빈 객체를 생성하여, mappings 맴버 변수에 키인 url에 대응하는 메소드 이름을 값으로 할당하여 특정 url에 매칭되는 메소드가 호출 될 수 있도록  한다.

  - ViewResolver는 model에 setViewName 메소드로 저장된 이름에 prefix와 postfix 붙여 

    [**prefix** **setViewName** **postfix**] 인 뷰를 지정할 수 있도록 뷰의 경로를 만들어주는 역할을 한다.

  - Dispatcher는 viewResolver로부터 받은 뷰 경로를 클라이언트한테 response 한다.

