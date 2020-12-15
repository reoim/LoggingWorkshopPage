---
title: "실습 리소스 정리"
weight: 1000
pre: "<b>7. </b>"
---

{{% notice warning %}}
이 실습을 마치면 사용한 AWS 계정에 비용이 추가로 발생하지 않도록 사용한 리소스를 삭제해야 합니다.
{{% /notice %}}

***


***
## Cloud9
[Cloud9 콘솔](https://us-east-2.console.aws.amazon.com/cloud9/home)로 이동하여 사용 했던 IDE 환경을 삭제 합니다.
![Delete resources](/images/cleanup/cloud9.png)
***
## CloudFormation
[CloudFormation](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false)으로 이동하여 cdk를 이용해 배포한 스택들을 모두 삭제합니다.
