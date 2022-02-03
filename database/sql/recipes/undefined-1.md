---
description: UNION ALL 구문, ROLLUP 구문, SUM(~), OVER(ORDER BY ~), width_bucket 함수
---

# 다면척인 축을 사용해 데이터 집약하기

> 구매 이력 데이터

| dt         | order\_id | user\_id | item\_id | price  | category       | sub\_category |
| ---------- | --------- | -------- | -------- | ------ | -------------- | ------------- |
| 2017-01-18 | 48291     | usr33395 | lad533   | 37300  | ladys\_fashion | bag           |
| 2017-01-18 | 48291     | usr33395 | lad329   | 97300  | ladys\_fashion | jacket        |
| 2017-01-18 | 48291     | usr33395 | lad102   | 114600 | ladys\_fashion | jacket        |
| 2017-01-18 | 48291     | usr33395 | lad886   | 33300  | ladys\_fashion | bag           |
| 2017-01-18 | 48292     | usr52832 | dvd871   | 32800  | dvd            | documentary   |
| 2017-01-18 | 48292     | usr52832 | gam167   | 26000  | game           | accessories   |
| 2017-01-18 | 48292     | usr52832 | lad289   | 57300  | ladys\_fashion | bag           |
| 2017-01-18 | 48293     | usr28891 | out977   | 28600  | outdoor        | camp          |
| 2017-01-18 | 48293     | usr28891 | boo256   | 22500  | book           | business      |
| 2017-01-18 | 48293     | usr28891 | lad125   | 61500  | ladys\_fashion | jacket        |
| 2017-01-18 | 48294     | usr33604 | mem233   | 116300 | mens\_fashion  | jacket        |
| 2017-01-18 | 48294     | usr33604 | cd477    | 25800  | cd             | classic       |
| 2017-01-18 | 48294     | usr33604 | boo468   | 31000  | book           | business      |
| 2017-01-18 | 48294     | usr33604 | foo402   | 48700  | food           | meats         |
| 2017-01-18 | 48295     | usr38013 | foo134   | 32000  | food           | fish          |
| 2017-01-18 | 48295     | usr38013 | lad147   | 96100  | ladys\_fashion | jacket        |

## 1. 카테고리별 매출과 소계 계산하기

<details>

<summary>UNION ALL을 활용한 방식</summary>

```sql
WITH sub_category_amount AS (
    -- 소 카테고리 매출 집계하기
    SELECT category,
           sub_category,
           SUM(price) AS amount
    FROM purchase_detail_log
    GROUP BY category, sub_category
)
   , category_amount AS (
    -- 대 카테고리의 매출 집계하기
    SELECT category,
           'all'       AS sub_category,
           SUM(amount) AS amount
    FROM sub_category_amount
    GROUP BY category
)
   , total_amount AS (
    -- 전체 매출 집계하기   
    SELECT 'all'       AS category,
           'all'       AS sub_category,
           SUM(amount) AS amount
    FROM category_amount
)
SELECT category, sub_category, amount FROM sub_category_amount
UNION ALL
SELECT category, sub_category, amount FROM category_amount
UNION ALL
SELECT category, sub_category, amount FROM total_amount
;
```

</details>

<details>

<summary>ROLLUP을 활용한 방식</summary>

```sql
SELECT COALESCE(category, 'all')     AS category,
       COALESCE(sub_category, 'all') AS sub_category,
       SUM(price)                    AS amount
FROM purchase_detail_log
GROUP BY ROLLUP (category, sub_category)
ORDER BY category, sub_category
;
```

</details>

| category       | sub\_category | amount |
| -------------- | ------------- | ------ |
| all            | all           | 861100 |
| book           | all           | 53500  |
| book           | business      | 53500  |
| cd             | all           | 25800  |
| cd             | classic       | 25800  |
| dvd            | all           | 32800  |
| dvd            | documentary   | 32800  |
| food           | all           | 80700  |
| food           | fish          | 32000  |
| food           | meats         | 48700  |
| game           | accessories   | 26000  |
| game           | all           | 26000  |
| ladys\_fashion | all           | 497400 |
| ladys\_fashion | bag           | 127900 |
| ladys\_fashion | jacket        | 369500 |
| mens\_fashion  | all           | 116300 |
| mens\_fashion  | jacket        | 116300 |
| outdoor        | all           | 28600  |
| outdoor        | camp          | 28600  |

## 2. ABC 분석으로 잘 팔리는 상품 판별하기

<details>

