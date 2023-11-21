---
title: "[Better Way #19] 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라"
date: 2023-06-26 20:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

- 함수가 여러 값을 반환하거나 언패킹할 때 값이나 변수를 **네 개 이상 사용하면 안 된다**. (세 개까지만!)
- 더 많은 값을 언패킹해야 한다면 다음의 방법을 사용한다.
    - **경량 클래스(lightweight class)**를 반환한다.
    - **`namedtuple` 인스턴스**를 반환한다.
