---
title: "[Python] asyncio 파헤치기 - ④ 동시성을 지원하는 애플리케이션 구현하기"
date: 2024-07-28 19:50:00 +0900
categories: [Python, Concurrency in Python]
tags: [python, asyncio]
math: true
---
> 본문은 “Python Concurrency with Asyncio”를 읽고 재구성한 글임을 밝힙니다.
> 

<br>

본 포스팅에서는 `sleep` 함수로 long operation을 simulate 하는 것에 그치지 않고, 실제로 **single-threaded socket-based echo server 애플리케이션**을 구현하면서 **multiple users**를 concurrent 하게 다루는 방법을 알아본다.

본문은 다음의 단계로 진행된다.

1. <span class="blue">blocking</span> socket의 원리 & 이를 이용한 multi-client echo server 구현
2. single-thread에서 blocking socket을 이용하여 multiple users를 동시에 처리할 때의 문제점
3. <span class="blue">non-blocking</span> socket으로 변경하기 & 문제점
4. 운영체제의 <span class="blue">event notification system</span>을 이용한 `selectors` 모듈로 변경하기
5. `asyncio` <span class="blue">event loop</span>로 변경하기

<br>

## 1. Blocking Socket 기반 Echo Server 구현하기

### 1-1. Blocking Socket

**socket**이란 <span class="shl">네트워크를 통해 데이터를 읽고 쓰는 방법</span>으로, 우편함으로 생각하면 쉽고 기본적으로 blocking 이다.

echo server를 구현하기 위해서는 우선 메인 우편함 소켓인 **server socket**을 생성한다. 이 socket은 통신을 원하는 clients로부터 connection message를 받아들인다. 이러한 server socket을 통해 connection이 인식되면, client와 통신하기 위한 **socket**을 생성한다.

> 즉, 우리의 서버는 여러 PO boxes를 가진 우체국과 같은 역할을 한다.
> 

![client-server](/assets/img/posts/Python/More-About-Python/2024-07-28-01.png)

```python
import socket

# Create a TCP server socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# Set the address of the socket to 127.0.0.1:8000
server_address = ("127.0.0.1", 8000)
# Listen for connections or open the "post office"
server_socket.bind(server_address)
server_socket.listen()

# Wait for a connection and assign the client a PO box
connection, client_address = server_socket.accept()
print(f"I got a connection from {client_address}!")
```


> 여기에서 **`accept()` 메서드**는 다음과 같은 역할을 한다.
>
> - connection을 받기 전까지 **block** 한다.
> - connection을 받으면 해당 **connection**과 연결한 **client address**를 반환한다.
> - 이 connection 또한 **또 다른 socket**이다.
{: .prompt-info}

<br>

### 1-2. Telnet

위에서 만든 socket application에 연결하기 위한 도구들 중 telnet command-line application을 살펴보자.

telnet이란 teletype network의 줄임말로, 이를 통해 서버와 TCP 연결을 맺은 후 터미널을 통해 데이터를 주고 받을 수 있다.

터미널 두 개를 열고 하나에서는 socket application을, 다른 하나에서는 telnet command-line application을 실행하면 다음과 같은 결과를 확인할 수 있다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-02.png)

> server(socket application) ↔ client 간 데이터를 주고 받는 기능을 추가하려면, client socket의 **`sendall` & `recv` 메서드**를 이용한다.
{: .prompt-tip}

<br>

### 1-3. `recv` 메서드: 연결로부터 데이터 읽기

일반적으로 모든 데이터를 한번에 socket을 통해 읽어들일 수 없기 때문에, 입력의 끝에 도달할 때까지 데이터를 buffer에 저장해두어야 한다.

buffer의 동작을 확인하기 위해 buffer size를 작게 해두고 실습해보자.

telnet에서 사용자가 <kbd>Enter</kbd>를 누르면 carriage return + line feed (`\r\n`)이 입력되므로, 다음의 코드와 같이 루프를 돌며 입력의 끝에 도달할 때까지 데이터를 `connection.recv(2)`로 2byte씩 읽어들인다.

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ("127.0.0.1", 8000)
server_socket.bind(server_address)
server_socket.listen()

