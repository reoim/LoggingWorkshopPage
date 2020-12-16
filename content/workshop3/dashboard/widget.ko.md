---
title: "위젯 추가"
weight: 420
pre: "<b>6-4.b. </b>"
---
***

대시보드에 다양한 위젯을 추가할 수 있습니다.

이 페이지에서는 로그 데이터와 몇가지 지표들로 위젯을 만들어 볼 것입니다.

## 로그 테이블

최근에 발생한 에러 로그들, 혹은 특정 쿼리로 반한 된 로그 데이터들을 그룹별로 대시보드에서 게시하려 합니다. 어떻게 할까요?

`로그 테이블` 위젯을 만들어 대시보드에 추가할 수 있습니다.

먼저 `Logs Insight` 콘솔 에서 원하는 쿼리를 작성합니다.

다음 예시는 최근 12시간 동안 `Exception`이나 `Error` 라는 단어를 포함한 CloudTrail 로그를 검색하는 쿼리 입니다. 

filter 구문에는 정규식을 사용했으며 `(?i)`는 대소문자를 구별하지 않는다는 의미 입니다. 

```
fields awsRegion, @timestamp, @message
| sort @timestamp desc
| limit 50
| filter @message like /(?i)Exception/ or @message like /(?i)Error/
```

쿼리를 실행한 후 `Add to dashboard` 버튼을 클릭 합니다.

![widget](/images/workshop3/log-table.png)

&nbsp;

대시보드를 선택하고 위젯 타입은 `Logs table`을 선택 합니다.

위젯 이름을 설정하고 `Add to dashboard` 버튼을 클릭 합니다.

![widget](/images/workshop3/log-table2.png)

&nbsp;

## 지표 위젯

다양한 지표들을 그래프 위젯으로 대시보드에 추가할 수 있습니다.

[실습 1, CDK로 로깅 인프라 구축 - Unified CloudWatch Agent](../../../workshop1/webserver/cwagent) 에서 추가한 CloudWatch agent 지표를 대시보드에 추가 해봅니다.

Metrics 메뉴에서 `CWAgent` 커스텀 네임스페이스를 클릭 합니다.

![metric](/images/workshop3/metric1.png)

`mem_used_percent` 지표를 선택하고 대시보드에 추가합니다.

![metric](/images/workshop3/metric2.png)

대시보드 이름과 라인 그래프 타입을 선택하고 대시보드에 추가합니다.

![metric](/images/workshop3/metric3.png)

대시보드에 위젯이 추가 되었습니다.

![metric](/images/workshop3/metric4.png)