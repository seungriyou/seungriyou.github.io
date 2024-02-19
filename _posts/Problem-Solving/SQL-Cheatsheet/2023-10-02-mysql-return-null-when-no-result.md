---
title: "[MySQL] SELECT 결과가 비어있을 때 NULL을 반환하는 방법"
date: 2023-10-02 21:50:00 +0900
categories: [Problem Solving, SQL Cheatsheet]
tags: [sql, mysql, tip, subquery]
math: true
---

일반 SELECT를 사용하면 결과가 없는 경우는 아예 출력되지 않는다.

**결과 값이 없는 경우에도 빈 테이블이 아닌 `NULL`을 반환**하고 싶다면, 다음과 같이 `SELECT` 절 안에 subquery를 넣는 <span class="hl">**중첩 SELECT 문**</span>을 사용해야 한다.

<br>

> 관련 문제: <https://leetcode.com/problems/biggest-single-number/>

```sql
SELECT (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
    ORDER BY num DESC LIMIT 1
) AS num;
```

<br>

> 관련 문제: <https://leetcode.com/problems/second-highest-salary/>

```sql
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1, 1
) AS SecondHighestSalary;
```
