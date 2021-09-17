# 복습

- ***~/*** : 운영체제가 설치되어 있는 드라이브의 user 폴더에 현재 로그인된 계정의 폴더가 home

  ex) c:/사용자/skyl/

- 특정 디렉토리가 repository로 지정되어 있으면 master 표시가 된다.

- git init(Working Directory) ->git add(Staging Area) -> git commit -m ->(Repository)

- ***git remote rm origin*** 오리진을 삭제하는 명령어

- [Free Bootstrap Themes, Templates, Snippets, and Guides - Start Bootstrap](https://startbootstrap.com/) 

  html, css, js 템플릿

# Github Pages

- 깃헙에서 제공하는 무료 웹 서비스

- 레포지토리에 푸시를 하게 되면 일정 시간이 지난 후에 자동으로 깃헙 페이지로 올라감.

- 각각의 레포지토리에 깃헙 페이지를 따로 만들 수 있다.

- username.github.io로 repository를 만든 후 html 파일로 내용을 작성하고 푸시를 하면 

  https://username.github.io 로 깃헙 페이지가 만들어짐.

- 크롬에서 오른쪽 마우스 클릭 검사 항목에 들어가면 수정하고 싶은 코드가 어디에 위치해 있는지 알 수 있다.

# git pull origin mater

​	origin에서origin에 있는 *(remote repository)master branch* 에 있는 변경 사항을 당겨오는 것.

- pull을 해오기 전에 변경 사항이 있으면 git에서 오류를 내고 pull을 먼저하라고 하며, 변경 사항을 저장한 상태에서 pull을 하면 git repository에 저장된 내용은 그대로 pull 해오고 local에 변경 사항과 당겨온 사항을 정리하라는 기능이 발동함

# 취소 명령어

- git add를 통해 Staging Area에 올린 파일 중 commit을 아직하지 않을 파일을 staging area에서 다시 내릴 때 **git restore --staged file명** 로 해당 파일에 대한 add를 취소할 수 있다.
- 현재 local repository에서 현재 작업한 상태를 취소하고, 가장 최신으로 commit된 상태로 원복 할 때 **git restore 파일명** 으로 할 수 있다. (현재 작업하고 commit하지 않은 모든 변경 사항이 없어짐) -> add를 통해 Staging Area에 올린 변경 사항은 취소 되지 않음. 
  - untracked 상태일 경우에는 추적이 안되기 때문에 오류



- **git reset** 

  - commit을 이전버전으로 되돌리는 역할

  - *git reset -hard 커밋id* : 커밋id 상태로 파일 상태를 돌림.

    -> Working Directory, Staging Area, 를 전부 해당 커밋으로 만듬

    -> git pull origin master 하면 가장 최신 커밋까지 다시 돌릴 수 있다.

  - *-mixed*  : Staging Area 와 Repository를 해당 커밋 상태로 만든다. 

  - *-soft* : Repository만 해당 커밋 상태로 돌린다.

    

- **.gitignore** 

  - id, pw 같은 개인 정보가 들어있는 파일을 git에 연동시키지 못하도록 한다.

  - .gitignore는 .git이 있는 directory에서 생성해줘야 한다.

  - gitignore 파일을 열고 git에 연동하고 싶지 않은 데이터를 적음, 하위 디렉토리에 있는 파일도 경로 설정 없이 파일명만 적어주면 ignore됨

  - Repository를 만들어주고 (git init) 한 직후에 바로 .gitignore를 만들어주고, 필요할 때마다 목록을 작성한다.

    - 특정파일명 ( 파일명.확장자명 )
    - 특정 폴더 ( 폴더명/ )
    - 특정 확장자 ( *.확장자명 )
    - 모든 png 확장자들은 ignore하되, 특정 파일은 ignore하지 않음 ( !profile.png)

    -> 빅데이터를 관리할 때 데이터는 git 연동에서 제외하고 소스 코드만 관리하는 경우가 많음

  - https://www.toptal.com/developers/gitignore 에 가면 소스 코드를 올릴 때 불필요한 파일을 ignore 해주는 템플릿을 제공

# Shared Repositroy Model

동일한 저장소(Repository)를 공유하여 활용하는 방식

- 팀장 : repository owner(project manager)
- 팀원 : collaborator(a.k.a no-yeah-contributor)

- collaborator는 팀장이 만든 repository에 접근하여 push할 권한이 생긴다. 그 외에는 public의 경우 소스 코드를 보는 것만 가능
- 팀장과 팀원이 push pull로 서로 작업을 함.

# Branch

​	특정 커밋을 가리키는 **'포인터'** 

- master branch로 commit들을 이어 나가는 형태

- master branch에서 feature-A라는 새로운 branch를 만들고 새로운 branch를 원래 branch와 다른 목적으로 사용할 수 있다.

  ex) master branch에는 오류 없이 동작하는 코드만을 잇고, 새로운 branch에는 테스트용 코드를 만든다.

- 새로운 branch 개발이 끝나 완전히 동작하게 되면

  1. 완성된 branch를 master branch로 merge 하는 방법
  2. 완성된 branch를 master branch와 합쳐서 수정하고 merge 방법

  

  

  ## Branch basic commands

  1. 브랜치 생성

     (master) $ git branch {branch name}

  2. 브랜치 이동

     (master) $ git checkout {branch name}

  3. 브랜치 생성 및 이동

     (master) $ git checkout -b {branch name}

  4. 브랜치 목록

     (master) $ git branch

  5. 브랜치 삭제

     (master) $ git branch -d {branch name}



- **git merge 브랜치이름**

  - master branch가 뻗어나간 branch의 가장 최신 커밋을 가르키게 됨

  - 뻗어나간 branch도 master branch에 포함되어 있기 때문에 master branch가 뻗어나간 branch의 가장 최신 commit을 가리키는게 가능하다.
  - **fast forward merge** : 기존 master branch에서 뻗어나온 branch와 master branch가 충돌이 없어 바로 합쳐질 수 있는 병합
  - fast forward merge가 안되는 상황 : 뻗어나간 branch는 master branch와 동일한 파일을 가지고 서로 다른 수정을 하는 경우가 많기 때문에 내용에 충돌이 발생할 수 있어 master branch에 merge를 할 때 합쳐주는 직접 수정이 필요할 수 있다. -> **merge commit** 

  

  

  - **git log --graph** 브랜치 그래프
  - **git log -- graph --oneline** 브랜치 그래프 간략화

- **branch** 는 만들어야 하는지 고민이 될 때는 일단 만들면 된다.

  -> 복사가 되는게 아니라 포인터가 생성되는 것이기 때문에 브랜치를 만들고 새로운 시도를 함

  -> 잘 작동되는 소스 코드는 master branch에 남아 있기 때문에 뻣어나간 branch는 지우고    	          	master branch로 돌아가면 된다.

- 협업 시 git branch 브랜치명 으로 새로운 브랜치를 만들고 해당 브랜치를 git push origin 브랜치명으로 푸시를 한 후에 pull request 요청

- 협업 하는 방법

  1. collaborator로 지정된 repo를 내 local repo로 clone함

  2. 해당 local repo로 가서 새로운 브랜치를 만들고 해당 브랜치로 이동한 다음에

  3. 소스 코드 수정을 하고

  4. 해당 코드를 owner repo로 새로 만든 브랜치 명으로 add commit push를 하고

  5. 새로 만든 브랜치를 pull request를 통해 merge 요청을 한다.

     **master branch는 

## Git Flow

- Master(main) : 배포 가능한 상태의 코드

- Develop(main) : 개발 진행, 발생된 버그 수정 등 개발 진행, 개발 이후 release branch로 갈라짐

- feature_branches(supporting) : 기능별 개발 브랜치, 기능이 반영되거나 드랍되는 경우 브랜치 삭제

- release_branches(supporting) : 개발 완료 이후 얻어진 버그를 배포 전 fix

- Hotfixes(supporting) : 긴급하게 반영 해야하는 bug fix

  -> release branch는 다음 버전을 위한 브랜치

  -> hotfix branch는 현재 버전을 위한 브랜치

  -> main은 항상 살려 놓는 branch이고 supporting은 그때 그때 필요할 때 만들었다 끝나면 없애는     	branch

# Fork & Pull Model

- 내가 해당 프로젝트 repository에 직접적인 push 권한이 있는지 여부
  - Shared Repository Model : push 권한이 있음
  - Fork & Pull Model : push 권한 없이 Pull Request로만 코드 수정이 가능
- 공개된 repo를 내 remote repo로 클론하는 것을 fork
- local로 clone을 하여 새로운 branch로 작업을 함.
- collarborator로 지정해주지 않았기 때문에 원래 repo로는 push하지 못하고, clone 해 온 내 remote repo에만 push 가능.
- 내 remote repo에서 PR로 원본 repo 소유자에게 merge 요청을 하고, repo 소유자가 코드 리뷰 후 괜찮으면 merge를 해줌.



- Fork 순서

  1. 원하는 타인 Repository에 가서 Fork 버튼을 누름

  2. 내 Repo에 오면 타인 Repo와 동일한 이름을 가진 저장소가 클론됨

  3. 해당 파일을 내 Local Repo에 클론시키고

  4. 새로운 브랜치를 만들어 브랜치 이동을 한 다음에

  5. 내가 원하는 방식으로 코드를 수정하고

  6. 수정한 코드를 새로 생성한 branch 명으로 add commit push 하고

  7. 내 git repo로 와서 pull request를 원작자 master branch로 보내면 된다. 

     이 때 내 repo 브랜치는 새로 만든 branch로 선택





추가 학습

git reset

git rebase

https://aidenlim-edu.github.io/class-git-advanced/#64

https://hphk.notion.site/hphk/Git-Github-eb889d145316478fb57d0123b698a982