---
title: "[Docker] Container 내 Shared Memory /dev/shm 메모리 관리"
date: 2022-11-30 20:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [shell, linux, docker, troubleshooting, tips]     # TAG names should always be lowercase
---

## ⚠️ 문제 상황
도커 컨테이너 내에서 `multiprocessing` 캐싱을 활용하여 딥러닝 학습을 수행할 때, 사용하는 shared memory인 `/dev/shm`의 용량이 가득 차는 문제가 발생했다.

따라서, `/dev/shm` 내에서 삭제할 파일을 **파일명**(나의 경우, `torch`가 포함된 이름) 및 **수정일시**를 기준으로 검색하여 삭제해야 했다.

<br> 

## 🔍 해결 방법

`/dev/shm`으로 이동하기

```shell
cd /dev/shm
```

오늘 하루동안 수정된 파일들 중, `torch` 라는 단어가 포함된 파일명만 뽑기

```shell
find -type f -mtime -1 | grep torch 
```

검색된 파일들의 전체 용량 확인하기

```shell
find -type f -mtime -1 | grep torch | du -ch
```

검색된 파일들을 한 번에 삭제하기

```shell
find -type f -mtime -1 | grep torch | xargs rm
```

용량이 얼마나 정리되었는지 확인하기

```shell
du /dev/shm -sh
```

<br>

## 📚 참고 자료
- <https://datawookie.dev/blog/2021/11/shared-memory-docker/>
- <https://funfunit.tistory.com/185>
