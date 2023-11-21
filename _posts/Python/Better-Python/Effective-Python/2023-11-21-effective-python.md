---
title: "[Effective Python] 모아보기"
date: 2023-11-21 15:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**을 읽고 정리한 내용입니다.

<br>

## Chapter 01. Pythonic Thinking

| # | Title | Post |  |
| --- | --- | --- | --- |
| 1 | 사용 중인 파이썬의 버전을 알아두라 | [LINK](/posts/effective-python-01-better-way-01) | |
| 2 | PEP 8 스타일 가이드를 따르라 | [LINK](/posts/effective-python-01-better-way-02) | |
| 3 | `bytes`와 `str`의 차이를 알아두라 | [LINK](/posts/effective-python-01-better-way-03) | ✅ |
| 4 | C 스타일 형식 문자열을 `str.format`과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라 | [LINK](/posts/effective-python-01-better-way-04) | |
| 5 | 복잡한 식을 쓰는 대신 도우미 함수를 작성하라 | [LINK](/posts/effective-python-01-better-way-05) | |
| 6 | 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라 | [LINK](/posts/effective-python-01-better-way-06) | |
| 7 | `range` 보다는 `enumerate`를 사용하라 | [LINK](/posts/effective-python-01-better-way-07) | |
| 8 | 여러 이터레이터에 대해 나란히 루프를 수행하려면 `zip`을 사용하라 | [LINK](/posts/effective-python-01-better-way-08) | |
| 9 | `for`나 `while` 루프 뒤에 `else` 블록을 사용하지 말라 | [LINK](/posts/effective-python-01-better-way-09) | ✅ |
| 10 | 대입식을 사용해 반복을 피하라 | [LINK](/posts/effective-python-01-better-way-10) | ✅ |

<br>

## Chapter 02. Lists and Dictionaries

| # | Title | Post |  |
| --- | --- | --- | --- |
| - |  파이썬의 딕셔너리(`dict`) 타입 | [LINK](/posts/effective-python-02-dictionary) | |
| 11 | 시퀀스를 슬라이싱하는 방법을 익혀라 | [LINK](/posts/effective-python-02-better-way-11) | |
| 12 | 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라 | [LINK](/posts/effective-python-02-better-way-12) | |
| 13 | 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라 | [LINK](/posts/effective-python-02-better-way-13) | ✅ |
| 14 | 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라 | [LINK](/posts/effective-python-02-better-way-14) | ✅ |
| 15 | 딕셔너리 삽입 순서에 의존할 때는 조심하라 | [LINK](/posts/effective-python-02-better-way-15) | |
| 16 | `in`을 사용하고 딕셔너리 키가 없을 때 `KeyError`를 처리하기보다는 `get`을 사용하라 | [LINK](/posts/effective-python-02-better-way-16) | ✅ |
| 17 | 내부 상태에서 원소가 없는 경우를 처리할 때는 `setdefault` 보다 `defaultdict`를 사용하라 | [LINK](/posts/effective-python-02-better-way-17) | ✅ |
| 18 | `__missing__`을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라 | [LINK](/posts/effective-python-02-better-way-18) | ✅ |

<br>

## Chapter 03. Functions

| # | Title | Post |  |
| --- | --- | --- | --- |
| 19 | 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라 | [LINK](/posts/effective-python-03-better-way-19) | |
| 20 | `None`을 반환하기보다는 예외를 발생시켜라 | [LINK](/posts/effective-python-03-better-way-20) | ✅ |
| 21 | 변수 영역과 클로저의 상호작용 방식을 이해하라 | [LINK](/posts/effective-python-03-better-way-21) | ✅ |
| 22 | 가변 위치 인자를 사용해 시각적인 잡음을 줄여라 | [LINK](/posts/effective-python-03-better-way-22) | ✅ |
| 23 | 키워드 인자로 선택적인 기능을 제공하라 | [LINK](/posts/effective-python-03-better-way-23) | |
| 24 | `None`과 독스트링을 사용해 동적인 디폴트 인자를 지정하라 | [LINK](/posts/effective-python-03-better-way-24) | ✅ |
| 25 | 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라 | [LINK](/posts/effective-python-03-better-way-25) | |
| 26 | `functools.wraps`을 사용해 함수 데코레이터를 정의하라 | [LINK](/posts/effective-python-03-better-way-26) | ✅ |

<br>

## Chapter 04. Comprehensions and Generators

| # | Title | Post |  |
| --- | --- | --- | --- |
| 27 | `map`과 `filter` 대신 컴프리헨션을 사용하라 | [LINK](/posts/effective-python-04-better-way-27) | |
| 28 | 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라 | [LINK](/posts/effective-python-04-better-way-28) | |
| 29 | 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라 | [LINK](/posts/effective-python-04-better-way-29) | |
| 30 | 리스트를 반환하기보다는 제너레이터를 사용하라 | [LINK](/posts/effective-python-04-better-way-30) | ✅ |
| 31 | 인자에 대해 이터레이션할 때는 방어적이 돼라 | [LINK](/posts/effective-python-04-better-way-31) | ✅ |
| 32 | 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라 | [LINK](/posts/effective-python-04-better-way-32) | ✅ |
| 33 | `yield from`을 사용해 여러 제너레이터를 합성하라 | [LINK](/posts/effective-python-04-better-way-33) | |
| 34 | `send`로 제너레이터에 데이터를 주입하지 말라 | [LINK](/posts/effective-python-04-better-way-34) | ✅ |
| 35 | 제너레이터 안에서 `throw`로 상태를 변화시키지 말라 | [LINK](/posts/effective-python-04-better-way-35) | ✅ |
| 36 | 이터레이터나 제너레이터를 다룰 때는 `itertools`를 사용하라 | [LINK](/posts/effective-python-04-better-way-36) | ✅ |

<br>

## Chapter 05. Classes and Interfaces

| # | Title | Post |  |
| --- | --- | --- | --- |
| 37 | 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라 | [LINK](/posts/effective-python-05-better-way-37) | ✅ |
| 38 | 간단한 인터페이스의 경우 클래스 대신 함수를 받아라 | [LINK](/posts/effective-python-05-better-way-38) | ✅ |
| 39 | 객체를 제너릭하게 구성하려면 `@classmethod`를 통한 다형성을 활용하라 | [LINK](/posts/effective-python-05-better-way-39) | ✅ |

<br>
