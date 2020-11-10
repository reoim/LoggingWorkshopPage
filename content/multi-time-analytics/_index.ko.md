---
date: 2020-07-29
title: "MLOps 파이프라인 구축하기"
chapter: false
weight: 300
pre: "<b>4. </b>"
---

## 실습 소개
이번 실습에서는 아래와 같은 MLOps 아키텍처를 배포 해 볼 것입니다.

AWS 관리형 서비스들을 이용하여 직접 인프라를 관리할 필요 없이 간편하게 serverless MLOps 아키텍처를 구축 하실 수 있습니다.
## 전체 아키텍처
***
![Architecture](/images/mlops/archi.png)
***
### Step functions
이 실습에서는 [Amazon Step Functions](https://aws.amazon.com/ko/step-functions/)를 사용하여 MLOps 파이프라인을 구축 합니다. **Step Functions**의 Workflow는 다음과 같습니다.
![Workflow](/images/mlops/stepfunction_workflow.png)

각 Step들은 다음과 같은 작업을 수행 합니다.

1. **Create-Dataset** – Forecast dataset을 생성 합니다. 

2. **Create-DatasetGroup** – Forecast Dataset group을 생성 합니다.

3. **Import-Data** – Data를 dataset group 안의 dataset으로 import 합니다.

4. **Create-Predictor** – Predictor를 생성 합니다. **3-2 노트북에서 Forecast 사용하기** 실습처럼 직접 POC작업을 하며 적합한 ML 알고리즘을 찾을 수도 있지만, 이번 실습에서는 AutoML을 사용하여 dataset을 대상으로 가장 최적의 성능을 내는 ML 알고리즘으로 predictor를 생성하게 할 것 입니다.

5. **Create-Forecast** – Forecast 데이터를 생성하고 S3로 export 합니다.

6. **Update-Resources** – Creates the necessary Athena resources and transforms the exported forecasts to the same format as the input dataset. 필요한 Athena 리소스들을 생성하고 export한 Forecast 데이터의 포멧을 input datset와 동일하게 변경 합니다.

7. **Notify Success** – 작업이 끝나면 Amazon SNS를 통해 이메일 알람을 보냅니다.

8. **Strategy-Choice**  – Forecast 리소스들을 삭제할지 체크 합니다. Parameter 파일(params.json)의 **PerformDelete** 값으로 설정 할 수 있습니다.

9. **Delete-Forecast** – Export한 data만 남기고 Forecast는 삭제 합니다.

10. **Delete-Predictor** – Predictor를 삭제 합니다.

11. **Delete-ImportJob** – Forecast의 Import job을 삭제 합니다.