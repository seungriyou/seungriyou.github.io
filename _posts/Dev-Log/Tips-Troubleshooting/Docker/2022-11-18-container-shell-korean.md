---
title: "[Docker] Container 내 Shell에서 한글 입력이 안 되는 문제 해결"
date: 2022-11-18 20:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [shell, linux, docker, troubleshooting]     # TAG names should always be lowercase
---

## ⚠️ 문제 상황
도커 컨테이너 내 shell에서 한글 입력이 되지 않는 문제가 발생했다.

<br> 

## 🔍 해결 방법

1. 컨테이너 내 shell에 접속한 후, 다음을 차례대로 수행한다.

    ```bash
   apt-get install locales 
   export LANGUAGE=ko_KR.UTF-8 
   export LANG=ko_KR.UTF-8 
   sudo locale-gen ko_KR ko_KR.UTF-8 
   sudo update-locale LANG=ko_KR.UTF-8 
   sudo dpkg-reconfigure locales
    ```

2. 작업을 완료했으면 **`exit` 후 컨테이너에 재접속**하여 확인해본다.

<br>

## 📚 참고 자료
<http://www.kwangsiklee.com/2018/08/문제해결-docker-bash-shell에서-한글-입력-안될-때/>
