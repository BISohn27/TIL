# JavaScript5

## Function

- fundamental building block
- subprogram
- `function name(param1, param2,...) { body ... return; }`
- 하나의 함수는 한가지의 일만 하도록 해야 한다.
- 함수의 이름은 동사 [+ 명사]
- 자바스크립트에서 함수는 객체이다. (변수에 할당할 수 있음, 변수가 일종의 함수 포인터)

---



### Parameters

- **Primitive Parameters**: passed by value

- **Object Parameters**: passed by reference

- **Default Parameters**

  ```javascript
  function showMessage(message, from ='unknown'){
      console.log('${message} by ${from}');
  }
  showMessage('Hi!');
  //Hi by unknown
  ```

- **Rest Parameters**

  ```javascript
  function printAll(...args){
      for(let i=0; i<args.length; i++) {
          console.log(args[i]);
      }
      
      for(const arg of args) {
          console.log(arg);
      }
      
      args.forEach((arg) => console.log(arg));
  }
  printAll('dream','coding','ellie');
  ```

  - ...args 형태로 매개변수를 정의하면 들어오는 인자가 배열 형태로 묶여서 전달된다.

---



### Local Scope

- 밖에서는 안이 보이지 않고 (전역 영역), 안에서는 밖을 볼 수 있다.(지역 영역)
- 밖에서는 지역 변수에 접근할 수 없으나 지역 영역 안에서는 전역 영역이나 자기보다 더 넓은 범위 scope에 접근할 수 있다.

---



### return

- 리턴 타입이 없으면 undefined return이 생략된 것과 똑같다.

---



### Early return, early exit

- 조건이 맞을 때만 실행할 경우에는 함수를 정의할 때 앞에 조건이 맞지 않으면 리턴이 되게끔 설계하는 것이 좋다.

  ```javascript
  //이것보다는
  function upgradeUser(user) {
      if(user.point > 10) {
          // long upgrade logic...
      }
  }
  
  //이게 더 좋다 (근데 어차피 위에 로직도 작으면 안에 안들어가고 바로 빠져나가는데?)
  function upgradeUser(user) {
      if(user.point <= 10) {
          return;
      }
      // long upgrade logic...
  }
  ```

---



## First-class function

- 함수는 다른 변수와 마찬가지로
  - 변수에 할당이 가능하다
  - 다른 함수에 argument로 전달 가능하다
  - 다른 함수로부터 반환되어질 수 있다.



### Function Expression

- 익명 함수

  ```javascript
  const print = function () {
      console.log('print');
  }
  print(); //변수로 호출 가능
  const printAgain = print; //다른 변수에 할당 가능
  ```

  - **function expression**(변수에 할당함으로써 정의되는 함수식)은 할당된 다음부터 호출이 가능하다.(hoisting이 되지 않는다.)
  - **function declaration**은 hoisting 되기 때문에 c언어처럼 프로토타입이 전방에 있는 효과가 있어 함수가 정의되기 전에 함수 호출이 가능하다. 



### Callback function using function expression

- 함수를 정의할 때 함수의 매개변수로 함수들을 할당하고, 정의된 함수 안에서 조건에 따라 특정 함수를 골라서 실행하도록 전달되는 함수를 call back function이라고 한다.

  ```javascript
  //printYes와 printNo는 callback 함수
  function randomQuiz(answer,printYes,printNo){
      if(answer === 'love you'){
        printYes();
      } else {
        printNo();  
      }
  }
  //anonymous function
  const printYes = function() {
      console.log('yes!');
  }
  //named function
  //debug 할때 함수 이름을 보려고 할때
  //재귀함수 사용시
  const printNo = function print() {
      console.log('no!');
  }
  
  randomQuiz('wrong', printYes, printNo);
  
  ```

  

### Arrow function

- 항상 익명 함수로 사용

```javascript
const simplePrint = function () {
    console.log('simplePrint');
}

const simplePrint = () => console.log('simplePrint!');
const add = (a,b) => a+b;
const simpleMultify = (a,b) => {
    return a*b;
}
```



### IIFE(Immediately Invoked Function Expression)

```javascript
(function hello(){
    console.log('IIFE');
})();
```

