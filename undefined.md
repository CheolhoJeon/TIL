---
description: CTE(WITH 구문)
---

# 계산된 테이블에 이름 붙여 재사용하기

* 복잡한 처리를 하는 SQL문을 작성할 때는 서브 쿼리의 중첩이 많아짐
* 비슷한 처리를 여러 번 하는 경우도 있는데, 이렇게 되면 쿼리의 가독성이 굉장히 낮아짐
* 이때 SQL99에서 도입된 <mark style="color:red;">공통 테이블 식(CTE: Common Table Expression)</mark>을 사용하면 일시적인 테이블에 이름을 붙여 재사용할 수 있음
* 그리고 이를 활용하면 코드의 가독성이 크게 높아짐

> 카테고리별 상품 매출 테이블

| category\_name | product\_id | sales |
| -------------- | ----------- | ----- |
| dvd            | D001        | 50000 |
| dvd            | D002        | 20000 |
| dvd            | D003        | 10000 |
| cd             | C001        | 30000 |
| cd             | C002        | 20000 |
| cd             | C003        | 10000 |
| book           | B001        | 20000 |
| book           | B002        | 15000 |
| book           | B003        | 10000 |
| book           | B004        | 5000  |

{% hint style="info" %}
아래의 예제(4-1 \~ 4-3)들은 순위의 상품을 가로로 나타내는 방법을 `WITH` 구문을 활용하여 단계적으로 설명함
{% endhint %}

### 4-1. 카테고리별 순위를 추가한 테이블에 이름 붙이기

아래의 코드는 `ROW_NUMBER` 함수를 사용해 카테고리별 순위를 붙임

```sql
WITH product_sale_ranking AS (
    SELECT category_name,
           product_id,
           sales,
           ROW_NUMBER()
           OVER (PARTITION BY category_name ORDER BY sales DESC) AS rank
    FROM product_sales
)
SELECT *
FROM product_sale_ranking
;
```

| category\_name | product\_id | sales | rank |
| -------------- | ----------- | ----- | ---- |
| book           | B001        | 20000 | 1    |
| book           | B002        | 15000 | 2    |
| book           | B003        | 10000 | 3    |
| book           | B004        | 5000  | 4    |
| cd             | C001        | 30000 | 1    |
| cd             | C002        | 20000 | 2    |
| cd             | C003        | 10000 | 3    |
| dvd            | D001        | 50000 | 1    |
| dvd            | D002        | 20000 | 2    |
| dvd            | D003        | 10000 | 3    |

### 4-2. 카테고리들의 순위에서 유니크한 순위 목록을 계산하는 쿼리

```sql
WITH product_sale_ranking AS (
    SELECT category_name,
           product_id,
           sales,
           ROW_NUMBER()
           OVER (PARTITION BY category_name ORDER BY sales DESC) AS rank
    FROM product_sales
)
   , mst_ranks AS (
    SELECT DISTINCT rank
    FROM product_sale_ranking
    ORDER BY rank
)
SELECT *
FROM mst_ranks
;
```

| rank |
| ---- |
| 1    |
| 2    |
| 3    |
| 4    |

### 4-3. 카테고리들의 순위를 횡단적으로 출력하는 쿼리

```sql
WITH product_sale_ranking AS (
    SELECT category_name,
           product_id,
           sales,
           ROW_NUMBER()
           OVER (PARTITION BY category_name ORDER BY sales DESC) AS rank
    FROM product_sales
)
   , mst_ranks AS (
    SELECT DISTINCT rank
    FROM product_sale_ranking
    ORDER BY rank
)
SELECT r.rank,
       p1.product_id AS dvd,
       p1.sales      AS dvd_sales,
       p2.product_id AS cd,
       p2.sales      AS cd_sales,
       p3.product_id AS book,
       p3.sales      AS book_sales
FROM mst_ranks AS r
         LEFT JOIN product_sale_ranking AS p1
                   ON r.rank = p1.rank AND p1.category_name = 'dvd'
         LEFT JOIN product_sale_ranking AS p2
                   ON r.rank = p2.rank AND p2.category_name = 'cd'
         LEFT JOIN product_sale_ranking AS p3
                   ON r.rank = p3.rank AND p3.category_name = 'book'
;
```

| rank | dvd  | dvd\_sales | cd   | cd\_sales | book | book\_sales |
| ---- | ---- | ---------- | ---- | --------- | ---- | ----------- |
| 1    | D001 | 50000      | C001 | 30000     | B001 | 20000       |
| 2    | D002 | 20000      | C002 | 20000     | B002 | 15000       |
| 3    | D003 | 10000      | C003 | 10000     | B003 | 10000       |
| 4    | NULL | NULL       | NULL | NULL      | B004 | 5000        |
