---
title: "Log subscription filter로 중앙 로깅 설정"
chapter: true
weight: 200
pre: "<b>5-2. </b>"
---

{{% notice info %}}
AWS 콘솔에서 로그아웃 한 뒤, 로그를 생성할 두번째 계정으로 로그인 합니다.
{{% /notice %}}

이번 실습에서는 `CloudWatch Log`에 저장된 로그들을 [Subscription Filter](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/SubscriptionFilters.html) 기능을 이용하여 로그 버킷 계정의 Log destination으로 로그를 전송 할 것입니다.

샘플 로그는 간단한 서버리스 워크로드를 배포하여 발생하는 API gateway 액세스 로그를 사용할 것입니다.