# 버킷

* Amazon S3에 데이터(사진, 동영상, 무서 등)를 업로드하려면 우선 하나의 AWS 리전에 S3 버킷을 만들어야 함
* <mark style="color:blue;">**버킷은 Amazon S3에 저장된 객체를 관리하는 컨테이너**</mark>
* **버킷에 저장할 수 있는 객체 수에는 제한이 없음**
* 또한 계정에 버킷을 최대 100개까지 포함할 수 있으며, AWS에 요청하면 최대치를 증가시킬 수도 있음



* 모든 객체는 어떤 버킷에 포함됨
* 예를 들어 `photos/puppy.jpg`로 명명된 객체는 미국 서부(오레곤) 리전의 `DOC-EXAMPLE-BUCKET` 버킷에 저장되며 URL `https://DOC-EXAMPLE-BUCKET.s3.us-west-2.amazonaws.com/photos/puppy.jpg`를 사용하여 주소를 지정할 수 있음
* 자세한 내용은 [버킷 액세스](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/access-bucket-intro.html)를 참조



* <mark style="color:blue;">**구현 측면에서 버킷과 객체는 AWS 리소스에 해당되며 Amazon S3는 이를 관리하기 위한 API를 제공함**</mark>
* 예를 들면 Amazon S3 API를 사용하여 버킷을 만들고 객체를 업로드할 수 있음
* 이러한 작업은 Amazon S3 콘솔을 사용하여 수행할 수도 있음
* 콘솔은 Amazon S3 API를 사용하여 요청을 Amazon S3로 보냄

### 권한

* AWS 계졍 루트 사용자 자격 증명을 사용하여 버킷을 만들고 기타 Amazon S3 작업을 수행할 수 있음
* <mark style="color:blue;">**하지만 버킷 생성 등의 요청을 할 때는 AWS 계정의 루트 사용자 자격 증명을 사용하지 않는 것이 좋음**</mark>
* 대신 AWS Identity and Access Management(IAM) 사용자를 만들고 이 사용자에게 모든 액세스 권한을 부여함(기본적으로 사용자는 권한이 없음)
* 자세한 내용은 _AWS 일반 참조_의 [AWS 계정 루트 사용자 자격 증명 및 IAM 사용자 자격 증명](https://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html)과 _IAM 사용 설명서_의 [IAM 보안 모범 사례](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)를 참조



* **리소스를 만든 AWS 계정은 해당 리소스의 소유자가 됨**
* 예를 들어, 자신의 AWS 계정에 IAM 사용자를 만들고 버킷을 만들 수 있는 사용자 권한을 부여하면, 이 사용자가 버킷을 만들 수 있음
* <mark style="color:blue;">**하지만 사용자는 버킷을 소유하지 않으며, 이 사용자가 속한 AWS 계정에서 버킷을 소유함**</mark>
* 사용자가 다른 버킷 작업을 수행하려면 리소스 소유자로부터 추가 권한을 받아야 함
* Amazon S3 리소스 권한 관리에 대한 자세한 내용은 [Amazon S3의 Identity and Access Management](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/s3-access-control.html) 단원을 참조

### 버킷 구성 옵션



