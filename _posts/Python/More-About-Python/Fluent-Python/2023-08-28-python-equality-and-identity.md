---
title: "[Python] ==와 is의 차이점"
date: 2023-08-28 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, equality, identity, variable]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

## TL;DR 📌

1. 파이썬에서 변수(variable)는 객체(attach)에 부여(attach, bind)된 라벨(label)일 뿐이며, 값을 담고 있는 상자(box)가 아니다.
2. `==`은 **equality**를 비교하는 연산자이고, `is`는 **identity**를 비교하는 연산자이다.

<br>

## 변수(Variable)

> 파이썬의 `==` 연산자와 `is` 연산자의 차이점에 대해 알기 위해서는 먼저 변수가 무엇인지 알아야 한다.
> 

파이썬에서 **변수(variable)**란 어떠한 값을 담고 있는 상자(box)가 아니라 <span class="hl">객체(object)에 부여(attach, bind)된 라벨(label)</span>에 불과하며, 객체는 변수에 의해 **참조(referenced)**된다.

즉, 다음과 같이 `b`에 `a`를 할당하는 식(assignment)은 상자 `a`의 내용을 상자 `b`로 복사하는 것이 아닌, **라벨 `a`를 가지는 객체에 라벨 `b`를 부여(= attach, bind)**하는 것이다.

```python
b = a  # 가리키는 객체가 list라면, 변수 a와 b는 모두 동일한 list에 대한 참조를 가진다.
```

<br>

이와 같은 assignment 상황에서는 <span class="hl">**우변에서 생성 또는 참조된 객체**</span>에 <span class="hl">**좌변에 위치한 라벨을 부여**</span>하게 되므로, **우변의 객체는 라벨이 부여되기 이전에 존재**해야 한다. 따라서 다음의 코드에서는 우변 코드가 평가되기 전에 예외가 발생하므로 변수 `y`가 생성되지 않는다.

![](/assets/img/posts/Python/Fluent-Python/2023-08-28-01.png){: .w-75}

<br>

정리하면, 변수에 할당하는 식에서 우변과 좌변은 각각 다음과 같이 정리할 수 있다.

| Position       | Meaning  | Description                              |
| -------------- | -------- | ---------------------------------------- |
| lefthand side  | <span class="shlp">variable</span> | bound to the object, like a label        |
| righthand side | <span class="shlp">object</span>   | where the object is created or retrieved |

<br>

## `==` vs. `is`

파이썬에서 비교를 위해서 사용하는 연산자에는 `==`과 `is`가 있다. 두 연산자의 차이점은 무엇일까?

바로 우리가 흔히 **값**을 비교할 때 사용하는 **`==`**은 **equality**를 비교하는 연산자이고, **`is`**는 **identity**를 비교하는 연산자라는 점이다.

표를 통해 자세히 비교하면 다음과 같다.

> **`id()` 함수**: CPython의 경우, 객체의 메모리 주소를 유일한 정수 값으로 반환한다.
{: .prompt-info}

| Operator | Comparison Criterion | Description | Speed |
| ----- | ----- | ----- | ----- |
| `==` | value(<span class="red">**equality**</span>)      | 해당 클래스의 `__eq__` 메서드의 구현대로 동작<br>(즉, `a == b` ↔ `a.__eq__(b)`) | ⬇️ |
| `is` | <span class="red">**identity**</span> | - `id()` 함수를 이용하여 구한 객체의 ID 비교<br>- ID는 객체의 생애주기 동안 변하지 않음 | ⬆️ |

<br>

### `is` 연산자

1. **alias 관계**인 두 변수는 같은 객체에 부여되므로, `is`로 비교하면 `True`를 반환한다.
    
    > alias 관계란, 둘 이상의 참조가 하나의 객체에 부여될 때 참조들 간의 관계를 뜻한다.
    {: .prompt-info}
    
    ```python
    charles = {"name": "Charles L. Dodgson", "born": 1832}
    lewis = charles # -- lewis는 charles의 alias
    lewis is charlse # -- True
    ```
    
2. 변수를 **싱글톤(singleton) 객체**와 비교하는 경우, `is`를 사용해야 한다.
    
    > 싱글톤 객체의 예시로는 **`None`**, **[sentinel objects](https://python-patterns.guide/python/sentinel-object/)** 등이 있다.
    {: .prompt-info}
    

<br>

### `is`는 `==` 보다 빠르다!

**`is` 연산자**의 동작이 **단순히 정수로 이루어진 두 ID를 비교하는 것에 불과**하기 때문이다.

반면, **`==` 연산자**는 **`a.__eq__(b)`의 syntactic sugar**로서 동일하게 동작한다. `object`로부터 상속받은 `__eq__` 메서드는 `is`와 마찬가지로 두 객체의 ID를 비교하지만, 대다수의 built-in 타입에서는 `__eq__` 메서드를 다른 의미있는 구현으로 **오버라이드(override)**하기 때문에 객체의 값을 고려하게 된다. 따라서 큰 collection 타입이나 중첩된 구조를 비교할 때 많은 연산을 필요로 한다.

<br>

> **오버로딩(overloading) vs. 오버라이딩(overriding)**
>
> - **오버로딩**: 여러 함수가 같은 이름을 가지고 있으나 매개변수의 타입 및 개수가 다른 것
> - **오버라이딩**: 상위 클래스에서 정의한 메서드를 하위 클래스에서 재정의하는 것
{: .prompt-info}

<br>

### 사용자 정의 클래스 관련 주의사항

**사용자 정의 클래스**에서는 `__eq__` method를 override하여 `==`가 instance에서 의미하는 바를 결정할 수 있다.

하지만 `__eq__` method를 override 하지 않는다면 **`object`에서 상속 받은 `__eq__` method(= comparing object IDs)가 실행**되므로, 모든 instances가 다르다고 판단될 수 있다.

<br>

## References

- “Fluent Python (2nd Edition)”, Ch06. Object References, Mutability, and Recycling
