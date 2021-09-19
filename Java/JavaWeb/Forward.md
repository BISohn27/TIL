# Forward

- 하나의 서블릿에서 다른 서블릿이나 jsp에 연동하는 방법
- 용도
  - 요청에 대한 추가적인 작업을 다른 서블릿에 수행시킴
  - 요청에 포함된 정보를 다른 서블릿이나 jsp와 공유
  - 요청에 정보를 포함시켜 다른 서블릿에 전달
  - 모델2 개발 시 서블릿에서 jsp로 데이터 전달

---



1. ## Redirect

   - 서블릿의 요청이 클라이언트의 웹 브라우저를 다시 거쳐 요청되는 방식

     1. 클라이언트 웹 브라우저에서 첫 번째 서블릿을 요청
     2. HttpServletResponse 객체의 **sendRedirect("원하는 서블릿 또는 jsp")** 로 클라이언트 웹 브라우저를 통해서 요청
     3. 웹 브라우저는 sendRedirect 메소드가 지정한 두 번째 서블릿을 다시 요청

     ---

     

2. ## Refresh

   - redirect처럼 클라이언트 웹 브라우저를 거쳐서 요청을 수행

     1. 클라이언트의 웹 브라우저에서 첫 번째 서블릿을 요청
     2. HttpServletResponse 객체 **addHeader("Refresh","경과시간(초);url=요청할 서블릿 또는 jsp")** 메소드로 웹 브라우저를 통해서 요청
     3. 웹 브라우저는 addHeader 메소드가 지정한 두 뻔째 서블릿을 다시 요청

     ---

     

3. ## Location

   - 자바스크립트 location 객체의 href 속성을 이용

   - 자바스크립트에서 재요청하는 방식

   - **location.href='요청할 서블릿 또는 jsp'** 

     ```java
     out.print("<script type='text/javascript'>");
     out.print("location.href='요청할 서블릿 또는 jsp';");
     out.print("</script>")
     ```

   ---

   

4. ## redirect 방식으로 다른 서블릿에 데이터 전달

   - redirect를 할 때 get 방식으로 데이터를 담아서 다른 서블릿에 전달할 수 있다.

     ```java
     PrintWriter out = response.getWriter();
     response.sendRedirect("요청할 서블릿?id=데이터값");
     ```

   - 요청 되어지는 두번째 서블릿에서는 getParameter("id") 를 통해 데이터를 받아 올 수 있다.

   - 이런 방식은 refresh나 location에서도 동일하게 get 방식으로 url에 데이터를 담아 보낼 수 있음

   ---

   

5. ## Dispatch

   - dipatch는 redirect와 다르게 두번째 서블릿이 클라이언트 웹 브라우저를 거치지 않고, 서버에서 포워딩이 진행

   - 포워드가 수행되어 다른 웹페이지로 넘어가더라도 웹 브라우저 주소창의 url이 변하지 않음

     -> 클라이언트 측에서는 포워드가 진행이 되어도 이를 알 수 없다

   - 일반적인 포워드 기능

   - **RequestDispatcher** 클래스의 메소드를 이용

     ```java
     RequestDispatcher dis = request.getRequestDispatcher("포워드할 서블릿 or JSP");
     dis.forward(request,response);
     ```

   - get 방식으로 데이터를 전송할 경우 

     `RequestDispatcher dis = request.getRequestDispatcher("서블릿?name=값");` 

   ---

   

6. ## Binding

   - 전달하는 데이터 양이 작을 때에는 get 방식으로 할 수 있으나 많은 양의 데이터를 다음 서블릿이나 JSP로 넘길 때에는 바인딩 기능을 사용

   - 웹 프로그램 실행 시 자원을 서블릿 관련 객체에 저장하는 방법

     **HttpServletRequest** 객체, **HttpSession** 객체, **ServletContext** 객체는 프로그램 시작 시 서블릿이나 JSP에서 공유하여 사용하기 때문에 여기에 데이터를 바인딩 시켜 전달

     

   1. ### HttpServletRequest를 이용한 redirect 포워딩 바인딩

      - *request.setAttribute(String name, Object obj)* 로 웹 브라우저에서 요청한 request 객체에 name 값으로 obj를 바인딩 시키고
      - response.sendRedirect("두번째 서블릿") 으로 두번째 서블릿에 redirect 방식으로 전달
      - 두번째 서블릿에서는 request.getAttribute(String name) 으로 전달된 데이터를 얻어옴
      - 하지만 실제 두번째 서블릿에서 해당 값을 가져오면 null이 들어 있다. 이는 redirect의 경우 첫번째 서블릿에서 두번째 서블릿을 실행할 때 클라이언트 측 웹 브라우저에 재요청을 해서 두번째 서블릿으로 오기 때문에 데이터가 담겨있는 첫번째 request 객체와 재 요청에 의해 두번째 서블릿 매개변수로 전달되는 request 객체가 다르기 때문에 나타나는 현상. 따라서 redirect 방식으로 바인딩 데이터 전달은 할 수 없다.

   2. ### HttpServletRequest를 이용한 dispatch 포워딩 바인딩

      - ```java
        request.setAttribute("address","서울시 성북구");
        RequestDispatcher dis = request.getRequestDispatcher("second");
        dispatch.forward(request,response);
        ```

      - ```java
        Stirng address=(String)request.getAttribute("address");
        ```

      - dispatch 방식을 사용하면 클라이언트가 첫번째 서블릿을 통해 요청한 request 객체를 그대로 다음 서블릿이나 JSP로 넘기기 때문에 request 객체에 정상적으로 정보가 담겨서 넘겨진다.



## 