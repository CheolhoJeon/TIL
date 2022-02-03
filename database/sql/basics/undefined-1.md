---
description: CONCAT 함수, || 연산자
---

# 문자열 다루기

## 1. 문자열 연결하기

> 사용자의 주소 정보 테이블

| user\_id | pref\_name | city\_name |
| -------- | ---------- | ---------- |
| U001     | 서울특별시      | 강서구        |
| U002     | 경기도수원시     | 장안구        |
| U003     | 제주도특별자치도   | 서귀포시       |

<details>

<summary>SQL</summary>

```sql
SELECT user_id,
       concat(pref_name, city_name) AS pref_city1,
       pref_name || city_name       AS pref_city2
FROM mst_user_location;
```

</details>

| user\_id | pref\_city1  | pref\_city2  |
| -------- | ------------ | ------------ |
| U001     | 서울특별시강서구     | 서울특별시강서구     |
| U002     | 경기도수원시장안구    | 경기도수원시장안구    |
| U003     | 제주도특별자치도서귀포시 | 제주도특별자치도서귀포시 |
