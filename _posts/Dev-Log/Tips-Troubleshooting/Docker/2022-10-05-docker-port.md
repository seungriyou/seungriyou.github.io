---
title: "[Docker] Container 내부에서 사용 중인 포트 번호 확인하기"
date: 2022-10-05 20:00:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [docker, linux, tips]     # TAG names should always be lowercase
---

<br>

원격 서버에 Docker container를 띄워놓고 그 내부에서 작업하던 도중, 사용 중인 포트 번호를 확인해야 하는 상황이 발생했다.
하지만 container 내부에 `netstat`이 설치되어 있지 않았고 설치도 되지 않아 확인하기 어려웠다.

<br>

이렇게 **Docker container 내에서 실행 중인 특정 프로세스**가 **몇 번 포트로 LISTEN** 중인지 확인하려는 경우, `nsenter` 명령어를 통해 확인할 수 있다.

<br>

> 다음의 작업은 Docker container 외부에서 수행해야 한다.
{: .prompt-info }

<br>

우선 해당 Docker container의 PID를 확인한다. 
{% raw %}
```bash
docker inspect -f '{{.State.Pid}}' [container-name]
```
{% endraw %}

<br>

PID를 이용하여 해당 container 내부에서 LISTEN 중인 포트와 PID를 확인한다.
```bash
sudo nsenter -t [container-pid] -n netstat -tupln
# Active Internet connections (only servers)
# Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
# tcp        0      0 127.0.0.1:33763         0.0.0.0:*               LISTEN      3610005/python
# tcp        0      0 127.0.0.1:34533         0.0.0.0:*               LISTEN      3610005/python
# tcp        0      0 127.0.0.11:37061        0.0.0.0:*               LISTEN      2042/dockerd
# tcp        0      0 127.0.0.1:55147         0.0.0.0:*               LISTEN      3610005/python
# tcp        0      0 127.0.0.1:48493         0.0.0.0:*               LISTEN      3610005/python
# tcp        0      0 127.0.0.1:49397         0.0.0.0:*               LISTEN      3610005/python
# tcp        0      0 127.0.0.1:46519         0.0.0.0:*               LISTEN      3314040/node
# tcp        0      0 127.0.0.1:8888          0.0.0.0:*               LISTEN      3606719/python
# tcp        0      0 127.0.0.1:8889          0.0.0.0:*               LISTEN      3810385/python
# tcp        0      0 127.0.0.1:38749         0.0.0.0:*               LISTEN      3610005/python
# udp        0      0 127.0.0.11:45617        0.0.0.0:*                           2042/dockerd
```

<br>

해당 프로세스를 종료하고 싶다면, 확인한 PID를 `kill` 하면 된다.

```bash
kill [PID]
kill -9 [PID] # 위 명령어로도 안 되면 -9로 kill (권장 X)
```
