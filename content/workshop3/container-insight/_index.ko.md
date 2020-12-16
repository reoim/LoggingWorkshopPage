---
title: "Container Insight"
weight: 300
chapter: true
pre: "<b>6-3. </b>"
---

CloudWatch Container Insights는 컨테이너화된 응용 프로그램이나 마이크로서비스에서 메트릭 및 로그를 수집하여 집계, 요약합니다. 메트릭은 CPU, 메모리, 디스크, 네트워크 등의 리소스 사용률이 포함됩니다. 메트릭은 CloudWatch 대시 보드에서도 사용할 수 있습니다. 
Container Insights에서 사용할 수 있는 전체 메트릭 목록은 Aamazon CloudWatch User Guide의 [Amazon ECS Container Insights Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-ECS.html) 를 확인하세요.

운영 데이터로 성능 로그 이벤트가 수집됩니다. 카디널리티가 높은 데이터를 대규모로 수집하고 저장할 수 있도록 구조화 된 JSON 스키마를 사용합니다. CloudWatch는 이 데이터들, 클러스터(cluster) 및 서비스 레벨(service level)에서 집계 된 메트릭을 이용하여 더 높은 수준의 CloudWatch 메트릭을 생성합니다. 더 많은 정보가 필요하면 Amazon CloudWatch User Guide의 [Container Insights Structured Logs for Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)를 확인하세요.

&nbsp;

## References
[User Guide: Amazon ECS CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-container-insights.html)