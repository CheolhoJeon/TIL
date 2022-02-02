---
description: GROUP BY 구문, SUM 함수, AVG 함수, OVER(PARTITION BY ~ ORDER BY ~), SUM(CASE~END)
---

# 시계열 기반으로 데이터 집계하기

> 구매 로그 데이터(2014-01-01 \~ 2014-01-10)

| dt         | order\_id | user\_id   | purchase\_amount |
| ---------- | --------- | ---------- | ---------------- |
| 2014-01-01 | 1         | rhwpvvitou | 13900            |
| 2014-01-01 | 2         | hqnwoamzic | 10616            |
| 2014-01-02 | 3         | tzlmqryunr | 21156            |
| 2014-01-02 | 4         | wkmqqwbyai | 14893            |
| ...        | ...       | ...        | ...              |
| 2014-01-09 | 23        | zzgauelgrt | 16475            |
| 2014-01-09 | 24        | qrzfcwecge | 6469             |
| 2014-01-10 | 26        | cyxfgumkst | 11339            |
| 2014-01-10 | 25        | njbpsrvvcq | 16584            |

## 1. 날짜별 매출 집계하기

<details>

<summary>SQL</summary>

```sql
SELECT dt,
       COUNT(*)                       AS purchase_count,
       round(SUM(purchase_amount), 2) AS totoal_amount,
       round(AVG(purchase_amount), 2) AS avg_amount
FROM purchase_log
GROUP BY dt
ORDER BY dt;
```

</details>

| dt         | purchase\_count | totoal\_amount | avg\_amount |
| ---------- | --------------- | -------------- | ----------- |
| 2014-01-01 | 2               | 24516          | 12258       |
| 2014-01-02 | 2               | 36049          | 18024.5     |
| 2014-01-03 | 3               | 53029          | 17676.33    |
| ...        | ...             | ...            | ...         |
| 2014-01-08 | 3               | 19760          | 6586.67     |
| 2014-01-09 | 2               | 22944          | 11472       |
| 2014-01-10 | 2               | 27923          | 13961.5     |

## 2. 이동평균을 사용한 날짜별 추이 보기

날짜 별 매출과 7일 간의 이동 평균을 구하는 예제

<details>

<summary>SQL</summary>

```sql
SELECT dt,
       SUM(purchase_amount)                AS total_amount,
       -- 최근 최대 7일 동안의 평균 계산하기
       AVG(SUM(purchase_amount))
       OVER (ORDER BY dt ROWS 6 PRECEDING) AS seven_day_avg_amount,
       -- 최근 7일 동안의 평균을 확실하게 계산하기
       CASE
           WHEN 7 = COUNT(*) OVER (ORDER BY dt ROWS 6 PRECEDING)
           THEN AVG(SUM(purchase_amount)) OVER (ORDER BY dt ROWS 6 PRECEDING)
       END                                 AS seven_day_avg_amount_strict
FROM purchase_log
GROUP BY dt
ORDER BY dt
;
```

</details>

| dt         | total\_amount | seven\_day\_avg\_amount | seven\_day\_avg\_amount\_strict |
| ---------- | ------------- | ----------------------- | ------------------------------- |
| 2014-01-01 | 24516         | 24516                   | NULL                            |
| 2014-01-02 | 36049         | 30282.5                 | NULL                            |
| 2014-01-03 | 53029         | 37864.66                | NULL                            |
| 2014-01-04 | 29299         | 35723.25                | NULL                            |
| 2014-01-05 | 48256         | 38229.8                 | NULL                            |
| 2014-01-06 | 29440         | 36764.83                | NULL                            |
| 2014-01-07 | 47679         | 38324                   | 38324                           |
| 2014-01-08 | 19760         | 37644.57                | 37644.57                        |
| 2014-01-09 | 22944         | 35772.42                | 35772.42                        |
| 2014-01-10 | 27923         | 32185.85                | 32185.85                        |

## 3. 당월 매출 누계 구하기

<details>

<summary>SQL</summary>

```sql
WITH daily_purchase AS (
    SELECT dt,
           substring(dt, 1, 4)  AS year,
           substring(dt, 6, 2)  AS month,
           substring(dt, 9, 2)  AS date,
           SUM(purchase_amount) AS purchase_amount
    FROM purchase_log
    GROUP BY dt
)
SELECT dt,
       concat(year, '-', month)                                             AS year_month,
       purchase_amount,
       SUM(purchase_amount)
       OVER (PARTITION BY year, month ORDER BY dt ROWS UNBOUNDED PRECEDING) AS agg_amount
FROM daily_purchase
ORDER BY dt
;
```

</details>

