---
title: "[Better Way #15] 딕셔너리 삽입 순서에 의존할 때는 조심하라"
date: 2023-06-21 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary, duck typing]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## 파이썬의 `dict` 인스턴스

- **파이썬 3.5 이전**: 딕셔너리에 대해 이터레이션 수행 시, 그 순서는 원소가 삽입된 순서와 일치하지 않았다.
- **파이썬 3.6 이후**: 딕셔너리가 삽입 순서를 보존하도록 동작이 개선되었다. 특히, 3.7+에서는 아예 파이썬 언어 명세에 해당 내용이 포함되었다.
- **파이썬 내 `dict` 타입의 활용 사례** (= 파이썬 3.6+부터 삽입 순서가 보존되는 동작)
    - 함수에 대한 키워드 인자(`**kwargs`)
    - 클래스의 인스턴스 딕셔너리 (`instance.__dict__`)

> **`collections.OrderedDict`**
> 
> - 파이썬 3.7+의 표준 `dict`의 동작과 비슷하기는 하지만, 성능 특성은 많이 다르다.
> - 키 삽입과 `popitem()` 호출을 매우 자주 처리해야 한다면, 표준 `dict` 보다 `OrderedDict`가 더 낫다.
{: .prompt-info}

<br>

## 덕 타이핑 (Duck Typing)

- 파이썬은 정적 타입 지정 언어가 아니므로, 대부분의 경우 엄격한 클래스 계층보다는 <span class="hl">**객체의 동작이 객체의 실질적인 타입을 결정**</span>하는 **덕 타이핑**에 의존한다.
- 덕 타이핑이란 동적 타입 지정의 일종으로, <span class="hl">객체가 실행 시점에 어떻게 행동하는지를 기준으로 객체의 타입을 판단</span>하는 타입 지정 방식이다.
- **`collections.abc` 모듈을 사용**하여 파이썬에서 제공하는 클래스와 비슷하지만 다르게 동작하는 클래스를 새롭게 정의할 수 있다.

<br>

### 덕 타이핑을 고려하여 딕셔너리와 비슷한 클래스를 조심스럽게 다루는 방법

1. `dict` 인스턴스의 삽입 순서 보존에 의존하지 않고 코드를 작성하는 방법 (가장 보수적임)

    ```python
    def get_winner(ranks):
        for name, rank in ranks.items():
            if rank == 1:
                return name
    ```
    
2. 실행 시점에 명시적으로 `dict` 타입을 검사하는 방법
    
    ```python
    def get_winner(ranks):
        if not isinstance(ranks, dict):
            raise TypeError("dict 인스턴스가 필요합니다.")
        return next(iter(ranks)) # dict 인스턴스의 삽입 순서 보존에 의존
    ```
    
3. 타입 어노테이션(annotation)과 정적 분석(static analysis)을 사용해 `dict` 값을 강제하는 방법 (ex. `mypy`)
    
    ```python
    from typing import Dict
    
    def get_winner(ranks: Dict[str, int]) -> str:
        return next(iter(ranks))
    ```
    