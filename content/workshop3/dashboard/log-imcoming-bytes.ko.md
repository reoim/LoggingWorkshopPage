---
title: "로그 수집량 모니터링"
weight: 430
pre: "<b>6-4.c. </b>"
---
***

특정 로그 그룹에서 생각지 못한 많은 로그 데이터가 수집 된다면 `CloudWatch Logs` 청구 금액이 증가될 수 있습니다.

이런 상황을 미리 알고 방지하기 위해 로그 데이터 수집량을 모니터 할 필요가 있습니다.

## 로그 수집량 모니터링 위젯 만들기

먼저 앞 실습에서 위젯 만들기에서 사용한 지표를 해제합니다.

![incomingBytes](/images/workshop3/log-bill-1.png)

&nbsp;

`Logs` 네임스페이스를 선택합니다.

![incomingBytes](/images/workshop3/log-bill-2.png)

&nbsp;

`Log Group Metrics`를 선택합니다.

![incomingBytes](/images/workshop3/log-bill-3.png)

&nbsp;

모니터링할 로그 그룹들의 `IncomingBytes` 지표들을 선택합니다.

![incomingBytes](/images/workshop3/log-bill-4.png)

&nbsp;

`Graphed metrics` 탭을 클릭하고 `Statistic`을 `Sum`으로 변경합니다.

`Period`는 `30 Days`로 변경합니다.

![incomingBytes](/images/workshop3/log-bill-5.png)

&nbsp;

`Graph options` 탭을 클릭하고 Widget type을 `Number`로 선택합니다. 

우측 상단에 `custom`을 클릭하고 `Absolute`를 선택 후 현재 날짜부터 지난 30일까지의 날짜 범위를 지정합니다.

![incomingBytes](/images/workshop3/log-bill-6.png)

&nbsp;

`Add to dashbaord`를 클릭합니다.

![incomingBytes](/images/workshop3/log-bill-7.png)

&nbsp;

위젯 이름을 `LogGroups IncomingBytes`로 변경하고 대시보드에 추가합니다.

![incomingBytes](/images/workshop3/log-bill-8.png)

&nbsp;

대시보드에 정상적으로 추가 되었습니다.

![incomingBytes](/images/workshop3/log-bill-9.png)