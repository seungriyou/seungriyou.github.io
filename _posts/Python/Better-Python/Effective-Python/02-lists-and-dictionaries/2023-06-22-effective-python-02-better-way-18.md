---
title: "[Better Way #18] __missing__을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라"
date: 2023-06-22 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## `setdefault`와 `defaultdict`의 한계점

- 다음의 경우에는 **`setdefault` 메서드를 사용하지 않아야** 한다.
    
    > 키 값의 존재 여부와 상관 없이 디폴트 값을 생성하는 동작이 항상 호출되기 때문이다.
    
    1. 디폴트 값을 생성하는 **비용**이 큰 경우
    2. 디폴트 값을 생성하는 과정에서 **예외가 발생**할 수 있는 상황인 경우
- **`defaultdict`에 전달되는 함수는 인자를 전달 받을 수 없으므로**, 접근에 사용한 키 값과 관련된 디폴트 값을 생성하는 것은 불가능하다.

<br>

## `dict` 타입의 하위 클래스 내 `__missing__` 특별 메서드

```python
def open_picture(profile_path):
    try:
        return open(profile_path, "a+b")
    except OSError:
        print(f"경로를 열 수 없습니다: {profile_path}")
        raise

class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)   # (1) 키에 해당하는 디폴트 값을 생성
        self[key] = value           # (2) 딕셔너리에 삽입
        return value                # (3) 호출한 쪽에 그 값을 반환

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

- 키가 없는 경우를 처리하는 로직을 커스텀화 할 수 있다.
- **`__missing__` 메서드**는 다음의 동작을 구현해야 한다.
    1. 키에 해당하는 디폴트 값을 생성
    2. 딕셔너리에 삽입
    3. 호출한 쪽에 그 값을 반환
- 딕셔너리에 접근 시 **해당 키가 없는 경우 `__missing__` 메서드가 호출**되며, 해당 키가 존재하는 경우에는 호출되지 않는다.
