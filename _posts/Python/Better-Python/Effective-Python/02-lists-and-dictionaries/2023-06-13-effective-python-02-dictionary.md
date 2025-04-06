---
title: "[Effective Python] 파이썬의 딕셔너리(dict) 타입"
date: 2023-06-13 21:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## 딕셔너리(`dict`) 타입

- 일반적으로 **해시 테이블(hash table)**이나 **연관 배열(associative array)** 구조 안에 값을 저장한다.
- 분할상환(amortization) 복잡도로 **상수 시간**($$O(1)$$)에 원소를 삽입하고 찾을 수 있다.
    
    > **분할상환 복잡도**: 각 연산이 발생하는 빈도와 시간을 함께 고려해 평균적으로 비용을 계산함으로써 산정한 복잡도
    {: .prompt-info}

    - **데이터 삽입 연산**: 중간중간 테이블을 재배치하기 위한 시간이 오래 걸릴 수 있다.
    - **데이터 읽기 연산**: 매우 빠르며, 일반적으로 데이터 삽입 연산보다 훨씬 많이 사용된다.
