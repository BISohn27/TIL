# JSP2

- 내장 객체의 영역: 내장 객체의 영역은 객체가 얼마 동안 살아있는가(life cycle)을 지정해 주는 것
  - 서블릿과 똑같이 page, request, session, application



## 액션 태그

- 액션 태그는 XML 문법을 따른다. 즉 시작 태그와 함께 반드시 종료 태그를 포함해야 함.

- 표현

  - `<jsp: . . . 속성="값">내용</jsp...>`

    - **`<jsp:forward page="URL"/>`**

      - 현재 JSP 페이지에서 URL로 지정된 특정 페이지로 넘어갈 때 사용하는 태그

      -  ```jsp
         <% RequestDispatcher dispatcher = request.getRequestDispatcher("URL");
         
         dispatcher.forward(request,response); %>
         ```

        위의 코드를 대체하는 액션태그

    - **`<jsp: include page="URL"/>`**

      - 한 페이지 내에서 시멘틱 태그를 모듈화 시켜서 각각의 영역을 서로 다른 JSP로 구축하고 최종적인 페이지를 include를 통해 완성하는 태그

      - ```jsp
        <jsp:include page="header.jsp"/>
        		~~~
        		~~~
        		~~~
        <jsp:include page="footer.jsp"/>
        ```

      - include에서는 한 페이지를 모듈화 시킨 것이기 때문에 변수 공유가 가능하다.

      - <%@ include file='URL' %> 와의 차이 디렉티브 태그를 사용하여 JSP 파일을 포함시키면 동적인 바인딩이 아닌 정적인 바인딩으로 JSP에서 서블릿으로 변경하는 과정에 코드의 해당 라인에서 url에 적혀진 코드가 삽입되는 방식으로 코드를 병합한다. 한편 액션태그의 include는 JSP에서 서블릿에 해당하는 JAVA 파일로 만드는 과정에서 코드가 삽입되는 것이 아니라 JAVA 파일은 따로 만들어지고, 페이지를 만드는 과정에서 해당 JAVA 파일로 페이지 생성 권한은 넘겼다가 다시 찾아오는 방식으로 페이지를 만든다.

    