try:
    # Wait for a connection and assign the client a PO box
    connection, client_address = server_socket.accept()
    print(f"I got a connection from {client_address}!")

    buffer = b""

    # Check each iteration to see if our buffer ends in a carriage return and a line feed
    while buffer[-2:] != b"\r\n":
        # Try to receive two bytes
        data = connection.recv(2)
        if not data:
            break
        else:
            print(f"I got data: {data}!")
            # Store it in our buffer
            buffer = buffer + data

    print(f"All the data is: {buffer}")
finally:
    # Close the server socket, ensuring that we close the connection even if an exception occurs
    server_socket.close()
```

<br>

telnet 터미널에서 `testing123`을 입력하면, server application에서는 다음과 같이 2byte 씩 읽어들여 `buffer`에 저장하는 것을 알 수 있다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-03.png)

<br>

### 1-4. `sendall` 메서드: 클라이언트에게 데이터 쓰기

다음과 같이 `buffer`에 저장된 데이터를 `sendall()` 메서드로 클라이언트에게 전송하면, telnet 터미널에서 `buffer`의 내용이 출력되는 것을 확인할 수 있다. 이렇게 간단한 echo server를 socket으로 작성할 수 있는 것이다!

```python
try:
    ...
    print(f"All the data is: {buffer}")
    # Write data back to the client
    connection.sendall(buffer)
finally:
    ...
```

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-04.png)

<br>

## 2. Blocking Socket의 문제점

지금까지 작성한 애플리케이션의 server socket은 한 순간에 하나의 client만 handle 할 수 있다. 이는 server socket이 <span class="red">**blocking socket**</span>이기 때문인데, 그렇다면 현재 코드에서 다수의 client가 연결을 시도하려 한다면 무슨 일이 발생할까?

<br>

### 2-1. 코드 살펴보기

socket의 listen 모드는 여러 client의 동시 연결을 허용한다. 즉, `socket.accept()` 메서드를 반복적으로 호출한다면, 각 client가 연결을 시도할 때마다 새로운 connection socket을 얻을 수 있다. 따라서 무한 루프를 돌며 `socket.accept()`를 통해 connection을 열고, **열린 모든 connection을 순회**하며 데이터를 읽고 쓰는 코드를 작성할 수 있다.

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ("127.0.0.1", 8000)
server_socket.bind(server_address)
server_socket.listen()

connections = []

try:
    while True:
        connection, client_address = server_socket.accept()
        print(f"I got a connection from {client_address}!")
        connections.append(connection)

        for connection in connections:
            buffer = b""

            while buffer[-2:] != b"\r\n":
                data = connection.recv(2)
                if not data:
                    break
                else:
                    print(f"I got data: {data}!")
                    buffer = buffer + data

            print(f"All the data is: {buffer}")
            connection.send(buffer)
finally:
    server_socket.close()
```

<br>

### 2-2. 문제점

하지만 위의 코드에서는 첫 번째 client는 제대로 동작하여 echo 동작을 잘 수행하지만, **두 번째 client는 아무 것도 반환하지 않는다**는 문제가 발생한다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-05.png)

<br>

이는 socket의 **`accept` 메서드와 `recv` 메서드**가 <span class="red">데이터를 받기 전까지 영원히 **block**</span> 하기 때문이다.

> 첫 번째 client가 연결한 후 데이터를 보낼 때까지 `connection.recv(2)`에서 blocked 되므로, 두 번째 client는 다음 iteration에서 `socket.accept()`를 하기 위해서 (첫 번째 client가 데이터를 보낼 때까지) 꼼짝없이 기다리게 되는 것이다.
> 

![two clients](/assets/img/posts/Python/More-About-Python/2024-07-28-06.png)

<br>

실제로 첫 번째 client가 데이터를 다시 보내면 해당 데이터가 먼저 echo 되고, 그 후 두 번째 client가 보낸 데이터 또한 echo 되는 것을 확인할 수 있다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-07.png)

<br>

## 3. Non-Blocking Socket으로 Echo Server 구현하기

위에서 살펴본 문제점을 해결하기 위해서는 **socket을 <span class="red">non-blocking</span> mode로** 사용해야 한다.

<br>

### 3-1. 코드 살펴보기

non-blocking socket을 사용하기 위해서 다음과 같이 코드를 변경한다.

