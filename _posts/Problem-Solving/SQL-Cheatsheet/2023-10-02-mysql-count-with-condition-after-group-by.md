---
title: "[MySQL] GROUP BY 후 조건에 부합하는 값들의 개수 및 비율 구하기"
date: 2023-10-02 19:50:00 +0900
categories: [Problem Solving, SQL Cheatsheet]
tags: [sql, mysql, tip, group by]
math: true
---

> 관련 문제: <https://leetcode.com/problems/queries-quality-and-percentage/>

<br>

다음과 같이 `query_name` 컬럼을 기준으로 `GROUP BY` 후, `rating < 3`이라는 특정 조건에 부합하는 값들의 비율을 `poor_query_percentage` 라는 컬럼으로 반환하려 한다.

```sql
SELECT query_name
    , ROUND(AVG(rating / position), 2) AS quality
    , <?> AS poor_query_percentage    -- <?> 부분을 채워보자!
FROM Queries
GROUP BY query_name
```

<br>

## 조건에 부합하는 값들의 개수 세기

우선, `GROUP BY` 후 `rating < 3` 이라는 조건에 부합하는 값들을 세는 방법에 대해 알아보자.

대표적으로 다음의 세 가지의 방법이 있다.

1. **`CASE`**-`WHEN`-`THEN`-(`ELSE`)-`END` 절과 `COUNT()` 사용하기

    ```sql
    COUNT(CASE WHEN rating < 3 THEN 1 END)
    ```

2. **`IF()`**와 `SUM()` 사용하기

    ```sql
    SUM(IF(rating < 3, 1, 0))
    ```

3. **`SUM(조건절)`** 사용하기 (→ ✅)

    > 조건절 내의 <span class="red">**비교 연산자 자체**</span>가 `0`, `1` 값을 반환하므로, `COUNT()` 함수가 필요하지 않다!

    ```sql
    SUM(rating < 3)
    ```

<br>

## 조건에 부합하는 값들의 비율 구하기

조건에 부합하는 값들의 비율을 구하기 위해서는 위에서 다룬 방법들로 값들의 개수를 세고 전체 데이터 개수(`COUNT(*)`)로 나누는 방법도 있으나, **`AVG(조건절)`**을 사용하면 전체 데이터 개수로 나누지 않아도 된다.

따라서 조건에 부합하는 값들의 비율을 구하는 총 네 가지 방법의 예시를 보면 다음과 같다.

1. **`CASE`**-`WHEN`-`THEN`-(`ELSE`)-`END` 절과 `COUNT()` 사용하기

    ```sql
    ROUND(COUNT(CASE WHEN rating < 3 THEN 1 END) / COUNT(*) * 100, 2)
    ```

2. **`IF()`**와 `SUM()` 사용 후 `COUNT(*)`로 나누기

    ```sql
    ROUND(SUM(IF(rating < 3, 1, 0)) / COUNT(*) * 100, 2)
    ```

3. **`SUM(조건절)`** 사용 후 `COUNT(*)`로 나누기
    
    ```sql
    ROUND(SUM(rating < 3) / COUNT(*) * 100, 2)
    ```

4. **`AVG(조건절)`** 사용하기 (→ ✅)

    > `COUNT(*)`로 나눌 필요가 없으므로 <span class="red">**코드가 가장 간단**</span>해진다!

    ```sql
    ROUND(AVG(rating < 3) * 100, 2)
    ```

<br>

## 가장 간단한 정답 코드

마지막 방법을 사용하면 다음과 같이 답안을 작성할 수 있다.

```sql
SELECT query_name
    , ROUND(AVG(rating / position), 2) AS quality
    , ROUND(AVG(rating < 3) * 100, 2) AS poor_query_percentage  -- AVG(조건절) 사용
FROM Queries
GROUP BY query_name
```

<br>

> 정리하면, `IF()` 나 `CASE`-`WHEN`-`THEN`-(`ELSE`)-`END` 절로 조건을 걸 수도 있으나, <span class="hl">비교연산자 자체가 0, 1 값을 반환</span>하므로 그대로 `SUM`, `AVG` 연산을 이용해도 된다.
{: .prompt-info}
