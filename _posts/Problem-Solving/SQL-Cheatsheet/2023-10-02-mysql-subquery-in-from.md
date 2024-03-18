---
title: "[MySQL] FROM 절에 중첩 SELECT 문 사용하는 예제"
date: 2023-10-02 21:50:00 +0900
categories: [Problem Solving, SQL Cheatsheet]
tags: [sql, mysql, tip, subquery]
math: true
---

> 관련 문제: <https://leetcode.com/problems/delete-duplicate-emails/>

<br>

GROUP BY 후 함수를 적용한 결과를 WHERE 조건으로 사용하고 싶다면 **중첩 SELECT 문**을 작성한다.

```sql
DELETE FROM Person
WHERE id NOT IN (
    SELECT min_id
    FROM (
        SELECT MIN(id) AS min_id
        FROM Person
        GROUP BY email
    ) M
);
```