1. `server_socket`과 `connection`에서 `setblocking(False)`를 호출한다.
2. 아직 데이터를 받지 않을 때 발생하는 `BlockingIOError`를 catch 하여 무시한다.

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ("127.0.0.1", 8000)
server_socket.bind(server_address)
server_socket.listen()
# Mark the server socket as non-blocking
server_socket.setblocking(False)

connections = []

try:
    while True:
        # Catch the exception and ignore it and keep looping until we have data
        try:
            connection, client_address = server_socket.accept()
            # Mark the client socket as non-blocking
            connection.setblocking(False)
            print(f"I got a connection from {client_address}!")
            connections.append(connection)
        except BlockingIOError:
            pass

        for connection in connections:
            # Catch the exception and ignore it and keep looping until we have data
            try:
                buffer = b""

                while buffer[-2:] != b"\r\n":
                    data = connection.recv(2)
                    if not data:
                        break
                    else:
                        print(f"I got data: {data}!")
                        buffer = buffer + data

                print(f"All the data is: {buffer}")
                connection.send(buffer)
            except BlockingIOError:
                pass
finally:
    server_socket.close()
```

<br>

위와 같이 코드를 변경하면, 기존에 발생하던 **blocking 문제 상황은 해결**되는 것처럼 보인다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-08.png)

<br>

### 3-2. 문제점

하지만 이러한 코드에도 다음과 같은 측면에서 **비용이 발생한다**는 문제점이 존재한다.

1. <span class="shl">**코드 품질**</span>: 데이터를 아직 받지 않은 상황에서 발생할 exception을 처리하는 로직을 추가함으로써 verbose & potentially error-prone 해진다.
2. <span class="shl">**자원 활용**</span>: 계속해서 루프를 돌며 exception을 검사하므로 CPU 사용률이 100%에 근접하게 치솟는다.
    
    ![cpu utilization](/assets/img/posts/Python/More-About-Python/2024-07-28-09.png){: .w-75}
    

> 이렇게 애플리케이션 레벨에서 socket을 사용하는 것과 다르게, 다음 섹션에서 살펴볼 **event notification system**에서는 socket이 데이터를 받게 되면 **운영체제**가 사용자 애플리케이션에 알려준다. 하지만 이는 **하드웨어 레벨**에서 일어나는 동작이므로, 위의 코드에서처럼 `while` loop에서 polling 하는 것이 아니다. 파이썬에는 이미 이러한 event notification system이 `selectors` 모듈에 내장되어 있으며, 이를 통해 socket event를 위한 **event loop**를 구현하면 **CPU 사용률 이슈를 해결**할 수 있다.
{: .prompt-info}

<br>

## 4. `selectors` 모듈을 이용하여 Socket Event Loop 만들기

운영체제는 socket이 데이터 혹은 이벤트를 받는 것에 대한 내장 API를 가지고 있으며, 이러한 **I/O notification system**은 운영체제 종류마다 구현에는 차이가 있으나 원리는 유사하다.

> 이벤트를 monitor 하는 socket들의 리스트를 넘기면 (각 socket을 계속해서 직접 확인할 필요 없이) **운영체제가 socket에 데이터가 들어왔을 때** 알려준다.
{: .prompt-tip}

이러한 동작은 하드웨어 레벨에서 구현되어 있어 monitoring 시 **CPU를 적게 사용**하므로 자원 효율성에 좋으며, 이는 **`asyncio`에서 concurrency를 달성하는 방법**의 핵심이기도 하다.

운영체제마다 구체적인 구현은 다르지만, 추상화가 되어 있는 **파이썬의 `selectors` 모듈**을 사용하면 이러한 API를 직접 사용할 수 있다.

<br>

### 4-1. 파이썬의 `selectors` 모듈

`selectors` 라이브러리에서 제공하는 주요 클래스에는 다음의 것들이 있다.

- **`BaseSelector`**: 각 event notification system으로 구현되는 abstract base class
- **`DefaultSelector`**: 자동으로 현재 시스템에 가장 효율적인 구현을 고르는 클래스

<br>

이 중에서, **`BaseSelector`의 주요 컨셉**에는 다음과 같은 것들이 있다.

> ref: <https://docs.python.org/ko/3/library/selectors.html#selectors.BaseSelector>
 
1. <span class="shl">**register**</span>: notification을 받고 싶은 socket을 등록함으로써 어떤 event(ex. read, write)에 관심이 있는지 지정한다. 반대로 더이상 관심이 없는 socket을 unregister 할 수도 있다.
2. <span class="shl">**select**</span>: `select`는 어떤 event가 발생할 때까지 **block** 하며, event가 발생하면 이와 **함께 처리될 준비가 된 socket들의 리스트**를 반환한다. 또한, **timeout**을 지정함으로써 감시하고 있는 socket이 준비되지 않더라도 해당 시간 이후에는 block 하지 않도록 할 수 있다.

<br>

이러한 컨셉을 사용해서 **<span class="red">CPU 사용률</span>에 악영향을 미치지 않는 non-blocking echo server**를 만들 수 있다. 설계 핵심은 다음과 같다.

- client로부터의 connection에 대해 listen 하는 server socket을 **default selector**에 등록한다.
- 누군가가 server socket에 연결하면, 그 client의 connection socket을 데이터를 감시하기 위한 **selector**에 등록한다.
- server socket이 아닌 그 외의 socket으로부터 데이터를 받게 되면 client가 data를 보낸 것을 알 수 있으므로, 해당 데이터를 받아 client에게 write back 해준다.
- **timeout**을 추가함으로써 대기하는 동안 다른 코드를 실행할 수 있도록 한다.

<br>

### 4-2. 코드 살펴보기

다음과 같이 `selectors` 모듈을 사용해서 구현할 수 있다.

```python
import selectors
import socket
from selectors import SelectorKey

