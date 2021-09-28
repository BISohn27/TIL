# JavaScript2

- [MDN Web Docs (mozilla.org)](https://developer.mozilla.org/ko/)

  자바스크립트의 공식 사이트를 대신하는 역할

- head 태그 안에 external 방식으로 자바 스크립트 파일 실행

  ![async defer 1](https://jess2.xyz/static/c1080099bafd9156190cc6c2e5da9820/23313/async-defer-1.jpg)

  - head 태그 안에 포함된 태그들은 브라우저가 한줄 한줄 parsing하여 분석하고 이해하며 이렇게 분석한 내용을 CSS와 병합하여 DOM요소로 변환하게 된다. 이때 스크립트 태그를 만나고 스크립트 태그 안에 자바스크립트 코드가 포함된 url을 만나게 되면(external) 해당 url의 파일을 다운로드 받기 위하여 html 태그를 parsing 하는 것을 멈추고 해당 스크립트 파일을 서버에서 다운 받아 코드를 실행한 다음에 다시 다음 내용을 parsing 하게 된다.
  - head에 external 방식으로 자바스크립트를 실행하게 되면(클라이언트 측 웹 브라우저에서 해당 스크립트 파일을 다운로드 받게 만드는 방식) 스크립트 파일의 크기가 크고, 클라이언트 인터넷 속도가 느릴 때 웹 브라우저의 동작 속도가 느려지는 단점이 발생

- body 태그 가장 마지막에 자바스크립트 파일을 실행

  ![async defer 2](https://jess2.xyz/static/222eb0d4f1704dbe5a17d7e56283bd7e/23313/async-defer-2.jpg)

  - body의 가장 마지막에 external 방식으로 자바스크립트 파일을 실행하게 되면 html 코드를 모두 parsing하여 클라이언트 측 웹 브라우저에 표시 한 후에 자바스크립트 파일을 fetching 하여 실행하기 때문에 사용자가 컨텐츠를 접하면서 웹 브라우저를 이용 가능하다.
  - 자바스크립트에 의존적인 웹 페이지라면 기본 html 페이지와 자바스크립트 파일이 받은 후 표현되는 페이지 간 차이가 발생하여 사용자가 보는 컨텐츠와 실제 서버가 제공하고자 하는 페이지가 특정 시간동안 다를 수가 있다.

- head 안에 **asyn** 속성값을 적용하여 external 방식으로 자바스크립트 파일을 실행

  ![async defer 3](https://jess2.xyz/static/7427dae98074c9f48b9b0f375fa7aec1/23313/async-defer-3.jpg)

  -  **asyn** 라는 불리언 타입의 속성 값을 사용하여(선언만으로 true로 되어 asyn 속성 값을 사용 가능) 자바스크립트 파일을 실행한다. 이럴 경우 해당 파일을 다운로드 하는 것과 html 태그들을 parsing 하는 것이 병렬로 처리되어 html 태그 parsing과 파일 다운로드가 병렬적으로 진행되어 파일 다운로드가 완전히 종료되었을 때 parsing을 멈추고, 다운로드된 파일을 실행한다. 그 후에 다시 남은 parsing을 진행.
  - body 태그의 끝에 스크립트 태그를 넣었을 때와는 달리 html 태그가 parsing되는 과정에도 스크립트 파일의 다운로드와 fetching이 병렬적으로 진행되기 때문에 fetching의 속도가 좀 더 빠를 수는 있다.
  - 하지만 fetching이 완료되었을 때 자바스크립트에서 발생시키는 이벤트가 아직 완전히 parsing되지 않아 html 태그가 실행되기도 전에 자바스크립트 이벤트가 발생하는 위험이 생길 수 있고, 자바스크립트 파일의 fetching이 끝나 parsing을 막고 자바스크립트 파일을 실행하는 동안 여전히 사용자가 페이지 지연을 경험할 수 있다.

- head 안에 **defer** 속성값을 적용하여 external 방식으로 자바스크립트 파일을 실행

  ![async defer 5](https://jess2.xyz/static/862f098cb6be83ebcf3596912338cee0/23313/async-defer-5.jpg)

  - **defer** 속성 값은 **asyn** 속성 값과 비슷하게 html parsing 과정에서 defer을 만나면 html parsing은 계속 진행하면서 자바스크립트 파일 fetching을 병렬적으로 처리하지만, fetching이 끝남과 동시에 parsing을 막고 자바스크립트 파일을 실행하는 asyn과 다르게 defer는 우선 html parsing을 완전히 끝내놓고, 이전에 fetching된 스크립트 파일을 실행한다.

  - 가장 효율적인 방식

    

- 다수의 스크립트 파일을 asyn 방식으로 실행하게 되면 각각의 스크립트 파일이 fetching될 때마다(정의된 순서와 관계없이 먼저 다운되는 순서로) parsing이 멈추는 비효율적인 상황이 발생하고 또한 자바스크립트 파일이 정의된 순서대로 page에 적용되는 flow인 경우 먼저 선언된 자바스크립트 코드가 필요한 시점에 다른 자바스크립트 코드가 실행되는 상황이 발생할 수 있다.

  ![async defer 4](https://jess2.xyz/static/b68b95e5ada9e533844f710190813249/23313/async-defer-4.jpg)

  

- defer 방식의 경우 다운로드가 먼저 된다고 하더라도, html parsing이 완전히 끝난 이후에 선언된 순서대로 자바스크립트 파일이 실행되기 때문에 다운로드 순서에 영향을 적게 받는다.

  ![async defer 6](https://jess2.xyz/static/ca0e70cca3c3b4d5046503a25f6104a1/23313/async-defer-6.jpg)



- 'use strict';
  - 선언되지 않은 변수에 값을 할당하거나 기존에 존재하는 프로토타입을 변경하는 등 정상적이지 못한 부분을 막아줌.

