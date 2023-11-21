---
title: "[Better Way #25] 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라"
date: 2023-06-28 20:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

## `*` - 위치 기반 인자의 마지막 & 키워드만 사용하는 인자의 시작

```python
# ignore_overflow와 ignore_zero_division은 키워드를 사용해서만 지정할 수 있음
def safe_division_c(number, divisor, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    ...
```

- **키워드만 사용하는 인자**: 반드시 키워드를 사용해 지정해야 하며, 위치를 기반으로는 지정할 수 없다.
- 인자 목록에 있는 `*` 기호는 다음을 구분한다.
    1. 위치 인자의 마지막
    2. 키워드만 사용하는 인자의 시작
- `*` 기호 앞의 필수 인자들의 경우, 위치와 키워드를 혼용할 수 있으므로 문제가 발생할 수 있다. (필수 인자들의 이름에 의존할 것이라고 생각하지 않아야 한다.)

<br>

## `/` - 위치로만 지정하는 인자의 마지막

```python
# numerator와 denominator는 키워드를 사용해 지정할 수 없음
def safe_division_d(numerator, denominator, /, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    ...
```

- **위치로만 지정하는 인자**: 반드시 위치 기반으로만 인자를 지정해야 하며, 키워드 인자로는 쓸 수 없다.
- 인자 목록에 있는 `/` 기호는 위치로만 지정하는 인자의 끝을 표시한다.

> 인자 목록에서 **`/`와 `*` 기호 사이에 있는 모든 파라미터**는 위치를 기반으로 전달할 수도 있고, 키워드를 사용해 전달할 수도 있다. (→ 파이썬 함수 파라미터의 기본 동작)
{: .prompt-info}
