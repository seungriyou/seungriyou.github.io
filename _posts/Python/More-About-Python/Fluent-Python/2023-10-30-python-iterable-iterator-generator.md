---
title: "[Python] 이터러블(Iterable), 이터레이터(Iterator), 제너레이터(Generator)"
date: 2023-10-30 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, iterable, iterator, generator]
math: true
---

## TL;DR 📌

[[📢] Iterable vs. Iterators vs. Generators](/posts/python-iterable-iterator-generator/#-iterable-vs-iterators-vs-generators)

<br>

## 1. 개요

파이썬에서는 **이터레이션(iteration)**을 통해 데이터 시리즈(data series)에 연산을 적용할 수 있으며, **이터레이터(iterator)**를 통해 데이터가 메모리에 수용 가능하지 않은 경우, 필요에 따라 아이템을 lazy 하게 fetch 할 수 있다.

파이썬의 **모든 표준 컬렉션(standard collection)**은 **이터러블(iterable)**인데, 이때 **이터러블은 이터레이터를 제공하는 객체**로 다음과 같은 연산을 제공한다.

- `for` loop
- list/dict/set comprehensions
- unpacking assignments
- construction of collection instances

<br>

## 2. 이터러블(Iterable)

### 2-1. 이터레이터와 이터러블의 관계 (+ `iter()` 함수)

파이썬이 어떤 객체 `x`에 대해 **이터레이트(iterate) 할 때, 자동으로 `iter(x)`를 호출하여 이터레이터(iterator)를 얻는다**. 이러한 **`iter()` 함수**는 built-in 함수이며, 다음과 같이 동작한다.

1. 해당 객체가 **`__iter__` 메서드를 구현**했는지 확인 후 호출하고, 그 결과로 이터레이터를 얻는다.
2. `__iter__` 메서드는 구현되지 않았으나 **`__getitem__` 메서드는 구현**된 경우, `iter()`는 0부터 시작하는 인덱스를 통해 아이템을 fetch 하는 이터레이터를 생성한다.
    
    > 따라서 파이썬의 시퀀스(sequence)는 모두 시퀀스 프로토콜을 따르므로 `__getitem__`을 구현하기 때문에 이터러블이다. 또한, 표준 시퀀스는 `__iter__`도 구현한다.
    > 
    > > **시퀀스 프로토콜 (sequence protocol)**
    > >
    > > `__len__` 메서드와 `__getitem__` 메서드를 가지는 클래스는 시퀀스 프로토콜을 따른다고 할 수 있다.
    > {: .prompt-info}
    
3. 모두 실패하면 `TypeError`를 발생시킨다.

<br>

이때, 이러한 **`iter()` built-in 함수에 전달했을 때 이터레이터를 생성할 수 있는 객체**를 <span class="red">**이터러블(iterable)**</span>이라 한다. 즉, 이터러블은 다음의 두 조건 중 하나를 만족하는 객체를 의미한다.

1. **`__iter__` method**를 구현하고 있어 iterator를 반환할 수 있다.
2. 0-based index를 허용하는 **`__getitem__` method**를 구현하고 있다.
    
    > 따라서 파이썬의 시퀀스는 항상 이터러블이다.
    > 

<br>

> 파이썬은 **이터러블(iterable)**로부터 **이터레이터(iterator)**를 얻는다!
>
> > 이터레이터는 다음 아이템에서 다뤄본다.
{: .prompt-info}

<br>

### 2-2. 이터러블의 타입 체킹

<span class="shl">**덕 타이핑(duck typing)**</span>의 관점에서, 어떠한 객체는 다음의 두 경우 중 하나에 해당하면 이터러블로 간주된다.

1. `__iter__`를 구현한 경우
2. `__getitem__`을 구현한 경우
    
    > 어떤 class가 `__getitem__`을 제공하면, `iter()` built-in은 class의 instance를 iterable로 받아 iterator를 반환한다. 이때, `__getitem__`은 index 0부터 호출되며, 더이상 item이 없으면 `IndexError`를 발생시킨다.
    > 
    
    ```python
    class Spam:
        def __getitem__(self, i):
            print('->', i)
            raise IndexError()
    
    spam_can = Spam()
    iter(spam_can)
    # <iterator at 0x106800640>
    
    list(spam_can)
    # -> 0
    # []
    ```
    

<br>

하지만 <span class="shl">**구스 타이핑(goose typing)**</span>의 관점에서는 `__iter__` 메서드를 구현한 경우에만 이터러블로 간주된다.

```python
from collections import abc

# === __getitem__를 구현한 Spam === #
isinstance(spam_can, abc.Iterable)
# False

# === __iter__를 구현한 GooseSpam === #
class GooseSpam:
    def __iter__(self):
        pass

issubclass(GooseSpam, abc.Iterable)
# True

goose_spam_can = GooseSpam()
isinstance(goose_spam_can, abc.Iterable)
# True
```

위 예제에서 `__getitem__`을 구현한 `spam_can`은 이터러블이지만, `isinstance`로는 `abc.Iterable`로 인식되지 않는다. 

**구스 타이핑 관점에서는 `__iter__`를 구현해야만 이터러블로 간주**되기 때문이다. 이때, `abc.Iterable`은 **`__subclasshook__`**을 구현하므로, subclassing이나 registration이 필요하지 않다.

<br>

> 파이썬의 런타임에서는 <span class="red">**덕 타이핑**</span>이 적용되므로, 파이썬에서 **어떤 객체 `x`가 이터러블인지 확인하는 가장 정확한 방법**은 **`iter(x)`를 호출**하고 **`TypeError` exception을 처리**하는 것이다.
>
> - `isinstance(x, abc.Iterable)`을 사용하면 구스 타이핑으로 인해 `__getitem__`을 구현한 경우를 이터러블로 판단할 수 없기 때문이다.
> - 어떤 객체를 이터러블인지 확인한 후 바로 iterate 하려는 경우, 굳이 명시적으로 따로 확인할 필요 없이 **iterate 하는 코드에서 `try`/`except` block을 통해 `TypeError` exception 처리**를 하면 된다. 명시적으로 타입을 확인하는 것은 나중에 해당 객체를 iterate 하고 싶은 경우에, 미리 에러를 catch 할 수 있다는 점에서 적합하다.
{: .prompt-info}

<br>

### 2-3. `iter()`를 Callable과 함께 사용하기

`iter()`에 다음과 같은 두 인자를 제공함으로써 **함수 혹은 callable 객체**로부터 이터레이터를 만들 수 있다.

1. **첫 번째 인자**: 반복해서 호출되며 값을 생성하는 callable (인자 없이!)
2. **두 번째 인자**: 해당 값을 callable이 생성하는 경우 `StopIteration`을 raise 하도록 하는 sentinel

<br>

다음의 예시 코드에서 `callable_iterator`인 `d6_iter`는 **sentinel value**로 설정된 값인 `1`을 반환하지 않는다. 또한, 한 번 exhausted 된 후 다시 해당 이터레이터를 사용하려면 이터레이터를 rebuild 해야 한다.

```python
from random import randint

def d6():
    return randint(1, 6)

d6_iter = iter(d6, 1)
```

```python
d6_iter
# <callable_iterator at 0x103ccb040>

for roll in d6_iter:
    print(roll)
# 2
# 6
# 4
# 5
# 3
```

<br>

만약 첫 번째 인자로 넘길 **callable에 인자가 필요**한 경우, **`partial()` 함수**를 사용할 수도 있다.

다음은 `iter()`를 통해 block-reader를 구현한 예제이다. **sentinel value**로 설정된 empty `bytes` object가 등장하면 더이상 읽을 byte가 없다는 것이므로 동작을 멈춘다.

```python
from functools import partial

with open('mydata.db', 'rb') as f:
    read64 = partial(f.read, 64)
    for block in iter(read64, b''): # -- empty bytes object is the sentinel
        process_block(block)
```

<br>

### 2-4. `for` 루프의 원리

파이썬에서 `for` 루프는 <span class="blue">**이터러블로부터 이터레이터를 얻어 동작**</span>한다.

다음은 `for` 루프를 통해 `str`(시퀀스, 즉 이터러블)을 iterate 하는 예시이다.

```python
s = 'ABC'
for char in s:
    print(char)
# A
# B
# C
```

이를 **`for` 루프 없이** 직접 구현해보면 다음과 같다.

```python
s = 'ABC'
it = iter(s)    # -- (1) iterable로부터 iterator를 얻는다.
while True:
    try:
        print(next(it))     # -- (2) iterator에서 next를 호출함으로써 다음 item을 얻는다.
    except StopIteration:   # -- (3) iterator가 exhausted 되면 StopIteration이 발생한다.
        del it      # -- (4) StopIteration이 발생하면 iterator object를 discard 한다.
        break       # -- (5) while loop를 빠져나온다.
```

이러한 내부 동작들은 **`for` 루프** 뿐만 아니라 **list comprehension**, **iterable unpacking** 등 다른 iteration context의 로직에 구현되어 있다.

<br>

## 3. 이터레이터(Iterator)

> 그렇다면, 이터러블에서 얻을 수 있다는 이터레이터란 무엇일까?
> 

### 3-1. 이터레이터 인터페이스: `abc.Iterator`

**이터레이터의 파이썬 표준 인터페이스**는 다음의 두 가지 method를 가진다.

1. **`__next__` 메서드**: 시리즈에서 다음 아이템을 반환하며, 다음 아이템이 없다면 `StopIteration`을 raise 한다.
2. **`__iter__` 메서드**: `self`(= 자기자신)를 반환하여 이터러블이 예상되는 곳에서 이터레이터가 사용될 수 있도록 한다.

<br>

이러한 인터페이스는 **`collections.abc.Iterator` ABC**에 나타나있다. 이는 `__next__` abstract method를 선언하고, `__iter__` abstract method가 선언되어 있는 `Iterable`을 상속받는다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-30-01.png){: .w-75}

