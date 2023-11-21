---
title: "[Better Way #33] yield from을 사용해 여러 제너레이터를 합성하라"
date: 2023-07-02 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

### `yield from` 식

- 파이썬 인터프리터가 사용자 대신 `for` 루프를 내포시키고 `yield` 식을 처리하도록 한다.
- 여러 제너레이터를 모아서 하나의 제너레이터로 합성할 때 사용할 수 있다.
- 직접 제너레이터를 `for` 문으로 처리할 때보다 개선된 성능을 보인다.

```python
# 다음의 두 함수는 동일한 제너레이터를 반환한다.

def animate():            # -- for 루프 사용
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

def animate_composed():   # -- yield from 사용
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)
```
