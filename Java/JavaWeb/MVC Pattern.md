# MVC Pattern

1. ## Model

   - 데이터를 가진 객체로, DB와 연동되어 있다.

2. ## View

   - 사용자가 요청한 정보를 보여주는 화면(User Interface)으로 사용자가 요구한 정보를 Model 객체를 통해 DB에서 가져오면 DB는 View에 넘기게 되고, View는 가져온 정보를 사용자에게 보여주기 위한 웹페이지를 만든다. (html, css, js)

3. ## Controller

   - UI를 통해 사용자가 특정 정보를 요청하면, 해당 DB와 연동되어 있는 Model에 해당 정보를 가져오는 메소드를 실행하고, 실행된 결과에 맞는 UI가 실행되도록 뷰의 메소드를 실행한다.

---

1. ## Model 1 Pattern

   - 모델 1 패턴은 비지니스 로직 영역(컨트롤러)에 프레젠테이션 영역(뷰)를 같이 구현하는 것
   - 모델 1패턴은 복잡하지 않은 비니지스 로직에서 구현
   - 빠르고 쉽게 개발할 수 있으나 컨트롤러와 뷰가 혼재되어 있어 유지 보수에 어려움이 있다.

2. ## Model 2 Pattern

   - 모델 2 패턴은 컨트롤러 영역과 뷰 영역가 분리되어 동작하는 것
   - 디자이너와 개발자의 분업이 가능하다. 개발자는 컨트롤러(프론트 엔드)와 모델(백엔드)를 구현하고, 디자이너는 뷰 부분을 구현한다.
   - 설계 단계가 모델 1보다 복잡하여 개발 난이도가 높다.
   - 설계를 잘못하면 웹 애플리케이션 전체가 망가진다.

---

![Diagram to show the different parts of the mvc architecture.](https://developer.mozilla.org/en-US/docs/Glossary/MVC/model-view-controller-light-blue.png)

