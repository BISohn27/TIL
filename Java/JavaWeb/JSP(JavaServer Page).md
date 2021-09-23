# JSP(JavaServer Page)

- 인터넷을 사용하는 사용자에게 정보를 제공하기 위해서는 html 태그를 사용하여 웹 브라우저 내에 정볼르 표시하는데 jsp에서는 이러한 html 태그를 사용할 수 있어 웹 애플리케이션의 프레젠테이션 역할을 한다.

- 기본 베이스는 html이고, 필요할 때마다 자바 코드를 넣어 동적 페이지(정보를 가져와 표시하는)를 만드는 기능

- html 태그 사이에 <% %>를 추가

  ```jsp
  <html>
      <body>
          <%
             int su1=20;
             int su2=10;
             int sum = su1+su2;
             out.println(sum);
             %>
      </body>
  </html>
  ```

- jsp는 html로 작성된 코드를 servlet으로 바꿔주는 일종의 printwriter 기능을 한다.

- jsp는 servlet으로 변환되어 java 파일이 만들어지고 이를 class 파일로 컴파일하여 웹 애플리케이션이 동작할 때 이 class 파일이 실행된다.

- [처리과정]

  개발자가 JSP를 작성 -> MyJSP.jsp -> 웹 컨테이너가 Servlet으로 변환-> MyJSP_jsp.java 

  ![JSP 처리 과정](https://media.vlpt.us/images/jsj3282/post/e078fef3-616e-4023-ab9e-d828e750f79a/%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.png)

​		WAS에서 JSP가 Java 코드로 변환되는 과정

![MVC 아키텍쳐 · Extra Studies](https://dorothy-koo.gitbooks.io/extra-studies/content/assets/mvc-5.png)

---



## JSP 태그

| 종류              | 사용용도                                                 | 형식         |
| ----------------- | -------------------------------------------------------- | ------------ |
| **s*criptlet***   | ***자바 코드를 기술함***                                 | ***<% %>***  |
| ***declaration*** | ***변수(member field)와 메소드를 선언함***               | ***<%! %>*** |
| ***expression***  | ***계산식이나 함수를 호출한 결과를 문자열 형태로 출력*** | ***<%= %>*** |
| comment           | JSP 페이지에 설명을 넣음                                 | <%-- --%>    |
| directive         | JSP 페이지의 속성을 지정함                               | <%@ %>       |

- expression <%= %>

  - out.println() 코드를 줄이는 역할
  - 자바 코드를 html 의미에 맞게 태그 형식으로 처리하기 위한 방법
  - expression은 해당 태그 안에 내용을 out.print() 에 매개변수로 자동으로 넣어주는 역할을 하기 때문에 <%= 변수; %> 처럼 내부에 ; 가 들어가면 오류가 생긴다. 

  ```jsp
  <%
  	int su1 =20;
  	int su2 =10;
  	int sum = su1 + su2;
  	out.println(sum);
  %>
  <%= sum %>
  ```

- comment

  - JSP에서 주석을 사용할 경우 html 주석(<-- -->)를 사용할 경우 html 태그만 주석이 적용되고 JSP 태그들은 주석 처리가 안된다
  - 반대로 JSP 주석은 html 태그까지 주석 처리가 가능하다.

- directive

  - Java에서 import문처럼 해당 페이지 내에서 특정 기술을 적용하기 위한 문법
  - 종류
    1. page directive
       - page directive 내에 import 문을 통해 특정 클래스를 사용 가능하다(결국 JSP도 클래스이기 때문에)
    2. include
    3. page library

---



- jsp 중간 중간 html 구문을 삽입이 가능하고, 삽입된 html 구문은 jsp 구문의 흐름에 따라 실행이 될 수도 있고, 실행이 되지 않을 수도 있다. (if 문을 위에서 사용했을 때 if 문 안에 포함된 html문은 조건이 맞을 경우에만 실행된다)
