---
title: "[Better Way #01] 사용 중인 파이썬의 버전을 알아두라"
date: 2023-05-31 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>


이제 파이썬 3 버전을 사용해야 하며, 파이썬 버전은 다음과 같이 확인할 수 있다.

```bash
python --version
# Python 3.10.11
```

```python
import sys
print(sys.version_info)
print(sys.version)

# sys.version_info(major=3, minor=10, micro=11, releaselevel='final', serial=0)
# 3.10.11 | packaged by conda-forge | (main, May 10 2023, 19:01:19) [Clang 14.0.6 ]
```
