---
title: "[Better Way #06] 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라"
date: 2023-06-09 23:30:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## 언패킹 구문의 동작 원리

> **언패킹(unpacking)**이란, 한 문장 안에서 여러 값을 대입할 수 있는 문법이다.
{: .prompt-info}


```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i - 1]:
                a[i - 1], a[i] = a[i], a[i - 1] # 언패킹

names = ["프레즐", "당근", "쑥갓", "베이컨"]
bubble_sort(names)
print(names) # ['당근', '베이컨', '쑥갓', '프레즐']
```

1. 대입문의 우항 `(a[i], a[i - 1])`이 계산된다.
2. 그 결과값이 이름이 없는 새로운 tuple에 저장된다.
3. 대입문의 좌항에 있는 언패킹 패턴 `(a[i - 1], a[i])`을 통해 임시 tuple의 값이 각각의 변수에 저장된다.
4. 이름이 없는 임시 tuple이 사라진다.

