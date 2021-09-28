# JavaScript4

## String concatenation

- ```javascript
  console.log('my' + 'cat');
  console.log('1' +2);
  console.log(`string literals: 1 +2 = ${1+2}
  싱글커트 대신 백키를 이용하면 엔터를 사용할 때 \n이 자동적으로 인식이 되며
  """"""""''''''' 같은 쿼터들을 사용할 수 있다. `);
  ```

  - 그 외에 사용법들은 자바나 c랑 비슷함 (' ', " ") 사용 시 \n \t \ ' 

---

## Numeric operators

- 2 ** 3 를 사용하면 2^3 계산 가능함

---

## Logical operations

- or 연산자는 처음에 true가 나오면 뒤에 나오는 관계 연산자에 상관없이 연산을 멈추고 true 값을 반환함
  - 만약 or 연산자를 사용하는 곳에서 뒤에 나오는 함수도 실행되는 것이 필요하다면 or 연산자의 제일 앞에 true일 때에 뒤에 함수가 실행되지 않을 수도 있다.
  - 또한 굳이 실행될 필요가 없는 함수라면 연산 시간을 생각 했을 때 가장 연산이 빠른 것을 제일 앞에 둬서 연산 시간을 줄이는 것이 좋다. (expression이나 function을 가장 뒤에 배치)

- and 연산자는 앞에서 false가 나오면 뒤에 연산은 생략하고 바로 false값을 반환한다.

  - 이 때에도 뒤에 연산이 긴 expression이나 function을 배치하여 불필요한 연산 시간을 소모하는 것을 방지하는게 좋다.

  - nullableObject && nullableObject.something 을 and 연산자로 묶으면 앞에 null일 때 false가 되어 뒤에 연산이 생략되고, 앞이 null이 아닐 때 뒤에 연산이 실행되는 트릭을 사용하여 

    `if(nullableObject) nullableObject.someting;` 코드를 대신할 수 있다.

---

## Equality

- **==** : loose equality, with type conversion

  - 내용이 같으면 타입에 상관없이 true를 반환 (`'5' == 5` 를 true로 반환)

- **===** : strict equality, no type conversion

  - 내용도 같고, 타입도 같아야 true로 나온다.
  - 주로 사용

  ```javascript
  const ellie1 = { name: 'ellie' };
  const ellie2 = { name: 'ellie' };
  const ellie3 = ellie2;
  ellie1 == ellie2;  //false (reference에 들어 있는 내용이 아니라 reference값 비교)
  ellie1 === ellie2; //false
  ellie3 === ellie2; //true
  ```

---

## Conditional operators

- if, else if, else

---

## Ternary operator: ?

- condition ? value1 : value2;