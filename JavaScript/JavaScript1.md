# JavaScript

- LiveScript로서 클라이언트 레이어에서 데이터를 서버로 넘길 때 데이터를 정적으로 넘기는 것이 아니라 동적으로 데이터를 넘기기 위한 목적으로 만들어짐 (데이터 유효성 검사 등)

- CSS와 비슷하게 inline, internal, external 방식으로 정의할 수 있으며, internal일 경우 CSS처럼 head 태그 안에 삽입하여 사용

- 특징

  - script 언어

  - 간단한 application

  - 사용자 event 처리 가능

  - Object-Based Programming(객체기반) 언어 

    (자바는 Object-Oriented Programming)

- 장점

  - 신속한 개발
  - 배우기 쉽다
  - 플랫폼에 독립적
  - 시스템에 부하가 적다

- 단점

  - 내장 Method의 한계 - 기능 추가를 더 할 수 없다.
  - Code의 보완성 - 이를 위해서 .js 파일로 외부에 저장
  - debugging 및 개발 툴의 부재 (최근에는 툴의 발전으로 가능)

- 함수

  - 함수에는 항상 괄호가 있다

    (**F(x) = b** : x에 값을 넣으면 f가 기능을 하여 어떤 값을 출력하게 된다)

    - built in function : 매뉴얼에 이미 만들어져 있는 함수
    - user-defined function :  사용자가 필요에 의해 만드는 함수
    
  - 함수 선언 function 함수명{ }, return 타입은 함수 내부에서 return을 쓰는지에 따라 결정

    별도로 return 타입을 줄 필요는 없음

  - document: 해당 페이지에서 사용하는 객체 (document로 html에 작성한 각각의 element를 가져올 수 있다.)

  - 부모의 객체를 변수로 가져오면, 부모 객체에 소속된 자식 객체에 할당된 값을 가져올 수 있다.

  - input button 태그에 onclick="function명" 속성 값을 주면 버튼을 눌렀을 때 자바스크립트 function에 접근할 수 있다.

  - 태그에 속성값은 **`부모객체.속성="속성값"`**로 자바스크립트를 통해 부여할 수 있다.

    ```javascript
    /*example*/
    frmMember.method="post";
    frmMember.action="/webapps/UserInfoAddServlet";
    frimMember.submit();
    ```

  - `document.getElementById("name").value = 'val';`

    html 속성값을 통해 id=name인 값에 접근하는 방법

  - function에 this를 매개변수로 넣으면 해당 태그의 부모 객체가 넘어온다.

  - **window.open('url', 'pop', 'width=300,height=300')** 객체 : 팝업창을 띄우는 객체

    - 팝업창을 띄우기 위해서 main 창이 반드시 필요. (java의 다이얼로그처럼 기본 생성자가 없음)
    - pop: 팝업창의 이름, 임의로 정의할 수 있다.
    - 3번째 매개변수로 팝업창의 크기를 넣어준다.
    - window.open('url?' + 값) 을 통해 get 방식으로 '값'을 전달할 수 있다.

    

- 서버와 클라이언트(프론트엔드) 간 데이터를 주고 받을 때 
  - xml 방식으로 데이터를 주고 받거나
  - json 방식으로 데이터를 주고 받을 수 있다.
  - 이 때 서로의 통신 방식이 결정되면, parse를 통해 전달받은 데이터를 분석(해석)한다.
