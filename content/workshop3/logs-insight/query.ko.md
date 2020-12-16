---
title: "로그 쿼리"
weight: 210
---
***

## 쿼리 실행

[CloudWatch Logs Insight 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#logsV2:logs-insights)로 이동하여 `TrailLog`로그 그룹을 선택합니다.

![Logs Insight - TrailLog](/images/workshop3/trail-log.png)

`Run query` 버튼을 클릭하여 샘플 쿼리를 실행 합니다.

다음과 같은 결과를 확인할 수 있습니다.

![Logs Insight - TrailLog](/images/workshop3/query1.png)

시간에 따른 로그 이벤트 분포 그래프와 가장 최근 20개의 로그 메세지를 확인 할 수 있습니다.

&nbsp;

## 필드 검색

`Logs Insight`는 로그 안의 로그 필드 검색 기능을 지원 합니다. AWS 서비스 로그들(VPC flow 로그, CloudTrail 로그 등)과 Json 포멧의 로그들은 자동으로 필드 검색이 되며 이밖의 타입들의 로그들도 기본적으로 `@timestamp, @ingestionTime, @logStream, @message, @log` 의 시스템 필드가 생성되며 로그 내용은 `@message` 필드로 검색됩니다. 

필드 검색에 대한 좀 더 자세한 내용은 [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_AnalyzeLogData-discoverable-fields.html)를 참고하시길 바랍니다.

&nbsp;

`Logs Insight` 콘솔 우측의 `Fields` 버튼을 클릭합니다.

![Logs Insight Fields](/images/workshop3/field.png)

검색된 필드를 샘플 쿼리에 추가하여 실행해 봅시다.

샘플 쿼리를 다음과 같이 수정 후 실행합니다.

```
fields awsRegion, @timestamp, @message
| sort @timestamp desc
| limit 20
```

![query](/images/workshop3/query-region1.png)

## 필터 기능

필터 기능을 사용하여 us-east-2 리전에서 발생한 CloudTrail 로그만을 쿼리 해봅니다.

아래 쿼리를 복사하여 붙여넣고 실행 해봅니다.

```
fields awsRegion, @timestamp, @message
| sort @timestamp desc
| limit 20
| filter awsRegion="us-east-2"
```

![query](/images/workshop3/filter.png)

## 집계 기능

쿼리 에디터에 다음 쿼리를 복사해서 붙여 넣고 실행 합니다.

CloudTrail 로그 중 이벤트 소스와, 이벤트 이름, 리전 별로 집계한 숫자를 나타내는 쿼리 입니다.

```
stats count(*) by eventSource, eventName, awsRegion
```

![query](/images/workshop3/aggregation.png)

Visualization 탭을 이용하여 집계 결과를 그래프로 확인할 수도 있습니다.

![pie](/images/workshop3/pie.png)

## 자동 완성 기능

쿼리에디터의 자동 완성 기능으로 필드 및 명령어를 쉽게 선택할 수 있습니다.

![Auto complete](/images/workshop3/auto.png)

## 샘플 쿼리

`Logs Insight` 콘솔 우측에 `Queries` 버튼을 클릭하면 다양한 샘플 쿼리들을 확인 할 수 있습니다.

로그 그룹을 선택하고 `Lambda`와 `VPC Flow Logs`, `CloudTrail` 등 샘플 쿼리들을 실행 해봅니다.

![sample query](/images/workshop3/sample.png)

쿼리 문법에 대한 자세한 내용은 [CloudWatch Logs Insights 쿼리 구문](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)을 참고 하시길 바랍니다.
