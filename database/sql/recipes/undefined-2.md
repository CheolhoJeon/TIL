---
description: COUNT 함수, COUNT(DISTINCT ~), ROLLUP 구문, CASE 식, CAST, JOIN, GROUP BY
---

# 사용자 전체의 특징과 경향 찾기

> 사용자 마스터 테이블

| user\_id | sex | birth\_date | register\_date | register\_device | withdraw\_date |
| -------- | --- | ----------- | -------------- | ---------------- | -------------- |
| U001     | M   | 1977-06-17  | 2016-10-01     | pc               | NULL           |
| U002     | F   | 1953-06-12  | 2016-10-01     | sp               | 2016-10-10     |
| U003     | M   | 1965-01-06  | 2016-10-01     | pc               | NULL           |
| ...      | ... | ...         | ...            | ...              | ...            |
| U008     | F   | 2006-12-09  | 2016-10-10     | sp               | NULL           |
| U009     | M   | 2004-10-23  | 2016-10-15     | pc               | NULL           |
| U010     | F   | 1987-03-18  | 2016-10-16     | pc               | NULL           |

> 액션 로그 테이블

| session  | user\_id | action    | category | products  | amount | stamp               |
| -------- | -------- | --------- | -------- | --------- | ------ | ------------------- |
| 989004ea | U001     | purchase  | drama    | D001,D002 | 2000   | 2016-11-03 18:10:00 |
| 989004ea | U001     | view      | NULL     | NULL      | NULL   | 2016-11-03 18:00:00 |
| 989004ea | U001     | favorite  | drama    | D001      | NULL   | 2016-11-03 18:00:00 |
| 989004ea | U001     | review    | drama    | D001      | NULL   | 2016-11-03 18:00:00 |
| ...      | ...      | ...       | ...      | ...       | ...    | ...                 |
| 87b5725f | U001     | add\_cart | action   | A005      | NULL   | 2016-11-04 12:00:00 |
| 87b5725f | U001     | add\_cart | action   | A006      | NULL   | 2016-11-04 12:00:00 |
| 9afaf87c | U002     | purchase  | drama    | D002      | 1000   | 2016-11-04 13:00:00 |
| 9afaf87c | U001     | purchase  | action   | A005,A006 | 1000   | 2016-11-04 15:00:00 |

## 1. 사용자의 액션 수 집계하기

### 1-1. 액션과 관련된 지표 집계하기

{% hint style="info" %}
**UU(Unique Users):** 중복 없이 집계된 사용자 수&#x20;

**Usage Rate:** 특정 액션 UU를 전체 액션 UU로 나눈 것
{% endhint %}

<details>

<summary>SQL</summary>

```sql
WITH stats AS (
    -- 로그 전체의 UU 구하기
    SELECT COUNT(DISTINCT session) AS total_UU
    FROM action_log
)
SELECT l.action,
       -- 액션 UU
       COUNT(DISTINCT session)                    AS action_uu,
       -- 액션의 수
       COUNT(*)                                   AS acount_count,
       -- 전체 UU
       s.total_UU                                 AS total_uu,
       -- 사용률: <액션 UU> / <전체 UU>
       100.0 * COUNT(DISTINCT session) / total_UU AS usage_rate,
       -- 1인당 액션 수: <액션 수> / <액션 UU>
       1.0 * COUNT(*) / COUNT(DISTINCT session)   AS count_per_user
FROM action_log AS l
         CROSS JOIN stats AS s
GROUP BY l.action, s.total_UU;
```

</details>

| action    | action\_uu | acount\_count | total\_uu | usage\_rate | count\_per\_user |
| --------- | ---------- | ------------- | --------- | ----------- | ---------------- |
| add\_cart | 3          | 12            | 4         | 75          | 4                |
| favorite  | 1          | 1             | 4         | 25          | 1                |
| purchase  | 3          | 5             | 4         | 75          | 1.67             |
| review    | 1          | 1             | 4         | 25          | 1                |
| view      | 1          | 1             | 4         | 25          | 1                |

### 1-2. 로그인 사용자와 비로그인 사용자를 구분해서 집계하기

{% hint style="info" %}
샘플 데이터에는 로그인하지 않은 사용자가 없어 결과 데이터에 드러나지 않음
{% endhint %}

<details>

<summary>SQL</summary>

```sql
WITH action_log_with_status AS (
    SELECT session,
           user_id,
           action,
           -- user_id가 NULL 또는 빈 문자가 아닌 경우 login이라고 판정하기
           CASE WHEN COALESCE(user_id, '') <> '' THEN 'login' ELSE 'guest' END AS login_status
    FROM action_log
)
SELECT COALESCE(action, 'all')       AS action,
       COALESCE(login_status, 'all') AS login_status,
       COUNT(DISTINCT session)       AS action_uu,
       COUNT(*)                      AS action_count
FROM action_log_with_status
GROUP BY ROLLUP (action, login_status)
;
```

