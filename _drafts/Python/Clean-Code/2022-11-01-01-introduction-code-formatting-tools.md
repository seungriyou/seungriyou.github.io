---
title: "[Clean Code] 01. Introduction, Code Formatting, Tools"
date: 2022-11-01 10:50:00 +0900
categories: [Python, Clean Code in Python]
tags: [python, clean code, book]     # TAG names should always be lowercase
---

> 본문은 파이썬 클린코드, 마리아노 아나야(1판)와 Clean Code in Python, Mariano Anaya(2nd Edition)를 정리한 내용입니다.

<br>

## 클린 코드

클린 코드란 품질 좋은 소프트웨어를 개발하고, 견고하고 유지보수가 쉬운 시스템을 만들고, 기술 부채를 회피하는 것을 뜻한다. 이를 통해 **민첩한 개발**과 **예측 가능한 속도로 지속적인 배포**가 가능하다.

클린 코드인지 아닌지는 **다른 엔지니어가 코드를 읽고 유지 관리할 수 있는지 여부**에 달려있다.

<br>

### 프로젝트 코딩 스타일 가이드 준수

품질 표준을 지키기 위해 프로젝트에서 따라야만 하는 최소한의 요구사항을 **코딩 가이드라인**이라 한다.

좋은 코드 레이아웃에서 가장 필요한 특성은 **일관성**이며, 팀에서 표준화된 구조를 사용한다면 신속하게 패턴을 파악할 수 있다.

파이썬이 따라야 하는 코딩 스타일은 **PEP-8**이다. 이는 파이썬 구문의 특성을 고려하여 작성되었으므로, 다른 표준을 고안하기 보다는 순수 PEP-8을 사용하든 PEP-8을 확장해서 사용할 것을 권장한다.

<br>

> **PEP-8의 특징**
> 
> - **검색 효율성(grepability)** - 키워드 인자(keyword argument)에 값을 할당할 때는 띄어쓰기를 사용하지 않지만, 변수에 값을 할당할 때는 띄어쓰기를 사용하도록 권고한다.
>     
>     ```bash
>   grep -nr "location=" .  # 키워드 인자에 값을 할당하는 경우를 검색
>   grep -nr "location =" . # 변수에 값을 할당하는 경우를 검색
>     ```
>     
> - **일관성** - 코드가 일정한 포맷(코드 레이아웃, 문서화, 이름 작명 규칙 등)을 가지면 훨씬 쉽게 읽을 수 있다.
>     
> - **코드 품질** - 버그와 실수를 쉽게 찾을 수 있으며, 코드 품질 도구도 사용할 수 있다.


<br>

---

## Docstring과 Annotation

### Docstring

코드에 **주석(comment)**을 다는 것은 외부 라이브러리에 오류가 있는 경우 정도를 제외하면 **지양**해야 한다. 그 이유는 다음과 같다.

1. 코드로 아이디어를 제대로 표현하지 못했음을 나타내는 것이다.
2. 실제 동작하는 것과 주석의 다른 점을 파악하는 경우, 오해의 소지가 있다.

**docstring**은 **소스코드에 포함된 문서**로, 특정 컴포넌트(ex. 모듈, 클래스, 메서드, 함수 등)에 대한 문서화이다.

docstring이 필요한 이유 중 하나는 <span class="hl">파이썬이 동적 타이핑</span>을 하기 때문인데, 예상되는 함수의 입력과 출력을 문서화하면 사용자가 사용할 때 함수가 어떻게 동작하는지 이해하기 쉬워진다.

IPython 대화형 인터프리터에서는 함수 뒤에 물음표 두 개 `??`를 사용하여 함수의 docstring을 가져올 수 있다.

객체에 docstring이 정의되어 있으면 `__doc__` 속성을 통해 접근이 가능하다.

docstring의 단점으로는 **지속적으로 수작업**을 해야한다는 점과 **상세하게 작성**해야 한다는 점이 있다. 코드를 변경한 경우, 위키, 사용자 매뉴얼, README, docstring 등 관련된 모든 내용을 업데이트하는 것이 중요하다.

<br>

