---
description: MAX(CASE ~) 구문, string_agg 함수, unnest 함수, CROSS JOIN, regexp_split_to_table 함수
---

# 가로 <-> 세로

## 1. 세로 기반 데이터를 가로 기반으로 변환하기

행 단위로 저장된 ‘세로 기반’을, 열 또는 쉼표로 구분된 문자열 등의 ‘가로 기반’으로 변환하는 방법을 설명함

### 1-1. 행을 열로 변환하기

* SQL에서 열은 고정적이어야 함
* 따라서 아래의 방법은 열로 전개할 데이터의 종류 또는 수를 명확하게 미리 알고 있어야 사용 가능

> 날짜별 KPI 데이터: 날짜별로 노출(impressions), 세션(sessions), 사용자(users)

| dt         | indicator   | val  |
| ---------- | ----------- | ---- |
| 2017-01-01 | impressions | 1800 |
| 2017-01-01 | sessions    | 500  |
| 2017-01-01 | users       | 200  |
| 2017-01-02 | impressions | 2000 |
| 2017-01-02 | sessions    | 700  |
| 2017-01-02 | users       | 250  |

<details>

<summary>SQL</summary>

```sql
SELECT dt,
       MAX(CASE WHEN indicator = 'impressions' THEN val END) AS impressions,
       MAX(CASE WHEN indicator = 'sessions' THEN val END)    AS sessions,
       MAX(CASE WHEN indicator = 'users' THEN val END)       AS uers
FROM daily_kpi
GROUP BY dt
ORDER BY dt;
```

</details>

| dt         | impressions | sessions | uers |
| ---------- | ----------- | -------- | ---- |
| 2017-01-01 | 1800        | 500      | 200  |
| 2017-01-02 | 2000        | 700      | 250  |

### 1-2. 행을 쉼표로 구분한 문자열로 집약하기

아래의 방법은 열의 종류와 수를 모르는 경우 사용할 수 있음

> 구매 이력

| purchase\_id | product\_id | price |
| ------------ | ----------- | ----- |
| 100001       | A001        | 3000  |
| 100001       | A002        | 4000  |
| 100001       | A003        | 2000  |
| 100002       | D001        | 5000  |
| 100002       | D002        | 3000  |
| 100003       | A001        | 3000  |

<details>

<summary>SQL</summary>

```sql
SELECT purchase_id,

       -- 상품 ID를 배열에 집약하고 쉼표로 구분된 문자열로 변환하기
       STRING_AGG(product_id, ','),
       SUM(price) AS amount
FROM purchase_detail_log
GROUP BY purchase_id
ORDER BY purchase_id;
```

</details>

| purchase\_id | string\_agg    | amount |
| ------------ | -------------- | ------ |
| 100001       | A001,A002,A003 | 9000   |
| 100002       | D001,D002      | 8000   |
| 100003       | A001           | 3000   |

## 2. 가로 기반 데이터를 세로 기반으로 변환하기

* 레코드로 저장된 세로 기반 데이터를 가로 기반으로 변환하는 것은 비교적 간단함
* 하지만 반대로 가로 기반 데이터를 세로 기반으로 변환하는 것은 간단한 일이 아님

### 2-1. 열로 표현된 값을 행으로 변환하기

| year | q1    | q2    | q3    | q4    |
| ---- | ----- | ----- | ----- | ----- |
| 2015 | 82000 | 83000 | 78000 | 83000 |
| 2016 | 85000 | 85000 | 80000 | 81000 |
| 2017 | 92000 | 81000 | NULL  | NULL  |

* 컬럼으로 표현된 가로 기반 데이터의 특징은 데이터의 수가 고정되어 있다는 것
* 행으로 전개할 데이터 수가 고정되었다면, 데이터 수와 같은 수의 일련 번호를 가진 피벗 테이블을 만들고 `CROSS JOIN` 하면 됨

<details>

<summary>SQL</summary>

