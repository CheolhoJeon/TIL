---
description: UNION ALL 구문, LEFT JOIN, 상관 서브 쿼리
---

# 테이블 결합하기

## 1. 세로로 결합하기

> 애플리케이션 1의 사용자 테이블

| user\_id | name   | email                                           |
| -------- | ------ | ----------------------------------------------- |
| U001     | Sato   | [sato@example.com](mailto:sato@example.com)     |
| U002     | Suzuki | [suzuki@example.com](mailto:suzuki@example.com) |

> 애플리케이션 2의 사용자 테이블

| user\_id | name   | phone         |
| -------- | ------ | ------------- |
| U001     | Ito    | 080-xxxx-xxxx |
| U002     | Tanaka | 070-xxxx-xxxx |

* 비슷한 구조를 가지는 테이블의 데이터를 일괄 처리하고 싶은 경우, 아래 처럼 `UNION ALL` 구문을 사용해 여러 개의 테이블을 세로로 결합하면 됨
* `UNION ALL`로 결합할 때는 테이블의 컬럼이 완전히 일치해야 하므로, 한쪽 테이블에만 존재하는 컬럼은 _phone_ 컬럼 처럼 SELECT 구문으로 제외하거나 _email_ 컬럼 처럼 디폴트 값을 할당해야함

```sql
SELECT 'app1' AS app_name, user_id, name, email
FROM app1_mst_users
UNION ALL
SELECT 'app2' AS app_name, user_id, name, NULL
FROM app2_mst_users;
```

| app\_name | user\_id | name   | email                                           |
| --------- | -------- | ------ | ----------------------------------------------- |
| app1      | U001     | Sato   | [sato@example.com](mailto:sato@example.com)     |
| app1      | U002     | Suzuki | [suzuki@example.com](mailto:suzuki@example.com) |
| app2      | U001     | Ito    | NULL                                            |
| app2      | U002     | Tanaka | NULL                                            |

{% hint style="info" %}
`UNION ALL` 구문 대신 `UNION DISTINCT` (또는 `UNION`) 구문을 사용하면, 데이터의 중복을 제외한 결과를 얻을 수 있음. 다만, `UNION ALL`에 비해 거의 사용되지 않고 계산 비용이 많이 들어감
{% endhint %}

## 2. 가로로 정렬하기

* 여러 개의 테이블을 가로 정렬할 때 가장 일반적인 방법은 `JOIN`을 사용하는 것
* 다만 마스터 테이블에 `JOIN`을 사용하면 결합하지 못한 데이터가 사라지거나, 반대로 중복된 데이터가 발생할 수 있음

> 카테고리 테이블

| category\_id | name |
| ------------ | ---- |
| 1            | dvd  |
| 2            | cd   |
| 3            | book |

> 카테고리 테이블

| category\_id | sales  |
| ------------ | ------ |
| 1            | 850000 |
| 2            | 500000 |

> 카테고리별 상품 매출 순위 테이블

| category\_id | rank | product\_id | sales |
| ------------ | ---- | ----------- | ----- |
| 1            | 1    | D001        | 50000 |
| 1            | 2    | D002        | 20000 |
| 1            | 3    | D003        | 10000 |
| 2            | 1    | C001        | 30000 |
| 2            | 2    | C002        | 20000 |
| 2            | 3    | C003        | 10000 |

### 2-1. JOIN을 사용해 정렬하기

* 아래의 코드는 `JOIN`을 사용하여 결합한 결과를 보여줌
* 데이터 손실과 데이터 중복이 모두 발생함
  * book 카테고리가 손실됨
  * 여러 개의 상품 ID가 부여된 DVD/CD 카테고리는 가격이 중복되어 출력되고 있음

```sql
SELECT m.category_id,
       m.name,
       s.sales,
       r.product_id AS sale_product
FROM mst_categories AS m
         -- 카테고리별로 매출액 결합하기
         JOIN category_sales AS s ON m.category_id = s.category_id
         -- 카테고리별로 상품 결합하기
         JOIN product_sale_ranking AS r ON m.category_id = r.category_id
;
```

| category\_id | name | sales  | sale\_product |
| ------------ | ---- | ------ | ------------- |
| 1            | dvd  | 850000 | D001          |
| 1            | dvd  | 850000 | D002          |
| 1            | dvd  | 850000 | D003          |
| 2            | cd   | 500000 | C001          |
| 2            | cd   | 500000 | C002          |
| 2            | cd   | 500000 | C003          |

{% hint style="warning" %}
이 예제에서 얻고자 하는 결과물 입장에서 데이터 중복이지. 만약 _product\_id_에 대한 모든 정보가 필요하다면 `JOIN`를 사용해아함
{% endhint %}

### 2-2. LEFT JOIN을 사용해 정렬하기

마스터 테이블의 행 수를 변경하지 않고 데이터를 가로로 정렬하려면, `LEFT JOIN`을 사용해 결합하지 못한 레코드를 유지한 상태로, 결합할 레코드가 반드시 1개 이하가 되게 하는 조건을 사용해야함

```sql
SELECT m.category_id,
       m.name,
       s.sales,
       r.product_id AS top_sale_product
FROM mst_categories AS m
         -- LEFT JOIN을 사용해서 결합하지 못한 레코드를 남김  
         LEFT JOIN category_sales AS s ON m.category_id = s.category_id
         -- LEFT JOIN을 사용해서 결합하지 못한 레코드를 남김
         -- 카테고리별 최고 매출 상품 하나만 추출해서 결합하기
         LEFT JOIN product_sale_ranking AS r
                   ON m.category_id = r.category_id AND r.rank = 1
;
```

| category\_id | name | sales  | top\_sale\_product |
| ------------ | ---- | ------ | ------------------ |
| 1            | dvd  | 850000 | D001               |
| 2            | cd   | 500000 | C001               |
| 3            | book | NULL   | NULL               |

### 2-3. 상관 서브쿼리를 사용해 정렬하기

* SELECT 구문 내부에서 상관 서브 쿼리를 사용할 수 있는 미들웨어의 경우, `JOIN`을 사용하지 않고 여러 테이블 값을 가로로 정렬할 수 있음
* `JOIN`을 사용하지 않아 원래 마스터 테이블의 행 수가 변할 걱정 자체가 없으므로, 테이블의 누락과 중복을 회피할 수 있음
* 방금 전 예제에서는 카테고리별 매출 테이블에 카테고리들의 _순위(rank)_를 사전에 컬럼으로 저장했지만, 상관 서브쿼리의 경우 내부에서 `ORDER BY` 구문과 `LIMIT` 구문을 사용하면 사전 처리를 하지 않고도 데이터를 하나로 압축할 수 있음

```sql
SELECT m.category_id,
       m.name,
       (SELECT sales
        FROM category_sales AS s
        WHERE m.category_id = s.category_id
       ) AS sales,
       (SELECT product_id
        FROM product_sale_ranking AS r
        WHERE m.category_id = r.category_id
        ORDER BY sales DESC
        LIMIT 1
       ) AS top_sale_product
FROM mst_categories AS m
;
```

| category\_id | name | sales  | top\_sale\_product |
| ------------ | ---- | ------ | ------------------ |
| 1            | dvd  | 850000 | D001               |
| 2            | cd   | 500000 | C001               |
| 3            | book | NULL   | NULL               |
