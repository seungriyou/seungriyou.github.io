---
title: "[Better Way #09] for나 while 루프 뒤에 else 블록을 사용하지 말라"
date: 2023-06-11 23:00:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## 루프 뒤의 `else`문

> 루프는 그 자체로 의미가 명확해야 하지만, `else` 블록을 사용하면 동작이 직관적이지 않고 혼동을 야기할 수 있다. 따라서 루프 뒤에 `else` 블록을 사용하지 말아야 한다.
{: .prompt-tip}

1. 루프 안에서 **`break` 문**을 사용하면 **`else` 블록이 실행되지 않는다**.

    > 즉, 루프가 반복되는 도중에 `break`를 만나지 않은 경우에만 실행된다.

2. **빈 시퀀스**에 대한 루프를 실행하면 **`else` 블록이 바로 실행**된다.

    > ex. `for x in []:`

3. `while` 루프의 조건이 **처음부터 `False`**인 경우에도 **`else` 블록이 바로 실행**된다.

    > ex. `while False:`

<br>

## 대체할 도우미 함수의 두 가지 구현

> **`else` 문을 이용하여 구현한 두 수가 서로소인지 검사하는 로직**
> 
> 
> → 공약수일 가능성이 있는 모든 수를 이터레이션하며 두 수를 나눌 수 있는지 검사
> 
> ```python
> a = 4
> b = 9
> 
> for i in range(2, min(a, b) + 1):
>     print("검사 중", i)
>     if a % i == 0 and b % i == 0:
>         print("서로소 아님")
>         break
> else:
>     print("서로소")
> 
> # 검사 중 2
> # 검사 중 3
> # 검사 중 4
> # 서로소
> ```


1. 원하는 조건을 찾자마자 빠르게 반환하는 방식
    
    ```python
    def coprime(a, b):
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
                return False
        return True

    assert coprime(4, 9)
    assert not coprime(3, 6)
    ```
    
2. 루프 안에서 원하는 대상을 찾았는지 나타내는 결과 변수를 도입하는 방식
    
    ```python
    def coprime_alternate(a, b):
        is_coprime = True
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
                is_coprime = False
                break
        return is_coprime

    assert coprime_alternate(4, 9)
    assert not coprime_alternate(3, 6)
    ```
    