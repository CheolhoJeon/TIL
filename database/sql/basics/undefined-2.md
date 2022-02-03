---
description: >-
  집약 함수, GROUP BY 구문, 윈도 함수, OVER(... PARTITION BY ~) 구문, OVER(...ORDER BY~) 구문,
  OVER(... ROWS~) 구문
---

# 그룹핑

## 1. 그룹의 특징 잡기

* SQL은 집약 함수라고 부르는 여러 가지 함수를 제공함
  * 여러 레코드를 기반으로 하나의 값을 리턴하는 함수
  * 레코드의 수, 합계, 평균, 최대, 최소를 계산
* SQL:2003에서 도입된 `윈도 함수`(분석 함수라고도 불림)를 사용하면, 기존의 SQL로는 하기 힘들었던 순서를 고려하는 처리, 여러 개의 레코드를 대상으로 하는 처리를 쉽게 할 수 있음

> 상품 평가 테이블

| user\_id | product\_id | score |
| -------- | ----------- | ----- |
| U001     | A001        | 4     |
| U001     | A002        | 5     |
| U001     | A003        | 5     |
| U002     | A001        | 3     |
| U002     | A002        | 3     |
| U002     | A003        | 4     |
| U003     | A001        | 5     |
| U003     | A002        | 4     |
| U003     | A003        | 4     |

### 1-1. 테이블 전체의 특징량 계산하기

* `COUNT 함수`는 지정한 컬럼의 레코드 수를 리턴하는 함수
* 컬럼 이름 앞에 `DISTINCT` 구문을 지정하면, 중복을 제외하고 수를 세어줌
* `SUM`, `AVG` 함수는 컬럼의 자료형이 정수 또는 실수 등의 숫자 자료형이어야함
* `MAX`, `MIN` 함수는 각각 최댓값과 최솟값을 구하는 함수
  * 따라서 대소 비교가 가능한 자료형(숫자, 문자열, 타임스탬프 등)에 적용할 수 있음

<details>

<summary>SQL</summary>

```sql
SELECT COUNT(*)                   AS total_count,
       COUNT(DISTINCT user_id)    AS user_count,
       COUNT(DISTINCT product_id) AS product_count,
       SUM(score)                 AS sum,
       AVG(score)                 AS avg,
       MAX(score)                 AS max,
       MIN(score)                 AS min
FROM review;
```

</details>

| total\_count | user\_count | product\_count | sum | avg               | max | min |
| ------------ | ----------- | -------------- | --- | ----------------- | --- | --- |
| 9            | 3           | 3              | 37  | 4.111111111111111 | 5   | 3   |

### 1-2. 그룹핑한 데이터의 특징량 계산하기

* 데이터를 조금 더 작게 분할하고 싶다면 `GROUP BY` 구문을 사용해 데이터를 분류할 키를 지정하고, 그러한 키를 기반으로 데이터를 집약할 수 있음
* 이때 `GROUP BY` 구문을 사용한 쿼리에서는, `GROUP BY` 구문에 지정한 컬럼 또는 집약 함수만 `SELECT` 구문의 컬럼으로 지정할 수 있음

{% hint style="info" %}
GROUP BY 구문을 사용한 쿼리에서는 GROUP BY 구문에 지정한 컬럼을 유니크 키로 새로운 테이블을 만들게 됨. 이 과정에서 GROUP BY 구문에 지정하지 않은 컬럼은 사라져버림. 따라서 집약 함수를 적용한 값과 집약 전의 값은 동시에 사용할 수 없는 것임
{% endhint %}

<details>

<summary>SQL</summary>

```sql
SELECT user_id,
       COUNT(*)                   AS total_count,
       COUNT(DISTINCT product_id) AS product_count,
       SUM(score)                 AS sum,
       AVG(score)                 AS avg,
       MAX(score)                 AS max,
       MIN(score)                 AS min
FROM review
GROUP BY user_id;
```

</details>

| user\_id | total\_count | product\_count | sum | avg  | max | min |
| -------- | ------------ | -------------- | --- | ---- | --- | --- |
| U001     | 3            | 3              | 14  | 4.66 | 5   | 4   |
| U002     | 3            | 3              | 10  | 3.33 | 4   | 3   |
| U003     | 3            | 3              | 13  | 4.33 | 5   | 4   |

### 1-3. 집약 함수를 적용한 값과 집약 전의 값을 동시에 다루기

