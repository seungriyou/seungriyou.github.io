---
title: "[Better Way #37] 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라"
date: 2023-11-19 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

- 파이썬 내장 딕셔너리 타입을 통해 동적(dynamic)인 내부 상태를 유지할 수 있으나, **내포 단계가 두 단계 이상**이 되면 더 이상 딕셔너리, 리스트, 집합, 튜플 등의 내장 타입 계층을 추가하지 말아야 한다.
    
    > ex) 튜플에 저장된 내부 원소에 위치를 통해 접근해야 하는 경우: 원소가 추가된다면 코드를 모두 수정해야 한다.
    
- 코드에서 **값을 관리하는 부분이 점점 복잡**해지고 있다면 **해당 기능을 여러 클래스로 분리**해야 한다. 이를 통해 인터페이스와 구현 사이에 추상화 계층을 만들 수 있다.

<br>

## `namedtuple` 클래스

- **장점**
    - `collection` 내장 모듈에서 제공하는 타입으로, **작은 불변 데이터 클래스**를 쉽게 정의할 수 있다.
    - `namedtuple` 클래스의 인스턴스를 만들 때는 **위치 기반 인자**를 사용해도 되고 **키워드 인자**를 사용해도 된다.
    - 필드에 접근할 때는 **애트리뷰트 이름을 사용**할 수 있으므로, 요구 사항이 바뀌는 경우에 일반 클래스로 변경하기 용이하다.

- **단점**
    - 디폴트 인자 값을 지정할 수 없으므로, 선택적인 프로퍼티가 많은 데이터에 사용하기는 어렵다.
        
        > 프로퍼티가 4~5개를 초과하면 **`dataclasses` 내장 모듈**을 사용하는 것이 낫다.
        
    - `namedtuple` 인스턴스의 애트리뷰트 값을 숫자 인덱스로 접근할 수 있고 이터레이션도 가능하므로, 외부에 제공하는 API의 경우 나중에 실제 클래스로 변경하기 어려워질 수 있다.
        
        > 모든 부분을 제어할 수 있는 상황이 아니라면 명시적으로 새로운 클래스를 정의하는 편이 낫다.
        > 

<br>

### [예시] `namedtuple`과 여러 클래스로 분리

```python
from collections import namedtuple, defaultdict

Grade = namedtuple("Grade", ("score", "weight"))
```

```python
class Subject:
    # -- 일련의 점수를 포함하는 단일 과목을 표현하는 클래스
    def __init__(self):
        self._grades = []
        
    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))
    
    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight

class Student:
    # -- 한 학생이 수강하는 과목들을 표현하는 클래스
    def __init__(self):
        self._subjects = defaultdict(Subject)
    
    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count

class Gradebook:
    # -- 모든 학생을 저장하는 컨테이너
    def __init__(self):
        self._students = defaultdict(Student)
    
    def get_student(self, name):
        return self._students[name]
```

```python
book = Gradebook()
albert = book.get_student("알버트 아인슈타인")
math = albert.get_subject("수학")
math.report_grade(75, 0.05)
math.report_grade(65, 0.15)
math.report_grade(70, 0.80)
gym = albert.get_subject("체육")
gym.report_grade(100, 0.40)
gym.report_grade(85, 0.60)
print(albert.average_grade()) # 80.25
```
