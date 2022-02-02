---
description: GROUP BY 구문, SUM 함수, AVG 함수, OVER(ORDER BY ~)
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



