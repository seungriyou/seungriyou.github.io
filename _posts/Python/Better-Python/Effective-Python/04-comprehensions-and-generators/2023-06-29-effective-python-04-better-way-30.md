---
title: "[Better Way #30] 리스트를 반환하기보다는 제너레이터를 사용하라"
date: 2023-06-29 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

```python
# -- 리스트를 반환하는 함수
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == " ":
            result.append(index + 1)
    return result
```
```python
# -- 이터레이터를 반환하는 함수 (제너레이터)
def index_words_it(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == " ":
            yield index + 1
```

<br>

## 리스트를 반환하는 함수의 문제점

1. 코드에 잡음이 많고 핵심을 알아보기 어렵다.
2. 반환하기 전에 **리스트에 모든 결과를 다 저장**해야 하므로, 입력이 매우 크면 프로그램이 **메모리를 소진**할 수 있다.

<br>

## <span class="hl">이터레이터의 특징</span>

- **`next` 내장 함수를 호출**할 때마다 **이터레이터는 제너레이터 함수를 다음 `yield` 식까지 진행**시킨다.
- 쉽게 **리스트로 변환**할 수 있다. (ex. `list(it)`)
- 리스트에 모든 값을 저장할 필요가 없으므로 사용하는 **메모리 크기를 어느정도 제한**할 수 있게 된다.
    
    > 작업 메모리에 모든 입력과 출력을 저장할 필요가 없으므로 입력이 아주 커도 출력 시퀀스를 만들 수 있다.
    {: .prompt-info}
- **이터레이터에 상태**가 있으므로, 호출하는 쪽에서 **재사용이 불가능**하다.
