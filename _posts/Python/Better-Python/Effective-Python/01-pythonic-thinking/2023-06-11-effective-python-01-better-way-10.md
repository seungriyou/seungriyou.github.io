---
title: "[Better Way #10] 대입식을 사용해 반복을 피하라"
date: 2023-06-11 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## 대입식 (Assignment Expression)

- 코드 중복 문제를 해결하고자 파이썬 3.8+부터 지원되고 있는 구문이다.
- **왈러스 연산자(walrus operator) `:=`** 를 사용한다.
- 일반 대입문 `=`이 쓰일 수 없는 위치에서 변수에 값을 대입할 수 있어 유용하다.
- 같은 식이나 같은 대입문을 여러 번 되풀이하는 부분을 대입식으로 대체할 수 있다.

<br>

### [사용 예시 1] 해당 블록 안에서만 사용하는 변수를 강조하지 않기

> 값을 가져와서 그 값을 조건에 맞게 검사한 이후, 해당 블록 안에서만 사용하는 패턴
> 
> 
> ```python
> fresh_fruit = {
>     "사과": 10,
>     "바나나": 8,
>     "레몬": 5
> }
> 
> def make_lemonade(count):
>     print("Make Lemonade")
> 
> def make_cider(count):
>     print("Make Cider")
>     
> def out_of_stock():
>     print("Out of Stock!")
>     
> def slice_bananas(count):
>     print("Slice Bananas")
> 
> class OutOfBananas(Exception):
>     pass
> 
> def make_smoothies(count):
>     print("Make Smoothies")
> ```


- `count` 변수는 `if` 블록 안에서만 사용하므로 `:=` 를 사용할 수 있다.
    
  ```python
  if count := fresh_fruit.get("레몬", 0):
      make_lemonade(count)
  else:
      out_of_stock()
  ```
    
- 대입식이 더 큰 식의 일부분으로 쓰일 때는 괄호로 둘러싸야 한다.
    
  ```python
  if (count := fresh_fruit.get("사과", 0)) >= 4:
      make_cider(count)
  else:
      out_of_stock()
  ```
    
- `if`-`else` 블록에서만 사용하는 `count` 변수를 블록 밖에 선언하지 않는다.
    
  ```python
  if (count := fresh_fruit.get("바나나", 0)) >= 2:
      pieces = slice_bananas(count)
  else:
      pieces = 0 # if 문 이전에 미리 선언해도 ok
      
  try:
      smoothies = make_smoothies(pieces)
  except OutOfBananas:
      out_of_stock()
  ```
    

<br>

### [사용 예시 2] switch/case 문 대신하기

```python
if (count := fresh_fruit.get("바나나", 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get("사과", 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get("레몬", 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = "nothing"
```

<br>

### [사용 예시 3] do/while 루프 대신하기

```python
def pick_fruit():
    print("Pick Fruit")

def make_juice(fruit, count):
    print("Make Juice")
```

- loop-and-a-half 방식으로 구현한 버전 (w/ 무한 루프)
    
  ```python
  bottles = []
  while True:
      fresh_fruit = pick_fruit()
      if not fresh_fruit:
          break # 중간에서 끝내기

      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  ```
    
- `:=`을 통해 구현한 버전 [✅ 권장!]
    
  ```python
  bottles = []
  while fresh_fruit := pick_fruit():
      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  ```
    

<br>

> 파이썬에서 제공하지 않는 **switch/case 문**이나 **do/while 루프**를 대입식을 통해 깔끔하게 구현할 수 있다.
{: .prompt-info}
