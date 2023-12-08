---
title: "[Better Way #45] 애트리뷰트를 리팩터링하는 대신 @property를 사용하라"
date: 2023-12-08 13:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, property]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

`@property` 데코레이터를 통해 **지능적인 로직을 수행하는 애트리뷰트를 정의**할 수 있다.

이것을 사용하는 흔한 케이스는 <span class="hl">**간단한 수치 애트리뷰트를 그때그때 요청에 따라 계산해 제공하도록 바꾸는 것**</span>이다.

이를 통해 **기존 클래스를 호출하는 코드를 전혀 바꾸지 않고도** <span class="hl">**클래스 애트리뷰트의 기존 동작을 변경하거나 새로운 기능을 제공**</span>할 수 있으므로 유용하다.

<br>

예시로 **leaky bucket 흐름 제어 알고리즘**을 구현함으로써 `@property` 데코레이터를 어떻게 활용할 수 있는지 알아보자!

<br>

## Leaky Bucket 흐름 제어 알고리즘 구현 예시

### 버킷 관련 연산

> 다음의 버킷 관련 연산들은 **버킷 클래스를 호출하는 코드**이다. 이를 전혀 바꾸지 않고도 버킷 클래스 애트리뷰트의 동작을 바꿀 수 있도록 `@property` 데코레이터를 사용해보자!

- 버킷에 가용 용량을 추가하는 함수
    
  ```python
  def fill(bucket, amount):
      now = datetime.now()
      # 주기가 끝났다면, 가용 용량 및 리셋 시간 초기화
      if (now - bucket.reset_time) > bucket.period_delta:
          bucket.quota = 0
          bucket.reset_time = now
      bucket.quota += amount
  ```
    
- 버킷으로부터 `amount` 만큼의 할당량을 할당 받는 함수 (할당 가능 여부를 반환)
    
  ```python
  def deduct(bucket, amount):
      now = datetime.now()
      # 새 주기가 시작됐는데 아직 버킷 할당량이 재설정되지 않았다면
      if (now - bucket.reset_time) > bucket.period_delta:
          return False
      # 버킷의 가용 용량이 충분하지 않다면
      if bucket.quota - amount < 0:
          return False
      # 버킷의 가용 용량이 충분하다면
      else:
          bucket.quota -= amount
          return True
  ```
    
- `deduct` 연산의 결과를 출력하는 함수
    
  ```python
  def use_quota(amount: int) -> None:
      if deduct(bucket, amount):
          print(f"{amount} 가용 용량 사용")
      else:
          print(f"가용 용량이 작아서 {amount} 용량을 처리할 수 없음")
      print(bucket)
  ```
    

<br>

### [BEFORE] 버킷이 시작할 때 가용 용량이 얼마인지 알 수 없는 코드

```python
from datetime import datetime, timedelta

class Bucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quota = 0      # 버킷의 남은 가용 용량
        
    def __repr__(self):
        return f"Bucket(quota={self.quota})"
```

```python
bucket = Bucket(60)
fill(bucket, 100)
print(bucket)
# Bucket(quota=100)

use_quota(99)
# 99 가용 용량 사용
# Bucket(quota=1)

use_quota(3)
# 가용 용량이 작아서 3 용량을 처리할 수 없음
# Bucket(quota=1)
```

해당 코드의 경우에는 `deduct`를 호출하는 쪽에서 할당량을 할당 받지 못 할 때, 그 이유가

1. `Bucket`에 할당된 가용 용량을 다 소진했기 때문인지
2. 이번 주기에 아직 버킷에 매 주기마다 재설정하도록 미리 정해진 가용 용량을 추가받지 못했기 때문인지

정확히 알 수 없다.

<br>

해당 이유를 정확히 알 수 있도록 클래스 애트리뷰트를 리팩토링하지 않고, `@property` 데코레이터를 이용하여 버킷 클래스를 수정해보자!

<br>

### [AFTER] `deduct`를 호출하는 쪽에서 할당량을 할당 받지 못하는 이유를 알 수 있는 코드

다음의 코드에서는 **`max_quota`와 `quota_consumed` 애트리뷰트**를 통해 현재 가용 용량 수준을 그때그때 계산하도록 하여, `deduct`를 호출하는 쪽에서 할당량을 할당 받지 못할 때 **해당 애트리뷰트들을 통해 그 이유를 알 수 있도록** 한다.

또한, `fill`과 `deduct` 함수에서 `quota` 애트리뷰트에 값을 할당할 때 현재 사용 방식에 맞춰 특별한 동작을 수행할 수 있도록 **`quota`의 setter**에 로직을 추가한다.

```python
class NewBucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0          # 이번 주기에 재설정된 가용 용량
        self.quota_consumed = 0     # 이번 주기에 버킷에서 소비한 용량의 합계
        
    def __repr__(self):
        return (f"NewBucket(max_quota={self.max_quota}, "
                f"quota_consumed={self.quota_consumed})")

    @property
    def quota(self):
        return self.max_quota - self.quota_consumed
    
    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        # 새로운 주기가 되어 가용 용량을 초기화하는 경우 (0으로 초기화 됨)
        if amount == 0:
            self.quota_consumed = 0
            self.max_quota = 0
        # 새로운 주기가 된 후 가용 용량을 추가하는 경우
        elif delta < 0:
            assert self.quota_consumed == 0
            self.max_quota = amount
        # 어떤 주기 안에서 가용 용량을 소비하는 경우
        else:
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta
```

```python
bucket = NewBucket(60)
print("최초", bucket)
fill(bucket, 100)
print("보충 후", bucket)
use_quota(99)
use_quota(3)
print("여전히", bucket)

# 최초 NewBucket(max_quota=0, quota_consumed=0)
# 보충 후 NewBucket(max_quota=100, quota_consumed=0)
# 99 가용 용량 사용
# NewBucket(max_quota=100, quota_consumed=99)
# 가용 용량이 작아서 3 용량을 처리할 수 없음
# NewBucket(max_quota=100, quota_consumed=99)
# 여전히 NewBucket(max_quota=100, quota_consumed=99)
```

<br>

## `@property` 메서드를 사용할 때의 특징

- `Bucket.quota`를 **사용하는 코드를 변경할 필요가 없이** 기존 동작을 변경할 수 있으므로, 호출하는 쪽에서 해당 클래스의 구현이 변경되었음을 알 필요도 없다.
- `@property`를 사용하면 **데이터 모델을 점진적으로 개선**할 수 있다.
- 하지만 `@property` 메서드를 **너무 과하게 사용**하고 있다면 설계한 코드의 단점을 포장하려 애쓰고 있는 것이므로, **클래스와 클래스를 사용하는 모든 코드를 리팩토링**하는 것을 고려해야 한다.
