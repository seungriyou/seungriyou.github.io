---
title: "[Python] match-case 문, else 절"
date: 2023-11-12 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, match, case, else]
math: true
---

## 1. 패턴 매칭(Pattern Matching)

C언어의 `switch` - `case` 문을 파이썬에서도 쓰고 싶다면? 패턴 매칭을 이용한다.

### 1-1. `match` 문과 `case` 절

![](/assets/img/posts/Python/Fluent-Python/2023-11-12-01.png){: .w-75}

1. **`|`** 를 사용하여 `or` 조건을 걸 수 있다.
2. **`['quote', x]`** : `'quote'`로 시작하는 list를 찾을 수 있다.
3. 각 조건의 끝 부분에 `if`로 guard를 추가할 수 있다.
4. **`case _`**: 모든 이전 cases에 걸리지 않는 경우를 의미한다.

<br>

### 1-2. OR-패턴 `|`

- `|`로 구분된 모든 서브 패턴들을 **OR-패턴**이라 한다.
- OR-패턴 내의 모든 서브 패턴들은 같은 변수**를 사용**해야 한다.
    
    > 그 중 어느 서브 패턴이 match 되든지간에, **guard expression**과 **`case` body** 내에서 해당 변수를 사용할 수 있음을 보장하기 위해서이다.
    > 
- `case` 절에서의 `|` 연산자는 `__or__` special method를 호출하는 것이 아니라, **set union이나 integer bitwise-or 같은 연산**을 수행한다.
- **서브 패턴 내에서도** `|`를 사용할 수 있다.
    
    ![](/assets/img/posts/Python/Fluent-Python/2023-11-12-02.png){: .w-50}
    

<br>

## 2. `for`, `while`, `try` 문과 `else` 절

> `else` 절을 `if` 문뿐만 아니라 `for`, `while`, `try` 문에서도 사용할 수 있다.
> 

### 2-1. `else` 절의 동작

- **`for` 문**
    - `for` 루프가 완료되면 실행된다.
    - `break`로 중단되는 경우에는 실행되지 않는다.

- **`while` 문**
    - 조건이 `False`가 되어 `while` 루프가 exit 될 때 실행된다.
    - `break`로 중단되는 경우에는 실행되지 않는다.

- **`try` 문**
    - `try` 블록에서 exception이 발생하지 않았을 때 실행된다.
    - 앞선 `except` 절에서는 `else` 절에서 발생하는 exception을 handling 하지 않는다.
    
    > **`try` 문 총 정리**
    > 
    > 
    > ```python
    > try:
    >     실행할 코드
    > except:
    >     예외가 발생했을 때 처리하는 코드
    > else:
    >     예외가 발생하지 않았을 때 실행할 코드
    > finally:
    >     예외 발생 여부와 상관없이 항상 실행할 코드
    > ```
    {: .prompt-danger}
    

> **exception** 혹은 **`return`, `break`, `continue` 문를 통해 실행이 종료**되는 경우에는 `else` 절이 실행되지 않는다.
{: .prompt-tip}

<br>

### 2-2. `else`의 활용

1. <span class="shlp">**루프**에서의 예시</span>
    
   ```python
   for item in my_list:
       if item.flavor == 'banana':
           break
   else:
       raise ValueError('No banana flavor found!')
   ```
    
2. <span class="shlp">**`try`/`except` 블록**에서의 예시</span>
    
    > 어디에서 exception이 발생했는지를 명확히 알기 위해서는 `try` 블록 내에는 expected exception을 생성하는 statements만 있어야 한다.
    {: .prompt-warning}
    
    다음의 코드에서는 exception이 `dangerous_call()`과 `after_call()` 중 어디에서 발생했는지 명확하게 알 수 없다.
    
   ```python
   try:
       dangerous_call()
       after_call()
   except OSError:
       log('OSError...')
   ```
   
    따라서 다음처럼 코드를 재구성하면 `dangerous_call()`에서 발생 가능한 error를 명확히 처리할 수 있으며, `try` 블록 내에서 exception이 발생하지 않았을 때에만 `after_call()`이 실행되도록 명시할 수 있다.
    
   ```python
   try:
       dangerous_call()
   except OSError:
       log('OSError...')
   else:
       after_call()
   ```
    

<br>

> **파이썬의 코딩 스타일: EAFP**
> 
> - Easier to ask for forgiveness than permission. (허락보다 용서가 쉽다.. 😅)
> - 우선은 valid 하다고 생각하고 실행한 다음, valid 하지 않다면 exception을 처리하는 스타일이다.
> - `try`, `except` 문으로 구현한다.
> - 반대되는 개념은 **LBYL**(Look before you leap)로, pre-condition을 `if` 문를 통해 명시적으로 검사한 후 실행한다.
>     
>     - 검사(looking) 후 동작(leaping)하므로, 그 사이에 multi-threaded 환경에서 race condition이 발생하면 동작이 실패할 수 있다. 이는 lock이나 EAFP 접근을 통해 해결할 수 있다.
{: .prompt-info}

<br>

## References

- “Fluent Python (2nd Edition)”, Ch18. with, match, and else Blocks