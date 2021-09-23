# CSS 수업

- css는 마크업 언어가 실제 브라우저에서 어떻게 표현되는지를 지정하는 언어
- 웹 디자이너는 css를 통해 html의 내용을 디자인
- 사용방법
  - 외부 파일 사용(css 파일) -> html에서 해당 파일을 불러와서 적용
  - 내부 파일 사용
  - 태그로 사용

- 표현

  - <link href="파일 경로" rel="stylesheet" type="text/css">
  - 내부 파일 사용

  `<style type="text/css">
      스타일 정리
  </style>`

  - `<p style ="color : blue;"></p>`

- selector를 통해 target을 정하고 해당 target에 적용할 내용을 style 태그 안에 적어줌

- id 속성: 태그들 중 같은 (많은 것들 중 유일한)id를 가진 것을 셀렉트할 수 있게 해줌

  style 적용:  **#id명 {}** 로 적용

- class 속성: 태그들 중 반복적으로 사용되는 것을 class로 묶고 해당 class에 일괄적으로 적용하기 위해 사용

  style 적용: **.class명 {}**로 적용

