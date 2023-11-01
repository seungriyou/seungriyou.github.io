---
title: "[Tips] SSH를 통해 원격 서버 내 Docker Container의 Shell에 곧바로 접속하기"
date: 2022-10-12 21:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [tips, shell, docker, container, ssh, remote, conda, vscode]     # TAG names should always be lowercase
---

나는 현재 원격 서버에 컨테이너를 띄워두고, 그 안에 접속하여 실험을 진행하고 있다.

이처럼 원격 서버에서 Shell 환경을 제공하는 컨테이너를 올려두고 사용 중이라면,  
원격 서버 접속 후 **`docker exec`을 실행**해야 하는 <span class="hl">**<로컬 → 원격 서버 → 도커 컨테이너 접속>**</span> 의 단계를 거치기보다,  
곧바로 <span class="hl">**<로컬 → 도커 컨테이너 접속>**</span> 의 단계를 거치는 편이 편리하다.

하지만 다음과 같이 별다른 작업 없이 **로컬 환경에서 `ssh` 명령어에 원격 서버에서 도커 컨테이너가 사용 중인 포트 번호를 입력**한다고 해서 접속이 가능한 것은 아니다. (나의 경우에는, **`Bad Connection` 에러**가 발생했다.)

```bash
ssh [host-name] -p [container-port]
```

이는 컨테이너 내부 shell의 <span class="hl">**shell server가 실행되고 있지 않기 때문**</span>이다!

<br>

## 원격 서버 내 Docker Container의 Shell Server 켜기

1. 원격 서버에 접속 후, 해당 도커 컨테이너에 들어간다.

2. **shell server의 상태를 확인**한다. 이때, `not running` 상태이면 켜주어야 한다.

    ```bash
   service ssh status # systemctl 명령어로도 가능
   # * sshd is not running
    ```

3. `sshd_config` 파일을 열고, **`PermitRootLogin` 항목을 `Yes`로** 변경한다.

    ```bash
   sudo vim /etc/ssh/sshd_config
    ```

    ![](/assets/img/posts/Dev-Log/Tips/2022-10-12-ssh-1.png){: style="max-width: 60%" .normal}

4. **shell server**를 켜주고 다시 상태를 확인한다.
    ```bash
   sudo service ssh start
   service ssh status
    ```

    ![](/assets/img/posts/Dev-Log/Tips/2022-10-12-ssh-2.png){: style="max-width: 70%" .normal}

5. 로컬 환경의 터미널에서 **`ssh -p` 옵션으로 컨테이너의 포트 번호**를 넣었을 때, 곧바로 컨테이너 내 shell로 접속되는지 확인한다. 

    > 접속 시에 password를 입력해야 한다면, 해당 password는 원격 서버의 password가 아닌 컨테이너의 password이다!
    {: .prompt-warning}

    ![](/assets/img/posts/Dev-Log/Tips/2022-10-12-ssh-3.png){: style="max-width: 90%" .normal}

6. `ssh` 접속 시 편리하게 별명을 사용하기 위해서 **`~/.ssh/config`에 컨테이너 포트 옵션을 추가한 호스트를 추가**한다.

    ```shell
   # 원격 서버
   Host remote-server
       HostName  123.456.789.000
       User sryou

   # 원격 서버 내 컨테이너
   Host remote-server-container
       HostName  123.456.789.000
       User sryou
       Port 2226   # -- container의 port
    ```

<br>

## VSCode에서 Docker Extension 없이 Remote-SSH로 곧바로 Container 접속하기

위의 설정을 마치면 로컬 환경에서 곧바로 원격 서버 내 컨테이너 내부로 접속할 수 있기 때문에 VSCode에서도 **Docker extension 없이, Remote-SSH 만으로도 컨테이너 환경**을 사용할 수 있다.

하지만 컨테이너 내부에 **`curl` 또는 `wget`**이 설치되어 있지 않다면, 첫 Remote-SSH 접속 시에 VSCode Server를 설치할 수 없다는 오류가 발생할 수 있다.

<br>

**`wget`을 설치하는 방법**은 다음과 같다.

1. 컨테이너 내부에서 다음을 수행한다.
    
    ```bash
   sudo apt-get update
   sudo apt-get upgrade
    ```
    
2. `wget`을 설치한다.
    
    ```bash
   sudo apt-get install wget
    ```

<br>

이렇게 설치 후 다시 Remote-SSH 접속을 시도해보면 VSCode Server가 잘 설치됨을 확인할 수 있다.

<br>

따라서 더이상 로컬 환경에서 `ssh`로 원격 서버 접근 후, `docker exec -it` 할 필요 없이, <span class="hl">로컬 환경에서 ssh 및 VSCode를 통해 곧바로 컨테이너 내의 shell로 접근 가능</span>하다.

이러한 점에서 더 편리할 뿐만 아니라, <span class="hl">VSCode 내에서 Jupyter Notebook을 사용할 때 걸리는 delay가 체감상 크게 감소</span>하였다.

<br>

## Container 내에서 `conda` 명령어가 동작하지 않는 문제 해결

위와 같이 설정을 완료했는데, 컨테이너로 곧바로 접속한 경우에는 파이썬 가상환경을 위해 사용하는 **`conda` 명령어가 동작을 하지 않는 이슈**가 발생했다. 이는 다음과 같이 해결했다.

1. 원격 컨테이너에 위의 방법들을 통해 곧바로 접속한 후, `conda`의 실행 파일이 저장된 디렉토리로 이동한다.
    
    ```bash
   cd /opt/conda/bin
    ```
    
2. 실행 파일을 이용하여 `conda init [shell-name]`을 실행한다. (나는 `zsh`을 이용 중이다.)
    
    ```bash
   ./conda init zsh    # shell 종류에 따라 다르게 입력한다.
    ```

    ![](/assets/img/posts/Dev-Log/Tips/2022-10-12-ssh-4.png)
    
3. `exit` 후 다시 해당 컨테이너에 곧바로 접속한다.
    
    ```bash
   exit
    
   ssh remote-server-container
    ```
    
4. `conda` 명령어가 동작함을 확인한다.
