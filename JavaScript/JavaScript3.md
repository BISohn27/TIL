# JavaScript3

## Variable (read/write)

- **let** (added in ES6) 변수 선언
- scope에 따라 전역 변수와 지역 변수가 존재(c와 비슷)
- **var**는 더 이상 사용하지 않는다. **var**는 변수를 선언해주기도 전에 변수에 값을 할당하고 그 후에  **var**로 해당 변수를 선언해주는 것이 가능한 약간 비정상적인 작동을 일으킨다. 또한 **var**는 블록 적용이 안된다. 따라서 그러한 비정상적인 작동을 에러로 인식하는 **let**을 현재는 사용한다. 
- hoisting : c언어에서 전방 선언과 비슷하게 정의부는 아래에 있으나 prototype을 위에 선언해줌으로써 정의부가 아직 나오지 않은 앞 부분에서도 아래에 정의된 것들이 인식이 가능한 것을 얘기하며, 직접 전방 선언을 해줘야 하는 c언어와는 달리 자바스크립트에서는 자동적으로 선언부를 위로 끌어 올려주는 hoisting 기능이 있다. **var**는 이런 hoisting 기능이 자동적으로 적용되어 변수가 선언되기도 전에도 변수에 값을 할당하는 비정상적인 상황이 발생한다.

---

## const(read only) 

- 상수 (<-> let), 최대한 변수들은 immutable variable로 만드는 것이 좋다.
- Immutable data types: primitive types, frozen objects (i.e object.freeze())
  - primitive 타입의 경우 만약 string을 선언했을 때 string을 전부 메모리에 올렸다가 변경시 다른 string을 메모리에 다시 올리는 방식으로 변경이 일어난다. 따라서 만약 'string' 에서  같은 string 내에서 i만 제외하도록 변경하는 것이 불가능
  - primitive 타입에서 value가 immutable 한 것이고, primitive를 참조하고 있는 variable은 mutable한 것으로 이해하면 된다. 기본적으로 자바스크립트에서 variable은 primitive value를 reference 하는 것이다.
  - variable을 변경한다는 것은 "When you "change the value of a variable" you are changing the string (or whatever) that the variable references, not the value itself."
- Mutable data types: all objects by default
  - 선언된 객체는 객체 안에 데이터를 변경할 수 있기 때문에 mutable
  - 모든 객체는 기본적으로 mutable

---

## Variable types

- primitive, single item : number, string, boolean, null, undefined, symbol
- object, box container
- function, first-class function

- Infinity = 1 / 0

  NegativeInfinity= -1 / 0

  NaN = 'not a number' / 2

- bigint : 123412341234123412432134n (숫자 뒤에 n을 붙이면 bigint가 됨)

  - 크롬과 파이어폭스만 지원

- 자바스크립트에서는 문자열이든 문자든 모두 string으로 인식됨

  - ```javascript
    const HelloBob = 'hi ${brendan}!';
    console.log('value: ${helloBob}, type: ${typeof helloBob}');
    console.log('value:' + helloBob + ' type: ' + typeof HelloBob);
    ```

- boolean

  - false: 0, null, undefined, Nan, ''
  - true: any other value

- null & undefined

  - null은 empty로 텅텅 빈 것을 의미하고, undefined는 선언은 되었지만 값이 초기화도 되지 않았고, 할당도 되지 않아 값이 비었는지 들어가 있는지 확인조차 불가능 한 것을 의미

- symbol : map이나 다른 자료구조에서 식별자용 key값이 필요하거나 여러 코드가 동시다발적으로 일어날 때 우선순위를 부여하는 고유한 식별자로 사용. String 타입으로 key 값을 설정해 줬을 때 같은 문자열이면 동일한 key 값으로 인식하지만 symbol의 경우 동일한 내용의 symbol로 만들었어도 다른 것으로 인식한다. (`Sysbol.('id')`)

  - `Symbol.for('id')` : id string에 대한 심볼을 만들어달라는 것으로 이 때에는 다른 곳에서

    `Symbol.for('id')` 로 심볼을 만들 경우 위의 심볼 선언과는 달리 두 경우가 동일한 것으로 인식된다.  

  - symbol을 출력할 때에는 `symbol1.description`을 사용하여 symbol을 description을 통해 string으로 변환해야 에러 없이 출력이 가능하다.
  
- 객체를 const로 선언하면 객체를 가지고 있는 참조변수가 const로 선언되어 있기 때문에 한번 참조하는 객체를 바꿀 수 없지만, 객체 안에 선언된 멤버 변수들은 멤버접근 연산자 . 을 사용해서 값을 변경할 수 있다.

---

## Dynamic typed language

- 자바스크립트는 c와 자바처럼 변수가 컴파일 타임에 결정되는 것이 아니라 런타임에 변수형이 결정되는 언어이다.
- 아이디어를 빠르게 구현하는 용으로 프로토타입을 만들 때에는 용이하지만 규모가 있는 프로젝트를 만들 때에는 문제가 생길 수 있다.



