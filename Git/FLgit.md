# Github

---

- 깃허브는 코드의 버전을 분산하여 관리하기 위한 시스템
- 중앙 집중형 시스템의 문제를 해결하기 위한 목적으로 사용
- 데이터를 하나의 서버가 아니라 *분산된 장소* 에 저장되어 있는 것
  - 블록체인의 경우 분산되어 장부가 저장되어 있기 때문에 한 사용자의 장부 조작에 대해 저항을 가짐
- 깃은 한 코드 소스에 버전을 부여하면 버전마다 *변경된 내용* 만을 저장 -> 속도 면에서 유리

### 분산형

- 하나의 컴퓨터가 모든 데이터를 저장하고 해당 데이터를 클라이언트가 요청할 때마다 주는 것이 아니라 클라이언트와 서버가 동시에 해당 파일을 소유하고 있는 형태

---

## Git Besh

- 윈도우에서 Git에 필요한 unix 명령어를 사용하기 위한 터미널

- 명령어
  - ***ls*** : 현재 디렉토리에 들어있는 파일 목록
  - ***cd <path>***  (change directory) : 디렉토리 변경
  - ***cd ..***  : 하나 상위 폴더로 이동
  - ***mkdir 생성폴더이름***  : (make directory) 폴더 생성
  - ***touch 생성파일이름.확장자***  : 생성파일이름의 확장자를 가진 파일 생성 (처음에는 파일의 날짜 변경으로 나옴)
  - ***rm 파일명.확장자***  : 파일을 삭제
  - ***rm -r 폴더명***  : 폴더 삭제 (rm 로는 directory 삭제 불가능)
  - ***cd ../RacingGround/*** : 상대경로

- README.md
  - git에서는 readme 파일에 설명을 넣음

---

## Repository

특정 디렉토리의 버전 관리하는 저장소

- ***git init*** : 명령어로 로컬 저장소를 생성
  - ***.git*** : 숨김 폴더로 저장되어 있는 디렉토리로 버전 관리를 하면 .git 폴더에 저장되어 추적
  - git한테 이제부터 해당 디렉토리를 버전 관리할 예정이라는 것을 알려줌

---

## Commit

- 특정 버전으로 남기는 것을 ***commit*** 임

- git은 commit을 3가지 영역을 바탕으로 관리를 함

  

  ### Working Directory : 내가 작업하고 있는 실제 디렉토리

  예) RacingGround에서는 RacingGround가 Working Directory (.git이 있는 디렉토리)

  

  ### Staging Area 

  ​	commit으로 남기고 싶은, 특정 버전으로 남기고 싶은 파일이 있는 곳

  

  ### Repository
  
  ​	commit들이 저장되는 곳

- untracked 상태(버전 관리 되지 않고 있음) (Working Directory) 

  ​	//.git이 생기더라도 아직 어느 파일이 버전 추적을 받을지 결정이 안되어 있기 때문에

  -> ***git add*** 를 통해 tracked하여 staged상태 만듬(Staging Area)

  ​	// git add 명령어를 통해 버전 추적 받을 파일을 명시하는 과정 , Staging Area는 작업 공간

  -> ***git commit*** 을 통해 commited된 상태로 만듬(Repository) ,  작업이 끝난 파일을 버전화함 

  ​	// 한번 commit을 하게 되면 더 이상 git한테 버전 관리 상태가 아님(저장된 상태)

  

- modified를 하게 되면 untracked 되는게 아니라 tracked 상태에서 modified됨

  ​	//tracked 상태이지만 modified되면 commit이 안되어 있기 때문에 일련의 과정을 다시 시행

- *git init* (디렉토리를 git으로 관리 시작) 

  -> *git add* (git으로 특정 파일을 버전 관리 시작)

  -> *git commit* (코드 작성이 완료되고 깃에 버전으로 올림)

- ***git add 파일명 파일명***  or ***git add .***  (.은 현재 폴더를 의미)

---





## Git 사용법

---

### Local Repository

- git bash와 계정 연결

  - git config --global user.email "skylord418@gamil.com"

  - git config --global user.name "BISohn27"

- ***git log***  : commit 뒤에는 사용자를 식별할 수 있는 고유 번호가 존재

- ***git status***  : 해당 directory에 commit 할 파일이 존재하는지 나옴
  - tree clean은 commit할 파일이 존재하지 않는 것

- ***git commit -m "commit message"*** 

  - 작성이 끝난 코드를 commit하는 명령어

  - commit message는 최대한 자세하게 (다음에도 어떤 의도로 남겼는지 알 수 있도록)

    

- 버전 업을 하고 싶은 파일만 commit으로 남기고 나머지는 그대로 두기 위해서 Staging Area를 사용. 

  - 수정 완료된 파일은 add를 통해 Staging Area 에 올리고 commit하면 버전 업이 가능

  - 수정은 했으나 아직 완료되지 않은 파일은 새로운 버전으로 올리지 않기 위하여 *add를 하지 않으면 commit 시 버전이 업 되지 않는다.* 

    ​	-> 추후 개발 완료 시 버전 업 하지 않는 파일만 add를 통해 Staging Area로 올리고 commit		하면 해당 파일만 버전 업으로 관리 가능

- ***git diff*** *A B* 

  - commit identifier에서 앞의 4자리만 적어주면 구별 가능
  - 앞의 아이디(A)를 기준으로 해서 뒤의 아이디(B)의 변경 내용을 마지막 문단에서 비교해줌

---

### Github Repository

- public : 올리는 것은 계정주만 보는 것은 가능
- private : 개인 저장소



- ***git remote add origin {github repository url}*** 
  - git repository에 올릴 때 파일들을 올리는게 주요 목적이 아니라 commit 단위로 변경 사항을 올리는 것이 목적
  - origin 
- ***git push -u origin master*** 
  - origin(remote repo)으로 master branch(local repo)에 있는 모든 commit들을 올린다
  - master라는 줄기가 만들어지고, 추가된 commit들이 전부 올라간다
  - 한번 git repository와 연결이 되면 -u 없이 명령어를 치면 push가 가능
- ***git clone {github repository url}*** 
  - git bash here 에서 git clone 주소를 복사해 넣으면 원격 레포가 만들어짐
  - git clone url . 하면 현재 디렉토리에 local repo를 형성



1. local을 먼저 만들고 master를 쌓은 다음에 git repo에 연결하여 push를 하거나

2. git repo를 먼저 만들고 local에 clone을 한 다음 master를 쌓는 것 두가지 가능

---



