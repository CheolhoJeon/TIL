---
description: 날짜/시간 함수, date 자료형, timestamp 자료형, 문자열 함수, interval 자료형
---

# 날짜와 타임스탬프 다루기

## 1. 현재 날짜와 타임스탬프 추출하기

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

## 2. 지정한 값의 날짜/시각 데이터 추출하기

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

## 3. 날짜/시각에서 특정 필드 추출하기

타임스탬프 자료형의 데이터에서 년과 월 등의 특정 필드 값을 추출할 때는 EXTRACT 함수를 사용함

### 3-1. 타임스탬프 자료형의 데이터에서 연, 월, 일 등을 추출하는 쿼리

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

### 3-2. 타임스탬프를 나타내는 문자열에서 연, 월, 일 등을 추출하는 쿼리

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



## 4. 날짜/시간 계산하기

> 등록 시간과 생일을 포함하는 사용자 마스터 테이블

| user\_id | register\_stamp     | birth\_date |
| -------- | ------------------- | ----------- |
| U001     | 2016-02-28 10:00:00 | 2000-02-29  |
| U002     | 2016-02-29 10:00:00 | 2000-02-29  |
| U003     | 2016-03-01 10:00:00 | 2000-02-29  |

#### 4-1. 미래 또는 과거의 날짜시간 계산하기

```sql
SELECT user_id,
       register_stamp::timestamp                          AS register_stamp,
       register_stamp::timestamp + '1 hour'::interval     AS after_1_hour,
       register_stamp::timestamp - '30 minutes'::interval AS before_30_minutes,
       register_stamp::date                               AS register_date,
       register_stamp::date + '1 day'::interval           AS after_1_day,
       register_stamp::date - '1 month'::interval         AS before_1_month
FROM mst_users_with_dates;
```

| user\_id | register\_stamp            | after\_1\_hour             | before\_30\_minutes        | register\_date | after\_1\_day              | before\_1\_month           |
| -------- | -------------------------- | -------------------------- | -------------------------- | -------------- | -------------------------- | -------------------------- |
| U001     | 2016-02-28 10:00:00.000000 | 2016-02-28 11:00:00.000000 | 2016-02-28 09:30:00.000000 | 2016-02-28     | 2016-02-29 00:00:00.000000 | 2016-01-28 00:00:00.000000 |
| U002     | 2016-02-29 10:00:00.000000 | 2016-02-29 11:00:00.000000 | 2016-02-29 09:30:00.000000 | 2016-02-29     | 2016-03-01 00:00:00.000000 | 2016-01-29 00:00:00.000000 |
| U003     | 2016-03-01 10:00:00.000000 | 2016-03-01 11:00:00.000000 | 2016-03-01 09:30:00.000000 | 2016-03-01     | 2016-03-02 00:00:00.000000 | 2016-02-01 00:00:00.000000 |

#### 4-2. 날짜 데이터들의 차이 계산하기

```sql
SELECT user_id,
       CURRENT_DATE                        AS today,
       register_stamp::date                AS register_date,
       CURRENT_DATE - register_stamp::date AS diff_days
FROM mst_users_with_dates;
```

| user\_id | today      | register\_date | diff\_days |
| -------- | ---------- | -------------- | ---------- |
| U001     | 2022-01-25 | 2016-02-28     | 2158       |
| U002     | 2022-01-25 | 2016-02-29     | 2157       |
| U003     | 2022-01-25 | 2016-03-01     | 2156       |

#### 4-3. 사용자의 생년월일로 나이 계산하기

```sql
SELECT user_id,
       CURRENT_DATE                                                   AS today,
       register_stamp::date                                           AS register_date,
       birth_date::date                                               AS birth_date,
       EXTRACT(YEAR FROM age(birth_date::date))                       AS current_age,
       EXTRACT(YEAR FROM age(register_stamp::date, birth_date::date)) AS register_age
FROM mst_users_with_dates;
```

| user\_id | today      | register\_date | birth\_date | current\_age | register\_age |
| -------- | ---------- | -------------- | ----------- | ------------ | ------------- |
| U001     | 2022-01-25 | 2016-02-28     | 2000-02-29  | 21           | 15            |
| U002     | 2022-01-25 | 2016-02-29     | 2000-02-29  | 21           | 16            |
| U003     | 2022-01-25 | 2016-03-01     | 2000-02-29  | 21           | 16            |