# Create a selector
selector = selectors.DefaultSelector()

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ("127.0.0.1", 8000)
# Mark the server socket as non-blocking
server_socket.setblocking(False)
server_socket.bind(server_address)
server_socket.listen()

# Register the default selector
selector.register(server_socket, selectors.EVENT_READ)

while True:
    # Create a selector that will timeout after 1 second
    events: list[tuple[SelectorKey, int]] = selector.select(timeout=1)

    # If there are no events, print it out
    if len(events) == 0:
        # This happens what a timeout occurs
        print("No events, waiting a bit more!")

    for event, _ in events:
        # Get the socket for the event, which is stored in the fileobj field
        event_socket = event.fileobj

        # If the event socket is the same as the server socket,
        # we know this is a connection attempt
        if event_socket == server_socket:
            connection, address = server_socket.accept()
            connection.setblocking(False)
            print(f"I got a connection from {address}")
            # Register the client that connected with our selector
            selector.register(connection, selectors.EVENT_READ)
        # If the event socket is not the server socket,
        # receive data from the client, and echo it back
        else:
            data = event_socket.recv(1024)
            print(f"I got some data: {data}")
            event_socket.send(data)
```

<br>

실행 결과는 다음과 같다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-10.png)

- connection event를 받기 전까지 `No events, waiting a bit more!`이라는 문구가 매 초마다 출력된다.
- connection을 가지게 되면 read event를 listen 하도록 register 한다.
- client가 데이터를 보내면, selector는 데이터가 준비된 event를 반환하므로 이를 `socket.recv`를 통해 읽을 수 있다.

<br>

이렇게 작성한 echo server 애플리케이션은 다음과 같은 **장점**이 있다.

1. 사용할 수 있는 데이터가 있을 때만 데이터를 read/write 하므로 <span class="shl">**blocking 문제가 발생하지 않는다**</span>.
2. 운영체제의 event notification system을 사용하므로 <span class="shl">**CPU를 아주 적게 사용한다**</span>.
    
    ![cpu utilization](/assets/img/posts/Python/More-About-Python/2024-07-28-11.png){: .w-75}
    

<br>

### 4-3. `asyncio`의 Event Loop과 비교하기

위에서 `selectors` 모듈로 구현한 것은 `asyncio`의 event loop 동작과 유사하다.

- **event**는 데이터를 받는 **socket**이다.
- 각 **iteration**은 발생하는 **socket event 혹은 timeout**에 의해 수행된다.

<br>

`asyncio` event loop의 경우, **<span class="red">socket event</span> 혹은 <span class="red">timeout</span>**이 발생하면 **실행 대기중인 coroutine이 실행**된다. 그리고 이 coroutine은 <span class="shlb">(1) 자기자신이 완료될 때까지</span>, 혹은 <span class="shlb">(2) 다음 `await` 문을 만날 때까지</span> 진행된다.

이때, **non-blocking socket을 사용하는 coroutine** 내에서 `await` 문을 만나면 <span class="shlb">(1) 해당 socket을 시스템의 selector에 등록</span>하고 <span class="shlb">(2) 그 coroutine이 결과를 기다리기 위해 중지되었다는 것을 추적</span>한다.

<br>

다음은 이러한 동작을 구현한 pseudocode이다.

```python
paused = []
ready = []