<br>

실제로 `abc.Iterator`의 코드를 살펴보면 다음과 같다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-30-02.png){: .w-75}

<br>

> 앞서 언급했듯, 파이썬에서 어떤 객체 `x`가 <span class="red">**이터러블**</span>인지 확인하는 가장 정확한 방법은 **`iter(x)`를 호출**하고 **`TypeError` exception을 처리**하는 것이었다.
> 
> > `isinstance(x, abc.Iterable)`을 사용한다면 구스 타이핑 관점에서 확인하므로, `__iter__` 대신 `__getitem__`을 구현한 경우를 이터러블로 판단할 수 없기 때문이다.
> 
> 반면, 어떤 객체 `x`가 <span class="red">**이터레이터**</span>인지 확인하는 가장 정확한 방법은 **`isinstance(x, abc.Iterator)`를 호출**하는 것이다.
>
> > `Iterator.__subclasshook__`에서 `__next__` 메서드와 `__iter__` 메서드가 구현되어 있는지 확인하기 때문이다.
{: .prompt-tip}

<br>

### 3-2. 이터레이터의 특징

> 코드에 등장하는 `Sentence` 클래스는 다음 섹션에서 다루는 “예제용 `Sequence` 클래스”이다. 이는 시퀀스 프로토콜을 따르므로 이터러블에 해당한다.
> 

