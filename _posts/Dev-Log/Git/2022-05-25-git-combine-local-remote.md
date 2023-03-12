---
title: "[Git] Local에서 작업하던 디렉토리에 Remote 디렉토리 파일 합치기"
date: 2022-05-25 22:50:00 +0900
categories: [Dev-Log, Git]
tags: [git, tips]     # TAG names should always be lowercase
---

<br>

local에서는 `frontend` 디렉토리 내 파일을 처음부터 작업 중이고, remote에는 다른 local에서 생성한 `backend` 디렉토리가 위치하는 상황을 생각해보자. **local 환경에 remote에서 작업한 내용을 불러와서 합치는 것**을 실험한 결과를 기록하려 한다. (정석적인 방법으로 한 것인지는 모르겠다. 그냥 기록용이다... 😭)

<br>

1. repository의 루트 디렉토리로 이동한다.

2. `git init`

3. `git remote add origin [url]`

4. `git fetch`

5. `git pull origin main`

6. `git branch -a`

    > 아직 `-> origin/main`이 나오지 않는다.

7. `git branch -m master main`

8. `git branch -a`

    > `master` 브랜치의 이름이 `main`으로 바뀐다.

9. `git remote set-head origin -a`

10. `git branch -a`

    > `-> origin/main`이 나온다.

11. `git checkout develop`

12. `git checkout -b feat/[issue-num]`

13. `git add .`

14. `git commit -m [commit-message]`

15. `git push -u origin feat/[issue-num]`

