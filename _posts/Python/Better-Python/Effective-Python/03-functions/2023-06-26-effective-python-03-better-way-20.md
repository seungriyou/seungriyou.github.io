---
title: "[Better Way #20] None을 반환하기보다는 예외를 발생시켜라"
date: 2023-06-26 21:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

- 파이썬에서는 `None`이 `False`, `0`, `""` 등으로 잘못 해석될 수 있다. 따라서 특별한 의미를 표시하는 `None`을 반환하는 함수를 사용하면 **조건문에서 `False`로 평가될 수 있으므로** 실수하기 쉽다.
- 따라서 예외 발생 등의 특별한 경우에 **`None`을 절대로 반환하지 않아야** 하며, 대신 **`Exception`을 호출한 쪽으로 발생**시켜서 호출자가 이를 처리하게 한다.
- **type annotation**과 **docstring**을 통해, 절대로 `None`을 반환하지 않는다는 사실과 발생시키는 `Exception`을 문서에 명시할 것을 권장한다.

<br>

```python
def careful_divide(a: float, b: float) -> float:
    """a를 b로 나눈다.

    Args:
        a (float): 나누어지는 값
        b (float): 나누는 값

    Raises:
        ValueError: b가 0이어서 나눗셈을 할 수 없는 경우

    Returns:
        float: 나눗셈 결과 값
    """
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError("잘못된 입력") # -- 예외 발생
    
x, y = 5, 2
try:
    result = careful_divide(x, y)
except ValueError as e: # -- 호출자 측에서 예외 처리
    print("잘못된 입력")
else:
    print(f"결과는 {result:.1f} 입니다.")
```
