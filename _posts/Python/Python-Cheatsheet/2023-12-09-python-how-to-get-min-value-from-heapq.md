---
title: "[Python] heapq 우선순위 큐에서 최솟값 추출하기"
date: 2023-12-09 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, heapq]
math: true
---

`heapq` 우선순위 큐, 즉 최소힙에서 최솟값을 삭제하지 않고 얻으려면 <span class="shl">`[0]` 인덱스</span>를 통해 접근하면 된다.

왜냐하면 최소힙을 구성하는 과정(`heapify`)에서 매번 새로운 최솟값을 `[0]` 인덱스에 위치시키기 때문이다.

<br>

하지만 <span class="shl">`[1]` 인덱스에 두 번째로 작은 원소, `[2]` 인덱스에 세 번째로 작은 원소가 들어있는 것은 아니다</span>.

두 번째로 작은 원소를 얻으려면 `heappop()`을 한 번 수행해서 최솟값을 삭제한 후 `[0]` 인덱스에 접근해야 한다.
