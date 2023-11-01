---
title: "[Linux] scp를 이용하여 로컬에서 원격 서버로 파일 전송하기 (+ 파일명 검색)"
date: 2022-10-13 20:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [linux, tips]     # TAG names should always be lowercase
---

## `scp` 명령어 기본 사용법
파일을 전송하고 싶은 원격 서버의 ssh config가 다음과 같다고 가정하겠다.

```shell
Host remote-server
    HostName  123.456.789.000
    User root
```

- 파일 단위로 전송하기
    ```bash
  scp <로컬 파일 경로> root@123.456.789.000:<원격 디렉토리 경로>
    ```

- 디렉토리 단위로 전송하기
    ```bash
  scp -r <로컬 디렉토리 경로> root@123.456.789.000:<원격 디렉토리 경로>
    ```

<br>

## `find` 명령어를 활용하여 특정 단어가 포함된 파일명 검색 및 전송하기
다음과 같이 `find` 명령어를 사용하여 **특정 단어가 포함된 파일명**을 검색할 수 있다.

```bash
scp `find . -type f | grep -e '검색어'` root@123.456.789.000:<원격 디렉토리 경로>
```

전송 전에, 검색된 파일명을 확인해보려면 **`find` 명령어만**을 사용하면 된다.

```bash
find . -type f | grep -e '검색어'
```

**한글이 깨져서 일반적으로 검색이 되지 않는 경우**라면, 다음의 결과에서 원하는 검색어를 찾아 깨진 상태 그대로 복사하여 검색어에 넣는다.

```bash
find . -type f | grep -e '[가-힣]'
```

<br>

## 참고 자료
- <https://superuser.com/questions/51330/i-would-like-to-pipe-output-of-find-into-input-list-of-scp-how>
- <https://reakwon.tistory.com/92>
