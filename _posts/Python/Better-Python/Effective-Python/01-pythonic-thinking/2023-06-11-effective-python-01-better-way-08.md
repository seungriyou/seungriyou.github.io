---
title: "[Better Way #08] 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라"
date: 2023-06-11 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## `zip` 내장 함수

- 둘 이상의 이터레이터를 지연 계산 제너레이터를 사용해 묶어준다.
- 각 이터레이터의 다음 값이 들어있는 튜플을 반환하며, 이를 `for` 문에서 바로 언패킹할 수 있다.
- 입력 이터레이터의 길이가 서로 다른 경우, **아무 경고 없이 가장 짧은 이터레이터의 길이까지만** 튜플을 반환한다.
    
    > 따라서 이터레이터의 길이가 서로 같지 않을 것으로 예상한다면 **`itertools.zip_longest()` 함수**를 사용하자.
    > 해당 함수는 존재하지 않는 값을 전달된 `fillvalue` 파라미터의 값으로 대신한다. (default: None)
    > 
    {: .prompt-info}
    
