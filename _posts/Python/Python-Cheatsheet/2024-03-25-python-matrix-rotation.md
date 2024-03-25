---
title: "[Python] 간편하게 행렬 뒤집기 & 회전하기 (+ zip(), [::-1])"
date: 2024-03-25 10:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, matrix, zip, slicing]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

주어진 `matrix`의 row가 다음과 같이 구성되어 있다고 가정한다.

```python
[1, 2, 3, 4]
[5, 6, 7, 8]
[9, 10, 11, 12]
```

<br>

## 1. 뒤집기
    
```python
transpose = list(zip(*matrix))
```

```python
(1, 5, 9)
(2, 6, 10)
(3, 7, 11)
(4, 8, 12)
```

<br>

## 2. 시계방향으로 90도 회전하기
    
```python
clockwise = list(zip(*matrix[::-1]))
```

```python
(9, 5, 1)
(10, 6, 2)
(11, 7, 3)
(12, 8, 4)
```

<br>
    
## 3. 반시계방향으로 90도 회전하기
    
```python
counter_clockwise = list(zip(*matrix))[::-1]
```

```python
(4, 8, 12)
(3, 7, 11)
(2, 6, 10)
(1, 5, 9)
```
