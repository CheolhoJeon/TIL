---
description: 날짜/시간 함수, date 자료형, timestamp 자료형, 문자열 함수
---

# 날짜와 타임스탬프 다루기

### 현재 날짜와 타임스탬프 추출하기

```sql
SELECT CURRENT_DATE      AS dt,
       CURRENT_TIMESTAMP AS stamp,
       -- 타임존을 적용하고 싶지 않으면 LOCALTIMESTAMP 사용하기
       LOCALTIMESTAMP    AS localstamp
;
```

| dt         | stamp                             | localstamp                 |
| ---------- | --------------------------------- | -------------------------- |
| 2022-01-25 | 2022-01-25 10:56:40.729976 +00:00 | 2022-01-25 10:56:40.729976 |



### 지정한 값의 날짜/시각 데이터 추출하기

문자열을 날짜 자료형 혹은 타임스탬프 자료형으로 변경하는 방법

```sql
SELECT CAST('2016-01-30' AS date)               AS dt,
       CAST('2016-01-30 12:00:00' AS timestamp) AS stamp
--     :: 연산자는 SQL 표준을 따르지 않음
--     '2016-01-30'::date                       AS dt,
--     '2016-01-30 12:00:00'::timestamp         AS stamp
;
```

| dt         | stamp                      |
| ---------- | -------------------------- |
| 2016-01-30 | 2016-01-30 12:00:00.000000 |

{% embed url="https://www.postgresqltutorial.com/postgresql-cast" %}
PostgreSQL이 제공하는 형변환 방법
{% endembed %}



### 날짜/시각에서 특정 필드 추출하기

타임스탬프 자료형의 데이터에서 년과 월 등의 특정 필드 값을 추출할 때는 EXTRACT 함수를 사용함

#### 타임스탬프 자료형의 데이터에서 연, 월, 일 등을 추출하는 쿼리

```sql
SELECT stamp,
       EXTRACT(YEAR FROM stamp)  AS year,
       EXTRACT(MONTH FROM stamp) AS month,
       EXTRACT(DAY FROM stamp)   AS day,
       EXTRACT(HOUR FROM stamp)  AS hour
FROM (SELECT CAST('2016-01-30 12:00:00' AS timestamp) AS stamp) as t;
```

| stamp                      | year | month | day | hour |
| -------------------------- | ---- | ----- | --- | ---- |
| 2016-01-30 12:00:00.000000 | 2016 | 1     | 30  | 12   |

#### 타임스탬프를 나타내는 문자열에서 연, 월, 일 등을 추출하는 쿼리

```sql
SELECT stamp,
       substring(stamp, 1, 4)  AS year,
       substring(stamp, 6, 2)  AS month,
       substring(stamp, 9, 2)  AS day,
       substring(stamp, 12, 2) AS hour,
       -- 연과 월을 함께 추출하기
       substring(stamp, 1, 7)  AS year_month
FROM (SELECT CAST('2016-01-30 12:00:00' AS text) as stamp) AS t;
```

| stamp               | year | month | day | hour | year\_moth |
| ------------------- | ---- | ----- | --- | ---- | ---------- |
| 2016-01-30 12:00:00 | 2016 | 01    | 30  | 12   | 2016-01    |