| dt         | year\_month | purchase\_amount | agg\_amount |
| ---------- | ----------- | ---------------- | ----------- |
| 2014-01-01 | 2014-01     | 24516            | 24516       |
| 2014-01-02 | 2014-01     | 36049            | 60565       |
| 2014-01-03 | 2014-01     | 53029            | 113594      |
| ...        | ...         | ...              | ...         |
| 2014-01-08 | 2014-01     | 19760            | 288028      |
| 2014-01-09 | 2014-01     | 22944            | 310972      |
| 2014-01-10 | 2014-01     | 27923            | 338895      |

## 4. 월별 매출의 작대비 구하기

> 구매 로그 테이블(2014-01-01 \~ 2015-12-05)

| dt         | order\_id | user\_id   | purchase\_amount |
| ---------- | --------- | ---------- | ---------------- |
| 2014-01-01 | 1         | rhwpvvitou | 13900            |
| 2014-02-08 | 95        | chtanrqtzj | 28469            |
| 2014-03-09 | 168       | bcqgtwxdgq | 18899            |
| ...        | ...       | ...        | ...              |
| 2015-10-07 | 1591      | trgjscaajt | 13398            |
| 2015-11-06 | 1666      | ccfbjyeqrb | 6213             |
| 2015-12-05 | 1741      | onooskbtzp | 26024            |

<details>

<summary>SQL</summary>

```sql
WITH daily_purchase AS (
    SELECT dt,
           substring(dt, 1, 4)  AS year,
           substring(dt, 6, 2)  AS month,
           substring(dt, 9, 2)  AS date,
           SUM(purchase_amount) AS purchase_amount
    FROM purchase_log
    GROUP BY dt
)
SELECT month,
       SUM(CASE WHEN year = '2014' THEN purchase_amount ELSE 0 END) AS amount_2014,
       SUM(CASE WHEN year = '2015' THEN purchase_amount ELSE 0 END) AS amount_2015,
       100 * SUM(CASE WHEN year = '2015' THEN purchase_amount ELSE 0 END) /
       SUM(CASE WHEN year = '2014' THEN purchase_amount ELSE 0 END) AS rate
FROM daily_purchase
GROUP BY month
ORDER BY month
;
```

</details>

| month | amount\_2014 | amount\_2015 | rate   |
| ----- | ------------ | ------------ | ------ |
| 1     | 13900        | 22111        | 159.07 |
| 2     | 28469        | 11965        | 42.02  |
| 3     | 18899        | 20215        | 106.96 |
| ...   | ...          | ...          | ...    |
| 10    | 6716         | 13398        | 199.49 |
| 11    | 16444        | 6213         | 37.78  |
| 12    | 29199        | 26024        | 89.12  |

## 5. Z 차트로 업적의 추이 확인하기

{% hint style="info" %}
Z 차트란?

