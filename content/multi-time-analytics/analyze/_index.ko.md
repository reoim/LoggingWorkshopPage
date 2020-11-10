---
date: 2020-09-2
title: "예측 데이터 분석 및 시각화"
chapter: false
weight: 250
pre: "<b>4-5 </b>"
---


이번 장 에서는 S3에 저장되어 있는 history data(학습에 쓰인 과거 데이터)와 forecast data(Amazon Forecast로 생성한 예측 데이터)를 **Athena**로 쿼리하여 Data set을 생성하고 **QuickSight**로 시각화 하는 작업을 할 것 입니다.

{{% notice note %}}
[Amazon Athena](https://aws.amazon.com/ko/athena/?nc1=h_ls&whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)는 표준 SQL을 사용해 Amazon S3에 저장된 데이터를 간편하게 분석할 수 있는 대화식 쿼리 서비스입니다. Athena는 서버리스 서비스이므로 관리할 인프라가 없으며 실행한 쿼리에 대해서만 비용을 지불하면 됩니다.
Athena는 사용이 쉽습니다. Amazon S3에 저장된 데이터를 가리키고 스키마를 정의한 후 표준 SQL을 사용하여 쿼리를 시작하기만 하면 됩니다. 

[Amazon Quicksight](https://aws.amazon.com/ko/quicksight/?nc2=type_a)는 조직 내 모든 구성원에게 통찰력을 손쉽게 제공할 수 있게 지원하는 빠른 클라우드 기반 BI(Business Intelligence) 서비스입니다. 완전관리형 서비스인 QuickSight를 사용하면 ML Insights가 포함된 대화형 대시보드를 손쉽게 생성 및 게시할 수 있습니다.
{{% /notice %}}

***
### QuickSight 가입
콘솔 서비스 창에서 QuickSight를 검색하여 QuickSight 콘솔로 이동 합니다.
![QuickSight](/images/mlops/quicksight.png)

실습 진행을 위해 **QuickSight** 가입이 필요 합니다.

이미 가입을 하신 분은 [Permissions](#permissions) 섹션부터 진행 하시길 바랍니다.
![Signup](/images/mlops/sign-up.png)

Standard 플랜을 선택 후 다음 단계로 진행 합니다.
![Signup](/images/mlops/standard.png)

**Region**은 **서울 region**을 선택하고 **QuickSight account name, email 주소**를 입력 후 **Finish** 버튼을 클릭 합니다.
![Signup](/images/mlops/signup3.png)

***
### Permissions
{{% notice info %}}
**QuickSight**가 데이터가 저장 된 **S3**와 **Athena**로 쿼리하여 만든 데이터 셋에 접근 할 수 있도록 권한을 추가 해줘야 합니다.
{{% /notice %}}
**QuickSight** 콘솔 우측 상단의 유저 버튼을 클릭하여 **Manage QuickSight** 메뉴를 클릭 합니다.

**Security & permissions** 메뉴를 클릭 후, **Add or remove** 버튼을 클릭 합니다.
![Permissions](/images/mlops/permission.png)

**Amazon S3** 의 **Details** 를 클릭 후 **Select S3 buckets**를 클릭 합니다.
![Permissions](/images/mlops/permission2.png)

**CloudFormation**으로 생성 된 **AthenaBucket**과 **ForecastBucket**을 선택하고 **Write permission for Athena Workgroup** 체크박스도 체크 한 뒤 **Finish** 버튼을 클릭 합니다.
![Permissions](/images/mlops/permission3.png)

아래와 같이 잘 설정 되었는지 확인 후 **Update** 버튼을 클릭 합니다.
![Permissions](/images/mlops/permission4.png)

***
### Data set 생성
**QuickSight** 콘솔에서 **New Analysis** 버튼을 클릭 합니다.
![New Analaysis](/images/mlops/new-analysis.png)

**New Dataset** 버튼을 클릭 합니다.
![New Dataset](/images/mlops/new-dataset.png)

Data Source로 **Athena**를 선택 합니다.
![Dataset Athena](/images/mlops/dataset-athena.png)

**Data source name**을 입력하고 **Athena workgroup**은 **ForecastGroup**을 선택 후 **Validate connection**을 테스트 합니다.
![Dataset Athena](/images/mlops/athena1.png)

**Connection** 테스트가 정상이면 **Create data source** 버튼을 클릭 합니다.
![Dataset Athena](/images/mlops/athena2.png)

**Use custom SQL**을 클릭 합니다.
![Dataset Athena](/images/mlops/customsql.png)

다음 SQL 코드를 복사하여 입력 합니다. 학습에 쓰인 history 데이터와 forecast(예측) 데이터를 union 하는 쿼리 입니다.
```sql
SELECT LOWER(forecast.item_id) as item_id,
         forecast.target_value,
         date_parse(forecast.timestamp, '%Y-%m-%d') as timestamp,
         forecast.type
FROM default.forecast
UNION ALL
SELECT LOWER(train.item_id) as item_id,
         train.target_value,
         date_parse(train.timestamp, '%Y-%m-%d') as timestamp,
         'history' as type
FROM default.train
```
**Confirm query**를 클릭 하여 다음 단계를 진행 합니다.
![Dataset Athena](/images/mlops/customsql2.png)

데이터를 바로 쿼리하거나, **SPICE**로 import해서 사용 하는 옵션을 선택 할 수 있습니다.

**SPICE**는 **Amazon QuickSight**의 매우 빠른 병렬 인 메모리 컴퓨팅 엔진으로서 신속하게 고급 계산을 수행하고 데이터를 처리하도록 설계 되었습니다.

원하는 옵션을 선택 후 **Visualize** 버튼을 클릭 합니다.
![Dataset Athena](/images/mlops/spice.png)

***
### Visualize

**SPICE** 옵션을 선택 하였다면 **SPICE**로 데이터를 import 하는 시간이 조금 걸립니다. 몇 분 뒤 import가 완료 된 것을 확인 할 수 있습니다.
![Visualize](/images/mlops/spice2.png)

좌측 하단의 **Visual types** 메뉴에서 **Line chart** 를 선택 합니다.

**Fields List**의 **Field**를 선택하여 **Line chart**로 drag & drop 할수 있습니다.

**timestamp**는 **X axis**로, **target_value**는 **Value**로 **type**은 **Color**로 drag & drop 해줍니다.
![Visualize](/images/mlops/vis1.png)

**Filter** 기능을 이용 하여 각 필드 별로 조건 값을 추가하여 대시보드에 반영 할 수 있습니다.

예를들어, 개별 아이템의 판매 히스토리와 예측 데이터를 확인 하시려면 다음과 같이 **Filter** 메뉴로 들어가서 **item_id**로 **Filter**를 추가 합니다.
![Visualize](/images/mlops/filter.png)

개별 아이템을 선택 후 **Apply** 버튼을 클릭하여 그래프에 필터 조건이 반영 된 것을 확인 할 수 있습니다.
![Visualize](/images/mlops/filter2.png)