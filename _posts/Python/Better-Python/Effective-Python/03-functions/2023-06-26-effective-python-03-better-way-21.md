---
title: "[Better Way #21] 변수 영역과 클로저의 상호작용 방식을 이해하라"
date: 2023-06-26 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

## 예시 코드를 통해 알아보는 파이썬의 특성

```python
def sort_priority(values, group):
    """group에 속하는 숫자에는 우선순위를 부여하여 정렬하는 함수"""
    def helper(x):
        if x in group: # -- helper 함수 영역 밖의 변수에 접근
            return (0, x)
        return (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)
# [2, 3, 5, 7, 1, 4, 6, 8]
```

1. 파이썬은 **클로저(closure)**를 지원한다.
    - 자신이 정의된 **영역 밖의 변수를 참조**하는 함수이다.
2. 파이썬에서는 **함수가 first-class citizen 객체**이다.
    - 직접 가리킬 수 있고, 변수에 대입하거나 다른 함수에 인자로 전달할 수 있다.
3. 파이썬에서는 **시퀀스(ex. 튜플)를 비교하는 규칙**이 있다.
    - 시퀀스를 비교할 때, 0번 인덱스부터 결과가 정해질 때까지 계속 차례로 비교한다.
{: .prompt-info}

<br>

## 파이썬에서 변수를 참조하는 순서

1. 현재 함수의 영역
2. 현재 함수를 둘러싼 영역 (현재 함수를 둘러싸고 있는 함수 등)
3. 현재 코드가 들어있는 모듈의 영역 (= 전역 영역, global scope)
4. 내장 영역(built-in scope, `len`, `str` 등의 함수가 들어있는 영역)

<br>

## 파이썬에서 변수에 값을 대입하는 방법

| **변수가 현재 영역에 이미 정의되어 있는 경우** | 그 변수의 값만 새로운 값으로 바뀐다. |
| **변수가 현재 영역에 정의되어 있지 않는 경우** | 변수 대입을 <span class="hl">변수 정의</span>로 취급한다.<br>새로 정의된 변수의 영역은 해당 대입문/식이 들어있던 함수가 된다. |

> **영역 지정 버그 (scoping bug)**
> 
> → 기본적으로 **클로저 내부에 사용한 대입문**은 <span class="hl">클로저를 감싸는 영역(ex. 모듈 영역)에 영향을 끼칠 수 없다</span>.
{: .prompt-info}

```python
def sort_priority2(numbers, group):
    found = False           # 영역: "sort_priority2"
    def helper(x):  
        if x in group:
            found = True    # 영역: "helper" -- "sort_priority2"에 영향 X
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```

<br>

## 간단한 함수가 아니라면 `nonlocal` 구문을 사용하지 말라

- `nonlocal` 문이란 **클로저 밖으로 데이터를 끌어내는 구문**으로, <span class="hl">**클로저가 자신을 감싸는 영역의 변수를 변경한다**</span>는 사실을 표시한다. (ex. `nonlocal found`)
- 모듈 수준 영역까지 변수 이름을 찾아 올라가지 않는다는 한계점이 있다.
    
    > `global` 문은 변수 대입 시 직접 모듈 영역(전역 영역)을 사용해야 한다고 지정한다.
    {: .prompt-tip}
- 간단한 함수가 아닌 경우에는 `nonlocal` 구문을 사용하기보다는 **도우미 함수로 상태를 감싸는 편**이 더 낫다.
    
  ```python
  class Sorter:
      def __init__(self, group):
          self.group = group
          self.found = False
          
      def __call__(self, x):
          if x in self.group:
              self.found = True
              return (0, x)
          return (1, x)

  sorter = Sorter(group)
  numbers.sort(key=sorter)
  assert sorter.found is True
  ```

<br>

> **클로저 함수 내부에서는..**
>
> - **참조**: 자신이 정의된 영역 외부에서 정의된 변수도 참조할 수 있다.
> - **대입**: 기본적으로 자신이 정의된 영역 외부에 영향을 끼칠 수 없으므로 대입이 아닌 정의로 취급된다.
{: .prompt-info}