```python
s3 = Sentence('Life of Brian')
it = iter(s3) # -- build an iterator!
it
# <iterator at 0x1067bdb10>

next(it)
# 'Life'

next(it)
# 'of'

next(it)
# 'Brian'

next(it)
# StopIteration 발생

list(it)
# []

list(iter(s3)) # -- rebuild the iterator!
# ['Life', 'of', 'Brian']
```

1. 이터레이터 `it`에서 `next(it)`로 아이템을 fetch 하다가 더이상 아이템이 없으면 **`StopIteration`**을 raise 한다. 이렇게 되면 해당 **“이터레이터가 exhausted 되었다”**고 표현하며, 이 상태의 이터레이터는 비어있다.

2. 다시 이터레이션을 수행하고 싶다면 `iter(iterable)`을 통해 **이터레이터를 rebuild** 해야 한다. (새로 rebuild 하지 않는 이상 reset은 불가능하다!)
    
    > `Iterator.__iter__`는 `self`를 반환하므로 `iter(iterator)`는 도움이 되지 않는다.
    {: .prompt-info}
    
3. 이터레이터가 필수로 가지는 메서드는 `__next__`와 `__iter__` 뿐이므로, **`StopIteration`이 발생될 때까지 `next()`를 호출**해야 이터레이터에 아이템이 남아있는지 여부를 확인할 수 있다.

<br>

## 4. 표준 이터러블 프로토콜(Standard Iterable Protocol)

표준 이터러블 프로토콜을 구현하는 방법에 대해서 예제 코드를 통해 알아보도록 하자.

