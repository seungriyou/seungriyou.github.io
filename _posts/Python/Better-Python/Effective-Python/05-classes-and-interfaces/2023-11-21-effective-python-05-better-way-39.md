---
title: "[Better Way #39] 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라"
date: 2023-11-21 12:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

파이썬에서는 **객체**뿐 아니라 **클래스**도 **다형성**을 지원한다. 

이처럼 클래스의 다형성을 이용하면 **같은 인터페이스**를 만족하거나 **같은 추상 기반 클래스**를 공유하는 많은 클래스가 **서로 다른 기능**을 제공할 수 있게 된다.

<br>

> 클래스의 다형성을 이용하여 `MapReduce`를 구현해보자!
{: .prompt-tip}

<br>

## [방법 #1] 인스턴스 메서드 다형성 활용 & 도우미 함수 생성
> 간단하지만 제너릭 X

### (1) <span class="hl">인스턴스 메서드 다형성</span>을 활용하여 `MapReduce`의 각 요소를 구현한다.

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

<br>

### (2) 도우미 함수를 활용하여 각 부분에 해당하는 <span class="hl">객체를 직접 만들고 연결</span>한다.
 -  <details>
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
     

<br>

> **객체를 구성할 수 있는 제네릭한 방법**이 필요하다. 다른 언어에서는 **다형성**을 활용하여 이 문제를 해결할 수 있지만, **파이썬에서는 생성자 메서드가 `__init__`** 밖에 없다. 하지만 하위 클래스가 똑같은 생성자만 제공해야 하는 것은 불합리하다!
> 
> → **클래스 메서드 다형성**을 사용하자!!
{: .prompt-warning}

<br>

## [방법 #2] 클래스 메서드 다형성 활용 (제너릭한 방법)

> **클래스 메서드(class method) 다형성**을 사용한다면, 다형성이 클래스로 만들어낸 개별 객체에 적용되는 것이 아니라 **클래스 전체에 적용**된다.

### (1) <span class="hl">클래스 메서드 다형성</span>을 활용하여 `MapReduce`의 각 요소를 구현한다.

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

<br>
            
### (2) <span class="hl">완전히 제너릭</span>한 `mapreduce` 함수를 작성한다.

> 제너릭한 동작을 위해 **클래스 자체를 파라미터로** 받도록 한다!
    
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
    

<br>

> 각 하위 클래스의 인스턴스 캑체를 결합하는 코드를 변경하지 않아도, `GenericInputData`와 `GenericWorker`의 하위 클래스를 원하는 대로 변경할 수 있다!
{: .prompt-info}