while True:
    paused, new_sockets = run_ready_tasks(ready)
    selector.register(new_sockets)
    timeout = calculate_timeout()
    events = selector.select(timeout)
    ready = process_events(events)
```

1. 실행할 준비가 된 coroutines을 실행하다가 **`await` 문에서 중지**되었으면 `paused`에 저장한다.
2. 해당 coroutines를 실행하지 못하도록 감시해야 할 새로운 sockets를 추적하고, 이를 selector에 등록한다.
3. `select`에 필요한 적합한 timeout 값을 계산한다.
4. `select`를 호출한 후, **socket event 혹은 timeout**을 기다린다. 그 중 하나가 발생한다면 해당 event를 처리하여 **실행 가능한 coroutine의 리스트 `ready`**로 바꾼다.


> **`selectors`를 사용하는 주요 컨셉:**
>
> 감시할 socket을 등록하고, 처리하길 원하는 event가 발생했을 때에만 해당하는 socket이 깨어나도록 한다.
{: .prompt-danger}

<br>

이어서 이러한 동작을 `asyncio`의 event loop를 이용해서 구현해보자.

<br>

## 5. `asyncio` Event Loop로 구현하기

`select`를 사용해서 직접 event loop를 구현하는 것은 너무 low-level 하다. 따라서 `asyncio`에 이미 구현된 event loop를 사용할 수 있다. 또한, `asyncio`의 coroutine과 task는 `selectors`에 대한 추상화를 제공하기 때문에 코드를 더 쉽게 구현/유지보수 할 수 있도록 돕는다.

<br>

### 5-1. Socket을 다루기 위한 Coroutines

`asyncio`에서 socket을 다루기 위해 제공하는 **세 가지 주요 coroutine**을 살펴보자.

> 이들은 이전에 사용했던 socket methods들과 유사하지만, <span class="shl">(1) socket을 인자로 받고</span> <span class="shl">(2) 활용할 수 있는 데이터를 받을 때까지 `await` 할 수 있는 coroutines를 반환</span>한다는 차이점이 있다.
{: .prompt-info}

1. **`sock_accept(sock)`**: `socket.accept()`의 async 버전이다.
    
    > doc: <https://docs.python.org/ko/3/library/asyncio-eventloop.html#asyncio.loop.sock_accept>
    
    - `(socket_connection, client_address)`로 이루어진 tuple을 반환한다.
    - 감시하려는 socket에 전달하면 반환되는 coroutine을 `await` 할 수 있고, 해당 coroutine이 완료되면 connection과 address를 얻을 수 있다.
    - 전달하는 socket `sock`은 **non-blocking**이어야 하며, 이미 port에 바인딩 되어 있어야 한다.
    
    ```python
connection, address = await loop.sock_accept(socket)
    ```
    
2. **`sock_recv(sock, nbytes)`**: `socket.recv()`의 async 버전이다.
    
    > doc: <https://docs.python.org/ko/3/library/asyncio-eventloop.html#asyncio.loop.sock_recv>
    
    - socket이 처리할 수 있는 bytes를 가질 때까지 `await` 한다.
      
    ```python
data = await loop.sock_recv(socket)
    ```
    
3. **`sock_sendall(sock, data)`**: `socket.sendall()`의 async 버전이다.
    
    > doc: <https://docs.python.org/ko/3/library/asyncio-eventloop.html#asyncio.loop.sock_sendall>
    
    - socket으로 전송하려는 모든 데이터가 전송될 때까지 대기하고, 성공 시 `None`을 반환한다.
      
    ```python