### [1] 예제용 `Sequence` 클래스

다음과 같이 단순히 파이썬의 **시퀀스 프로토콜(sequence protocol)**을 만족하는 `Sentence` 클래스를 생각해보자.

> **시퀀스 프로토콜 (sequence protocol)**
>
> `__len__` 메서드와 `__getitem__` 메서드를 가지는 클래스는 시퀀스 프로토콜을 따른다고 할 수 있다.
{: .prompt-info}

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')

class Sentence:
    
    def __init__(self, text) -> None:
        self.text = text
        self.words = RE_WORD.findall(text)  # -- all nonoverlapping matches of the regex
    
    def __getitem__(self, index):
        return self.words[index]
    
    def __len__(self):  # -- iterable을 만드는 데에는 필요 없으나 sequence protocol에 필요
        return len(self.words)
    
    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text) # -- abbreviated string representation

```

이러한 `Sentence` 클래스를 실제로 사용하는 예시는 다음과 같다.

```python
s = Sentence('"The time has come," the Walrus said.')
print(s)
# Sentence('"The time ha... Walrus said.')

for word in s:
    print(word)
# The
# time
# has
# come
# the
# Walrus
# said

print(list(s))
# ['The', 'time', 'has', 'come', 'the', 'Walrus', 'said']
```

<br>

### [2] 이터러블 프로토콜을 적용한 `Sentence` 클래스

우리는 지금까지 **“어떤 객체 `x`에 대해, `iter(x)`를 호출함으로써 이터레이터를 반환받을 수 있는 객체 `x`를 이터러블이라 한다”**는 것을 알게 되었다.

그리고 [1]번에서 다룬 `Sentence` 클래스는 시퀀스 프로토콜을 따르므로 `__getitem__`을 구현하기 때문에 **이터러블**에 해당한다.

따라서 다음과 같이 `Sentence` 클래스를 **이터레이터 디자인 패턴(iterator design pattern)을 적용**하여 **표준 이터러블 프로토콜(standard iterable protocol)을 구현**함으로써 변경할 수 있다.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')

class Sentence:
    
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)
    
    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'
    
    def __iter__(self):
        return SentenceIterator(self.words)

class SentenceIterator:
    
    def __init__(self, words):
        self.words = words
        self.index = 0
    
    def __next__(self):
        try:
            word = self.words[self.index]
        except IndexError:
            raise StopIteration()
        self.index += 1
        return word
    
    def __iter__(self):
        return self
```

위의 코드에서 `Sentence` 클래스의 `__iter__` 메서드에서 **(1) 이터레이터인 `SentenceIterator`를 instantiate** 한 후 **(2) return** 하므로 이터러블 프로토콜을 만족한다고 할 수 있다.

> 이때, `SentenceIterator`를 `abc.Iterator`를 상속 받아 만든다면 `__iter__` 메서드는 구현할 필요가 없다.
{: .prompt-info}

<br>

하지만 **이터러블 혹은 이터레이터를 구현할 때, 다음의 것들에 주의**해야 한다.

| 메서드 | 주의할 점 |
| --- | --- |
| 이터러블의 `__iter__`   | 매번 새로운 이터레이터를 instantiate 해야 한다!<br>(같은 객체를 계속 반환하면 X) |
| 이터레이터의 `__next__` | 개별적인 아이템을 반환해야 한다. |
| 이터레이터의 `__iter__` | self를 반환해야 한다. |

> 즉, **(1) 하나의 <span class="red">이터러블</span> 객체로부터 여러 개의 독립적인 <span class="red">이터레이터</span>**를 얻을 수 있어야(ex. `iter(my_iterable)`) 하며, **(2) 각 이터레이터는 자신만의 internal state**를 가져야 한다.
{: .prompt-tip}

이러한 이유로 `Sentence` 클래스에 `__next__`를 구현하지 않고, `SentenceIterator` 클래스를 따로 생성하여 `Sentence` 클래스의 `__iter__`에서는 **이 이터레이터를 새롭게 생성**하도록 구현하는 것이다.

<br>

### [3] 제너레이터 함수를 통해 이터러블 프로토콜을 구현한 `Sentence` 클래스

[2]번에서 구현한 것과 같이 이터러블 프로토콜을 구현할 수도 있으나, `yield` 키워드가 포함된 **제너레이터 함수(generator function)**를 통해 `SentenceIterator` 없이도 이터러블 프로토콜을 구현할 수도 있다.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')

