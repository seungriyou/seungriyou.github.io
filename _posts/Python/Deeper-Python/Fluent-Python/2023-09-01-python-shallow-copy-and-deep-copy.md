---
title: "[Python] 얕은 복사(Shallow Copy)와 깊은 복사(Deep Copy)"
date: 2023-09-01 19:50:00 +0900
categories: [Python, Deeper Python]
tags: [python, fluent python, copy, shallow copy, deep copy]
math: true
---

## TL;DR 📌

깊은 복사(deep copy)는 참조로만 채워지는 얕은 복사(shallow copy)와는 다르게, 원본 컨테이너에서 가지고 있는 **실제 객체 자체가 복제**되는 것이다.

<br>

## 복사(Copy)

파이썬에서 복사(copy)란 **다른 ID를 가지는 동일한 객체**를 의미하며, 다음과 같이 두 가지의 종류가 있다.

1. 얕은 복사 (shallow copy)
2. 깊은 복사 (deep copy)

<br>

## 얕은 복사(Shallow Copy)

얕은 복사(shallow copy)는 <span class="hlp">**원본 컨테이너에서 가지고 있는 객체들에 대한 참조로만**</span> 채워지게 되며, 이는 파이썬 복사의 **디폴트** 동작이다.

<br>

얕은 복사는 다음의 방법들 중 하나로 생성할 수 있다.

1. built-in 생성자 사용하기 (ex. `list()`)
2. slicing이 가능한 경우, `[:]` 사용하기
3. `copy` 모듈의 `copy()` 함수 사용하기

<br>

이러한 얕은 복사는 <span class="hl">**메모리를 절약**</span>할 수 있다는 장점이 있다.

하지만 컨테이너 내 모든 객체가 immutable 하다면 상관이 없으나, <span class="hl">**mutable 하다면 문제가 발생**</span>할 수 있다.

![Untitled](/assets/img/posts/Python/Fluent-Python/2023-09-01-01.png){: .w-75}

<br>

## 깊은 복사(Deep Copy)

깊은 복사(deep copy)는 원본 컨테이너에서 가지고 있는 <span class="hlp">**객체들의 참조를 공유하지 않는 복제본**</span>으로, 참조로만 채워지는 얕은 복사(shallow copy)와는 다르게 **실제 객체 자체가 복제**되는 것이다.

<br>

깊은 복사는 `copy` 모듈의 **`deepcopy()` 함수**를 통해 생성할 수 있다.

> 객체가 **순환 참조(cyclic reference)**를 가지는 경우에도, `copy.deepcopy()` 함수를 사용하면 무한 루프에 빠지는 문제 없이 처리할 수 있다.
> 

![Untitled](/assets/img/posts/Python/Fluent-Python/2023-09-01-02.png){: .w-75}

<br>

> 만약 컨테이너 내 객체가 복제되어서는 안되는 외부 리소스나 싱글톤 객체를 참조한다면, `copy` 및 `deepcopy` 함수의 동작을 제어하기 위해 **`__copy__()`나 `__deepcopy()__` 특별 메서드**를 구현하면 된다.
{: .prompt-tip}

<br>

## References

- “Fluent Python (2nd Edition)”, Ch06. Object References, Mutability, and Recycling