```sql
SELECT q.year,
       -- Q1에서 Q4까지의 레이블 이름 출력하기
       CASE
           WHEN p.idx = 1 THEN 'q1'
           WHEN p.idx = 2 THEN 'q2'
           WHEN p.idx = 3 THEN 'q3'
           WHEN p.idx = 4 THEN 'q4'
       END AS quarter,
       -- Q1에서 Q4까지의 매출 출력하기
       CASE
           WHEN p.idx = 1 THEN q.q1
           WHEN p.idx = 2 THEN q.q2
           WHEN p.idx = 3 THEN q.q3
           WHEN p.idx = 4 THEN q.q4
       END AS sales
FROM quarterly_sales AS q
     CROSS JOIN
     -- 행으로 전개하고 싶은 열의 수만큼 순번 테이블 만들기
     (
         SELECT 1 AS idx
         UNION ALL
         SELECT 2 AS idx
         UNION ALL
         SELECT 3 AS idx
         UNION ALL
         SELECT 4 AS idx
     ) AS p
;
```

</details>

| year | quarter | sales |
| ---- | ------- | ----- |
| 2015 | q1      | 82000 |
| 2015 | q2      | 83000 |
| 2015 | q3      | 78000 |
| 2015 | q4      | 83000 |
| 2016 | q1      | 85000 |
| 2016 | q2      | 85000 |
| 2016 | q3      | 80000 |
| 2016 | q4      | 81000 |
| 2017 | q1      | 92000 |
| 2017 | q2      | 81000 |
| 2017 | q3      | NULL  |
| 2017 | q4      | NULL  |

### 2-2. 임의의 길이를 가진 배열을 행으로 전개하기

* 고정 길이의 데이터를 행으로 전개하는 것은 비교적 간단함
* 하지만 데이터의 길이가 확정되지 않은 경우는 조금 복잡함

> 구매 로그

| purchase\_id | product\_ids   |
| ------------ | -------------- |
| 100001       | A001,A002,A003 |
| 100002       | D001,D002      |
| 100003       | A001           |

#### unnest 함수

* 테이블 함수를 구현하고 있는 미들웨어라면 배열을 쉽게 레코드로 전개할 수 있음
* 테이블 함수란 함수의 리턴 값이 테이블인 함수를 의미함
* 대표적인 테이블 함수로는 `unnest` 함수가 있음

```sql
SELECT UNNEST(ARRAY['A001', 'A002', 'A003']) AS product_id;
```

| product\_id |
| ----------- |
| A001        |
| A002        |
| A003        |

#### 구매 로그 테이블을 행으로 전개하기

* 실제 데이터에 테이블 함수를 사용할 경우 한 가지 주의할 것이 있음
* 일반적인 SELECT 구문 내부에는 레코드에 포함된 스칼라 값을 리턴하는 함수와 컬럼 이름을 지정할 수 있지만, 테이블 함수는 ‘테이블’을 리턴함
* 스칼라 값과 테이블 함수의 리턴 값을 동시에 추출하고 싶은 경우, 테이블 함수를 FROM 구문 내부에 작성하고, JOIN 구문을 사용해 원래 테이블과 테이블 함수의 리턴 값을 결합해야함

<details>

<summary>SQL</summary>

```sql
SELECT purchase_id,
       product_id
FROM purchase_log AS p
         CROSS JOIN UNNEST(string_to_array(product_ids, ',')) AS product_id
;
```

</details>

| purchase\_id | product\_id |
| ------------ | ----------- |
| 100001       | A001        |
| 100001       | A002        |
| 100001       | A003        |
| 100002       | D001        |
| 100002       | D002        |
| 100003       | A001        |

* `PostgreSQL`의 경우 SELECT 구문 내부에 스칼라 값과 테이블 함수를 동시에 지정할 수 있음
* 또한 문자열을 구분자로 분할해서 테이블화하는 `regexp_split_to_table` 함수가 구현되어 있으므로, 아래와 같이 간단하게 행으로 전개할 수 있음

<details>

<summary>SQL</summary>

```sql
SELECT purchase_id,
       -- 쉼표로 구분된 문자열을 한 번에 행으로 전개하기
       regexp_split_to_table(product_ids, ',') AS product_id
FROM purchase_log;
```

</details>

| purchase\_id | product\_id |
| ------------ | ----------- |
| 100001       | A001        |
| 100001       | A002        |
| 100001       | A003        |
| 100002       | D001        |
| 100002       | D002        |
| 100003       | A001        |
