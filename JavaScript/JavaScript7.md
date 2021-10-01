# JavaScript7

## Objects

- 연관이 있는 데이터나 함수들을 묶어서 놓은 것

- 객체 생성법

  **`const obj1 = { name : 'anonymous', age: 4 }`** : 'object literal' syntax

  **`const obj2 = new Object()`;**  : 'object constructor' syntax

  ​	-> 클래스에서 정의된 생성자를 통해 객체를 생성

- 자바스크립트는 런타임에 타입이 결정되기 때문에 객체를 생성한 후에도 뒤에 객체에 또다른 데이터를 만들어줄 수 있다. (가능하다는 것이지 유지,보수 측면에서는 권장은 아님)

- `delete 객체명.변수명;` 를 통해 객체에 이미 존재하는 변수를 삭제할 수도 있다.

- **object = {key : value}** !!

  - 자바스크립트에서 객체는 키(변수명)과 그에 대응하는 값으로 이뤄져 있기 때문에 마치 해시맵에서 그때 그때 키와 값을 메소드를 통해 만들어주는 것처럼 자바스크립트에서 객체가 설계되어 있기 때문에 객체가 생성된 이후에도 변수를 새로 추가해주거나 삭제할 수 있는 것이다. (이는 해시맵에서 필요할 때마다 키와 값을 넣고 필요 없을때는 삭제하는 것과 똑같다.)

## Computed properties

- `console.log(sohn.name);` 와 `console.log(sohn['name']);`은 동일하다.

  - 앞은 멤버 접근 연산자를 이용하는 것이고, 위에는 키를 통해 배열처럼 접근하는 방식

- key should be always string (becuase it is key)

- 멤버 접근 연산자를 통해 값을 받아 올 때에는 우리가 접근하고자 하는 값의 키를 정확하게 할 때 사용하고, 우리가 가져오고자 하는 값의 키를 정확하게 모를 때(정확하게는 computed properties는 런타임에 결정되는 특성이 있기 때문에 코딩하는 시점에는 어떤 키를 사용해야 할지 모르지만 웹 서버 구동 시 실시간으로 특정 값에 접근해야 하는 경우에 사용)에는 computed properties를 통해 값에 접근한다.

  - 예를 들어 사용자에게 input을 받아 값을 출력해야 하는 경우(런타임) computed properties를 사용해서 값에 접근하도록 만들어준다.
  - 이는 키값은 항상 string으로 넣어줘야 하는데 만약 key에 변수를 할당하면 undefined 값을 던져주게 된다. 이는 자바스크립트가 자료형이 런타임에 결정되기 때문에 매개변수로 변수를 넣어주면 이 변수는 런타임에 값을 받아야만 자료형이 결정되기 때문이다.

  ```javascript
  function printValue(obj,key){
      //undefined로 정상적인 기능 수행이 불가능
      console.log(obj.key);  
      //아래처럼 computed properties를 사용해야 한다.
      console.log(obj[key]);
  }
  ```

## Property value shorthand

- 키와 value의 명이 동일하다면 둘 중 한가지만 작성해서 객체를 생성할 수 있다.

  ```javascript
  function makePerson(name,age) {
      return {
          name : name;
          age : age;
      };
  }
  function makePerson(name,age) {
      return{
          name,
          age,
      };
  }
  //Constructor Function
  function Person(name,age){
      this.name = name;
      this.age = age;
      return this;
  }
  ```

## in operator : property existence check (key in obj)

- `console.log('name' in sohn);`

- 키가 객체 안에 있으면 true, 키가 객체 안에 없으면 false
-  정의되지 않은 key(객체의 필드가 아닌)를 출력하면 undefined가 된다.

## for ... in , for ... of

- for (key in obj)

  - 객체 안에 있는 키가 무엇인지 하나하나 다 출력할 때 for ... in 사용

- for (value of iterable)

  - 객체가 아니라 순차적으로 iterable한 배열이나 리스트에 사용

    ```javascript
    const array = [1,2,3,4];
    for(let i = 0; i< array.length; i++){}
        console.log(array[i]);
    }
    for(value of array){
        cosole.log(value);
    }
    ```

    

## cloning

- deep copy

- old way

  ```javascript
  const user3 ={};
  for(key in user)
  	user3[key] = user[key];
  ```

- new way

  ```javascript
  const user4 ={};
  Object.assign(user4,user);
  
  const user5= Object.assign({},user);
  ```

- cloning은 뒤에 나오는 obj가 앞의 obj를 계속해서 덮어 씌우는 형태로 copy가 일어나기 때문에 동일한 key 값이 있다면 가장 마지막에 온 obj의 값이 저장된다.