* SQL:2003 이후에 정의된 윈도 함수가 지원되는 환경이라면, 윈도 함수를 사용해서 쉽고 효율적으로 집약 함수의 결과와 원래 값을 조합합 수 있음
* 집약 함수로 윈도 함수를 사용하려면, 집약 함수 뒤에 `OVER 구문`을 붙이고 여기에 윈도 함수를 지정함
* `OVER 구문`에 매개 변수를 지정하지 않으면 테이블 전체에 집약 함수를 적용한 값이 리턴됨
* 매개 변수에 `PARTITION BY <컬럼이름>`을 지정하면 해당 컬럼 값을 기반으로 그룹화하고 집약 함수를 적용함

<details>

<summary>SQL</summary>

```sql
SELECT user_id,
       product_id,
       -- 개별 리뷰 점수
       score,
       -- 전체 평균 리뷰 점수
       AVG(score) OVER ()                             AS avg_score,
       -- 사용자의 평균 리뷰 점수
       AVG(score) OVER (PARTITION BY user_id)         AS user_avg_score,
       -- 개별 리뷰 점수와 사용자 평균 리뷰 점수의 차이
       score - AVG(score) OVER (PARTITION BY user_id) AS user_avg_score_diff
FROM review;
```

</details>

| user\_id | product\_id | score | avg\_score | user\_avg\_score | user\_avg\_score\_diff |
| -------- | ----------- | ----- | ---------- | ---------------- | ---------------------- |
| U001     | A001        | 4     | 4.11       | 4.67             | -0.67                  |
| U001     | A002        | 5     | 4.11       | 4.67             | 0.33                   |
| U001     | A003        | 5     | 4.11       | 4.67             | 0.33                   |
| U002     | A001        | 3     | 4.11       | 3.33             | -0.33                  |
| U002     | A002        | 3     | 4.11       | 3.33             | -0.33                  |
| U002     | A003        | 4     | 4.11       | 3.33             | 0.67                   |
| U003     | A001        | 5     | 4.11       | 4.33             | 0.67                   |
| U003     | A002        | 4     | 4.11       | 4.33             | -0.33                  |
| U003     | A003        | 4     | 4.11       | 4.33             | -0.33                  |

## 2. 그룹 내부의 순서

* SQL의 테이블은 기본적으로 순서라는 개념이 없음
* 따라서 SQL로 순위를 작성하거나 시간 순서로 데이터를 다루려면 복잡한 방법을 사용해야 했음
* 하지만 윈도 함수가 등장하면서 SQL로 순서를 다루는 것이 굉장히 쉬워졌음

| product\_id | category | score |
| ----------- | -------- | ----- |
| A001        | action   | 94    |
| A002        | action   | 81    |
| A003        | action   | 78    |
| A004        | action   | 64    |
| D001        | drama    | 90    |
| D002        | drama    | 82    |
| D003        | drama    | 78    |
| D004        | drama    | 58    |

### 2-1. ORDER BY 구문으로 순서 정의하기

* 윈도 함수란 테이블 내부에 ‘윈도'라고 부르는 범위를 정의하고, 해당 범위 내부에 포함된 값을 특정 레코드에서 자유롭게 사용하려고 도입한 것
* 다만 윈도 내부에서 특정 값을 참조하려면 해당 값의 위치를 명확하게 지정해야함
* 윈도 함수에서는 `OVER` 구문 내부에 `ORDER BY` 구문을 사용할 수 있음
* 이를 사용하면 윈도 내부에 있는 데이터의 순서를 정의할 수 있음

`ORDER BY <컬럼이름>`으로 테이블 내부의 순서를 지정했을 경우, 아래의 함수를 사용할 수 있음:

{% tabs %}
{% tab title="ROW_NUMBER()" %}
순서에 유일한 순위 번호를 붙임
{% endtab %}

{% tab title="RANK()" %}
같은 순위의 레코드가 있을 때 순위 번호를 같게 붙임\
단, 같은 순위의 레코드 뒤의 순위 번호를 건너뜀
{% endtab %}

{% tab title="DENSE_RANK()" %}
같은 순위의 레코드가 있을 때 순위 번호를 같게 붙임\
단, 같은 순위의 레코드 뒤의 순위 번호를 건너뛰지 않음
{% endtab %}

{% tab title="LAG()" %}
현재 행을 기준으로 앞의 행의 값을 추출
{% endtab %}

{% tab title="LEAD()" %}
현재 행을 기준으로 뒤의 행의 값을 추출
{% endtab %}
{% endtabs %}

<details>

<summary>SQL</summary>