class Sentence:
    
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)
    
    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)
    
    def __iter__(self):
        for word in self.words:
            yield word
```

위의 코드에서 살펴볼 수 있는 특징은 다음과 같다.

- `__iter__`에서 `return` 이 필요하지 않으며, `StopIteration` 또한 발생하지 않는다.
- `Sentence.__iter__` 는 **제너레이터 함수(generator function)**이므로, 호출 시 `Iterator` 인터페이스를 구현하는 **제너레이터 객체(generator object)**를 생성한다. 따라서 `SentenceIterator`가 더 이상 필요하지 않다.

> 이렇게 제너레이터를 통해 구현한 방식에서 **lazy 버전**, **제너레이터 식을 통해 구현한 버전**을 추가로 더 살펴볼 예정이다.
{: .prompt-info}

<br>

그렇다면, 제너레이터(generator)란 무엇일까?

<br>

## 5. 제너레이터(Generator)

제너레이터(generator)는 간단하게 말하자면 <span class="shl">**파이썬 컴파일러를 통해 생성된 이터레이터**</span>이다. 제너레이터 객체를 생성하려면 두 가지 방법을 사용할 수 있다.

1. `yield` 키워드를 사용하여 **제너레이터 함수(generator function)** 만들기
2. **제너레이터 식(generator expression)** 작성하기

이러한 제너레이터 객체는 **`__next__` 메서드를 제공**하기 때문에 이터레이터라고 볼 수 있다.

<br>

### 5-1. 제너레이터 함수(Generator Function)

우선, 제너레이터 함수와 제너레이터 객체에 대해 정리하면 다음과 같다.

| Term | Description | Action |
| ---- | ---- | ---- |
| **제너레이터 함수**<br>(generator function) | 내부에 **`yield` 키워드**를 가지고 있는 파이썬 함수<br>(즉, 제너레이터 팩토리) | 제너레이터를 반환한다. |
| **제너레이터 객체**<br>(generator object)   | **제너레이터 함수** 혹은 **제너레이터 식**으로부터 생성되는 객체  | 값을 yield 한다. |

<br>

이어서 코드 예시를 살펴보도록 하자.

다음의 코드에서 **`for` 루프의 동작**은 **(1) `g = iter(gen_AB())`로 제너레이터 객체**를 얻은 후, **(2) 각 iteration에서 `next(g)`를 호출**하는 것과 같다.

```python
def gen_123():
    yield 1
    yield 2
    yield 3

gen_123
# <function __main__.gen_123()>

gen_123()
# <generator object gen_123 at 0x103d3f610>

for i in gen_123():
    print(i)
# 1
# 2
# 3

g = gen_123()
next(g)
next(g)
next(g)
next(g)
# StopIteration
```

<br>

다음과 같이 제너레이터 함수 안의 `yield` 사이에 `print` 문이 섞여있는 경우도 살펴보자.

```python
def gen_AB():
    print('start')
    yield 'A'
    print('continue')
    yield 'B'
    print('end.')

for c in gen_AB():
    print('-->', c)

# start
# --> A
# continue
# --> B
# end.
```

위 코드에서는 세 번째 `next()` 호출 시 `'end.'`가 출력되는데, 이때의 동작은 다음과 같다.

1. **제너레이터 함수 body**의 **맨 마지막**에 도달한다. 
2. **제너레이터 객체**는 **`StopIteration`**을 raise ****한다.
3. `for` 루프 machinery는 exception을 catch하여 루프를 끝낸다.

<br>

### 5-2. 지연 제너레이터(Lazy Generator)

> **lazy implementation**은 가능한 마지막 순간까지 값을 생성하는 것을 미룬다.
{: .prompt-info}

eager implementation의 경우, 전체 데이터를 처리해야 하므로 많은 양의 메모리가 요구된다.

하지만 **`re.findall`의 lazy version**인 **`re.finditer`**를 사용하면, `re.MatchObejct` 객체를 **on demand로 yield** 하는 generator를 얻을 수 있다. 이때, 많은 matches가 존재한다면, `re.finditer`는 **많은 양의 메모리를 아낄** 수 있다.

따라서 `Sentence` 클래스를 다음과 같이 lazy 버전으로 수정할 수 있다.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')

class Sentence:
    
    def __init__(self, text):
        self.text = text    # -- words list를 가질 필요가 없다.
    
    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'
    
    def __iter__(self):
        for match in RE_WORD.finditer(self.text):
            yield match.group()     # -- MatchObject instance로부터 matched text를 추출한다.
```

