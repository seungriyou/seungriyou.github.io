---
title: "[Tips] 사용 중인 포트 번호 확인하기 (for Mac)"
date: 2022-06-27 22:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [mac, linux, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---

<br>

Mac에서 `netstat` 명령어를 이용해 사용 중인 포트 번호를 보고싶을 때, `netstat -tnlp | grep [port-number]` 명령어를 실행하면 다음과 같이 에러가 발생한다.

![](/assets/img/posts/Dev-Log/Tips/2022-06-27-port-number-1.png)

<br>

따라서 다음의 명령어를 사용하면 된다.

```bash
netstat -p tcp -van | grep LISTEN | grep [port-number]
```

![](/assets/img/posts/Dev-Log/Tips/2022-06-27-port-number-2.png)

<br>

`kill` 하고 싶은 프로세스의 PID를 찾아 다음의 명령어로 종료할 수 있다.  
**프로세스의 PID**는 **오른쪽에서 4번째 숫자**이다.

```bash
kill [PID]
kill -9 [PID] # 위 명령어로도 안 되면 -9로 kill (권장 X)
```

<br>

---

<br>

`netstat`이 깔려있지 않은 경우에는 다음의 명령어로 설치할 수 있다.

```bash
sudo apt-get install net-tools
```

<br>

---

<br>

`lsof` 명령어를 사용하면 더 편리하다.

```bash
lsof -i:[port-number]
```

![](/assets/img/posts/Dev-Log/Tips/2022-06-27-port-number-3.png)
