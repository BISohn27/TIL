# HTML2

- 각 태그들을 적절한 기준에 맞게 상사로 분리하여 작성하면 labeling하기 편하다

  

  ![HTML Elements and Tags |](https://www.bluekatanasoft.com/wp-content/uploads/element-structure.png)



![HTML Attributes | OnlineDesignTeacher](https://4.bp.blogspot.com/-B5vUzJXNAoE/Vuay2ygsN2I/AAAAAAAAG5o/-qOAVBa3LRkJ0fPWywYzkAcmezRAY2Rxg/s640/html-syntax.png)



---

## Box

- 사용자에게 보여지지 않지만 태그들을 나누기 위해 작성
- header, nav, aside, main, footer로 나뉘고 각각의 영역에 세부 상자들을 만듬
- main -> 여러개의 section -> 각 section안에 여러개의 article
- **article** : 여러 개의 태그들을 그룹화하여 재사용 가능하게 모아 놓은 것 (내부에는 재사용되는 태그들로 구성되어 있음)
- **div** : 묶어서 스타일링을 할 필요가 있을 때 함께 스타일링을 적용할 태그들을 같은 div에 넣음

## Item

- 사용자에게 보여지는 부분
- a(anchor) , button, input, label, img, video, audio, map, canvas, table 등 사용자에게 보여지는 것
- 사용자에게 보여지는 item도 **Block** 과 **Inline** 으로 분류 할 수 있음
  - **Block** :  블럭 형태로 되어 있는 item은 그 다음 태그가 올 때 개행이 되어 배치
  - **Inline** : 인라인 형태로 되어 있는 item은 그 다음 태그가 바로 옆 남은 공간으로 배치

![Differences between html inline and blocks elements. - DEV Community](https://res.cloudinary.com/practicaldev/image/fetch/s--y9knEwLf--/c_imagga_scale,f_auto,fl_progressive,h_500,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/t6vbjui1jm8q52osgfcj.png)



---



## Tag

- **a(anchror)** : <a href= url>

- **ol** : 순서가 있는 리스트 [ (ol>li*3) and tab typing : 3개의 리스트가 바로 만들어짐]

- **ul** : 순서가 없는 리스트

- **input** : 원하는 데이터를 사용자로부터 받는 태그 , **label** 태그와 함께 사용하여 정확하게 입력받고 싶은 정보를 표시

  ```html
  <label for="input_name">Name: </label>
  <input id = "input_name" type="text">
  ```

  label에 for와 input의 id로 라벨과 인풋을 연결해준다.

  