</details>

| action    | login\_status | action\_uu | action\_count |
| --------- | ------------- | ---------- | ------------- |
| add\_cart | login         | 3          | 12            |
| add\_cart | all           | 3          | 12            |
| favorite  | login         | 1          | 1             |
| favorite  | all           | 1          | 1             |
| purchase  | login         | 3          | 5             |
| purchase  | all           | 3          | 5             |
| review    | login         | 1          | 1             |
| review    | all           | 1          | 1             |
| view      | login         | 1          | 1             |
| view      | all           | 1          | 1             |
| all       | all           | 4          | 20            |

### 1-3. 회원과 비회원을 구분해서 집계하기

<details>

<summary>SQL</summary>

```sql
WITH action_log_with_status AS (
    SELECT session,
           user_id,
           action,
           CASE
               WHEN COALESCE(MAX(user_id)
                             OVER (PARTITION BY session ORDER BY stamp
                                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW),
                             '') <> '' THEN 'member'
               ELSE 'none' END AS member_status,
           stamp               AS stamp
    FROM action_log
)
SELECT *
FROM action_log_with_status
;
```

</details>

| session  | user\_id | action    | member\_status | stamp               |
| -------- | -------- | --------- | -------------- | ------------------- |
| 47db0370 | U002     | add\_cart | member         | 2016-11-03 19:00:00 |
| 47db0370 | U002     | purchase  | member         | 2016-11-03 20:00:00 |
| 47db0370 | U002     | add\_cart | member         | 2016-11-03 20:30:00 |
| 87b5725f | U001     | add\_cart | member         | 2016-11-04 12:00:00 |
| ...      | ...      | ...       | ...            | ...                 |
| 989004ea | U001     | purchase  | member         | 2016-11-03 18:10:00 |
| 989004ea | U001     | purchase  | member         | 2016-11-03 18:10:00 |
| 9afaf87c | U002     | purchase  | member         | 2016-11-04 13:00:00 |
| 9afaf87c | U001     | purchase  | member         | 2016-11-04 15:00:00 |

## 2. 연령별 구분 집계하기

> 연령별 구분 목록

| 연령별 구분 | 설명             |
| ------ | -------------- |
| C      | 4\~12세 남성과 여성  |
| T      | 13\~19세 남성과 여성 |
| M1     | 20\~34세의 남성    |
| M2     | 35\~49세의 남성    |
| M3     | 50세 이상의 남성     |
| F1     | 20\~34세의 여성    |
| F2     | 35\~49세의 여성    |
| F3     | 50세 이상의 여성     |

<details>

<summary>SQL</summary>

```sql
WITH mst_users_with_int_birth_date AS (
    SELECT *,
           20220223                                                        AS int_specific_date,
           CAST(REPLACE(SUBSTRING(birth_date, 1, 10), '-', '') AS integer) AS int_birth_date
    FROM mst_users
),
     mst_users_with_age AS (
         SELECT *,
                FLOOR((int_specific_date - int_birth_date) / 10000) AS age
         FROM mst_users_with_int_birth_date
     ),
     mst_users_with_category AS (
         SELECT user_id,
                sex,
                age,
                CONCAT(
                        CASE WHEN age <= 20 THEN '' ELSE sex END,
                        CASE
                            WHEN age BETWEEN 4 AND 12 THEN 'C'
                            WHEN age BETWEEN 13 AND 19 THEN 'T'
                            WHEN age BETWEEN 20 AND 34 THEN '1'
                            WHEN age BETWEEN 35 AND 49 THEN '2'
                            WHEN age >= 50 THEN '3'
                        END
                    ) AS category
         FROM mst_users_with_age
     )
SELECT category,
       COUNT(*) AS user_count
FROM mst_users_with_category
GROUP BY category
ORDER BY category;
```

</details>

| category | user\_count |
| -------- | ----------- |
| F1       | 1           |
| F3       | 4           |
| M1       | 1           |
| M2       | 1           |
| M3       | 1           |
| T        | 2           |

## 3. 연령별 구분의 특징 추출하기

<details>

<summary>SQL</summary>

```sql
WITH mst_users_with_int_birth_date AS (
    // '2.연령별 구분 집계하기'와 동일
),
     mst_users_with_age AS (
         // '2.연령별 구분 집계하기'와 동일
     )
SELECT p.category AS product_category,
       u.category AS user_category,
       COUNT(*)   AS purchase_count
FROM mst_users_with_category u
         JOIN action_log p ON u.user_id = p.user_id
WHERE action = 'purchase'
GROUP BY p.category, u.category;
```

</details>

| product\_category | user\_category | purchase\_count |
| ----------------- | -------------- | --------------- |
| action            | M2             | 1               |
| drama             | F3             | 2               |
| drama             | M2             | 2               |

