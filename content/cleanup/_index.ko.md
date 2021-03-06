---
title: "실습 리소스 정리"
weight: 1000
pre: "<b>7. </b>"
---

{{% notice warning %}}
이 실습을 마치면 사용한 AWS 계정에 비용이 추가로 발생하지 않도록 사용한 리소스를 삭제해야 합니다.
{{% /notice %}}

***

## CDK destroy

터미널 창에 다음 코드를 입력하여 cdk 앱을 삭제합니다.

```
cdk destroy --all
```

배포한 리소스들이 많기 때문에 이 작업은 약 10분 정도 시간이 소요될 수 있습니다.

자세한 진행상황을 모니터링 하고 싶다면 [CloudFormation](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false)으로 이동하여 진행상황을 체크합니다.

## Cloud9
[Cloud9 콘솔](https://us-east-2.console.aws.amazon.com/cloud9/home)로 이동하여 사용 했던 IDE 환경을 삭제 합니다.

## 두번째 계정
실습에서 사용한 두번째 계정에서도 `cdk destroy`, **Cloud9 IDE** 삭제 작업을 반복합니다.