```sql
SELECT product_id,
       score,

       -- 점수 순서로 유일한 순위를 붙임
       ROW_NUMBER() OVER (ORDER BY score DESC)        AS row,
       -- 같은 순위를 허용해서 순위를 붙임
       RANK() OVER (ORDER BY score DESC)              AS rank,
       -- 같은 순위가 있을 때 같은 순위 다음에 있는 순위를 건너뛰고 붙임
       DENSE_RANK() OVER (ORDER BY score DESC)        AS dense_rank,

       -- 현재 행보다 앞에 있는 행의 값 추출하기
       LAG(product_id) OVER (ORDER BY score DESC)     AS lag1,
       LAG(product_id, 2) OVER (ORDER BY score DESC)  AS lag2,
       -- 현재 행보다 뒤에 있는 행의 값 추출하기
       LEAD(product_id) OVER (ORDER BY score DESC)    AS lead1,
       LEAD(product_id, 2) OVER (ORDER BY score DESC) AS lead2
FROM popular_products
ORDER BY row;
```

</details>

| product\_id | score | row | rank | dense\_rank | lag1 | lag2 | lead1 | lead2 |
| ----------- | ----- | --- | ---- | ----------- | ---- | ---- | ----- | ----- |
| A001        | 94    | 1   | 1    | 1           | NULL | NULL | D001  | D002  |
| D001        | 90    | 2   | 2    | 2           | A001 | NULL | D002  | A002  |
| D002        | 82    | 3   | 3    | 3           | D001 | A001 | A002  | A003  |
| A002        | 81    | 4   | 4    | 4           | D002 | D001 | A003  | D003  |
| A003        | 78    | 5   | 5    | 5           | A002 | D002 | D003  | A004  |
| D003        | 78    | 6   | 5    | 5           | A003 | A002 | A004  | D004  |
| A004        | 64    | 7   | 7    | 6           | D003 | A003 | D004  | NULL  |
| D004        | 58    | 8   | 8    | 7           | A004 | D003 | NULL  | NULL  |

### 2-2. ORDER BY 구문과 집약 함수 조합하기

* `ORDER BY` 구문에 이어지는 `ROWS` 구문은 윈도 프레임 지정 구문임
* <mark style="color:red;">윈도 프레임 지정</mark>이란 현재 레코드 위치를 기반으로 상대적인 윈도를 정의하는 구문임
* 프레임 지정 구문에는 여러 가지 종류가 있음
* 가장 기본적인 형태은 `ROWS BETWEEN <START> AND <END>`임

`START`와 `END`에는 아래의 키워드들이 위치할 수 있음

{% tabs %}
{% tab title="CURRENT ROW" %}
현재의 행
{% endtab %}

{% tab title="[Something] PRECEDING" %}
* `n PRECEDING`: n행 앞
* `UNBOUNDED PRECEDING`: 이전 행 전부&#x20;
{% endtab %}

{% tab title="[Something] FOLLOWING" %}
* `n FOLLOWING`: n행 뒤
* `UNBOUNDED FOLLOWING`: 이후 행 전부
{% endtab %}
{% endtabs %}

<details>

<summary>SQL</summary>

```sql
SELECT product_id,
       score,
       -- 점수 순서로 유일한 순위를 붙임
       ROW_NUMBER() OVER (ORDER BY score DESC)                                             AS row,

       -- 순위 상부터의 누계 점수 계산하기
       SUM(score)
       OVER (ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)         AS cum_score,

       -- 현재 행과 앞 뒤의 행이 가진 값을 기반으로 평균 점수 계산하기
       AVG(score)
       OVER (ORDER BY score DESC ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)                 AS local_avg,

       -- 순위가 높은 상품 ID 추출하기
       FIRST_VALUE(product_id)
       OVER (ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS first_value,
       -- 순위가 낮은 상품 ID 추출하기
       LAST_VALUE(product_id)
       OVER (ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_value
FROM popular_products
ORDER BY row;
```

</details>

| product\_id | score | row | cum\_score | local\_avg        | first\_value | last\_value |
| ----------- | ----- | --- | ---------- | ----------------- | ------------ | ----------- |
| A001        | 94    | 1   | 94         | 92                | A001         | D004        |
| D001        | 90    | 2   | 184        | 88.66666666666667 | A001         | D004        |
| D002        | 82    | 3   | 266        | 84.33333333333333 | A001         | D004        |
| A002        | 81    | 4   | 347        | 80.33333333333333 | A001         | D004        |
| A003        | 78    | 5   | 425        | 79                | A001         | D004        |
| D003        | 78    | 6   | 503        | 73.33333333333333 | A001         | D004        |
| A004        | 64    | 7   | 567        | 66.66666666666667 | A001         | D004        |
| D004        | 58    | 8   | 625        | 61                | A001         | D004        |

