---
title: "[Effective Python] 05. Classes and Interfaces"
date: 2023-07-08 22:50:00 +0900
categories: [Python, Better Python]
tags: [python, effective python, book]
math: true
---

> 본문은 "파이썬 코딩의 기술 (Effective Python, 2판)"을 읽고 정리한 내용입니다.

<br>

## Better Way 37: 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

- 파이썬 내장 딕셔너리 타입을 통해 동적(dynamic)인 내부 상태를 유지할 수 있으나, **내포 단계가 두 단계 이상**이 되면 더 이상 딕셔너리, 리스트, 집합, 튜플 등의 내장 타입 계층을 추가하지 말아야 한다.
    
    > ex) 튜플에 저장된 내부 원소에 위치를 통해 접근해야 하는 경우: 원소가 추가된다면 코드를 모두 수정해야 한다.
    > 
- 코드에서 **값을 관리하는 부분이 점점 복잡**해지고 있다면 **해당 기능을 여러 클래스로 분리**해야 한다. 이를 통해 인터페이스와 구현 사이에 추상화 계층을 만들 수 있다.

### `namedtuple` 클래스

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

<br>

## Better Way 38: 간단한 인터페이스의 경우 클래스 대신 함수를 받아라

### 훅 (Hook)

- 파이썬에서 함수는 first-class citizen으로 취급되므로, 함수나 메서드를 다른 함수에 넘기거나 변수 등으로 참조할 수 있다.
- 파이썬 내장 API가 실행되는 과정에서 **전달된 함수를 실행**하는 경우, 이러한 함수를 **훅(hook)**이라 한다.
    
    ```python
    names = ["소크라테스", "아르키메데스", "플라톤", "아리스토텔레스"]
    names.sort(key=len) # -- 리스트를 문자열의 길이를 기준으로 정렬
    print(names)
    # ['플라톤', '소크라테스', '아르키메데스', '아리스토텔레스']
    ```
    

### [예시] `defaultdict`의 인자로 `__call__` 메서드를 정의한 클래스의 인스턴스 전달

- `defaultdict`의 경우, 딕셔너리 안에 없는 키에 접근할 경우에 호출되는 **인자가 없는 함수**를 전달할 수 있다. 이 함수는 **존재하지 않는 키에 해당하는 값이 될 객체를 반환**해야 한다.
- 다음과 같은 `log_missing` 함수를 이용하면 **정해진 동작(= 값 삽입)**과 **부수 효과(= 로그 남기기)**를 분리할 수 있다.
    
  ```python
  from collections import defaultdict
  
  def log_missing():
      print("키 추가됨")
      return 0
  
  current = {"초록": 12, "파랑": 3}
  increments = [
      ("빨강", 5),
      ("파랑", 17),
      ("주황", 9)
  ]
  result = defaultdict(log_missing, current) # -- hook 전달
  print("이전:", dict(result))
  for key, amount in increments:
      result[key] += amount
  print("이후:", dict(result))
  
  # 이전: {'초록': 12, '파랑': 3}
  # 키 추가됨
  # 키 추가됨
  # 이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}
  ```
    
- **존재하지 않는 키에 접근한 총 횟수를 카운트**하려는 경우, 다음과 같은 방법으로 접근할 수 있다.
  1. 추적하고 싶은 **상태를 저장하는 작은 클래스**를 정의한다.
  2. `defaultdict`에 전달할 디폴트 값 훅을 1번에서 정의한 클래스의 **`__call__` 특별 메서드**에 정의한다. 해당 메서드는 **상태를 저장하는 클로저 역할**을 하게 된다.
      
      > `__call__`을 사용하면 객체를 함수처럼 호출할 수 있으며(= **callable**), `callable` 내장 함수를 호출하면 `True`가 반환된다.
      > 
    
  ```python
  class BetterCountMissing:
      def __init__(self):
          self.added = 0
          
      def __call__(self):
          self.added += 1
          return 0

  counter = BetterCountMissing()
  result = defaultdict(counter, current)  # -- __call__에 의존
  for key, amount in increments:
      result[key] += amount
  assert counter.added == 2
  ```
    

> **`__call__` 메서드**
>
> - `__call__` 메서드를 정의함으로써 **함수가 인자로 쓰일 수 있는 부분**에 **해당 클래스의 인스턴스**를 사용할 수 있다. 이를 통해 외부에서는 `__call__` 내부에서 어떤 일이 벌어지는지에 대해 전혀 알 필요가 없게 된다.
> - 상태를 유지하기 위한 함수가 필요한 경우, **상태가 있는 클로저를 정의**하는 대신 **`__call__` 메서드가 있는 클래스를 정의**할지 고려해보자.
{: .prompt-info}

