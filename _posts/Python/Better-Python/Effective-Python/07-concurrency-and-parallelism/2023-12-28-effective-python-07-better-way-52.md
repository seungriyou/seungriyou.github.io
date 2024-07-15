---
title: "[Better Way #52] 자식 프로세스를 관리하기 위해 subprocess를 사용하라"
date: 2023-12-28 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch07 concurrency and parallelism, subprocess]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 07. Concurrency and Parallelism"**을 읽고 정리한 내용입니다.

<br>

> **개념 잡기**
> - <span class="ulr">**동시성(concurrency)**</span>
> 
>   - 컴퓨터가 같은 시간에 여러 다른 작업을 처리하는 것처럼 보이는 것이다.
>   - CPU 코어가 하나뿐인 컴퓨터에서 OS는 유일한 프로세스 코어에서 실행되는 프로그램을 아주 빠르게 번갈아가며 실행한다.
>   - 여러 프로그램이 마치 동시에 수행되는 것처럼 보인다.
>   
> - <span class="ulr">**병렬성(parallelism)**</span>
> 
>   - 같은 시간에 실제로 여러 다른 작업을 처리하는 것이다.
>   - 각 CPU 코어는 서로 다른 프로그램의 명령어를 실행하므로, CPU 코어가 여러 개인 컴퓨터는 여러 프로그램을 동시에 실행할 수 있다.
> 
> - <span class="ulg">**동시성 프로그래밍**</span>
>
>   - I/O 등으로 인해 **지연 시간**이 있는 실행 경로가 많다면, 동시성 프로그램에서도 속도 향상이 일어난다.
>
>     > C10K(1만 클라이언트 동시 접속) 문제 참고하기!
>
>   - 파이썬에서는 스레드(thread), 코루틴(coroutine) 등을 통해 동시성 프로그래밍을 수행할 수 있다.
{: .prompt-info}

<br>

파이썬이 시작한 **자식 프로세스**는 <span class="shl">파이썬 인터프리터 프로세스 및 서로와 병렬로 실행</span>되기 때문에 파이썬이 컴퓨터의 모든 CPU 코어를 사용할 수 있다.

파이썬이 자식 프로세스를 관리할 때는 **`subprocess` 내장 모듈**을 사용하는 것이 가장 좋다.

<br>

## `run` 함수

간단하게 자식 프로세스를 실행하는 방법이다. 이렇게 실행된 자식 프로세스는 **부모 프로세스(= 파이썬 인터프리터)와 독립적으로 실행**된다.

```python
import subprocess

result = subprocess.run(
    ["echo", "자식 프로세스가 보내는 인사!"], capture_output=True, encoding="utf-8"
)

result.check_returncode()
print(result.stdout)
# 자식 프로세스가 보내는 인사!
```

<br>

## `Popen` 클래스

### [1] Polling

`run` 함수 대신 `Popen` 클래스를 통해 하위 프로세스를 만들면, **파이썬이 다른 일을 하면서 주기적으로 자식 프로세스의 상태를 검사(polling)**할 수 있다.

```python
proc = subprocess.Popen(["sleep", "0.000001"])
while proc.poll() is None:
    print("작업 중...")
    # 시간이 걸리는 작업을 여기에서 수행!
    ...

print("종료 상태", proc.poll())

# 작업 중...
# ...
# 작업 중...
# 종료 상태 0
```

<br>

### [2] 병렬 실행

자식 프로세스와 부모 프로세스를 분리하면 부모 프로세스가 원하는 개수만큼 많은 자식 프로세스를 병렬로 한꺼번에 실행할 수 있다.

다음의 코드에서 각 프로세스가 순차적으로 실행된다면 지연 시간은 10초 이상일 것이지만, 병렬로 수행하기 때문에 약 1초이다.

> ref: <https://docs.python.org/ko/3/library/subprocess.html#subprocess.Popen.communicate>
> 

```python
import time

start = time.time()
sleep_procs = []

for _ in range(10):
    proc = subprocess.Popen(["sleep", "1"])
    sleep_procs.append(proc)

for proc in sleep_procs:
    proc.communicate()

end = time.time()
delta = end - start
print(f"{delta:.3}초 만에 끝남")
# 1.03초 만에 끝남
```

<br>

### [3] Pipe

파이썬 프로그램의 데이터를 <span class="shl">**파이프(pipe)**</span>를 사용해 **하위 프로세스로 보내거나, 하위 프로세스의 출력을 받을 수 있다**.

