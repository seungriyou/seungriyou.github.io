---
title: "[Tips] 터미널로 원격 서버에 접속하기"
date: 2022-04-15 19:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [ssh, terminal, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---
> 개발 환경: MacBook M1 Pro

<br>

딥러닝 학습을 수행하는 경우나 AWS, GCP 등의 클라우드 서비스를 이용한다면 원격 서버 인스턴스를 주로 사용하게 된다. 이때, SSH 명령어를 통해 로컬 터미널에서 원격 서버로 접속하게 되는데, 원격 서버의 IP 주소나 포트 번호를 매번 타이핑하기 귀찮을 수 있다. 따라서 `key` 파일을 `config`에 등록해두고 **별명을 이용해서 편리하게 SSH로 원격 서버에 접속하는 방법**을 기록하려 한다.

<br>

#### 1. `key` 파일의 permission 수정하기

```bash
chmod 0400 ./key 
# ./key는 key 파일이며, chmod 시 0600도 가능하다.
```

#### 2. ssh `config` 파일 수정하기
```bash
vim ~/.ssh/config
```

```config
Host [Host-Name]
    HostName [IP-Address]
    User root
    Port [Port-Number]
    IdentityFile [Key-File-Path]
```

#### 3. `Host`에 기입한 별명으로 ssh 접속하기
```bash
ssh [Host-Name]
```

<br>
