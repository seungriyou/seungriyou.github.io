---
title: "[Python] 문자열 내 다중 공백 하나로 줄이는 방법"
date: 2023-11-10 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, string]
math: true
---

파이썬에서 문자열 내에 여러 개의 whitespace가 연달아 있는 부분을 **하나의 공백으로 줄이거나 제거**해야 하는 경우가 발생한다.

이러한 경우에 시도해 볼 수 있는 방법에는 두 가지가 있다.

<br>

## [방법 1] 정규표현식 `re.sub()`

다음과 같이 문자열 내에 하나 이상의 whitespace로 이루어진 부분을 공백 하나로 치환할 수 있다.

```python
re.sub(r"\s+", " ", string)
```

> 만약, whitespace가 아닌 특정 문자(ex. `"0"`)를 기준으로 split 하고 싶다면, 다음과 같이 하면 된다.
> 
> 
> ```python
> num = '110011'
> print(num.split('0'))
> # ['11', '', '11']
> 
> num = re.sub('[0+]', ' ', num)
> print(num.split())
> # ['11', '11']
> ```
> 

<br>

## [방법 2] `join()`과 `split()`

`split()`을 통해 공백 기준으로 문자열을 split 후, 이를 다시 공백 하나를 이용하여 `join()` 할 수도 있다.

```python
num = "11   45 6  11"
print(num.split())
# ['11', '45', '6', '11']

num = " ".join(num.split())
print(num)
# 11 45 6 11
```

<br>

## References

- <https://jsikim1.tistory.com/213>
- <https://m.blog.naver.com/wideeyed/221906671758>
