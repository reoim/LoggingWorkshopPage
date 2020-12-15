---
title: "실습 1, CDK로 로깅 인프라 구축"
chapter: true
weight: 40
pre: "<b>4. </b>"
---

이번 실습에서는 `CDK`를 이용하여 다양한 데모 워크로드들을 AWS 위에 배포하고 

서비스를 운영하며 발생하는 여러가지 로그들을 `CloudWatch Log`로 통합 관리하는 인프라를 구성 해봅니다.

또한, 비실시간성 분석 및 보관을 위해 `S3 Bucket`에 로그를 저장하는 로깅 인프라도 구성 해볼 것입니다.

`CloudWatch Log`보다 `S3`의 보관비용이 더 저렴하고 보관 용도에 따라 [Standard-Infrequent Access](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html#sc-infreq-data-access), [Glacier](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html#sc-glacier)등 다양한 스토리지 클래스 선택이 가능하기 때문에, 실시간 분석이 필요한 로그들을 일정시간 `CloudWatch Log`에 보관한 뒤 `S3`로 내보내는 것이 모범사례 입니다.

실습의 편의를 위해서 하나의 로그 버킷으로 Standard 클래스를 사용할 것이지만 로그 타입별로 버킷을 분리하거나, 버킷의 prefix 별로 분리하여 서로 다른 스토리지 클래스를 적용하실 수도 있습니다.

`CloudTrail 로그`, `Vpc flow 로그` 등 몇몇 AWS 서비스들은 직접 S3로 로그를 내보낼 수 있습니다.

`CloudWatch Log`에서 S3로 로그를 내보내는 방법은 다음 링크를 참고하시길 바랍니다.

[Amazon S3로 로그 데이터 내보내기](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/S3Export.html)

&nbsp;

실습 1 은 다음과 같은 순서로 진행 됩니다.

{{% children showhidden="false" %}}