### Annotation

**annotation**은 코드 사용자에게 <span class="hl">함수 인자로 어떤 값이 와야 하는지 힌트를 주는 것</span>이다. 하지만 이를 통해 파이썬이 타입을 검사하거나 강제하지는 않는다.

**annotation으로 지정할 수 있는 것**에는 다음의 것들이 있다.

1. 예상 타입
2. 변수의 의도를 설명하는 문자열
3. 콜백이나 유효성 검사 함수로 사용할 수 있는 callable

annotation을 사용하면 `__annotations__`라는 속성이 생성되는데, 이는 annotation의 이름과 값을 매핑한 dictionary 타입의 값이다. 이를 통해 **문서 생성**, **유효성 검증** 또는 **타입 체크**를 할 수 있다.

<br>

<span class="hl">타입 힌팅(type hinting)</span>은 코드 전체에 올바른 타입이 사용되었는지 확인하고 호환되지 않는 타입이 발견되었을 때 사용자에게 힌트를 주는 것이다.

타입 힌팅을 통해 **타입 체크** 뿐만 아니라 **meaningful names**와 **abstractions**를 사용할 수 있다. 

```python
# [Before] delay_in_seconds 라는 인자의 이름이 verbose 하다.
def launch_task(delay_in_seconds):
    ...

# [After] meaningful name과 abstraction을 사용할 수 있고, 재사용도 할 수 있다.
Seconds = float
def launch_task(delay: Seconds):
    ...
```

```python
# simplest form
def process_clients(clients: list):
    ...

# a bit more detail
def process_clients(clients: list[tuple[int, str]]):
    ...

# meaning is clearer w/ alias
from typing import Tuple
Client = Tuple[int, str]
def process_clients(clients: list[Client]):
    ...
```

> **`typing.Tuple` vs. `tuple`**
>
> > 참고: <https://stackoverflow.com/questions/39458193/using-list-tuple-etc-from-typing-vs-directly-referring-type-as-list-tuple-etc>
> 
> - **Python 3.8**까지는 `tuple`과 `list`가 generic type으로 지원되지 않았으므로, 아직 Python 3.8 이하 버전을 지원하려면 `typing` generics(`typing.Tuple`, `typing.List`)를 이용해야 한다.
> 
> - **Python 3.9**부터는 standard collection을 이용한 type hinting이 지원된다.
{: .prompt-info}

파이썬 3.5부터는 **`typing` 모듈**이 도입되었으며, PEP-526, PEP-557의 도입 후부터는 **함수의 파라미터와 리턴 타입**뿐만 아니라 **변수에 직접 주석**을 달 수 있다. 

이를 통해 클래스 내에 **속성**을 선언하고, 각각의 **타입**을 지정하고, `@dataclass` decorator를 이용하여 `__init__` 메서드에 명시적으로 설정할 필요 없이 **instance 속성**으로 다룰 수 있다.

```python
from dataclasses import dataclass

@dataclass
class Point:
    lat: float
    long: float
```

<br>

> **annotation이 docstring을 대체하는 것일까?**
> 
> docstring에 포함된 정보의 일부를 annotation으로 대체할 수 있는 것은 사실이지만, docstring을 통해 더 나은 문서화를 위한 여지를 남겨두어야 한다. (ex. 파라미터와 함수 반환 값의 예상 형태가 dictionary인 경우, 더 상세한 형태를 docstring으로 문서화 한다.)
{: .prompt-info}

<br>

---

## 도구 설정

반복적인 확인 작업을 줄이기 위해 **코드 검사를 자동 실행**하는 도구를 설정할 수 있다. 이는 테스트/체크리스트의 일부가 되어 **지속적인 통합 빌드(CI, Continuous Integration)**의 하나가 되어야 하며, 검사에 실패하면 빌드도 실패해야 한다.

<br>

### `mypy`, `pytype` - Type Consistency

`mypy`는 파이썬에서 일반적으로 사용하는 정적 타입 검사 도구로, 프로젝트의 모든 파일을 검사하여 **타입 불일치를 검사**해준다. `mypy <파일명>`을 입력하면 타입 검사 결과를 제공해주며, 가끔 잘못된 탐지를 하는 경우, 다음과 같이 문장 끝에 주석을 추가하여 무시될 수 있도록 할 수 있다.

