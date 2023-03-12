---
title: "[Python] 2-1. Variable and List"
date: 2022-01-17 18:50:00 +0900
categories: [부스트캠프 AI Tech 3기, 01 - Python Basics for AI]
tags: [부스트캠프AITech, study-log, week01, python]     # TAG names should always be lowercase
math: true
mermaid: true
image: 
  src: /assets/img/posts/Boostcamp-AI-Tech/boostcamp_ai_tech_title.png
---
> 본문은 네이버 부스트캠프 AI Tech 3기의 강의와 자료를 바탕으로 작성된 글입니다.

# **Variable & Memory**
## **변수의 개요**
- 데이터(값)을 저장하기 위한 메모리 공간의 프로그래밍상 이름
  > (변수) = (값)

## **변수(Variable)란?**
- 프로그래밍에서 변수는 **값을 저장하는 장소**
- 변수는 **메모리 주소**를 가지고 있고, 변수에 들어가는 **값**은 **메모리 주소**에 할당됨

> **[참고] 컴퓨터의 구조 - 폰 노이만 아키텍처**
> 
> ![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-1-01.png){: width="50%" height="50%"}

## **변수와 메모리 주소**
![DRAM](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-1-02.png){: width="50%" height="50%"}
_DRAM_

## **메모리와 변수**
- **변수** - 프로그램에서 사용하기 위한 특정한 값을 저장하는 공간
- 선언 되는 순간 **메모리 특정 영역에 물리적인 공간**이 할당됨
- 변수에는 값이 할당되고 해당 값은 메모리에 저장됨
- A = 8의 의미는 “A는 8이다”가 아닌, A라는 이름을 가진 메모리 주소에 8을 저장하라 임

## **변수 이름 작명법**
- 알파벳, 숫자, 언더스코어(_)로 선언 가능
  - ex) `data = 0`, `_a12 = 2`, `_gg = ‘afaf’`
- 변수명은 **의미 있는 단어로 표기**하는 것이 좋다.
  - ex) `professor_name = ‘Sungchul Choi’`
- 변수명은 **대소문자가 구분**된다.
  - ex) ABC와 Abc는 같지 않다.
- 특별한 의미가 있는 **예약어는 쓰지 않는다**.
  - ex) for, if, else 등

<br>

---

# **Basic Operations**
## **Basic Operation (간단한 연산)**
- 복잡한 프로그램을 작성하기 앞서 **간단한 사칙연산**과 **문자열 처리** 등의 기초적인 연산을 알아야 함
- 이번 장에서 다루는 내용
    1. 기본 자료형 (primitive data type)
    2. 연산자와 피연산자
    3. 데이터 형변환

## **기본 자료형 (primitive data types)**
- data type에 따라 차지하는 메모리의 크기가 다르다.
  
  ![data type에 따라 차지하는 메모리의 크기가 다르다.](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-1-03.png){: width="70%" height="70%"}

- `type(a)`과 같이 자료형을 확인할 수도 있다.
  - type을 명시하지 않아도 자동으로 type이 결정된다.

<br>

---

# **Dynamic Typing**

> 코드 실행 시점에 데이터의 type을 결정하는 방법  
> - **컴파일러 언어**인 JAVA는 필요한 메모리 공간을 미리 파악할 수 있으나, 
> - **인터프리터 언어**인 python은 미리 파악할 수 없어 속도가 더 느리다.

## **연산자(operator)와 피연산자(operand)**
- `+`, `-`, `*`, `/` 같은 기호들을 연산자라고 칭함
- 연산자에 의해 계산이 되는 숫자들은 피연산자라 칭함
  - ex) `3 + 2`에서
    - 피연산자) `3`, `2`
    - 연산자) `+`
- 수식에서 **연산자의 역할 & 연산의 순서**는 수학에서 연산자와 동일
- 문자 간에도 `+` 연산이 가능 **⇒ concatenate**

## **제곱승과 나머지 구하기**
- `**` - 제곱승 계산 연산자
- `%` - 나머지 연산자

## **증가 또는 감소 연산**
- `a += 1`과 `a = a + 1`은 같은 의미 (`-=`도 마찬가지)
- `a`가 좌변에 있을 때는 **저장되는 공간**, `a`가 우변에 있을 때는 **그 공간에 들어있는 값**
- Python에서는 `++`, `--` 연산자는 없음

