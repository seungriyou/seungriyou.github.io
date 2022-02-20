---
title: "[Python] 2-2. Function and Console I/O"
date: 2022-01-18 18:50:00 +0900
categories: [부스트캠프 AI Tech 3기, 01 - Python Basics for AI]
tags: [부스트캠프AITech, study-log, week01, python]     # TAG names should always be lowercase
math: true
mermaid: true
image: 
  src: /assets/img/posts/Boostcamp-AI-Tech/boostcamp_ai_tech_title.png
---
> 본문은 네이버 부스트캠프 AI Tech 3기의 강의와 자료를 바탕으로 작성된 글입니다.

# **Function**
## **함수의 개요**
- 어떤 일을 수행하는 코드의 덩어리
- 반복적인 수행을 1회만 작성 후 호출
- 코드를 논리적인 단위로 분리
- **캡슐화** - 인터페이스만 알면 타인의 코드 사용

## **함수 선언 문법**
- 함수 이름, parameter, indentation, return value(optional)
![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-01.png){: width="60%" height="60%"}

## **함수 수행 순서**
- 처음에 함수 코드를 memory에 올림
    > 함수는 코드의 상단에 적는 것을 권장
    {: .prompt-note}
- 함수 부분을 제외한 메인 프로그램부터 시작
- 함수 호출 시 함수 부분을 수행 후 되돌아옴
- `return` 문을 사용해 값을 반환함
![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-02.png){: width="70%" height="70%"}

  > **python formatting:**  
  > 함수와 함수 사이에 **두 줄** 띄워주기
  {: .prompt-info}

## **parameter vs. argument**
- **parameter** - 함수의 입력 값 인터페이스 (ex. `x`)
- **argument** - 실제 parameter에 대입된 값 (ex. `2`)  
  ⇒ 사실 잘 구분해서 사용하지 않고 parameter로 통칭해서 부름

## **함수 형태**
- parameter 유무, 반환 값(return value) 유무에 따라 함수의 형태가 다름

![함수의 4가지 형태](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-03.png){: width="70%" height="70%"}

> **함수의 return 값 유무 차이**
> 
> ```shell
> >>> def f(x):
> ...   print(x + 10)
>
> >>> f(10)
> 20
> >>> c = f(10)
> 20
> >>> c
> >>> print(c)
> None
> ```
> _`c = f(10)` 했는데 왜 `c`가 `None`으로 출력될까? - return 값이 없기 때문임_
>
> ```shell
> >>> def f(x):
> ...   return x + 10
> 
> >>> c = f(10)
> >>> c
> 20
> >>> f(10)
> 20
> ```
> _함수 내에서 `return`하는 경우에만 값으로 치환됨_
>
> **ex) 리스트 정렬 함수**
> ```shell
> >>> list_ex = [5, 4, 3, 2, 1]
> >>> list_ex
> [5, 4, 3, 2, 1]
> >>> **sorted(list_ex)**
> [1, 2, 3, 4, 5]
> >>> list_ex
> [5, 4, 3, 2, 1]
> >>> **list_ex.sort()**
> >>> list_ex
> [1, 2, 3, 4, 5]
> ```
> - `sorted()` - return 값이 있어 실제 list는 변하지 X
> - `list.sort()` - list 자체를 inplace로 정렬

<br>

---

# **Console in/out**
> 어떻게 프로그램과 데이터를 주고 받을 것인가?

## **개요**
터미널 - mouse가 아닌 키보드로 명령을 입력 → 프로그램 실행

## **Command Line Interface**
- GUI와 달리 text를 사용하여 컴퓨터에 명령을 입력하는 인터페이스 체계
- **Windows** - CMD window, Windows Terminal
- **Mac, Linux** - Terminal
- 윈도우 cmder도 권장함

## **콘솔창 입출력 1 - input**
- `input()` 함수는 콘솔창에서 **문자열(str)**을 입력 받는 함수

## **콘솔창 입출력 2 - print**
- 콤마(`,`) 사용할 경우 `print`문이 연결됨
    
    > **type이 다른 값들을 연속적으로 출력할 때는 comma `,`를 이용해야 함**
    > 
    > ```python
    > print("Hello", "Hi", 100)
    > ```
    >
    > **`+`로 concatenate 하려면 type 변환해야 함**
    >
    > ```python
    > print("Hello", "Hi" + str(100))
    > ```
    
- 숫자 입력 받기
    - `input(str)` - str이 출력되고 그 옆에서 input 값을 받을 수 있음
    - `float(input(~))` - input 값을 float 형으로 변환함

<br>

---

# **Print Formatting**
> 형식(format)에 맞춰서 출력을 할 때가 있음  
> ⇒ `print` 문을 활용해서 결과 formatting 하기

## **print formatting**
> `print` 문은 출력의 양식의 형식을 지정할 수 있음

1. % string
2. `format` 함수
3. fstring (요즘 많이 사용)

![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-04.png){: width="60%" height="60%"}

## **old-school formatting**
> 일반적으로 **%-format**과 **str.format()** 함수를 사용함

- **%-format**
  - `%datatype %(variable)` 형태로 출력 양식 표현
- **str.format() 함수**
  - `"...{datatype}...".format(argument)`

## **padding**
- `{:<10s}` - 왼쪽 정렬, 10칸 padding
- `{:>10s}` - 오른쪽 정렬, 10칸 padding

## **naming**
- 해당 표시할 내용을 변수로 표시하여 입력

![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-05.png){: width="70%" height="70%"}

## **f-string**
- python 3.6 이후, PEP498에 근거한 formatting 기법
- `f"..{variable}..."`
    
    > 주로 이렇게 사용하자!
    {: .prompt-note}

![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-2-06.png){: width="70%" height="70%"}