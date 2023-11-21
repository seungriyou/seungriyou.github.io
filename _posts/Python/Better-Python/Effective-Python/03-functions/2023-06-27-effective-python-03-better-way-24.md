---
title: "[Better Way #24] None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라"
date: 2023-06-27 21:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

## 디폴트 인자

- **디폴트 인자 값**은 그 인자가 포함된 **함수 정의가 속한 모듈이 로드되는 시점에 단 한 번**만 평가 된다.
- **동적인 값**(ex. `{}`, `[]`, `datetime.now()` 등)의 경우, 이상한 동작이 일어날 수 있다.
- ex) `default={}`인 경우, 해당 딕셔너리 객체가 모든 함수 호출에서 공유된다.
    
  ```python
  import json

  def decode(data, default={}): # -- default 딕셔너리 객체가 공유됨
      try:
          return json.loads(data)
      except ValueError:
          return default

  foo = decode("잘못된 데이터")
  foo["stuff"] = 5
  bar = decode("또 잘못된 데이터")
  bar["meep"] = 1
  print(f"Foo: {foo}")
  print(f"Bar: {bar}")
  assert foo is bar # -- foo와 bar는 같은 객체

  # Foo: {'stuff': 5, 'meep': 1}
  # Bar: {'stuff': 5, 'meep': 1}
  ```

<br>

## 동적인 디폴트 인자를 지정하는 방법

1. 동적인 값을 가질 수 있는 **키워드 인자의 디폴트 값으로 `None`**을 지정한다.
2. 해당 인자는 **내부에서, 그 값이 `None`인 경우에 새로 생성**한다.
3. 실제 동작을 **독스트링**에 문서화한다.

<br>

```python
import json

def decode(data, default=None): # -- default 값을 None으로 지정
  """문자열로부터 JSON 데이터를 읽어온다.

  Args:
      data: 인코딩할 JSON 데이터.
      default: 디코딩 실패 시 반환할 값이다. 디폴트 값은 빈 딕셔너리다.
  """
  try:
      return json.loads(data)
  except ValueError:
      if default is None:
          default = {} # -- 내부에서 딕셔너리 객체 생성
      return default

foo = decode("잘못된 데이터")
foo["stuff"] = 5
bar = decode("또 잘못된 데이터")
bar["meep"] = 1
print(f"Foo: {foo}")
print(f"Bar: {bar}")
assert foo is not bar

# Foo: {'stuff': 5}
# Bar: {'meep': 1}
```

```python
from time import sleep
from datetime import datetime

def log(message, when=None): # -- default 값을 None으로 지정
  """메시지와 타임스탬프를 로그에 남긴다.

  Args:
      message: 출력할 메시지.
      when: 메시지가 발생한 시각(datetime). 디폴트 값은 현재 시간이다.
  """
  if when is None:
      when = datetime.now() # -- 내부에서 datetime.now() 실행
  print(f"{when}: {message}")

log("안녕!")
sleep(0.1)
log("다시 안녕!")
```

<br>

### 타입 어노테이션을 통해 키워드 인자의 디폴트 값 표현하기 (`None`)
    
```python
from typing import Optional

# when에 사용할 수 있는 값은 None과 datetime 객체 뿐임
def log_typed(message: str,
              when: Optional[datetime] = None) -> None: # -- Optional 어노테이션 & None 지정
    """메시지와 타임스탬프를 로그에 남긴다.

    Args:
        message (str): 출력할 메시지.
        when (Optional[datetime], optional): 메시지가 발생한 시각(datetime). 디폴트 값은 현재 시간이다.
    """
    if when is None:
        when = datetime.now()
    print(f"{when}: {message}")
```

<br>

> 다음은 모두 동일하다.
>
> - `Optional[str] = None`
> - `Union[str, None] = None`
> - (Python 3.10+ 권장 사항) `str | None = None`
{: .prompt-info}
