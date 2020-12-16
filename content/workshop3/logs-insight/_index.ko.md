---
title: "Logs Insight"
weight: 200
chapter: true
pre: "<b>6-2. </b>"
---

CloudWatch Logs Insights를 사용하면 Amazon CloudWatch Logs에서 로그 데이터를 대화식으로 검색해 분석할 수 있습니다. 운영상의 문제에 보다 효율적이고 효과적으로 대처할 수 있도록 쿼리를 수행할 수 있습니다. 문제가 발생하면 CloudWatch Logs Insights를 사용해 잠재적인 원인을 식별하고 배포된 수정 사항을 확인할 수 있습니다.

CloudWatch Logs Insights에는 간단하지만 강력한 몇 가지 명령을 포함한 특수 쿼리 언어가 포함되어 있고, CloudWatch Logs Insights는 샘플 쿼리, 명령 설명, 쿼리 자동 작성 및 로그 필드 검색 기능을 제공해 손쉽게 시작할 수 있도록 지원합니다. 여러 가지 유형의 AWS 서비스 로그에 대한 샘플 쿼리가 포함되어 있습니다.

CloudWatch Logs Insights는 AWS 서비스(예: Amazon Route 53, AWS Lambda, AWS CloudTrail 및 Amazon VPC)의 로그와 로그 이벤트를 JSON으로 출력하는 모든 애플리케이션 또는 사용자 지정 로그에서 필드를 자동으로 검색합니다.

CloudWatch Logs Insights를 사용하면 2018년 11월 5일 이후 CloudWatch Logs로 전송된 로그 데이터를 검색할 수 있습니다.

단일 요청은 최대 20개의 로그 그룹을 쿼리할 수 있습니다. 쿼리가 완료되지 않은 경우 15분 후에 쿼리가 시간 초과됩니다. 쿼리 결과는 7일 동안 사용할 수 있습니다.

생성한 쿼리를 저장할 수 있습니다. 그러면 복잡한 쿼리를 실행해야 할 때마다 다시 만들지 않고도 실행할 수 있습니다.

&nbsp;

## References
[AWS Document: Analyzing Log Data with CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)