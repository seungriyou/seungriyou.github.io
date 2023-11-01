---
title: "[Error] Visual Studio Code Remote-SSH 접속 시 무한 Password 문제 해결"
date: 2022-10-11 20:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [error, vscode, ssh, troubleshooting]     # TAG names should always be lowercase
---

## ⚠️ 문제 상황
VSCode에서 Remote-SSH를 사용하여 처음 원격 서버에 접속하던 도중, **VSCode Server가 설치되는 단계에서 강제 종료**를 한 뒤로 다시 접속을 시도할 때 **무한 Password 입력**을 받는 문제가 발생하였다.


<br>

## 🔍 해결 방법
VSCode Server 설치 작업이 중단되어 정상적으로 완료되지 않았는데, 그 후 접속 시도 때마다 설치 작업이 중복되어 문제가 발생한 것이었다.

다음과 같은 경로에 있는 **bin log 파일을 삭제하고 다시 접속**을 시도하니 해결되었다.

```
/home/#####/.vscode-server/bin/78a...03a/vscode-remote-lock.#####.78a...03a
```

<br>

## 📚 참고 자료
<https://github.com/microsoft/vscode-remote-release/issues/2518>