다음은 `openssl` 자식 프로세스와 I/O 파이프를 연결하는 예시이다.

```python
import os

def run_encrypt(data):
    env = os.environ.copy()
    env["password"] = "zf7ShyBhZ0raQDdE/FiZpm/m/8f9X+M1"
    proc = subprocess.Popen(
        ["openssl", "enc", "-des3", "-pbkdf2", "-pass", "env:password"],
        env=env,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
    )
    proc.stdin.write(data)
    proc.stdin.flush()  # 자식이 입력을 받도록 보장
    return proc
```

실전에서는 파이프를 통해 사용자 입력, 파일 핸들, 네트워크 소켓 등에서 받은 데이터를 암호화 함수에 보내게 될 것이다.

```python
procs = []
for _ in range(3):
    data = os.urandom(10)
    proc = run_encrypt(data)
    procs.append(proc)

for proc in procs:
    out, _ = proc.communicate()
    print(out[-10:])
    
# b'\x99\x1a\x99N*\xbb\x84\xf3.\x82'
# b'J\x8d1\x99\\\xa7\x18,\x83e'
# b'\xf4\x1ds_\x86A}\xf8\x93\xdc'
```

<br>

### [4] 파이프라인

한 자식 프로세스의 출력을 다음 프로세스의 입력으로 계속 연결시켜서 **여러 병렬 프로세스를 연쇄적으로 연결**할 수도 있다.

다음은 `openssl` 도구를 하위 프로세스로 만들어서, 입력 스트림의 `sha256` 해시를 계산하는 예시이다.

```python
def run_hash(input_stdin):
    return subprocess.Popen(
        ["openssl", "dgst", "-sha256", "-binary"],
        stdin=input_stdin,
        stdout=subprocess.PIPE,
    )
```

```python
encrypt_procs = []
hash_procs = []

for _ in range(3):
    data = os.urandom(100)

    encrypt_proc = run_encrypt(data)
    encrypt_procs.append(encrypt_proc)

    hash_proc = run_hash(encrypt_proc.stdout)
    hash_procs.append(hash_proc)

    # 자식 프로세스가 입력 스트림에 들어오는 데이터를 소비하고 communicate() 메서드가
    # 불필요하게 자식으로부터 오는 입력을 훔쳐가지 못하게 함
    # 또한, 다운스트림 프로세스(= 파이프라인을 읽는 프로세스, hash_proc)가 죽으면
    # SIGPIPE를 업스트림 프로세스(= 파이프라인에 쓰는 프로세스, encrypt_proc)에 전달함
    encrypt_proc.stdout.close()
    # close() 하지 않으면, 다운스트림 프로세스(hash_proc)가 먼저 종료되더라도
    # 파이썬 메인 프로세스가 파일 핸들을 가지고 있으므로 파이프라인이 끊어지지 않아 SIGPIPE가 발생하지 않음!
    encrypt_proc.stdout = None
```

```python
# 자식 프로세스들이 시작되면 프로세스 간 I/O가 자동으로 일어남
for proc in encrypt_procs:
    proc.communicate()
    assert proc.returncode == 0

for proc in hash_procs:
    out, _ = proc.communicate()
    print(out[-10:])

assert proc.returncode == 0

# b'\x16x\xde\xbfZ\xc1r\x99,\x81'
# b'\x8a\x8f\x0e\x82H\xf4}Y\x02\xc7'
# b'\xc0\xe1\xee\x0b\xaf\xdb\x80FP8'
```

<br>

### [5] Timeout

자식 프로세스가 영원히 끝나지 않는 경우, 입력 혹은 출력 파이프를 기다리며 <span class="shl">**block 되는 상황**이나 **교착 상태(deadlock)**</span>가 우려된다면 **`timeout` 파라미터를 `communicate` 메서드에 전달**할 수 있다.

`timeout` 값을 전달하면 **자식 프로세스가 주어진 시간 동안에 끝나지 않을 경우에 예외가 발생**하므로, 다음과 같이 해당 프로세스를 종료할 수 있다.

> ref: <https://docs.python.org/ko/3/library/subprocess.html#subprocess.Popen.returncode>
> 

```python
proc = subprocess.Popen(["sleep", "10"])
try:
    proc.communicate(timeout=0.1)
except subprocess.TimeoutExpired:
    proc.terminate()
    proc.wait()

print("종료 상태", proc.poll())
# 종료 상태 -15
```
