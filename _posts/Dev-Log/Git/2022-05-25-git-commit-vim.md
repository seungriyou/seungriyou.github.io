---
title: "[Git] Git Commit 메시지 에디터 Vim으로 설정하기"
date: 2022-05-25 19:50:00 +0900
categories: [Dev-Log, Git]
tags: [git, tips]     # TAG names should always be lowercase
---

<br>

각자 익숙한 텍스트 에디터가 있기 마련인데, `vim`이 익숙한 나는 설정이 되지 않은 외부 서버에서 `git commit` 할 때마다 단축키를 찾는 데에 애를 먹었다.

`git commit` 시 사용할 에디터를 `vim`으로 변경하려면 다음의 명령어를 입력하면 된다.

<br>

```bash
git config --global core.editor "vim"
```
