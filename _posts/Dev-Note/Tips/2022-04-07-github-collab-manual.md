---
title: "[Tips] Issue 기반 GitHub 협업 매뉴얼 (w/ Git Flow)"
date: 2022-04-07 19:50:00 +0900
categories: [Dev Note, Tips]
tags: [github, git-flow, collaboration, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---

<br>

부스트캠프 기간 동안 대회나 최종 프로젝트 등 GitHub을 이용하여 협업을 경험해볼 수 있는 기회가 정말 많기 때문에 이번을 기회로 팀원들과 GitHub을 제대로 활용해보면 좋겠다 싶어 **협업 규칙**을 제대로 세워보았다! 특히, 멘토님께서 알려주신 **Git Flow**와 **Issue**도 나중에 최종 프로젝트에서는 유용하게 활용할 수 있을 것 같아 초반 협업 규칙 수립에 상당한 시간을 투자한 것 같다.. ㅎㅎ 
본 포스팅에 담긴 방법은 우리 ❄️ #눈#사람 ❄️ 팀만의 방법이기 때문에 절대적인 정답은 아니겠지만, 이를 조금씩 고쳐나간다면 최적의 협업 방법을 찾을 수 있지 않을까 싶다 😎

<br>

![](/assets/img/posts/Dev-Notes/Tips/2022-04-07-github-01.png)
![](/assets/img/posts/Dev-Notes/Tips/2022-04-07-github-03.png)
_시작 전에 보고 가면 유익한(?) 나의 부끄러운 과거... 😅 커밋 메시지를 이렇게 쓰면 절대 안 된다!_

<br>

## 1. 사전 작업
### `.gitignore` 생성하기
> 사용하는 언어나 프로젝트 구성에 따라 `.gitignore`의 내용은 달라질 수 있다.

```
.DS_Store
__pycache__
*.pyc
.ipynb_checkpoints
*/.ipynb_checkpoints/*
```

### GitHub Project의 Automation 설정하기
To do, In progress, Review in progress, Done 등 **GitHub Project 칸반보드**에서 확인하길 희망하는 status에 대해 automation을 설정한다. (이를 설정하지 않고 manual 하게 칸반보드를 조작해도 괜찮지만 상당히 귀찮다!)

### `pre-commit` 설정하기
> Commit 할 때마다 자동으로 수행하고 싶은 동작(ex. 코드 style 검사 및 통일하기 등)을 `pre-commit`으로 등록할 수 있다.

1. `.pre-commit-config.yaml` 파일을 작성하고 repository에 push 한다. (우리 팀의 경우는 `flake8`과 `black`을 등록하였으며, `yaml` 파일을 `.github` 폴더 밑에 넣었다.)
   
   ```yaml
   # See https://pre-commit.com for more information
   # See https://pre-commit.com/hooks.html for more hooks
   repos:
   -   repo: https://github.com/pre-commit/pre-commit-hooks
       rev: v3.2.0
       hooks:
       -   id: trailing-whitespace
       -   id: check-yaml
       -   id: check-json
   -   repo: https://github.com/psf/black
       rev: stable
       hooks:
       - id: black
       language_version: python3.8
   -   repo: https://gitlab.com/pycqa/flake8
       rev: 4.0.1
       hooks:
       - id: flake8
   ```
   
2. `pre-commit`을 설치한다.
   
   ```bash
   pip install pre-commit
   cd .github
   pre-commit   # 설치 확인을 위한 명령어
   ```

3. git clone / pull을 통해 repository에 존재하는 `.pre-commit-config.yaml` 파일을 로컬로 불러온다.
   
4. 이후 commit 시, `git add`로 staging area에 변경 사항을 추가하고 다음과 같이 `pre-commit`을 실행하면 등록한 동작이 수행된다.
   
   ```bash
   pre-commit run
   ```

5. 본 예시에서는 `black`으로 코드를 수정까지 해주기 때문에 `git status`로 확인했을 때 staging area에서 누락된 파일들이 존재할 수 있다. 그렇다면 다시 `git add` 후 `pre-commit run`을 수행한다. 이후에 코드가 더이상 수정되지 않는다면 `git commit`을 하면 된다.
   
6. 이를 자동화하고 싶다면 다음의 명령어를 실행한다. 이렇게 하면 `pre-commit run` 명령어를 수행하지 않아도 자동으로 수행된다고 한다.
   
   ```bash
   pre-commit install
   ```

### Commit Message Template 적용하기
1. 팀원끼리 정한 Commit Message 템플릿을 `.gitmessage.txt` 파일에 작성한다. (우리 팀의 경우는 해당 파일도 `.github` 폴더 밑에 넣었다.)
   
2. git clone / pull을 통해 repository에 존재하는 `.gitmessage.txt` 파일을 로컬로 불러온다.
   
3. 다음의 명령어로 git commit template을 설정한다.
   
   ```bash
   git config --local commit.template .gitmessage.txt   
   # 프로젝트 루트 디렉토리에서 수행하려면 .github/.gitmessage.txt
   ```

4. 이렇게 하면 `git commit` 시 해당 내용이 보여지게 된다. 템플릿 내용대로 작성 및 저장한 후 `git push`를 수행하면 된다.

### `develop` 브랜치 생성하기
Git Branch 전략인 **Git Flow**를 도입하기 위해 `main` 브랜치를 복제한 `develop` 브랜치를 생성한다. 기능을 추가할 때마다 해당 `feat` 브랜치를 `develop` 브랜치에 merge 하는 형식이 된다.

```bash
git branch develop  # main 혹은 master 브랜치에서 수행
git push -u origin develop
```

![git-flow](/assets/img/posts/Dev-Notes/Tips/2022-04-07-github-05.png){: width="50%" height="50%"}
_Git Flow를 설명하는 유명한 그림_

### 기타 작업 (Issue / PR Message Template)
그 외에 **Issue Template**이나 **Pull Request Template**을 작성한다. 우리 팀의 경우에는 이 또한 모두 `.github` 폴더 밑에 넣어 관리했다.

![.github](/assets/img/posts/Dev-Notes/Tips/2022-04-07-github-04.png){: width="70%" height="70%"}
_`.github` 폴더 안의 파일들 (예시)_

![issue-template](/assets/img/posts/Dev-Notes/Tips/2022-04-07-github-06.png){: width="70%" height="70%"}
_`.github/ISSUE_TEMPLATE` 폴더 안의 파일들 (예시)_

<br>

## 2. 전체 과정
1. 새로운 기능을 추가하는 등 GitHub Repository에 나의 작업물을 반영하기 위해 **관련 Issue를 작성**한다. 이때, Assignees, Labels, Project를 설정하고, Project의 경우에는 In Progress로 옮긴다.
   
    > 작업 시작 전에 Issue를 먼저 올릴지 말지 고민이 된다면, 임시로 이름을 지어 브랜치를 만든다. 그 후에 Issue를 작성하고 해당 브랜치 이름을 규칙에 맞게 바꿔주면 된다.
    > ```bash
    > git branch -m [변경 전 이름] [변경 후 이름]
    > ```

2. `develop` 브랜치(추후 merge를 원하는 브랜치)를 **최신 내역으로 업데이트** 한 후, 내가 **작업할 브랜치를 생성**한다. 이때, 브랜치의 이름은 팀 내의 규칙대로 설정한다. 
   
    > 우리 팀의 경우, 새로운 기능을 추가하는 브랜치의 이름은 `feat/[issue-num]`으로 명명했다. `issue-num`은 GitHub Repository에서 Issue를 생성하면 확인할 수 있다.
   ```bash
   git pull    # 최신 내역으로 업데이트
   git status  # develop 브랜치에 있음을 확인
   git checkout -b feat/21
   git status  # feat/21 브랜치에 있음을 확인
   ```

3. 해당 브랜치에서 파일을 업로드하거나 수정하는 등 작업을 한다.
   
4. commit 시에는 파일들을 staging area에 올리고 `pre-commit`을 실행한다.
   ```bash
   git add [commit할 파일 경로]
   pre-commit run
   # 파일이 수정되었을 것이므로 한 번 더 반복
   ```

5. `pre-commit`이 완료되었다면 템플릿에 맞춰 커밋 메시지를 작성한다.
   
   ```bash
   git commit
   # vi, vim 등 에디터가 실행되면 커밋 메시지 작성 후 저장
   ```
   - 제목과 본문 사이에 한 줄 공백을 두어 구분한다.
   - 본문 혹은 꼬리말에 관련 issue number(ex. `issue #21`)를 `#`과 함께 기입하면 issue에서 해당 커밋들을 tracking 할 수 있다. merge 시 해당 issue가 자동으로 close 되도록 하는 closing keyword도 있다.
  
6. commit이 완료되면 push 한다.
   
   ```bash
   git push -u origin feat/21   # --set-upstream도 가능
   ```

7. 해당 작업이 마무리되어 `feat` 브랜치를 `develop` 브랜치에 merge 및 issue를 close 하고 싶은 경우, GitHub Repository에 들어가 **Pull Request를 생성**한다.
   - 이때, base branch(ex. `develop`)에 주의한다! `main`으로 되어 있으면 안 된다.
   - PR Message에도 관련 issue number를 포함(ex. `issue #21`, `close #21`)한다.

8. 팀원들에게 review를 받고 해당 브랜치를 merge 시킨다. 그 후에는 해당 issue를 close 하고, local과 remote에 있는 브랜치(ex. `feat/21`)를 삭제한다.
   
   ```bash
   git branch -d feat/21    # local 브랜치 삭제
   git push origin --delete feat/21 # remote 브랜치 삭제
   ```

9. 다른 팀원들은 `develop`에 merge 된 변경 사항을 받아보기 위해 `git pull`을 수행해야 한다.
   ```bash
   git checkout develop # develop 브랜치로 이동
   git pull # 변경 사항 업데이트
   ```

10. 또한, **GitHub Project의 칸반보드**를 통해 다른 팀원들의 작업 상황을 언제든지 시각적으로 확인할 수 있다.

<br>

## Reference
- [https://gmlwjd9405.github.io/2018/05/11/types-of-git-branch.html](https://gmlwjd9405.github.io/2018/05/11/types-of-git-branch.html)