{% hint style="info" %}
윈도 함수의 프레임 지정 범위를 이해하기에 아래의 예제는 매우 용이함
{% endhint %}

<details>

<summary>SQL</summary>

```sql
SELECT product_id,
       -- 점수 순서로 유일한 순위를 붙임
       ROW_NUMBER() OVER (ORDER BY score DESC)                                             AS row,

       -- 가장 앞 순위부터 가장 뒷 순위까지의 범위를 대상으로 상품 ID 집약하기
       ARRAY_AGG(product_id)
       OVER (ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS whole_agg,

       -- 가장 앞 순위부터 현재 순위까지의 범위를 대상으로 상품 ID 집약하기
       ARRAY_AGG(product_id)
       OVER (ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)         AS cum_agg,

       -- 순위 하나 앞과 하나 뒤까지의 범위를 대상으로 상품 ID 집약하기
       ARRAY_AGG(product_id)
       OVER (ORDER BY score DESC ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)                 AS local_agg
FROM popular_products
WHERE category = 'action'
ORDER BY row;
```

</details>

| product\_id | row | whole\_agg            | cum\_agg              | local\_agg       |
| ----------- | --- | --------------------- | --------------------- | ---------------- |
| A001        | 1   | {A001,A002,A003,A004} | {A001}                | {A001,A002}      |
| A002        | 2   | {A001,A002,A003,A004} | {A001,A002}           | {A001,A002,A003} |
| A003        | 3   | {A001,A002,A003,A004} | {A001,A002,A003}      | {A002,A003,A004} |
| A004        | 4   | {A001,A002,A003,A004} | {A001,A002,A003,A004} | {A003,A004}      |

### 2-3. PARTITION BY와 ORDER BY 조합하기

윈도 함수의 `PARTITION BY` 구문과 `ORDER BY` 구문을 조합해서 사용할 수도 있음

<details>

<summary>SQL</summary>

```sql
SELECT category,
       product_id,
       score,

       -- 카테고리별로 점수 순서로 정렬하고 유일한 순위를 붙임
       ROW_NUMBER() OVER (PARTITION BY category ORDER BY score DESC) AS row,

       -- 카테고리별로 같은 순위를 허가하고 순위를 붙임
       RANK() OVER (PARTITION BY category ORDER BY score DESC)       AS rank,

       -- 카테고리별로 같은 순위가 있을 때
       -- 같은 순위 다음에 있는 순위를 건너 뛰고 순서를 붙임
       DENSE_RANK() OVER (PARTITION BY category ORDER BY score DESC) AS dense_rank
FROM popular_products
ORDER BY category, row DESC;
```

</details>

| category | product\_id | score | row | rank | dense\_rank |
| -------- | ----------- | ----- | --- | ---- | ----------- |
| action   | A004        | 64    | 4   | 4    | 4           |
| action   | A003        | 78    | 3   | 3    | 3           |
| action   | A002        | 81    | 2   | 2    | 2           |
| action   | A001        | 94    | 1   | 1    | 1           |
| drama    | D004        | 58    | 4   | 4    | 4           |
| drama    | D003        | 78    | 3   | 3    | 3           |
| drama    | D002        | 82    | 2   | 2    | 2           |
| drama    | D001        | 90    | 1   | 1    | 1           |

#### 각 카테고리의 상위 n개 추출하기

<details>

<summary>SQL</summary>

```sql
SELECT *
-- 서브 쿼리 내부에서 순위 계산하기
FROM (SELECT category,
             product_id,
             score,
             ROW_NUMBER() OVER (PARTITION BY category ORDER BY score DESC) AS rank
      FROM popular_products
     ) AS popular_products_with_rank
-- 외부 쿼리에서 순위 활용해 압축하기
WHERE rank <= 2
ORDER BY category, rank;
```

</details>

| category | product\_id | score | rank |
| -------- | ----------- | ----- | ---- |
| action   | A001        | 94    | 1    |
| action   | A002        | 81    | 2    |
| drama    | D001        | 90    | 1    |
| drama    | D002        | 82    | 2    |

#### 각 카테고리의 최상위 추출하기

<details>

<summary>SQL</summary>

```sql
-- DISTINCT 구문을 사용해 중복 제거하기
SELECT DISTINCT category,
                -- 카테고리별로 순위 최상위 상품 ID 추출하기
                FIRST_VALUE(product_id)
                OVER (PARTITION BY category ORDER BY score DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM popular_products;
```

</details>

| category | first\_value |
| -------- | ------------ |
| drama    | D001         |
| action   | A00          |
