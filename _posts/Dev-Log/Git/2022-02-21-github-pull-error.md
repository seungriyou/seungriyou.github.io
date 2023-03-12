---
title: "[Git] GitHub 파일 삭제 관련 Pull 오류 해결 방법"
date: 2022-02-21 20:00:00 +0900
categories: [Dev-Log, Git]
tags: [git, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---

<br>

remote와 local에서 각각 파일 삭제를 다르게 진행한 경우, pull 시 오류가 발생한다.

이는 다음의 방법으로 해결할 수 있다.

```bash
git config pull.rebase false
git pull
```

`rebase`와 `pull`에 대해서 제대로 정리해봐야 할 것 같다.
