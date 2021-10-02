# JavaScript8

- 자료 구조: 비슷 특징을 가진 것들을 한 container에 넣는 것으로 어떤 형식으로 담는지가 중요하다.

- 예를 들어 토끼와 당근은 각각이 객체이고, 이런 객체들 중 비슷한 특징을 가진 것을 자료구조를 통해 한 곳에 넣을 수 있다.
- 자바스크립트는 dynamically tayped lanuage(런타임에 자료형이 결정되는 언어)이기 때문에 코드를 작성하고 있을 때에는 자료형이 결정되지 않아 모든 객체를 종류에 관계없이 자료구조에 담을 수 있다. (가능은 하나 실제로 프로그래밍에서 권장하지는 않는 방법)

## Array

- 인덱스가 부여된 자료구조 (기본적으로 스택같은 구조로 되어 있는듯함)

- 한 배열 안에 동일한 타입을 넣는 것이 좋다.

- 배열의 선언

  `const arr1 = new Array();`

  `const arr2 = [1,2];`

- 배열의 접근

  ```javascript
  //for
  for(let i = 0; i < array.length; i++){
      console.log(array[i]);
  }
  //for of
  for(let arr of array){
      console.log(arr);
  }
  //forEach
  array.forEach(function(arg,index,array){
      console
  });
  array.forEach((arg,index) => console.log(arg));

- 배열에 데이터 넣기,삭제,복제

  - (뒤에) 넣기

    `array.push('데이터1','데이터2')`

  - (뒤에서) 삭제

    `array.pop();`

  - (앞에) 넣기

    `array.unshift('데이터1','데이터2');`

  - (앞에 것을) 삭제

    `array.shift();`

  - shift와 unshift는 배열의 데이터를 뒤로 밀고 데이터를 넣고, 앞에 데이터를 빼고 뒤에 데이터들을 다시 앞으로 옳기는 작업을 수행하기 때문에 오버헤드가 생긴다.

  - 특정 인덱스의 데이터를 삭제

    `array.splice(start_index (,count);`

    - 시작하는 인덱스만 넣어주면 시작하는 인덱스 뒤에 모든 데이터를 삭제

    `array.aplice(start_index ,count,'데이터1','데이터2');`

    - 시작 인덱스부터 count개만큼 지우고, 그 자리에 뒤에 데이터1과 데이터2를 집어 넣음

- 배열의 결합

  ```javascript
  const array1 =[1,2]
  const array2 =[3,4];
  const newArray = array1.concat(array2); 
  ```

- 배열의 검색

  `array.indexOf(데이터)` : array에서 데이터가 몇번째 인덱스에 있는지를 반환 (없으면 -1)

  `array.include(데이터)` : array에서 데이터가 있는지 없는지를 불리언 타입으로 반환

  `array.lastIndexOf(데이터)` : 데이터가 중복되어 배열에 저장되어 있을 때 indexOf 메소드는 												가장 처음 검색되는 인덱스를 반환하고 lastIndexOf는 해당 데이												터의 가장 마지막 인덱스를 반환												

