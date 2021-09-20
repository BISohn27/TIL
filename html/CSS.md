# CSS

- **Cascading Style Sheet** 의 약자

  - 위에서 아래로 연속에서 떨어지는 방식으로 스타일링을 하는 것

    1. 처음에 사용자가 작성한 스타일링 시트가 있는지 확인

    2. 사용자가 작성한 스타일링 시트가 없으면 브라우저에서 기본 제공되는 것이 있는지 확인

  - Author style : html 작성자가 지정한 스타일링 (가장 우선 순위)

  - User style : 클라이언트가 지정한 스타일링 (중간 우선 순위)

  - Browser style : 브라우저에서 기본 값으로 제공하는 스타일링 (가장 하위 우선 순위)

- **!important** : css에서 기본으로 정의된 연결 고리를 무시하고 가장 우선으로 처리되는 스타일링

  보통의 경우 html 박스 구조에 문제가 있거나 스타일링이 제대로 적용이 안될 때 사용됨

- 커다란 body 안에 section들을 나누고 section들 안에도 작은 박스로 나눠서 labeling이 쉬운 구조로 만드는 것이 가장 좋음

  -> labeling을 잘해야 css 시 박스들을 골라서 스타일링 하기가 좋다.]

- 박스들을 적절한 자리에 배치하는 것이 가장 중요

