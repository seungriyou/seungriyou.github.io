---
title: "[Better Way #03] bytes와 str의 차이를 알아두라"
date: 2023-05-31 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

| 데이터 종류 | 인스턴스 타입 | 변환 시 사용하는 메서드 | 설명 |
| --- | --- | --- | --- |
| 이진 데이터 | `bytes` | `bytes.decode` | 부호가 없는 8바이트 데이터 |
| 유니코드 데이터 | `str` | `str.encode` | 사람이 사용하는 언어의 문자를 표현하는<br>유니코드 코드 포인트 |

<br>

## 유니코드 샌드위치

- 유니코드 데이터를 인코딩 및 디코딩하는 부분을 인터페이스의 가장 먼 경계 지점에 위치시키는 방식이다.
- 프로그램의 핵심 부분은 **유니코드 데이터 `str`**을 사용해야 하며, 문자 인코딩에 대해 어떠한 가정도 해서는 안 된다.
- 입력으로 다양한 텍스트 인코딩을, 출력으로 한 가지 인코딩(ex. UTF-8)을 제한할 수 있다.
- 일반적으로 시스템 디폴트 인코딩 방식은 `UTF-8`이다.

<br>

## 두 가지 도우미 함수

- **`to_str`**: 항상 유니코드 데이터 `str` 반환

  ```python
  def to_str(bytes_or_str):
      if isinstance(bytes_or_str, bytes):
          value = bytes_or_str.decode("utf-8")
      else:
          value = bytes_or_str
      return value      # -- str 인스턴스

  print(repr(to_str(b"foo")))           # 'foo'
  print(repr(to_str("bar")))            # 'bar'
  print(repr(to_str(b"\xed\x95\x9c")))  # '한'
  ```
  
- **`to_bytes`**: 항상 이진 데이터 `bytes` 반환
  
  ```python
  def to_bytes(bytes_or_str):
      if isinstance(bytes_or_str, str):
          value = bytes_or_str.encode("utf-8")
      else:
          value = bytes_or_str
      return value      # -- bytes 인스턴스
  
  print(repr(to_bytes(b"foo")))           # b'foo'
  print(repr(to_bytes("bar")))            # b'bar'
  print(repr(to_bytes(b"\xed\x95\x9c")))  # b'\xed\x95\x9c'
  ```
  

<br>

## `bytes`와 `str` 다룰 때 주의할 점

1. `>`, `==`, `+`, `%`와 같은 연산자를 사용할 때, 두 타입을 섞어서 사용하면 서로 호환되지 않는다.
2. 파일 핸들과 관련한 연산(`open`)들에서 모드를 다르게 지정해주어야 한다.
    
    | 데이터 종류 | open 시 사용하는 모드 |
    | --- | --- |
    | 유니코드 데이터 `str` | 텍스트 모드 사용 (`w`, `r`) |
    | 이진 데이터 `bytes` | 이진 모드 사용 (`wb`, `rb`) |

    > 시스템 인코딩을 항상 검사하고, 디폴트 인코딩이 의심스러운 경우에는 명시적으로 `open`에 `encoding` 파라미터를 전달한다.
    {: .prompt-tip}
