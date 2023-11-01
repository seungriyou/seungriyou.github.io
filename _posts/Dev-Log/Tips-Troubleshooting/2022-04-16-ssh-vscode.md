---
title: "[Tips] Visual Studio Code에서 원격 서버에 접속하기"
date: 2022-04-16 19:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [ssh, vscode, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---
> 개발 환경: MacBook M1 Pro

<br>

[**이전 포스팅**](https://seungriyou.github.io/posts/ssh-terminal/)에서는 터미널에서 원격 서버에 접속하는 법을 다루었는데, 이번 포스팅에서는 IDE인 **VSCode에서 원격 서버에 접속하는 방법**을 다룬다. 다른 IDE인 PyCharm도 사용해보았으나 SSH 원격 접속은 Community 버전에서는 불가능하기도 하고, 서버 내 파일에 접근하는 것도 VSCode가 편리하여 VSCode를 이용하는 방법을 기록하게 되었다.

<br>

### 1. VSCode에 `Remote-SSH` Extension 설치하기
![remote-ssh](/assets/img/posts/Dev-Log/Tips/2022-04-16-ssh-vscode.png)

<br>

### 2. ssh `config` 파일에 host 등록하기
> [**이전 포스팅**](https://seungriyou.github.io/posts/ssh-terminal/)을 따라서 등록해도 되고, 다음과 같이 VSCode 상에서 `config` 파일을 수정하여 `key` 파일을 등록할 수도 있다. 개인적으로는 이전 포스팅의 방법을 사용하는 편이 더 편리한 것 같다.
{: .prompt-info}

  1. `ctrl` + `shift` + `p` → `Remote-SSH: Add New SSH Host...`
  2. 입력 창에 다음과 같은 형식으로 원격 서버 정보 입력하기
        ```bash
root@123.45.678.90 -p 1234  # 예시
        ```
  3. `config` 파일 업데이트하기 (나의 경우, `/Users/[user-name]/.ssh/config`)
  4. 다시 `ctrl` + `shift` + `p` → `Remote-SSH: Connect to Host` → `Configure SSH Hosts` → 3번에서의 `config` 파일 선택
  5. `config` 파일에 `IdentityFile` 추가 (`key` 파일의 위치 지정)
        ```config
Host 123.45.678.90  # IP 대신 host 별명을 입력해도 된다!
    HostName 123.45.678.90
    User root
    Port 1234
    IdentityFile [Key-File-Path]
        ```

<br>

### 3. Host에 연결 및 디렉토리 선택
  1. `ctrl` + `shift` + `p` → `Remote-SSH: Connect to Host...` → 등록한 host 선택
  2. `Open Folder` → 위치 확인 후 `OK`


<br>

<br>

이렇게 하면 VSCode 내 터미널에서도 바로 원격 서버로 접속된다. 😆
