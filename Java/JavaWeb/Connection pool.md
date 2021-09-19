# Connection pool

- 데이터베이스를 필요시 마다 연동하게 하면 데이터 베이스와 어플리케이션 간 연동에 소요되는 시간 때문에 오버헤드가 생긴다.
- 오버헤드를 줄이기 위하여 웹 애플리케이션이 실행 됨과 동시에 데이터 베이스와 연결되고, 연결 된 데이터베이스(미리 만들어진 Connection 객체)를 필요할 때마다 가져와 쓰는 것이 커넥션풀

---

## 커넥션풀 동작 과정

1. 웹 어플리케이션 구동 시(톰캣 컨테이너 실행 시) 커넥션풀 객체를 생성
2. 생성된 커넥션풀과 DBMS 연결
3. 연동 작업이 필요하면 케넉션풀의 메소드를 통해 커넥션 객체를 받아옴

----

## 커넥션풀 구현

- 자바 api인 DataSource 클래스로 구현

- JNDI(Java Naming and Directory Interface)를 통해 객체에 접근

  - JNDI는 키/값(map형태)로 저장하고, 필요할 때마다 키로 값을 로딩해오는 방법
  - 웹 브라우저에서 name/value 쌍으로 전송한 후 서블릿에서 getParameter(name)로 값을 가져올 때
  - 해시맵, 해시테이블에서 값을 가져올 때
  - 웹 브라우저에서 도메인 네임으로 DNS 서버에 요청할 경우 해당 도메인 네임에 대응되는 ip 주소를 가져올 때 등에 사용

- 서버 설정 파일인 context.xml을 수정

  1. ```xml
     <REsource
         name="jdbc/mysql"
         auth ="Container"
         type="javax.sql.DataSource"
         driverClassName="com.mysql.cj.jdbc.Driver"
         url="jdbc:mysql://localhost:3306/스키마이름?serverTimezone=UTC"
         username = "root"
         password = "java"
         maxActive = "50"
         maxWait = "-1" />
     ```

     - name : JNDI 이름(key 값)
     - auth : 인증 주체
     - type : 데이터베이스 종류별 DataSource
     - maxActive : 동시에 최대로 데이터베이스에 연결할 수 있는 Connection 수
     - maxIdle : 동시에 idle 상태로 대기할 수 있는 최대 수
     - maxWait : 새로운 연결이 생길 때까지 기다릴 수 있는 최대 시간(-1: 무한대)