## **데이터 형변환: 정수형 ↔ 실수형**
- `float()`와 `int()` 함수를 사용하여 데이터의 형변환 가능
    
    > **주의할 점**
    >
    > ```shell
  > >>> a = 10
  > >>> type(a)
  > <class 'int'>
  > >>> float(a)
  > 10.0
  > >>> a
  > 10
    > ```
    >
    > - `float(a)`만 실행했기 때문에, 그 결과값인 `10.0`만 뱉어내고 종료
    > - `a` 값을 float 타입으로 바꾸려면 `a = float(a)`를 실행해야 함  
    >   **⇒ `a`에 다시 assign 해야 함**
    
- 실수형 → 정수형 변환 시 **소수점 이하 내림**
    

## **데이터 형 변환: 숫자 ↔ 문자열**
- 문자열로 선언된 값도 `int()`, `float()` 함수로 형변환 가능
- `a`와 `b`를 실수형으로 덧셈하거나 문자열로 연결하려면 **형을 맞춰야 함**

## **데이터 형 확인하기**
- `type()` 함수는 변수의 데이터 형을 확인하는 함수

## **컴퓨터의 반올림 오차**
- **Python 2.7에서만** 발생하는 현상으로, 3.X 버전에서는 발생하지 않음
- 컴퓨터의 모든 값은 이진수로 변환되어 메모리에 저장되는데, 0.1을 이진수로 변환하면 무한소수가 됨
- 반올림 오차는 충분히 작아 반올림을 하여 일반적으로 문제가 되지 않음

<br>

---

# **List**
> primitive type은 아니지만 거의 primitive type처럼 사용됨

## **List 또는 Array**
- 시퀀스 자료형, 여러 데이터들의 집합
- `int`, `float` 같은 다양한 데이터 타입 포함

## **list의 특징**
1. 인덱싱 (indexing)
2. 슬라이싱 (slicing)
3. 리스트 연산
4. 추가 / 삭제
5. 메모리 저장 방식
6. 패킹 / 언패킹
7. 2차원 리스트

## **1. 인덱싱 (indexing)**
- list에 있는 값들은 **주소(offset)**를 가짐
- 주소를 사용해 할당된 값을 호출

## **2. 슬라이싱 (slicing)**
- list의 값들을 잘라서 쓰는 것
- list의 주소 값을 기반으로 부분 값을 반환
- `list[a:b]` - 인덱스 `a` ~ `b-1`의 값을 반환
- **특징**
    - 범위를 넘어갈 경우 자동으로 최대 범위로 지정됨
    - 음수이면 거꾸로
    - 빈 채로 두면 처음 or 끝으로 설정됨
    - `[시작index : 끝index : step]` - 세 번째 숫자는 **간격**을 의미  
        ⇒ `-1`인 경우 거꾸로 뒤집는 것과 같은 동작

## **3. 리스트의 연산**
- concatenation, `in`, 연산 함수들
    - `+` - 두 list를 concat
    - `*` - list를 횟수만큼 반복
    - `in` - list 내에 해당 값이 있는지 여부 반환
    - 모든 연산의 결과는 변수에 저장되지 않기 때문에 따로 할당이 필요
- `append`, `extend`, `insert`, `remove`, `del` 등  
    ⇒ 반환만 되는 함수가 아니라 **연산 결과가 변수에 반영**되는 함수

## **4. Python 리스트만의 특징**
- 다양한 data type이 하나의 list에 들어갈 수 있음  
    ⇒ Python은 **메모리의 주소를 참조**하기 때문임

## **5. 리스트 메모리 저장 방식**
- 파이썬은 해당 리스트 변수에는 리스트 주소값이 저장됨
- `b = a`라고 선언하는 순간 `a`와 `b`는 같은 리스트를 가리키게 됨  
    ⇒ `a`를 복사해서 `b`에 넣고 싶다면 `b = a[:]`를 실행해야 함
- `a.sort()`는 inplace 연산

  > **주의**  
  > - `b = a` - 메모리 주소 할당 → 같은 object 가리킴
  > - `b = a[:]` - 복사
  {: .prompt-warning}

## **6. 패킹과 언패킹**
- **패킹** - 한 변수에 여러 개의 데이터를 넣는 것
- **언패킹** - 한 변수의 데이터를 각각의 변수로 반환  
    ⇒ 변수의 개수가 list의 원소 개수와 같아야 함
  
![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/2-1-04.png){: width="70%" height="70%"}

## **7. 이차원 리스트**
- list 안에 list를 만들어 행렬(matrix) 생성
- 이차원일 때는 `b = a[:]`로 복사가 불가능함  
    ⇒ `copy.deepcopy()`를 사용해야 함
    
    ```python
  import copy
  
  midterm_copy = copy.deepcopy(midterm_score)
    ```