<summary>SQL</summary>

```sql
WITH monthly_sales AS (
    SELECT category,
           -- 항목별 매출 계산하기
           SUM(price) AS amount
    FROM purchase_detail_log
    WHERE dt BETWEEN '2017-01-01' AND '2017-01-31'
    GROUP BY category
)
   , sales_composition_ratio AS (
    SELECT category,
           amount,
           -- 구성비: 100.0 * <항목별 매출> / <전체 매출>
           100.0 * amount / SUM(amount) OVER () AS composition_ratio,
           -- 구성비누계: 100.0 * <항목별 구계 매출> / <전체 매출>
           100.0 * SUM(amount) OVER (ORDER BY amount DESC ROWS UNBOUNDED PRECEDING) /
           SUM(amount) OVER ()                  AS cumulative_ration
    FROM monthly_sales
)
SELECT *,
       CASE
           WHEN cumulative_ration BETWEEN 0 AND 70 THEN 'A'
           WHEN cumulative_ration BETWEEN 70 AND 90 THEN 'B'
           WHEN cumulative_ration BETWEEN 90 AND 100 THEN 'C'
       END AS abc_rank
FROM sales_composition_ratio
ORDER BY amount DESC
;
```

</details>

| category       | amount | composition\_ratio | cumulative\_ration | abc\_rank |
| -------------- | ------ | ------------------ | ------------------ | --------- |
| ladys\_fashion | 497400 | 57.76              | 57.76              | A         |
| mens\_fashion  | 116300 | 13.50              | 71.26              | B         |
| food           | 80700  | 9.37               | 80.64              | B         |
| book           | 53500  | 6.21               | 86.85              | B         |
| dvd            | 32800  | 3.80               | 90.66              | C         |
| outdoor        | 28600  | 3.32               | 93.98              | C         |
| game           | 26000  | 3.01               | 97.00              | C         |
| cd             | 25800  | 2.99               | 100                | C         |

## 3. 히스토그램으로 구매 가격대 집계하기

* 히스토그램은 가로 축에 단계(데이터의 범위), 세로 축에 도수(데이터의 개수)를 나타내는 그래프
* 히스토그램을 사용하면 데이터가 어떻게 분산되어 있는지를 한 눈에 확인할 수 있음
* 데이터의 산에서 가장 높은 부분을 ‘최빈값'이라고 부름

<details>

<summary>SQL</summary>

```sql
WITH stats AS (
    SELECT MAX(price) + 1              AS max_price,
           MIN(price)                  AS min_price,
           MAX(price) + 1 - MIN(price) AS range_price,
           10                          AS bucket_num
    FROM purchase_detail_log
)
   , purchase_log_with_bucket AS (
    SELECT price,
           min_price,
           -- 정규화 금액: 대상 금액에서 최소 금액을 뺸 것
           price - min_price                                     AS diff,
           -- 계층 범위: 금액 범위를 계층 수로 나눈 것
           1.0 * range_price / bucket_num                        AS bucket_range,
           -- 계층 판정: FLOOR(<정규화 금액> / <계층 범위>)
--            FLOOR(
--                1.0 * (price - min_price) / (1.0 * range_price / bucket_num)
--            ) + 1                                                                   AS bucket
           width_bucket(price, min_price, max_price, bucket_num) AS bucket
    FROM purchase_detail_log,
         stats
)
SELECT bucket,
       -- 계층의 하한과 상한 계산하기
       min_price + bucket_range * (bucket - 1) AS lower_limit,
       min_price + bucket_range * bucket       AS upper_limit,
       -- 도수 세기
       COUNT(price)                            AS num_purchase,
       -- 합계 금액 계산하기
       SUM(price)                              AS total_amount
FROM purchase_log_with_bucket
GROUP BY bucket, min_price, bucket_range
ORDER BY bucket
;
```

</details>

| bucket | lower\_limit | upper\_limit | num\_purchase | total\_amount |
| ------ | ------------ | ------------ | ------------- | ------------- |
| 1      | 22500        | 31880.1      | 5             | 133900        |
| 2      | 31880.1      | 41260.2      | 4             | 135400        |
| 3      | 41260.2      | 50640.3      | 1             | 48700         |
| 4      | 50640.3      | 60020.4      | 1             | 57300         |
| 5      | 60020.4      | 69400.5      | 1             | 61500         |
| 8      | 88160.7      | 97540.8      | 2             | 193400        |
| 10     | 106920.9     | 116301       | 2             | 23090         |
