---
title: "[Python] pyproject.toml 파일의 역할 (+ poetry, 파이썬 패키징)"
date: 2024-02-17 19:50:00 +0900
categories: [Python, Deeper Python]
tags: [python, keyword, poetry, packaging]
math: true
---

의존성 관리를 위해 `poetry`를 사용하면 `pyproject.toml` 파일이 생성된다.

하지만 `pyproject.toml` 파일의 이름만 봤을 때는 `poetry`에 한정해서만 사용될 것 같지는 않다.

그렇다면 `pyproject.toml`의 파일의 본래 용도는 무엇인지, 어떻게 사용 가능한지에 대해 알아보자!

<br>

## 파이썬 패키징

우선, **파이썬 패키징**이 무엇인지에 대해 알아보자.

파이썬에서는 하나의 `.py` 파일을 **모듈(module)**이라 하고, 이러한 관련 있는 모듈의 집합을 **패키지(package)**라 한다.

> 파이썬 프로젝트에 대한 경험이 있다면 디렉토리마다 `__init__.py` 파일을 본 적이 있을 것이다. 바로 이러한 `__init__.py` 파일은 **해당 디렉터리가 파이썬 패키지의 일부**임을 알려주는 역할을 한다. (파이썬 3.3+부터는 필수가 아니라고 한다.)
> 

이처럼 파이썬 프로젝트를 패키징하여 패키지를 만들게 되면 해당 패키지를 [PyPI(Python Packaging Index)](https://pypi.org/)와 같은 곳에 게시하는 등의 방법을 통해 **다른 누군가도 설치하여 사용**할 수 있게 된다. 따라서 **다른 환경에서도 해당 패키지가 잘 동작할 수 있도록 보장**해야 한다.

<br>

> 파이썬 패키징에 대해 자세히 알고 싶다면 다음의 포스팅을 참고하는 것을 권장한다.
> 
> → **[파이썬 패키징의 과거, 현재, 그리고 미래](https://ryanking13.github.io/2021/07/11/python-packaging.html)**
{: .prompt-warning}

<br>

## `pyproject.toml`의 도입 배경

파이썬 패키지는 빌드될 때 여러 모듈들(ex. `setuptools`, `wheel`)을 필요로 한다. 따라서 작성한 패키지를 어딘가에 업로드하는 경우에는 다른 환경에서도 해당 패키지가 잘 동작할 수 있도록 보장하기 위한 코드를 추가로 작성해왔다.

이전에는 주로 `setuptools`를 위한 빌드 스크립트 파일인 `setup.py` 라는 파일에 패키지의 의존성, 파이썬 버전, 어플리케이션의 엔트리포인트 등와 같은 메타데이터들을 작성했었다. 하지만 이러한 **`setup.py` 파일에는 큰 문제점**이 있었다!

> `setuptools`는 패키지를 빌드하고 설치하는 기능을 제공할 뿐만 아니라, PyPI에 패키지를 업로드하고 테스트를 수행하는 등의 다양한 부가 기능도 지원하는 도구이다.
{: .prompt-tip}

<br>

바로 `setup.py`는 `setuptools`가 사용하는 설정 파일이지만, **`setup.py` 파일은 `setuptools`에 의존적**이기 때문에 chicken and egg problem이 발생하게 된다는 것이었다.

사실 `setuptools`는 파이썬의 공식적이며 유일한 빌드 시스템이 아니기 때문에, `setup.py`에 의존하는 파이썬 패키징 방식에서는 `setuptools`가 없으면 빌드가 불가능하다.

<br>

여기에서 **chicken and egg problem**이 발생하는 부분을 정리하면 다음과 같다.

1. 패키지를 빌드하기 위해 필요한 메타데이터를 `setup.py`에 명시하므로, `setuptools`가 아닌 또 다른 빌드 시스템을 이용해서 패키지를 빌드하려면 그 사실 또한 `setup.py`에 명시해야 한다.
2. 하지만 `setup.py`는 `setuptools`라는 빌드 시스템에 의존적이기 때문에, 파싱하기 위해서 `setuptools`가 필요하다.

<br>

따라서 <span class="red">**특정한 빌드 시스템에 종속되지 않는 설정 파일**</span>로서 `pyproject.toml` 파일이 도입되었다. 여기에는 파이썬 패키지를 어떻게 빌드하는지, 어떠한 빌드 시스템을 사용해야 하는지(물론 `setuptools`를 사용하는 것도 가능), 특정 버전이 필요하다면 어떤 버전을 지정해야 하는지 등의 정보를 명시할 수 있다.

<br>

## `pyproject.toml`의 또 다른 역할

위에서 살펴본 바와 같이, 본래 `pyproject.toml` 파일은 패키지 빌드와 관련된 설정 정보들을 명시하기 위해 도입되었다. 하지만 여러 프로젝트들을 살펴보면 `pyproject.toml` 파일에 빌드와 관련이 없는 정보들(ex. 빌드 전에 수행되어야 하는 테스트, 코드 포맷팅 등에 대한 정보)도 적혀있는 것을 어렵지 않게 찾아볼 수 있다. 이는 `pyproject.toml`의 용도가 확장된 것이라 볼 수 있다.

> 즉, 정리하면 `pyproject.toml`은 **패키지 빌드 도구** 뿐만 아니라, 린터, 타입 체커 등과 같은 **여러 개발과 관련된 도구들**에 의해서도 사용되는 **설정 파일**이라고 할 수 있다.
{: .prompt-tip}

<br>

## `pyproject.toml`에 작성할 수 있는 테이블

> 작성 예시는 <https://packaging.python.org/en/latest/guides/writing-pyproject-toml/> 을 참고한다.
{: .prompt-info}

`pyproject.toml` 파일에 **기입할 수 있는 TOML 테이블의 종류**에는 다음의 세 가지가 있다.

1. `[build-system]` : 패키지 및 프로젝트에 필요한 빌드 시스템을 명시한다. (작성 매우 권장)
2. `[project]` : 의존성, 이름 등과 같은 기본적인 메타데이터를 명시한다.
    
    > `poetry`에서는 `[project]` 테이블 대신 `[tool.poetry]` 테이블을 사용한다.
    > 
3. `[tool]` : 각 도구마다 설정을 추가한다.

<br>

## References

- <https://ryanking13.github.io/2021/07/11/python-packaging.html>
- <https://packaging.python.org/en/latest/guides/writing-pyproject-toml/>
- <https://crunchingthedata.com/what-is-a-pyproject-toml-file/>
- <https://crunchingthedata.com/cs01-create-python-package/>
- <https://stackoverflow.com/questions/62983756/what-is-pyproject-toml-file-for>
- <https://data-newbie.tistory.com/770>
- <https://tech.buzzvil.com/blog/setup.py-멈춰/>
- <https://wikidocs.net/1418>
