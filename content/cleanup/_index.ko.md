---
title: "실습 리소스 정리"
weight: 1000
pre: "<b>   </b>"
---

{{% notice warning %}}
이 실습을 마치면 사용한 AWS 계정에 비용이 추가로 발생하지 않도록 사용한 리소스를 삭제해야 합니다.
{{% /notice %}}

***
## QuickSight
**QuickSight**는 첫 가입시 60일간은 free trial로 사용 할 수 있습니다. 만약 가입 취소를 원하면 [Unsubscribe](#unsubscribe)를 진행 합니다.
### 리소스 삭제
다음과 같이 Analyses와 Dataset을 삭제 합니다.
![Delete resources](/images/cleanup/quicksight-delete1.png)
![Delete resources](/images/cleanup/quicksight-delete2.png)
### Unsubscribe
가입 취소를 원하시면 다음과 같이 **Admin** -> **Account settings** -> **Unsubscribe** 선택 하여 가입 취소를 할 수 있습니다.
![Delete resources](/images/cleanup/unsubscribe.png)

***
## S3
[S3 콘솔](https://s3.console.aws.amazon.com/s3/home?region=ap-northeast-2#)으로 이동하여 실습에서 만든 3개의 **S3 Bucket**들을 모두 삭제 합니다. 

**3-1 실습환경 구축**에서 생성한 S3, **4-2 MLOps 파이프라인 구축**에서 생성한 AthenaBucket과 ForecastBucket을 삭제 합니다.
![Delete resources](/images/cleanup/s3.png)

*** 
## Glue
[Glue 콘솔](https://ap-northeast-2.console.aws.amazon.com/glue/home?region=ap-northeast-2)에서 생성한 **Database**을 삭제 합니다.
![Delete resources](/images/cleanup/glue_db.png)
**Crawler**을 삭제 합니다.
![Delete resources](/images/cleanup/glue_crawler.png)
**Job**을 삭제 합니다.
![Delete resources](/images/cleanup/glue_job.png)

***
## IAM
[IAM 콘솔](https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/roles)로 이동하여 **Glue** job에 사용한 role을 삭제 합니다.
![Delete resources](/images/cleanup/iam.png)


***
## Cloud9
[Cloud9 콘솔](https://ap-northeast-2.console.aws.amazon.com/cloud9/home)로 이동하여 사용 했던 IDE 환경을 삭제 합니다.
![Delete resources](/images/cleanup/cloud9.png)
***
## CloudFormation
[CloudFormation](https://ap-northeast-2.console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false)으로 이동하여 **SAM template**로 배포한 stack을 삭제 합니다.
![Delete resources](/images/cleanup/cf.png)