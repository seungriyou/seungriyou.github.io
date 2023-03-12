---
title: "[Tips] 원격 서버의 Bash Highlighting 설정 및 한글 깨짐 문제 해결 방법"
date: 2022-05-21 22:50:00 +0900
categories: [Dev-Log, Tips]
tags: [bash, server, linux, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---


## 문제 상황
터미널에서 원격 서버(aistages)의 bash shell로 접근했을 때, highlighting이 되지 않았고 한글이 깨지는 문제가 발생했다.

![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-1.png){: width="70%" height="70%"}

![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-2.png){: width="70%" height="70%"}

<br>

## `.bashrc` 적용하기
1. 우선, `.bashrc` 파일이 존재하는지 확인한다.

    ```bash
    vim /etc/skel/.bashrc  # new file이 안 열리고 내용이 나온다면 존재하는 것이다.
    ```

2. 다음의 명령어로 `.bashrc` 파일을 `~/` 위치로 복사한다.
    
    ```bash
    cd ~        # home 디렉토리로 이동
    cp /etc/skel/.bashrc ~    # .bashrc 파일 복사
    source ~/.bashrc           # .bashrc 파일 적용
    ```

3. 터미널에서 파일 형식마다 highlighting이 적용되어 보이는지 확인한다.

4. 재부팅 할 때마다 `source ~/.bashrc` 명령어를 직접 실행하지 않기 위해 `.bash_profile` 파일을 생성한다.
    
    ```bash
    vim .bash_profile

    # === 다음의 내용 입력 ===
    # Get the aliases and functions
    
    if [ -f ~/.bashrc ]; then
        . ~/.bashrc
    fi
    ```

5. 터미널에서 highlighting이 적용되는 것을 확인한다.

    ![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-3.png){: width="80%" height="80%"}

<br>

## 한글 깨짐 현상 해결하기
1. `locale` 명령어로 현재의 언어 설정을 확인한다.

2. 다음의 명령어로 언어 패키지를 설치한다.

    ```bash
    apt-get install -y language-pack-ko
    ```

3. `.bashrc` 파일을 열고 다음을 추가한다.
    
    ```bash
    vim ~/.bashrc

    # === add ===
    # add 0521
    LANG=ko_KR.UTF-8
    export LANG

    cd ~
    # ===========

    source ~/.bashrc
    ```

    ![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-4.png){: width="70%" height="70%"}

4. `locale` 명령어로 다시 확인한다.

    ![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-5.png){: width="50%" height="50%"}

5. 서버에 재접속하여 한글을 입력해본다. VSCode 내의 터미널에서도 해본다.

    ![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-6.png){: width="70%" height="70%"}

    ![](/assets/img/posts/Dev-Log/Tips/2022-05-21-bash-7.png){: width="80%" height="80%"}

<br>

## 참고 자료
- [https://hwan-shell.tistory.com/100](https://hwan-shell.tistory.com/100)
- [http://cheonbrave.blogspot.com/2016/11/bash-shell.html](http://cheonbrave.blogspot.com/2016/11/bash-shell.html)
- [https://itbest.tistory.com/10](https://itbest.tistory.com/10)