success = await loop.sock_sendall(socket, data)
    ```
    
<br>

### 5-2. Coroutine이 적합한 상황과 Task가 적합한 상황

> 어느 상황에 그냥 **coroutine**을 사용할 것인지, 혹은 coroutine을 **task**로 감싸서 사용할 것인지 결정하려면 <span class="shl">**해당 동작이 동시에 처리될 필요가 있는지**</span>를 살펴보아야 한다.
{: .prompt-tip}

- **`listen_for_connections`**: 무한히 loop를 돌며 incoming connection을 listen 하는 coroutine
    
    > connections에 대해 listen 할 때, <span class="shlb">여러 connections를 동시에 처리할 필요가 없으므로</span> **하나의 <span class="red">coroutine</span>**이 무한히 반복되도록 하면 된다. 이를 통해 connection을 기다리면서 pause 되어 있는 동안 다른 코드를 실행할 수 있도록 허용할 수 있다.
    > 
    
  ```python
  async def listen_for_connections(server_socket: socket, loop: AbstractEventLoop):
      while True:
          connection, address = await loop.sock_accept(server_socket)
          connection.setblocking(False)
          print(f"Got a connection from {address}")
  ```
    
- **`echo`**: 무한히 loop를 돌며 connection으로부터 데이터를 read 후 write 하는 coroutine
    
    > multiple connections을 가질 것이며, 각각은 언제든 데이터를 보낼 수 있어야 한다. 즉, <span class="shlb">하나의 connection이 다른 하나를 block 하면 안 되기 때문</span>에 multiple clients가 있는 경우에 concurrent 하게 동작할 수 있어야 한다.
    > 
    > 
    > 따라서 `listen_for_connections` coroutine에서 각 connection에 대해 이 **`echo` coroutine을 wrap 한 <span class="red">task</span>**를 생성하여 데이터를 read/write 하도록 하는 것이 적합하다.
    > 
    
  ```python
  async def echo(connection: socket, loop: AbstractEventLoop) -> None:
      # Loop forever waiting for data from a client connection
      while data := await loop.sock_recv(connection, 1024):
          # Once we have data, send it back to that client
          await loop.sock_sendall(connection, data)
          
  
  async def listen_for_connections(server_socket: socket, loop: AbstractEventLoop) -> None:
      while True:
          ...
          # Whenever we get a connection, create an echo task to listen for client data
          asyncio.create_task(echo(connection, loop))
  ```
    
<br>

### 5-3. 코드 살펴보기

위에서 작성한 coroutine들을 가지고 echo server 애플리케이션을 작성하면 다음과 같다.

> client가 연결하면, incoming connection을 listen 하는 **`listen_for_connection` coroutine**이 각 client 별로 데이터를 read/write 하기 위한 **`echo` task**를 생성한다.
>
> ![coroutine and task](/assets/img/posts/Python/More-About-Python/2024-07-28-12.png)
{: .prompt-info}

```python
import asyncio
import socket
from asyncio import AbstractEventLoop

async def echo(connection: socket, loop: AbstractEventLoop) -> None:
    # Loop forever waiting for data from a client connection
    while data := await loop.sock_recv(connection, 1024):
        # Once we have data, send it back to that client
        await loop.sock_sendall(connection, data)

async def listen_for_connections(
    server_socket: socket, loop: AbstractEventLoop
) -> None:
    while True:
        connection, address = await loop.sock_accept(server_socket)
        connection.setblocking(False)
        print(f"Got a connection from {address}")
        # Whenever we get a connection, create an echo task to listen for client data
        asyncio.create_task(echo(connection, loop))

async def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    server_address = ("127.0.0.1", 8000)
    server_socket.setblocking(False)
    server_socket.bind(server_address)
    server_socket.listen()

    # Start the coroutine to listen for connections
    await listen_for_connections(server_socket, asyncio.get_event_loop())

