# Dependency Injection

- DI는 IoC 원칙을 구현하기 위한 디자인 패턴이다.

- DI는 의존 관계를 가지는 클래스 외부에서 의존하는 클래스를 주입해주는 방법론에 대한 것

- 3 class types of Dependency Injection

  1. Client Class : Service 클래스에 의존하는 클래스 (메소드를 요청하는 클래스,dependent)

  2. Service Class : Client 클래스에 서비스를 제공하는 클래스 (메소드를 주는 클래스, dependency)

  3. Injection Class : Client 클래스에 Service 클래스를 제공해주는 클래스

     ![Dependency Injection](https://www.tutorialsteacher.com/Content/images/ioc/DI.png)

- types of Dependency Injection : constructor way, property way, method way

  1. Constructor Injection: 클라이언트의 생성자를 통해 객체 전달
  2. Property Injection: 클라이언트의 public property로 데이터 접근 인터페이스를 만듬
  3. Method Injection: 

  

## Constructor Injection

- 생성자에서 생성시 매개변수로 데이터 접근 인터페이스를 비지니스 로직 클래스 내에 있는 프로퍼티로 할당 받아 사용한다.

- 만약 매개변수 없이 생성자가 호출되면 기본적으로 사용하는 concrete 데이터 접근 클래스를 생성하여 프로퍼티에 할당하는 default 생성자도 만들어준다.

  ``` java
  public class CustomerBusinessLogic{
      ICustomerDataAccess dataAccess;
      //생성자에 매개변수로 사용할 concrete 데이터 접근 클래스를 주입
      public CustomerBusinessLogic(IcustomerDataAccess dataAccess){
          this.dataAccess = dataAccess
      }
      //매개변수가 없는 생성자는 기본적으로 사용하는 concrete 데이터 접근 클래스 주입
      public CustomerBusinessLogic(){
          dataAccess = new CustomerDataAccess();
      }
      
      public String ProcessCustomerData(String id){
          return dataAccess.getCustomerName(id);
      }
  }
  
  interface ICustomerDataAccess{
      String getCustomerName(String id);
  }
  
  public class CustomerDataAccess implements ICustomerDataAccess {
      @Override
      public String getCustomerNmae(String id){
          //implement
          return "name";
      }
  }
  
  public class CustomerService{
  	CustomerBusinessLogic customerBL;
      
      public CustomerService(){
          customerBL = new CustomerBusinessLogic(new CustomerDataAccess());
      }
  }
  ```

  

## Property Injection

- 비지니스 로직 클래스 내에 데이터 접근 인터페이스를 필드가 있고, 외부에서 비지니스 로직 클래스를 사용할 때 해당 인터페이스에 비지니스 로직 클래스가 필요로 하는 concrete 데이터 접근 클래스를 생성하여 주입해주는 방식

  ```java
  public class CustomerBusinessLogic{
      public string getCustomerName(String id){
          return DataAccess.getCustomerName(id);
      }
      public ICustomerDataAccess dataAccess
  }
  
  interface ICustomerDataAccess{
      String getCustomerName(String id);
  }
  
  public class CustomerDataAccess implements ICustomerDataAccess {
      @Override
      public String getCustomerNmae(String id){
          //implement
          return "name";
      }
  }
  public class CustomerService{
  	CustomerBusinessLogic customerBL;
      
      public CustomerService(){
          customerBL = new CustomerBusinessLogic();
          customerBL.dataAccess = new CustomerDataAccess();
      }
  }
  ```



## Method Injection

- 인터페이스나 다른 클래스에 데이터 접근 클래스를 setting 해주는 메소드를 만들고, 비지니스 로직 클래스에 해당 클래스를 상속 받게 하여 injector에서 해당 클래스를 상속해준 클래스로 변환하여 사용할 데이터 접근 클래스를 매개변수로 넣고 메소드를 호출하는 방식

  ```java
  //메소드 방식으로 비지니스 로직 클래스에 데이터 접근 클래스를 주입할 인터페이스
  interface IDataAccessDependency{
      void setDependency(ICustomerDataAccess dataAccess);
  }
  
  public class CustomerBusinessLogic : IDataAccessDependency{
      ICustomerDataAccess dataAccess;
      
      public string getCustomerName(String id){
          return DataAccess.getCustomerName(id);
      }
      //위의 인터페이스에 메소드를 오버라이딩
      @Override
      public void setDependency(ICustomerDataAccess dataAccess){
          this.dataAccess = dataAccess;
      }
  }
  
  interface ICustomerDataAccess{
      String getCustomerName(String id);
  }
  
  public class CustomerDataAccess implements ICustomerDataAccess {
      @Override
      public String getCustomerNmae(String id){
          //implement
          return "name";
      }
  }
  public class CustomerService{
  	CustomerBusinessLogic customerBL;
      
      public CustomerService(){
          customerBL = new CustomerBusinessLogic();
          customerBL.dataAccess = new CustomerDataAccess();
      }
  }
  ```

  