<br>

## Better Way 39: 객체를 제너릭하게 구성하려면 `@classmethod`를 통한 다형성을 활용하라

파이썬에서는 **객체**뿐 아니라 **클래스**도 **다형성**을 지원한다. 

이처럼 클래스의 다형성을 이용하면 **같은 인터페이스**를 만족하거나 **같은 추상 기반 클래스**를 공유하는 많은 클래스가 **서로 다른 기능**을 제공할 수 있게 된다.

> 클래스의 다형성을 이용하여 `MapReduce`를 구현해보자!
{: .prompt-warning}

### [방법 #1] 인스턴스 메서드 다형성 활용 & 도우미 함수 생성 (간단하지만 제너릭 X)

1. **인스턴스 메서드 다형성**을 활용하여 `MapReduce`의 각 요소를 구현한다.
    1. **입력 데이터**와 **이 입력 데이터를 소비하는 워커**에 대한 **추상 인터페이스**
        
        > **하위 클래스에서 다시 정의해야 하는 메서드**, 즉 **공통 인터페이스**에 대해서는 **`NotImplementedError`**를 발생시키도록 한다.
        {: .prompt-info}
        
        - **입력 데이터**에 대한 추상 인터페이스
            
          ```python
          class InputData:
              def read(self):
                  raise NotImplementedError
          ```
            
        - **워커**에 대한 추상 인터페이스
            
          ```python
          class Worker:
              def __init__(self, input_data):
                  self.input_data = input_data
                  self.result = None
              
              def map(self):
                  raise NotImplementedError
              
              def reduce(self, other):
                  raise NotImplementedError
          ```
            
    2. 각 추상 인터페이스에 대한 **구체적인 하위 클래스**
        
        > **공통 인터페이스를 상속** 받아 하위 클래스를 작성한다. 이때, **공통 인터페이스를 구현**해야 한다.
        {: .prompt-info}
        
        - **입력 데이터**에 대한 구체적 하위 클래스 (디스크에서 파일을 읽는 동작)
            
          ```python
          class PathInputData(InputData):
              def __init__(self, path):
                  super().__init__()
                  self.path = path
              
              def read(self):
                  with open(self.path) as f:
                      return f.read()
          ```
          
        - **워커**에 대한 구체적 하위 클래스 (`\n` 문자의 개수를 세는 동작)
            
          ```python
          class LineCountWorker(Worker):
              def map(self):
                  data = self.input_data.read()
                  self.result = data.count("\n")
              
              def reduce(self, other):
                  self.result += other.result
          ```
            
    3. 동작 테스트를 위한 dummy file
      
        ```python
        import os
        import random
        
        def write_test_files(tmpdir):
            os.makedirs(tmpdir)
            for i in range(100):
                with open(os.path.join(tmpdir, str(i)), "w") as f:
                    f.write("\n" * random.randint(0, 100))
        
        tmpdir = "dummy_file"
        write_test_files(tmpdir)
        ```
        
2. 도우미 함수를 활용하여 각 부분에 해당하는 **객체를 직접 만들고 연결**한다.
    - <details>
      <summary>코드 전문</summary>
      <div markdown="1">

      ```python
      import os
      
      def generate_inputs(data_dir):
          for name in os.listdir(data_dir):
              yield PathInputData(os.path.join(data_dir, name))
      
      def create_workers(input_list):
          workers = []
          for input_data in input_list:
              workers.append(LineCountWorker(input_data))
          return workers
      ```
      
      ```python
      from threading import Thread
      
      def execute(workers):
          threads = [Thread(target=w.map) for w in workers]
          for thread in threads:  thread.start()
          for thread in threads:  thread.join()
          
          first, *rest = workers
          for worker in rest:
              first.reduce(worker)
          return first.result
      ```
      
      ```python
      def mapreduce(data_dir):
          inputs = generate_inputs(data_dir)
          workers = create_workers(inputs)
          return execute(workers)
      ```
      
      ```python
      tmpdir = "dummy_file"
      result = mapreduce(tmpdir)
      print(f"총 {result} 줄이 있습니다.") # 총 5008 줄이 있습니다.
      ```

      </div>
      </details>
        
    - `Worker` 인스턴스의 `map`을 여러 스레드에 공급하여 실행할 수 있고, 그 후 `reduce`를 반복적으로 호출하여 결과를 최종 값으로 합친다.
    - 잘 작동하지만, **함수가 전혀 제너릭(generic)하지 않다!**
        
        > 즉, 다른 `InputData`나 `Worker`의 하위 클래스를 사용하고 싶다면, 각각에 맞게 `generate_inputs()`, `create_workers()`, `mapreduce()` 함수를 재작성 해야 한다.
        

