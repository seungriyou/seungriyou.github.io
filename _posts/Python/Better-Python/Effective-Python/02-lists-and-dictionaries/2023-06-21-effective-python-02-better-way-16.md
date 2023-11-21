---
title: "[Better Way #16] in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 get을 사용하라"
date: 2023-06-21 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## 딕셔너리 키가 없는 경우를 처리하는 방법

1. **`if` 문과 `in`을 사용하는 방법**
    
    ```python
    if key in counters:
        count = counters[key]
    else:
        count = 0
    
    counters[key] = count + 1
    ```
    
2. **`KeyError` 예외를 활용하는 방법**
    
    ```python
    try:
        count = counters[key]
    except KeyError:
        count = 0
    
    counters[key] = count + 1
    ```
    
3. <span class="hl">**`dict` 내장 타입의 `get` 메서드를 사용하는 방법**</span> [→ 가장 코드가 짧고 깔끔한 방법 ✅]
    
    > **`get`의 두 번째 인자** = 키(첫 번째 인자)가 딕셔너리에 들어 있지 않을 때 돌려줄 디폴트 값
    > 
    
    ```python
    count = counters.get(key, 0)
    
    counters[key] = count + 1
    ```
    
<br>

## 딕셔너리 키가 없는 경우를 처리하는 방법: 복잡한 값(ex. `list`)이 저장되어 있는 경우

> **이중 대입문 `votes[key] = names = []`**
>
> → 참조를 통해 리스트 내용을 변경할 수 있으므로, `names.append()` 호출 후 다시 리스트를 딕셔너리에 대입할 필요가 없다.
{: .prompt-info}

1. **`if` 문과 `in`을 사용하는 방법**
    
    ```python
    if key in votes:
        names = votes[key]
    else:
        votes[key] = names = [] # 이중 대입문
    
    names.append(who)
    ```
    
2. **`KeyError` 예외를 활용하는 방법**
    
    ```python
    try:
        names = votes[key]
    except:
        votes[key] = names = []
    
    names.append(who)
    ```
    
3. **`dict` 내장 타입의 `get` 메서드를 사용하는 방법**
    
    ```python
    names = votes.get(key)
    if names is None:
        votes[key] = names = []
    
    names.append(who)
    ```
    
4. <span class="hl">**`dict` 내장 타입의 `get` 메서드와 대입식을 사용하는 방법**</span> [→ 가장 코드가 짧고 깔끔한 방법이므로 권장 ✅]
    
    ```python
    if (names := votes.get(key)) is None:
        votes[key] = names = []
    
    names.append(who)
    ```
    
5. **`dict` 타입이 제공하는 `setdefault` 메서드를 사용하는 방법**
    
    ```python
    names = votes.setdefault(key, [])
    
    names.append(who)
    ```
    
    - **`setdefault` 메서드**는 **딕셔너리에 키가 없으면 제공받은 디폴트 값을 키에 연관시켜 딕셔너리에 대입**한 다음, **키에 연관된 값을 반환**한다.
    - 전달하는 디폴트 값이 별도로 복사되지 않고 **딕셔너리에 직접 대입**되므로, 디폴트 값을 전달할 때마다 해당 **키의 존재 여부와 상관 없이 그 값을 새로 생성**해야 한다. 따라서 성능이 크게 저하될 수 있다.
        
        > 디폴트 값으로 이용할 객체를 새로 생성하지 않고 재활용한다면 이상한 동작을 하게 되고 버그가 발생할 수 있다.
        > 
    - 실제로는 `setdefault` 보다 `defaultdict`를 사용하는 것으로 충분할 수 있다.

