# Responsive Web

- 다양 사이즈의 모바일이 나와 데스트크탑 용으로만 만들면 다른 디바이스에서 컨텐츠에 문제가 생긴다.

- 반응형 웹은 디바이스 사이즈에 따라 컨텐츠의 크기가 자동으로 조절되는 것

- ***Content is like water.*** 

- 최근에는 반응형 웹을 만들기 위하여 **px** 수치를 사용하지 않고 아래와 같은 것을 사용

  - **Flex grid**
  - **Flex box**
  - **%**
  - **vw** (view width)
  - **vh** (view height)

- 사이즈만 아니라 레이아웃을 재배치하는 것은 CSS의 **Media Queries**를 사용

- 최근에는 해상도에 따른 디바이스 브레이크 포인트가 모호하나 대략적으로 나누면

  - mobile : 320px ~ 480px
  - tablet: 768px ~ 1024px
  - desktop: 1024px~

- ```css
  @media screen and (min-width:800px) {
      .container{
          width: 50%;
      }
  }
  ```

  - screen의 최소(적어도) 넓이가 800px이면 컨테이너의 스타일링을 넓이 50%로 하라
  - screen, speech, print, all
  - and only  ,(comma)
  - min-width, width, max-width