---
title: "실습 2, 멀티 계정 중앙 로깅"
chapter: true
weight: 50
pre: "<b>5. </b>"
---



많은 기업에서 보안, 분석, 모니터링 등의 이유로 중앙 집중 로깅을 고려합니다.

여러 부서, 팀에서 관리되는 멀티 계정 환경에서는 중앙에서 로그정보를 수집하는 로깅 전담 계정이 있는것이 모범사례 입니다.

이를 통해 보안팀 입장에서는 실시간으로 위험한 행위를 탐지하고 침해에 대처하는데 도움을 받을 수 있게 됩니다.

또한 로그 데이터가 사고나 혹은 의도적으로 지워질 경우도 방지할 수 있습니다. 애플리케이션 운영팀 입장에서도 여러 갱의 애플리케이션 티어 상에서 로그 데이터를 분석하고 연관짓는데 도움을 받을 수 있습니다.

이번 실습에서는 멀티 계정의 CloudWatch 로그 그룹에 적재되는 로그들을 중앙 로그 버킷에 전송하는 인프라를 만들어볼 것입니다.

{{% notice info %}}
이 실습을 시작하기 위해서는 총 두개의 AWS 계정, 또는 Event Engine 계정이 필요합니다. 편의상 중앙 로그 버킷이 위치할 첫번째 계정 ID를 `111111111111`, 로그를 생성하고 전송할 두번째 계정 ID를 `222222222222` 라고 가정합니다.
{{% /notice %}}

&nbsp;

실습은 다음과 같은 순서로 진행 됩니다.

{{% children showhidden="false" %}}

&nbsp;

## References
[Blog: Centralize Amazon CloudWatch Logs using AWS CDK](https://aws.amazon.com/ko/blogs/developer/build-infrastructure-for-centralized-logging-using-aws-cdk/)

[Blog: AWS 멀티 어카운트 환경을 위한 통합 로깅 방법](https://aws.amazon.com/ko/blogs/korea/central-logging-in-multi-account-environments/)

[Blog: Central Logging in Multi-Account Environments](https://awsfeed.com/uncategorized/central-logging-in-multi-account-environments)

[AWS Document: Cross-Account Log Data Sharing with Subscriptions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CrossAccountSubscriptions.html)
