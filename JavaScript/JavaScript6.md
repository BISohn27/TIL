# JavaScript

## Class

- 연관이 있는 것들을 한 곳에 묶어둔 일종의 container

  ```javascript
  class person {
      name;      //field
      age;	   //field
      speak();   //method
  }
  ```

- 자바스크립트에서도 상속, 다형성, 캡슐화가 가능

- **Class** : Template, Declare once, No data in 

  -> **Data** : Instance of a class, created many times, Data in

- ES6 이전에는 클래스 없이 object로 바로 만들어서 사용했음

- Prototype-based inheritance

- 표현

  ```javascript
  class Person {
      //constructor
      constructor(name,age) {
          this.name = name;
          this.age = age;
      }
      
      //methods
      speak() {
          console.log(`${this.name}: hello!`);
      }
  }
  
  const sohn = new Person('Sohn', '30');
  sohn.speak();
  ```

- 사용자가 직접 field에 접근하지 못하게 하고 getter와 setter를 통해서만 field에 접근하게 하여 비정상적인 값 수정을 막는 것이 encapsulation.

  ```javascript
  //Getter and Setter
  class User{
      constructor(firstName,lastName, age) {
          this.firstName = firstName;
          this.lastName = lastName;
          this.age = age;
      }
      //여기서 만약 getter의 함수명을 age로 하게 되면 위의 field의 age와 이름이 겹치기 때문에 여기서 부터 age를 호출하면 field의 age가 호출되는 것이 아니라 gettter method가 호출된다.
      get age() {
          return this._age; 
      }
       //여기서도 age가 setter로 바뀐다. 따라서 _age로 field를 설정한다. 이렇게 될 경우 생성자에서 this.age를 호출하면 내부적으로 setter를 통해 field에 값이 할당된다.
      set age(value) {
          if(value<0) {
              throw Error('age can not be negative');
          }
          this._age = value;
      }
  }
  ```

- Fields (public, private)

  - 추가된지 얼마 안되서 많이 사용되고 있지는 않다.

  ```javascript
  class Experiment {
      publicField =2;
  	//#field명을 하면 private이 됨.
  	#privateField = 0;
  }
  ```

- Static

  - c와 자바처럼 클래스에서 instantiation되면 객체가 만들어지면서 각각 메모리에 값이 할당된다. 따라서 static화 시켜 클래스끼리 공유하는 정적 필드와 정적 메소드를 만들 수 있다.
  - 이것도 최근에 추가된 것

  ```javascript
  class Article{
      static publicsher = 'Wow!';
  	constructor(articleNumber){
          this.articleNumber = articleNumber;
      }
  
  	static printPublisher(){
          console.log(Article.publisher);
      }
  }
  
  //정적 메소드나 정적 필드에 접근하기 위해서는 인스턴스로 접근하는게 아니라 클래스로 접근해야 한다.
  console.log(Article.publisher);
  Article.printPublisher();
  ```

- Inheritance

  ```javascript
  class shape{
      constructor(width, height, color){
          this.width = width;
          this.height = height;
          this.color = color;
      }
      
      draw(){
          console.log(`drawing ${this.color} color of`);
      }
      
      getArea(){
          return this.width * this.height;
      }
  }
  
  class Rectangle extends Shape {
  }
  class Triangle extends Shape{
      draw(){
          super.draw();
          console.log('ㅗ');
      }
      //다형성 가능
      getArea() {
          return (this.width * this.height)/2
      }
  }
  
  const rectangle = new Rectangle(20,20,'blue');
  rectangle.draw();
  ```

- Instance of

  - A instanceof B : A가 B에 속한 객체인지를 나타냄

  - 자식은 부모클래스에서 파생되었기 때문에 ''자식 instanceof 부모'' 는 true

    -> 다형성

  - 자바스크립트에서도 object라는 조상 클래스가 존재 (이브도 아니고 맨날 나와)