* [Z차트를 활용한 매출추이 분석](https://brownbears.tistory.com/536)
* [Z 차트로 매출의 추이 확인하기](https://brownbears.tistory.com/536)
{% endhint %}

<details>

<summary>SQL</summary>

```sql
WITH daily_purchase AS (
    SELECT dt,
           substring(dt, 1, 4)  AS year,
           substring(dt, 6, 2)  AS month,
           substring(dt, 9, 2)  AS date,
           SUM(purchase_amount) AS purchase_amount
    FROM purchase_log
    GROUP BY dt
),
     monthly_amount AS (
         -- 월별 매출 집계하기
         SELECT year,
                month,
                SUM(purchase_amount) AS amount
         FROM daily_purchase
         GROUP BY year, month
     ),
     calc_index AS (
         SELECT year,
                month,
                amount,
                -- 2015년의 누계 매출 집계하기
                SUM(CASE year WHEN '2015' THEN amount END)
                OVER (ORDER BY year, month ROWS UNBOUNDED PRECEDING)                  AS agg_amount,
                -- 당월부터 11개월 이전까지의 매출 합계(이동년계) 집계하기
                SUM(amount)
                OVER (ORDER BY year, month ROWS BETWEEN 11 PRECEDING AND CURRENT ROW) AS year_avg_amount
         FROM monthly_amount
         ORDER BY year, month
     )
-- 마지막으로 2015년의 데이터만 압축하기
SELECT concat(year, '-', month) AS year_month,
       amount,
       agg_amount,
       year_avg_amount
FROM calc_index
WHERE year = '2015'
ORDER BY year_month
;
```

</details>

| year\_month | amount | agg\_amount | year\_avg\_amount |
| ----------- | ------ | ----------- | ----------------- |
| 2015-01     | 22111  | 22111       | 160796            |
| 2015-02     | 11965  | 34076       | 144292            |
| 2015-03     | 20215  | 54291       | 145608            |
| 2015-04     | 11792  | 66083       | 145006            |
| 2015-05     | 18087  | 84170       | 160811            |
| 2015-06     | 18859  | 103029      | 169490            |
| 2015-07     | 14919  | 117948      | 180382            |
| 2015-08     | 12906  | 130854      | 187045            |
| 2015-09     | 5696   | 136550      | 188909            |
| 2015-10     | 13398  | 149948      | 195591            |
| 2015-11     | 6213   | 156161      | 185360            |
| 2015-12     | 26024  | 182185      | 182185            |

## 6. 매출을 파악할 때 중요 포인트

아래의 예제는 구매 로그 데이터를 기반으로 ‘판매 월’, ‘판매 횟수’, ‘평균 구매액’, ‘매출액’, ‘누계 매출액', ‘작년 매출액', ‘작년비'를 구함

<details>

<summary>SQL</summary>

```sql
WITH daily_purchase AS (
    SELECT dt,
           substring(dt, 1, 4)  AS year,
           substring(dt, 6, 2)  AS month,
           substring(dt, 9, 2)  AS date,
           SUM(purchase_amount) AS purchase_amount
    FROM purchase_log
    GROUP BY dt
),
     monthly_purchase AS (
         SELECT year,
                month,
                COUNT(*)             AS orders,
                AVG(purchase_amount) AS avg_amount,
                SUM(purchase_amount) AS monthly
         FROM daily_purchase
         GROUP BY year, month
     )
SELECT CONCAT(year, '-', month)                                         AS year_month,
       orders,
       avg_amount,
       monthly,
       SUM(monthly)
       OVER (PARTITION BY year ORDER BY month ROWS UNBOUNDED PRECEDING) AS agg_amount,
       -- 12개월 전의 매출 구하기
       LAG(monthly, 12) OVER (ORDER BY year, month)                     AS last_year,
       -- 12개월 전의 매출과 비교해서 비율 구하기
       100 * monthly / LAG(monthly, 12) OVER (ORDER BY year, month)     AS rate
FROM monthly_purchase
ORDER BY year_month
;
```

</details>

| year\_month | orders | avg\_amount | monthly | agg\_amount | last\_year | rate   |
| ----------- | ------ | ----------- | ------- | ----------- | ---------- | ------ |
| 2014-01     | 1      | 13900       | 13900   | 13900       | NULL       | NULL   |
| 2014-02     | 1      | 28469       | 28469   | 42369       | NULL       | NULL   |
| 2014-03     | 1      | 18899       | 18899   | 61268       | NULL       | NULL   |
| 2014-04     | 1      | 12394       | 12394   | 73662       | NULL       | NULL   |
| 2014-05     | 1      | 2282        | 2282    | 75944       | NULL       | NULL   |
| 2014-06     | 1      | 10180       | 10180   | 86124       | NULL       | NULL   |
| 2014-07     | 1      | 4027        | 4027    | 90151       | NULL       | NULL   |
| 2014-08     | 1      | 6243        | 6243    | 96394       | NULL       | NULL   |
| 2014-09     | 1      | 3832        | 3832    | 100226      | NULL       | NULL   |
| 2014-10     | 1      | 6716        | 6716    | 106942      | NULL       | NULL   |
| 2014-11     | 1      | 16444       | 16444   | 123386      | NULL       | NULL   |
| 2014-12     | 1      | 29199       | 29199   | 152585      | NULL       | NULL   |
| 2015-01     | 1      | 22111       | 22111   | 22111       | 13900      | 159.07 |
| 2015-02     | 1      | 11965       | 11965   | 34076       | 28469      | 42.02  |
| 2015-03     | 1      | 20215       | 20215   | 54291       | 18899      | 106.96 |
| 2015-04     | 1      | 11792       | 11792   | 66083       | 12394      | 95.14  |
| 2015-05     | 1      | 18087       | 18087   | 84170       | 2282       | 792.59 |
| 2015-06     | 1      | 18859       | 18859   | 103029      | 10180      | 185.25 |
| 2015-07     | 1      | 14919       | 14919   | 117948      | 4027       | 370.47 |
| 2015-08     | 1      | 12906       | 12906   | 130854      | 6243       | 206.72 |
| 2015-09     | 1      | 5696        | 5696    | 136550      | 3832       | 148.64 |
| 2015-10     | 1      | 13398       | 13398   | 149948      | 6716       | 199.49 |
| 2015-11     | 1      | 6213        | 6213    | 156161      | 16444      | 37.78  |
| 2015-12     | 1      | 26024       | 26024   | 182185      | 29199      | 89.12  |
