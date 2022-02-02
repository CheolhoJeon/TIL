---
description: GROUP BY 구문, SUM 함수, AVG 함수
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
