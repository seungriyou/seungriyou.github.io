---
title: "[Better Way #34] send로 제너레이터에 데이터를 주입하지 말라"
date: 2023-07-08 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

## `send` 메서드

- 파이썬 제너레이터가 지원하는 메서드로, `yield` 식을 양방향 채널로 만들어주어 <span class="hl">데이터를 제너레이터에 주입</span>할 수 있도록 한다.
- 제너레이터는 <span class="hl">**`send`로 주입된 값**을 **`yield` 식이 반환하는 값**을 통해 받으며</span>, 이 값을 **변수에 저장해 활용**할 수 있다.
    - 제너레이터가 재개(resume)될 때 `yield`가 `send`에 전달된 파라미터 값을 반환한다.
    - 따라서 **최초로 `send`를 호출**할 때 인자로 전달할 수 있는 유일한 값은 **`None` 뿐**이다. (아직 `yield` 식에 도달하지 못했으므로)
- `send`와 제너레이터를 합성하는 `yield from`을 함께 사용하면 각각의 내포된 제너레이터의 출력의 처음에 `None`이 등장하게 된다.

<br>

## `send` 메서드를 사용하지 않고 제너레이터에 데이터를 주입하는 방법

- **함수에 이터레이터를 전달하는 방법**
    
  ```python
  def wave_cascading(amplitude_it, steps):
      step_size = 2 * math.pi / steps
      for step in range(steps):
          radians = step * step_size
          fraction = math.sin(radians)
          amplitude = next(amplitude_it)  # -- 다음 입력 받기
          output = amplitude * fraction
          yield output
  ```
    
- **합성에 사용할 여러 제너레이터 함수에 같은 이터레이터를 넘기는 방법**
    - **이터레이터는 상태**가 있으므로, 내포된 각각의 제너레이터는 앞에 있는 제너레이터가 처리를 끝낸 시점부터 데이터를 가져와 처리한다.
    - 아무데서나 이터레이터를 가져올 수 있으며, 이터레이터가 완전히 동적인 경우에도 잘 작동한다.
    - 하지만 **제너레이터가 항상 thread-safe 한 것은 아니므로**, 스레드 경계를 넘나들면서 제너레이터를 사용해야 한다면 `async` 함수가 더 나은 해법일 수 있다.
    
  ```python
  def complex_wave_cascading(amplitude_it):
      # -- 동일한 이터레이터를 여러 제너레이터 함수에 넘김
      yield from wave_cascading(amplitude_it, 3)
      yield from wave_cascading(amplitude_it, 4)
      yield from wave_cascading(amplitude_it, 5)
  
  def run_cascading():
      amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
      it = complex_wave_cascading(iter(amplitudes))
      for amplitude in amplitudes:
          output = next(it)
          transmit(output)
  
  run_cascading()
  ```
