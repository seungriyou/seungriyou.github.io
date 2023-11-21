---
title: "[Better Way #04] C 스타일 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라"
date: 2023-06-09 21:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## 형식화 (Formatting)

> 미리 정의된 문자열에 데이터 값을 끼워넣어 사람이 보기 좋은 문자열로 저장하는 과정

1. **C 스타일 형식화**: `%` 형식화 연산자 사용 (tuple, dictionary로 값 전달)
2. **고급 문자열 형식화 (파이썬 3+)**: 내장함수 `format`, `str.format` 사용 (위치 지정자 `{}` 사용)
3. **f-문자열 (파이썬 3.6+) [이 방법을 권장 ✅]**: 형식 문자열 앞에 `f` 문자를 붙이고, 인터폴레이션 수행 

```python
key = "my_var"
value = 1.234

f_string = f"{key:<10} = {value:.2f}"
c_tuple = "%-10s = %.2f" % (key, value)
str_args = "{:<10} = {:.2f}".format(key, value)
str_kw = "{key:<10} = {value:.2f}".format(key=key, value=value)
c_dict = "%(key)-10s = %(value).2f" % {"key": key, "value": value}

assert c_tuple == c_dict == f_string
assert str_args == str_kw == f_string
```

> 연속된 string은 서로 연결되므로 다음과 같이 여러 줄로 나누어 작성할 수 있다.
> 
> ```python
> for i (item, count) in enumerate(pantry):
>   print(f"#{i+1}: "
> 	f"{item.title():<10s} = "
> 	f"{round(count)}")
> ```
{: .prompt-info}
