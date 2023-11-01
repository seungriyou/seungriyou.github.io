---
title: "[Linux] 파일 시스템에서 사용 중인 용량 확인하기"
date: 2022-10-13 22:50:00 +0900
categories: [Dev-Log, Tips & Troubleshooting]
tags: [linux, tips]     # TAG names should always be lowercase
---

## 전체 시스템에서 사용 중인 용량 확인하기
```bash
df -h
```

<br>

## 특정 디렉토리에서 사용 중인 용량 확인하기
```bash
sudo du -hs ./* # -- all directories under current directory
```

<br>

## 참고 자료
- <https://askubuntu.com/questions/1294495/dev-nvme0n1p1space-is-full-how-to-clean-it>
