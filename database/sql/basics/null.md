---
description: COALESCE 함수
---

# NULL 값을 디폴트 값으로 대치하기

> 쿠폰 사용 여부가 함께 있는 구매 로그(purchase\_log\_with\_coupon) 테이블

| purchase\_id | amount | coupon |
| ------------ | ------ | ------ |
| 10001        | 3280   | null   |
| 10002        | 4650   | 500    |
| 10003        | 3870   | null   |

<details>

<summary>SQL</summary>

```sql
SELECT purchase_id,
       amount,
       coupon,
       amount - coupon              AS discount_amount1,
       amount - COALESCE(coupon, 0) AS discount_amount2
FROM purchase_log_with_coupon
;
```

</details>

| purchase\_id | amount | coupon | discount\_amount1 | discount\_amount2 |
| ------------ | ------ | ------ | ----------------- | ----------------- |
| 10001        | 3280   | null   | null              | 3280              |
| 10002        | 4650   | 500    | 4150              | 4150              |
| 10003        | 3870   | null   | null              | 3870              |
