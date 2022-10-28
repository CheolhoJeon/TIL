# 정책 및 권한

### 기본 요소

* ****[**Resource**](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/s3-arn-format.html)****
  * 버킷, 객체, 액세스 포인트 및 작업은 권한을 허용하거나 거부할 수 있는 Amazon S3 리소스
  * 정책에서는 Amazon Resource Name(ARN)을 사용하여 리소스를 식별해야함
* ****[**Action**](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/using-with-s3-actions.html)****
  * 각 리소스에 대해 여러 작업을 취할 수 있음
  * 작업 키워드를 사용하여 허용(또는 거부)할 리소스 작업을 식별
  * 예를 들어 s3:ListBucket 권한은 사용자가 GET Bucket 작업할 수 있도록 허용
* ****[**Effect**](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_elements\_effect.html)****
  * 사용자가 특정 작업을 요청하는 결과 - '허용(allow)' 또는 '거부(deny)'
  * 명시적으로 리소스에 대한 액세스 권한을 부여(허용)하지 않는 경우, 액세스는 암시적으로 거부됨
  * 리소스에 대한 액세스를 명시적으로 거불할 수도 있음
  * 다른 정책에서 액세스 권한을 부여하더라도 사용자가 해당 리소스에 액세스할 수 없도록 하려고 할 때 이러한 작업을 수행할 수 있
* ****[**Principal**](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-bucket-user-policy-specifying-principal-intro.html)**(보안주체)**
  * 작업 및 리소스에 액세스할 수 있는 계정 또는 사용자
  * 버킷 정책에서 보안 주체는 사용자, 계정, 서비스 또는 이 권한의 수신자인 기타 주체임
* ****[**Condition**](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/amazon-s3-policy-keys.html)****
  * 정책이 적용되기 위한 조건
  * AWS 전역 키와 Amazon S3 전용 키를 사용하여 Amazon S3 액세스 정책에서 조건을 지정할 수 있음

### 예제

이 정책에서는 `Account-ID` 계정의 사용자인 Dave에게 `s3:GetObject` 버킷의 `s3:GetBucketLocation`, `s3:ListBucket` 및 `awsexamplebucket1` Amazon S3 권한을 허용함

```json
{
    "Version": "2012-10-17",
    "Id": "ExamplePolicy01",
    "Statement": [
        {
            "Sid": "ExampleStatement01",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Dave"
            },
            "Action": [
                "s3:GetObject",
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::awsexamplebucket1/*",
                "arn:aws:s3:::awsexamplebucket1"
            ]
        }
    ]
}
```