> **객체를 구성할 수 있는 제네릭한 방법**이 필요하다. 다른 언어에서는 **다형성**을 활용하여 이 문제를 해결할 수 있지만, **파이썬에서는 생성자 메서드가 `__init__`** 밖에 없다. 하지만 하위 클래스가 똑같은 생성자만 제공해야 하는 것은 불합리하다!
> 
> → **클래스 메서드 다형성**을 사용하자!!
{: .prompt-warning}

### [방법 #2] 클래스 메서드 다형성 활용 (제너릭한 방법)

> **클래스 메서드(class method) 다형성**을 사용한다면, 다형성이 클래스로 만들어낸 개별 객체에 적용되는 것이 아니라 **클래스 전체에 적용**된다.

1. **클래스 메서드** **다형성**을 활용하여 `MapReduce`의 각 요소를 구현한다.
    1. **입력 데이터**와 **이 입력 데이터를 소비하는 워커**에 대한 **추상 인터페이스**
        > 클래스 메서드를 사용하려면 제너릭 **`@classmethod`**를 적용한다.
        {: .prompt-info}
        
        - **입력 데이터**에 대한 추상 인터페이스
            
          ```python
          class GenericInputData:
              def read(self):
                  raise NotImplementedError
              
              @classmethod
              def generate_inputs(cls, config):
                  raise NotImplementedError
          ```
            
        - **워커**에 대한 추상 인터페이스
            
          ```python
          class GenericWorker:
              def __init__(self, input_data):
                  self.input_data = input_data
                  self.result = None
              
              def map(self):
                  raise NotImplementedError
              
              def reduce(self, other):
                  raise NotImplementedError
              
              @classmethod
              def create_workers(cls, input_class, config):   # input_class: GenericInputData의 하위 타입
                  workers = []
                  # 클래스 다형성: input_class.generate_inputs
                  for input_data in input_class.generate_inputs(config):
                      workers.append(cls(input_data)) # __init__ 메서드가 아닌, 제너릭 생성자 cls()를 호출함으로써 GenericWorker 객체를 만들 수 있다!
                  return workers
          ```
            
            > 파이썬 클래스의 **생성자**는 **`__init__` 메서드** 뿐인데, **`@classmethod`**를 사용하면 클래스에 `__init__` 메서드가 아닌 **다른 (제너릭) 생성자**를 정의할 수 있다.
            {: .prompt-warning}
            
    2. 각 추상 인터페이스에 대한 **구체적인 하위 클래스**
        - **입력 데이터**에 대한 구체적 하위 클래스 (디스크에서 파일을 읽는 동작)
            
          ```python
          class PathInputData(GenericInputData):
              def __init__(self, path):
                  super().__init__()
                  self.path = path
              
              def read(self):
                  with open(self.path) as f:
                      return f.read()
                  
              @classmethod
              def generate_inputs(cls, config):
                  data_dir = config["data_dir"]
                  for name in os.listdir(data_dir):
                      yield cls(os.path.join(data_dir, name))
          ```
            
        - **워커**에 대한 구체적 하위 클래스 (`\n` 문자의 개수를 세는 동작)
            
          ```python
          class LineCountWorker(GenericWorker):
              # 내용은 기존 LineCountWorker와 동일
          ```
            
2. 도우미 함수를 작성할 필요 없이, 다음과 같이 **완전히 제너릭한 `mapreduce` 함수**를 작성한다.   

    > 제너릭한 동작을 위해 **클래스 자체를 파라미터로** 받도록!
        
    ```python
    def mapreduce(worker_class, input_class, config):   # -- 완전히 제너릭하다!
        workers = worker_class.create_workers(input_class, config)
        return execute(workers)
    ```
      
    ```python
    config = {"data_dir": "dummy_file"}
    # 제너릭하게 작동하므로, 더 많은 파라미터가 필요하다.
    result = mapreduce(LineCountWorker, PathInputData, config)
    print(f"총 {result} 줄이 있습니다.") # 총 5008 줄이 있습니다.
    ```
    

> 각 하위 클래스의 인스턴스 캑체를 결합하는 코드를 변경하지 않아도, `GenericInputData`와 `GenericWorker`의 하위 클래스를 원하는 대로 변경할 수 있다!
{: .prompt-info}