<br>

### 5-3. 제너레이터 식(Generator Expression)

**간단한 제너레이터 함수는** **제너레이터 식**으로 교체(→ syntactic sugar)할 수 있기 때문에, 제너레이터 식 또한 마찬가지로 **제너레이터 객체**를 생성한다.

<br>

다음은 리스트 컴프리헨션을 제너레이터 식으로 변경하여 lazily iterate 하는 예시이다.

- **eagerly iterate 하는 예제**: 리스트 컴프리헨션
    
    ```python
    res1 = [x*3 for x in gen_AB()]  # -- eagerly iterates (list comprehension)
    # start
    # continue
    # end.

    for i in res1:
        print('-->', i)
    # --> AAA
    # --> BBB
    ```
    
- **lazily iterate 하는 예제**: 제너레이터 식
    
    ```python
    res2 = (x*3 for x in gen_AB())  # -- generator is not consumed here!

    for i in res2:
        print('-->', i)
    # start
    # --> AAA
    # continue
    # --> BBB
    # end.
    ```
    

<br>

이러한 제너레이터 식을 통해 `Sentence` 클래스의 `__iter__` 메서드를 다음과 같이 변경할 수도 있다.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')

class Sentence:
    
    def __init__(self, text):
        self.text = text    # -- words list를 가질 필요가 없다.
    
    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'
    
    def __iter__(self):
        # -- generator function이 아닌 generator expression을 이용한다! (no yield)
        return (match.group() for match in RE_WORD.finditer(self.text))
```

- `yield`가 없으므로 `__iter__`은 제너레이터 함수는 아니지만, 제너레이터 식을 사용한다.
- `__iter__`의 caller는 제너레이터 객체를 얻게 된다.

<br>

> **제너레이터 함수 vs. 제너레이터 식**
> 
> - **제너레이터 함수**: 더 유연하다.
>     - complex logic w/ multiple statements
>     - can be used as coroutines
> - **제너레이터 식**: 간단한 경우에 대해 가독성이 더 좋다.
>     - 제너레이터 식으로 2줄 이상 작성해야 한다면 제너레이터 함수를 사용하자.
{: .prompt-tip}

<br>

### 5-4. 서브 제너레이터(Sub-Generator) (w/ `yield from`)

> 관련 포스팅: [[Better Way #33] yield from을 사용해 여러 제너레이터를 합성하라](/posts/effective-python-04-better-way-33/)
{: .prompt-info}

<br>

## [📢] Iterable vs. Iterators vs. Generators
> 지금까지 다룬 내용을 간단히 정리하면 다음과 같다.

- <span class="shl">**이터러블(iterable)**</span>: **`iter()` built-in 함수**에 전달했을 때 **이터레이터(iterator)를 생성**할 수 있는 객체
    - 다음의 두 조건 중 하나를 만족하면 이터러블이다.
        1. 이터레이터를 반환하는 **`__iter__` 메서드**를 구현한다.
        2. 0-based index를 허용하는 **`__getitem__` 메서드**를 구현한다.
    - 파이썬은 **이터러블**로부터 **이터레이터**를 얻는다.

- <span class="shl">**이터레이터(iterator)**</span>: **`__iter__`**, **`__next__` 메서드**를 구현한 객체
    - 클라이언트 코드에서 소비되는 데이터를 생성하도록 설계되었다.
    - 파이썬의 대부분의 이터레이터는 제너레이터이다.

- <span class="shl">**제너레이터(generator)**</span>: 파이썬 컴파일러를 통해 생성된 이터레이터
    - **제너레이터 객체**를 생성하는 방법은 다음의 두 가지이다.
        1. `yield` 키워드를 통해 **제너레이터 함수**를 만든다.
        2. 간단한 경우라면 **제너레이터 식**을 작성한다.
    - 제너레이터 객체는 `__next__`를 제공하므로, 일종의 **이터레이터**이다.
    

<br>

## References

- “Fluent Python (2nd Edition)”, Ch17. Iterators, Generators, and Classic Coroutines
