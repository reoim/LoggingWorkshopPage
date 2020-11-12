---
title: "Stack 구성하기"
weight: 300
pre: "<b>4. </b>"
---
***
이 챕터에서는 AWS에서 발생하는 여러가지 로그들을 CloudWatch Log에 수집하여 분석, 관리, 모니터링 하고 여러 로그들을 중앙 S3 bucket에 수집하는 구성을 CDK를 이용하여 실습 할 것 입니다.


## 패키지 설치
실습에 필요한 패키지들을 설치 합니다.

아래와 같이 새로운 터미널 창을 하나 추가 합니다.
![New terminal window](/images/log-stack/new_term.png)
{{% notice warning %}}
**npm run watch** 명령어가 실행 되어 있는 터미널 창을 닫지 마세요.
{{% /notice %}}


새 터미널 창에 다음 명령어를 입력하여 프로젝트 폴더로 이동 합니다.
```bash
cd logging-workshop
```

터미널 창에 다음 명령어를 입력하여 실습에 필요한 패키지들을 설치 합니다.
```bash
npm i @aws-cdk/aws-s3@$AWS_CDK_VERSION \
@aws-cdk/aws-ec2@$AWS_CDK_VERSION \
@aws-cdk/aws-sns@$AWS_CDK_VERSION \
@aws-cdk/aws-sns-subscriptions@$AWS_CDK_VERSION \
@aws-cdk/aws-cloudtrail@$AWS_CDK_VERSION \
@aws-cdk/aws-cloudwatch-actions@$AWS_CDK_VERSION \
@aws-cdk/aws-apigateway@$AWS_CDK_VERSION
```

정상적으로 설치 되면 다음과 비슷한 결과 메세지를 확인 할 수 있습니다.
```term
+ @aws-cdk/aws-apigateway@1.72.0
+ @aws-cdk/aws-sns-subscriptions@1.72.0
+ @aws-cdk/aws-cloudwatch-actions@1.72.0
+ @aws-cdk/aws-sns@1.72.0
+ @aws-cdk/aws-ec2@1.72.0
+ @aws-cdk/aws-s3@1.72.0
+ @aws-cdk/aws-cloudtrail@1.72.0
added 14 packages from 1 contributor, updated 3 packages and audited 767 packages in 10.964s
```

패키지 설치시 발생하는 **Warning** 메세지들은 무시하셔도 좋습니다.