asyncio.run(main())
```

![termianl](/assets/img/posts/Python/More-About-Python/2024-07-28-13.png)

<br>

이렇게 작성한 코드 또한 **내부적으로 `selectors`를 사용**하기 때문에, <span class="shl">(1) multiple client와 concurrent 하게 동작</span>할 수 있으며 <span class="shl">(2) CPU utilization이 낮다</span>는 **장점**이 있다.

하지만, `echo` task가 실패하는 경우에 대비하여 error handling을 추가해줘야 한다.

<br>

### 5-4. Task 관련 Error Handling

`echo` task 내부에서 exception이 발생하는 상황을 가정하기 위해, `boom`이라는 단어를 받아들이면 exception을 raise 하는 코드를 추가하자.

```python
async def echo(connection: socket, loop: AbstractEventLoop) -> None:
    while data := await loop.sock_recv(connection, 1024):
        if data == b"boom\r\n":
            raise Exception("Unexpected network error")
        await loop.sock_sendall(connection, data)
```

그리고 하나의 터미널에서 `boom`을 보내 exception을 raise 하면, echo server 애플리케이션이 실행 중인 터미널에서 다음과 같은 traceback을 확인할 수 있다.

```
Task exception was never retrieved
future: <Task finished name='Task-2' coro=<echo() done, defined at .../app.py:6> exception=Exception('Unexpected network error')>
Traceback (most recent call last):
  File ".../app.py", line 10, in echo
    raise Exception("Unexpected network error")
Exception: Unexpected network error
```

여기에서 핵심은 **`Task exception was never retrieved`**이다. 이는 <span class="shl">task 내부에서 exception이 thrown 되면, 해당 task는 exception이라는 결과로 완료되었다고 간주되기 때문</span>이다. 따라서 exception은 call stack에 thrown up 되지 않으며, cleanup도 불가능하다.

<br>

그렇다면 어떻게 해야 task exception이 retrieved 될 수 있을까? task 내부에서 발생한 exception이 retrieved 되어 우리에게 도달할 수 있으려면 **task를 `await` 식에서 사용**해야 한다. 실패하는 task를 `await` 하면, <span class="shlb">(1) exception이 `await` 문을 실행 중인 곳으로 thrown</span> 되며 <span class="shlb">(2) traceback 또한 이를 반영</span>하게 된다.

> task를 `await` 하지 않는다면, 해당 task가 raise 하는 exception을 볼 수 없다.
{: .prompt-danger}

<br>

task를 `await` 하지 않아 exception이 보이지 않는 경우를 연출하면 다음과 같다.

```python
tasks = []

async def listen_for_connections(
    server_socket: socket, loop: AbstractEventLoop
) -> None:
    while True:
        ...
        tasks.append(asyncio.create_task(echo(connection, loop)))
```

이 경우에는 애플리케이션을 강제로 종료하기 전까지는 exception에 대해서 아무것도 출력되지 않게 된다. 그 이유는 `asyncio`는 **실패한 task에 대한 message와 traceback**을 <span class="shl">**오직 그 task가 garbage collected 되었을 때에만 출력**</span>할 수 있기 때문이다. 위 코드에서는 `tasks` 배열에 task에 대한 reference를 저장했기 때문에 garbage collected 되지 않아 exception에 대한 정보를 얻을 수 없었던 것이다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-14.png)

<br>

따라서 task에서 발생한 exception을 제대로 처리하려면 <span class="ulr">**task에 대해 `await`**</span>를 하거나 <span class="ulr">**task에서 발생 가능한 모든 exception을 handling**</span> 해야 한다. 여기에서는 다음과 같이 task 내부에서 `try` / `catch`를 통해 exception을 handling 하여 로그에 남기고, `finally` block에서 connection을 닫도록 코드를 변경해보자.

```python
async def echo(connection: socket, loop: AbstractEventLoop) -> None:
    try:
        while data := await loop.sock_recv(connection, 1024):
            if data == b"boom\r\n":
                raise Exception("Unexpected network error")
            await loop.sock_sendall(connection, data)
    # Retrieve exception from a task
    except Exception as ex:
        logging.exception(ex)
    # Shut down the socket properly
    finally:
        connection.close()
```

<br>

실행 결과, 다음과 같이 exception이 잘 retrieved 되어 출력되는 것을 확인할 수 있다.

![terminal](/assets/img/posts/Python/More-About-Python/2024-07-28-15.png)

<br>

## References

- “Python Concurrency with Asyncio”, Ch03. A First asyncio Application
