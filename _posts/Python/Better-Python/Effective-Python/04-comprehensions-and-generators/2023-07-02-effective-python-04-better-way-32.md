---
title: "[Better Way #32] 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라"
date: 2023-07-02 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

### 제너레이터 식 `()`

```python
roots = ((x, x**0.5) for x in it)
print(next(roots))
```

- **리스트 컴프리헨션의 문제점**: <span class="hl">입력이 커지면 메모리를 상당히 많이 사용하고 프로그램이 중단</span>될 수 있다.
- **제너레이터 식**이란 리스트 컴프리헨션과 제너레이터를 일반화한 것으로, **`()` 사이에 리스트 컴프리헨션과 비슷한 구문**을 넣어 제너레이터 식을 만들 수 있다.
- 제너레이터 식에 들어있는 식으로부터 원소를 하나씩 만들어내는 <span class="hl">**이터레이터가 생성**</span>된다.
    - 한 번에 원소를 하나씩 반환하므로 메모리 문제를 피할 수 있다.
    - `next` 내장 함수를 이용할 수 있다.
- **두 제너레이터 식을 합성**할 수 있다.
    - 이터레이터를 전진시킬 때마다 내부의 이터레이터도 전진한다.
    - 가능한 메모리를 효율적으로 사용하면서 동작한다.
- 아주 **큰 입력 스트림**에 대해 **여러 기능을 합성**해 적용해야 할 때 유용하다.
- 제너레이터가 반환하는 이터레이터에는 **상태**가 있으므로 <span class="hl">**이터레이터를 단 한 번만 사용**</span>해야 한다.
