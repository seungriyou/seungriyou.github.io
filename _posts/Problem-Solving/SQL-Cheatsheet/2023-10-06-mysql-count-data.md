---
title: "[MySQL] 테이블 내의 데이터 개수 구하기 (+ DISTINCT)"
date: 2023-10-06 19:50:00 +0900
categories: [Problem Solving, SQL Cheatsheet]
tags: [sql, mysql, tip, count]
math: true
---

`SELECT COUNT(*) FROM Product` 와 같이 **`COUNT()`를 SELECT** 하면 된다.

`SELECT COUNT(DISTINCT player_id) FROM Activity` 와 같이 `DISTINCT`를 `COUNT()` 내에 사용할 수도 있다.

> `COUNT()` 내에 `DISTINCT`를 써야하는 때에 주의해야 한다!!
{: .prompt-danger}

<br>

> 관련 문제: <https://leetcode.com/problems/customers-who-bought-all-products/>

```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

<br>

> 관련 문제: <https://leetcode.com/problems/game-play-analysis-iv/>

```sql
SELECT ROUND(COUNT(player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) AS fraction
FROM Activity
WHERE (player_id, event_date) IN (
    SELECT player_id, DATE_ADD(MIN(event_date), INTERVAL 1 DAY)
    FROM Activity
    GROUP BY player_id
);
```