- css 값을 적용할 때에는 모든 브라우저에서 해당 값을 지원하는지 먼저 확인해야 하는데 MDN사이트에서 확인을 해도 되고 [Can I use](https://caniuse.com/) 에서 확인할 수도 있다.

---

## Selectors

- css를 적용할 범위를 설정하는 것
- 종류
  - Universal '' ***** ''
  - Type '' **Tag** ''
  - ID '' **#id** ''
  - Class ''** .class** ''
  - State '' **:** ''
  - Attribute '' **[]** ''
- [CSS Diner](https://flukeout.github.io/)

----



## Span, Div

- Span
  - span은 안에 내용이 있어야 나타남
  - span은 inline level
  - display를 통해 레벨을 변경할 수 있음
    - block : 블럭 레벨로 변경 -> 블럭으로 변경될 경우 css 적용 값은 content에 상관없이 보여짐
    - inline : 인라인 레벨로 변경
    - inline-block

- Div

  - div는 블럭 레벨이기 때문에 안에 내용이 없어도 css 적용 값이 보여짐

  - div는 block level

  - display를 통해 레벨을 변경할 수 있음

    - block : 블럭 레벨로 변경
    - inline : 인라인 레벨로 변경 -> 인라인으로 변경될 경우 css 적용 값은 content가 있을 때 보여짐
    - inline-block

    

- **Inline** : content 자체만을 꾸며주는 것으로 안에 내용이 있어야 css 적용 값이 보여진다.

  -> css 내부에 정의한 width나 height는 무시되고 content에 따라 크기가 정해진다.

- **inline-block** : inline block은 기본적인 배치는 인라인 형태로 들어가나 css값이 inline과 달리 css 에서 설정된 값이 적용된다.

---



## Position

- **position** 값
  - *top left*
  - *top right*
  - *bottom left*
  - *bottom right*
- css 기본 값으로 **static**을 가지고 있으며 html에 정해진 순서대로 브라우저 상에 자연스럽게 보여지는 것으로 positon이 static 값을 가지면 position 값을 넣어도 default 값으로 배치된다.
- position 값을 **relative** 값으로 바꾸면 나보다 먼저 배치된 item과의 default 값만큼 떨어진 위치에서 css로 입력한 값만큼 이동하여 배치됨
- position 값을 **absolute** 값으로 바꾸면 해당 item이 담긴 box의 default 값(시작점)에서 css 값만큼 변경
- position 값을 **fixed** 값으로 바꾸면 box나 item의 위치에 관계없이 윈도우 상에서 가장 상단 시작점으로부터 css 값만큼 떨어진 위치에 배치된다. (웹 브라우저 상에 가장 상단에 표시되는 위치)
- position 값을 **sticky** 값으로 바꾸면 위치는 *static* 와 동일한 위치이나 스크롤링 시 sticky로 설정된 item이 고정된 위치에서 계속 보여진다.

---



## Flex box

- 박스와 아이템을 행이나 열로 자유롭게 배치시켜 줄 수 있는 기능

- 박스가 커지거나 작아질 때 박스 내부의 아이템들이 어떻게 배치되어야 하는지를 자유롭게 정의할 수 있다.

- 기존에는 박스 내 배치를 position, float, table 등을 통해 구성하였으나 너무 복잡하고 아이템들을 vertical 방향으로 정렬하거나 사이즈에 관계없이 동일한 간격으로 배치하거나 동일한 높이로 배치하는 것이 힘들어 제약 사항이 있었다.

- 2가지의 주된 내용

  1. flex box에는 **컨테이너(박스)에 적용되는 속성 값**과 **아이템에 적용되는 속성 값**이 존재한다.

     - container attribute

       - display

       - flex-direction : flex box 의 방향(column, reverse)

       - flex-wrap : item들을 한 줄에 가득차면 다음 줄로 넘어갈 것인지

       - flex-flow : direction과 wrap을 합친 것( `flex-flow: column , wrap` )

       - justify-content : 중심축에서 어떻게 item들을 배치할 것인지

         - `flex-start;` : 축의 왼쪽에 배치(row) , 축의 위쪽에 배치(column)
         - `flex-end;` : 축의 오른쪽에 배치, 축의 아래쪽에 배치(column)
         - `center;` : 축의 가운데에 배치
         - `space-around;` : 박스에 스페이스를 넣는 것, 맨 왼쪽과 오른쪽에는 스페이스 조금, 아이템 가운데에는 스페이스 좀 더 많이 넣어서 축이 꽉 차게
         - `space-evenly;` : space-around에서 간격이 일정하게
         - `space-between;` : space-around에서 시작과 끝을 화면에 딱 맞게 배치

       - align-items : 한축의 아이템들을 반대축에서 어떻게 item들을 배치할 것인지

         - `center` : 반대축에 가운데에 배치
         - `baseline` : 아이템의 중심이 각각의 반대축에 대해서 중앙에 오게끔 배치

       - align-content : 여러 줄의 아이템 축을 각각 어떻게 배치할 것인지

         - `space-between` : 가장 위와 가장 아래의 아이템 축은 화면 최상단과 최하에 붙

           ​							고 가운데에는 일정한 간격으로 배치

         - `center` : 여러개의 아이템 축이 가운데에 모두 몰리게 됨.

           

     - item attribute	

       - order : item의 표시 순서를 정함

       - flex-grow : 자신의 고유한 사이즈가 아니라 부여된 크기에 따라 커짐

       - flex-shrink : 화면이 작아서 아이템 크기가 줄어들 때 줄어드는 정도가 값으로 표기

       - flex-basis :  커질 때나 작아질 때 부여된 % 값에 따라 변함

       - align-self : 아이템 별로 정렬됨.

         

  2. flex box에는 **중심축과 반대축**이 존재

     - 한 축을 중심축으로 지정하면 그에 수직이 되는 축이 반대축으로 지정

     - 아이템들이 정렬되는 축이 중심축이 되고 그 중심축에 수직이 되는 축이 반대축으로 설정됨

       (만약 수직으로 아이템들이 정렬되면 중심축이 수직축이 되고 반대축이 수평축이 된다)

       

- flex 박스를 지정하기 위해서는 css attribute에 `display: flex;` 로 값을 줌

- [A Complete Guide to Flexbox | CSS-Tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

  

---



## Float

- 원래의 목적은 이미지와 텍스트를 어떻게 배치할 것인지 정리하는 기능

- 종류

  - **float: left** 

    이미지가 왼쪽에 배치되고 텍스트가 이미지를 감싸는 형태로 배치

  - **float: center** 

    이미지가 가운데에 배치되고 텍스트가 이미지를 감싸는 형태로 배치

  - **float: right** 

    이미지가 오른쪽에 배치되고 텍스트가 이미지를 감싸는 형태로 배치 

---



- 크기를 vh가 아닌 %로 주게 되면 %은 부모 높이에 %비율로 크기를 결정

- 만약 해당 태그가 최상위 태그라면 그 태그의 부모는 body

- vh(view point) : 보여지는 화면의 몇 %

- [Color Tool - Material Design](https://material.io/resources/color/#!/?view.left=0&view.right=0) 색상 확인 가능

  



