---
title: "[Better Way #14] 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라"
date: 2023-06-20 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>


> **`list` 내장 타입 `sort` 메서드**는 자연스러운 순서를 사용해 오름차순으로 정렬한다.
> 
> 
> → 거의 대부분의 내장 타입(문자열, 부동 소수점 수 등)에 대해 잘 작동한다.
{: .prompt-info}

<br>


## `sort` 메서드의 `key` 파라미터

원소 타입에 자연스러운 순서가 정의되어 있지 않는 경우, `key` 파라미터를 통해 리스트의 각 원소 대신 **비교에 사용할**(= 비교 가능한, 자연스러운 순서가 정의된) **객체를 반환하는 도우미 함수**를 정의할 수 있다. (ex. 원소 애트리뷰트에 접근, 인덱스를 사용하여 값 얻기 등)

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight
    
    def __repr__(self):
        return f"Tool({self.name!r}, {self.weight})"

tools = [
    Tool("수준계", 3.5),
    Tool("해머", 1.25),
    Tool("스크류드라이버", 0.5),
    Tool("끌", 0.25)
]

print("미정렬:", repr(tools))
tools.sort(key=lambda x: x.name)
print("정렬:", repr(tools))
```

<br>

### `key` 함수를 통해 원소 값을 변형하는 경우

ex) 대소문자를 구분하지 않고 알파벳 순으로 문자열 정렬하기

```python
places = ["home", "work", "New York", "Paris"]
places.sort()
print("대소문자 구분:", places)
places.sort(key=lambda x: x.lower()) # 소문자로 변환 후 정렬
print("대소문자 무시:", places)
```

<br>

## 여러 정렬 기준을 사용해야 하는 경우

1. **`key` 함수의 리턴 타입으로 `tuple` 타입을 사용하는 방법**
    - 정렬 기준으로 사용할 애트리뷰트를 우선순위 순서대로 튜플에 넣는다.
    - 모든 비교 기준의 정렬 순서가 같아야 한다. (모두 오름차순 or 모두 내림차순)
    - `reverse` 파라미터를 넘기면 튜플에 들어있는 기준들의 정렬 순서가 같이 영향을 받게 된다.
    - 여러 정렬 기준의 정렬 순서가 달라야 하는 경우, 부호를 바꿀 수 있는 타입(ex. 숫자값)이라면 **단항 부호 반전 연산자 `-`**를 사용해 정렬 방향을 혼합할 수 있다.
    
    ```python
    power_tools = [
        Tool("드릴", 4),
        Tool("원형 톱", 5),
        Tool("착암기", 40),
        Tool("연마기", 4)
    ]
    
    power_tools.sort(key=lambda x: (-x.weight, x.name)) # (내림차순, 오름차순)
    print(power_tools)
    # [Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
    ```
    
2. **정렬 기준의 우선순위가 점점 높아지는 순서대로 `sort` 함수를 호출하는 방법**
    - 부호를 바꿀 수 없는 타입이라면 **각 정렬 기준마다 `reverse` 파라미터로 정렬 순서를 지정**하면서 `sort` 함수를 여러 번 사용해야 한다.
    - 이것이 가능한 이유는 파이썬이 **stable sort** 알고리즘을 제공하기 때문이다.
        
        > **stable sort** = `key` 함수가 반환하는 값이 서로 같은 경우, 리스트에 들어있던 원래 순서를 그대로 유지
        {: .prompt-info}
    - 최종적으로 리스트에서 얻어내고 싶은 **정렬 기준 우선순위의 역순으로 정렬을 수행**해야 한다.
    - 꼭 필요할 때만 이 방법을 사용하는 것을 권장한다.
    
    ```python
    power_tools = [
        Tool("드릴", 4),
        Tool("원형 톱", 5),
        Tool("착암기", 40),
        Tool("연마기", 4)
    ]
    
    power_tools.sort(key=lambda x: x.name)
    power_tools.sort(key=lambda x: x.weight, reverse=True)
    print(power_tools)
    # [Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
    ```
    