```python
type_to_ignore = "something" # type: ignore
```

발생 가능한 런타임 에러에 대해서도 리포트해주는 `pytype`이라는 도구도 사용할 수 있다.

<br>

### `pylint`, `coala` - Generic Validation

`pycodestyle`, `flake8`, `pylint`는 **PEP-8을 준수했는지 여부**를 검사하는 도구이다. `pylint`는 그 중에서도 가장 완결성이 높고 엄격하며 설정 가능한 옵션이 많다. 이는 `.pylintrc` 파일을 통해 설정값을 바꿀 수 있으며, 다음과 같이 overrule을 통해 특정 restriction을 disable 할 수도 있다.

```
[DESIGN]
disable=missing-function-docstring
```

뿐만 아니라 error를 사용자에게 알리고 자동으로 수정하는 기능도 지원해주는 `Coala`라는 도구도 있다. 이와 같은 도구들은 predefined rules외에도 own rules를 추가할 수 있다.

<br>

### `black`, `yapf` - Automatic Formatting

`black`은 **자동으로 코드를 포맷팅**하는 도구로, 문자열에 항상 쌍따옴표를 사용하는 등 항상 같은 방법으로 결과를 출력한다. 다음과 같이 line의 최대 길이를 설정할 수 있다.

```python
black -l 79 *.py
```

default로는 코드를 포맷팅하지만, `--check` 옵션을 통해 validation을 수행할 수도 있다. 이는 automatic checks나 CI process에서 사용될 수 있다.

기존 프로젝트(ex. legacy project)에 `black`을 적용할 때, 마주할 수 있는 두 가지 시나리오는 다음과 같다.

![](/assets/img/posts/Python/Clean-Code/2022-11-01-01.png){: width="80%" height="80%"}
_Clean Code in Python, Mariano Anaya (2nd Edition), p25_

엄격한 `black`과 다르게 partial formatting을 지원하는 highly customizable한 `yapf` 라는 도구도 있다. 특히, legacy repository에 적용하는 경우에는 `yapf`가 더 적합할 수 있다. 이러한 도구들을 IDE나 `git pre-commit hook`에 설정하여 편리하게 사용할 수 있다.

<br>

### `Makefile` - Automatic Checks

`Makefile`은 Unix 개발환경에서 **해당 프로젝트 내에서 실행되어야 할 명령어들**(ex. compiling, running, …)을 설정하는 것을 돕는 도구이다. 이를 프로젝트 루트 디렉토리에 생성하면, 포맷팅 검사나 코딩 컨벤션 검사를 자동화할 수 있다.

가장 좋은 방법은 테스트를 위한 각각의 target을 만들고, 이것들을 모두 실행하는 또 다른 target을 만드는 것이다.

```makefile
.PHONY: typehint
typehint:
    mypy --ignore-missing-imports src/

.PHONY: test
test:
    pytest tests/

.PHONY: lint
lint:
    pylint src/

.PHONY: checklist
checklist: lint typehint test

.PHONY: black
black:
    black -l 79 *.py

.PHONY: clean
clean:
    find . -type f -name "*.pyc" | xargs rm -fr
    find . -type d -name __pycache__ | xargs rm -fr
```

개발 환경과 CI 빌드 시에 다음의 명령어를 사용하여 실행할 수 있다. 내부 동작 중 오류가 발생하면 중단된다.

```bash
make checklist
```

<br>

<span class="hl">Makefile의 장점</span>은 다음과 같다.

1. 반복적인 작업을 자동화할 수 있는 쉬운 방법이다. 
    
    > 내부에서 사용하는 도구와 파라미터에 상관없이 동일한 명령어로 실행할 수 있으므로 배우기 쉬우며, 도구가 바뀌더라도 명령어는 동일하다.
    
2. CI 도구에서도 이를 사용할 수 있다.
    
    > CI 도구에서 다시 설정해야 할 내용이 적어진다.
