---
title: "[MySQL] 두 테이블의 Cartesian Product를 구하는 세 가지 방법"
date: 2023-10-05 19:50:00 +0900
categories: [Problem Solving, SQL Cheatsheet]
tags: [sql, mysql, tip, join]
math: true
---

> 관련 문제: <https://leetcode.com/problems/students-and-examinations/>

<br>


두 table의 **데이터 조합에 대해 모든 경우의 수**를 가지는 Cartesian Product를 구하는 방법에 대해 알아보자.

<br>

## Tables

### Students

```sql
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
```

### Subjects

```sql
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
```

<br>

## Three Ways to Cartesian Product

### [1] `CROSS JOIN` 사용하기
    
```sql
SELECT *
FROM Students
CROSS JOIN Subjects
```

<br>
    

### [2] `(INNER) JOIN` 사용하기
    
```sql
SELECT *
FROM Students
INNER JOIN Subjects ON 1=1   -- 1=1은 항상 True임을 나타내어 아무것도 제거하지 않는다는 의미
```
   
<br>
 
### [3] `FROM` 절에 나열하기

```sql
SELECT *
FROM Students, Subjects
```

<br>

> 2, 3번 방법의 경우, `ON` 또는 `WHERE` 조건을 설정하면 INNER JOIN으로 동작한다.
{: .prompt-tip}

<br>

## Result

```sql
| student_id | student_name | subject_name |
| ---------- | ------------ | ------------ |
| 1          | Alice        | Programming  |
| 1          | Alice        | Physics      |
| 1          | Alice        | Math         |
| 2          | Bob          | Programming  |
| 2          | Bob          | Physics      |
| 2          | Bob          | Math         |
| 13         | John         | Programming  |
| 13         | John         | Physics      |
| 13         | John         | Math         |
| 6          | Alex         | Programming  |
| 6          | Alex         | Physics      |
| 6          | Alex         | Math         |
```
