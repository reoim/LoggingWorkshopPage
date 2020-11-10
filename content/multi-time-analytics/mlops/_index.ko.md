---
date: 2020-08-11
title: "MLOps 파이프라인 구축"
chapter: false
weight: 220
pre: "<b>4-2 </b>"
---

### 1. 샘플 코드 다운로드
Cloud9 IDE terminal에서 다음의 명령을 실행하여 샘플 소스 코드를 가져옵니다.
``` bash
git clone https://github.com/aws-samples/amazon-forecast-samples.git
```
코드 다운로드가 완료되면 **amazon-forecast-samples/ml_ops/visualization_blog/template.yaml** 파일을 열어 봅니다.

![MLOps yaml file](/images/mlops/mlops_yaml.png)
이 소스 코드와 아키텍처는 [MLOps Sample repo](https://github.com/aws-samples/amazon-forecast-samples/tree/master/ml_ops/visualization_blog)에서도 확인 할 수 있습니다.

해당 파일을 본 실습에 사용할 수 있도록 몇가지 내용들을 수정하여야 합니다.

Cloud9 IDE 메뉴 중 **Find->Replace** 를 선택 합니다.
![python version replace](/images/mlops/replace.png)

코드 하단에 생긴 replace field에 **Python3.7**을 **python3.6** 으로 변경 하게끔 입력하고 **Replace All** 을 클릭하여 수정 합니다.
![python version replace](/images/mlops/replace_python_version.png)

**youremail@yourprovider.com** 를 본인의 이메일 주소로 변경 하고 저장 합니다.
![replace email](/images/mlops/email.png)

***
### 2. Parameter 설정
MLOps 파이프라인 배포와 실행에 필요한 parameter들을 설정 합니다. 해당 parameter들은 SAM template로 배포시, 그리고 step function 기동시 참고하는 parameter들 입니다.

**amazon-forecast-samples/ml_ops/visualization_blog/testing-data/params.json** 파일을 열어서 아래와 같이 코드를 수정 후 저장 합니다.
``` json
{
  "DatasetGroup": {
    "DatasetGroupName":"ForecastDemoGroup",
    "Domain": "CUSTOM"
  },
  "Predictor": {
    "PredictorName": "ForecastDemoPredictor",
    "ForecastHorizon": 60, 
    "FeaturizationConfig":{
      "ForecastFrequency":"D" 
    },
    "PerformAutoML": true 
  },
  "Forecast": {
    "ForecastName": "ForecastDemo",
    "ForecastTypes":[
      "0.10", 
      "0.50", 
      "0.90"  
    ]
  },
  "TimestampFormat": "yyyy-MM-dd",
  "Datasets": [
    {
      "DatasetName": "ForecastDemoTargetTimeSeries",
      "Domain": "CUSTOM",
      "DatasetType": "TARGET_TIME_SERIES",
      "DataFrequency": "D", 
      "Schema": {
        "Attributes": [
          {
             "AttributeName":"timestamp",
             "AttributeType":"timestamp"
          },
          {
             "AttributeName":"item_id",
             "AttributeType":"string"
          },
          {
             "AttributeName":"target_value",
             "AttributeType":"float"
          }
        ]
      }
    }
  ],
  "PerformDelete": false
}
```

***
### 3. Build & Deploy
Cloud9 터미널에서 다음 명령어를 실행하여 yaml 파일의 위치로 이동 후 솔루션을 배포 합니다.
``` bash
cd amazon-forecast-samples/ml_ops/visualization_blog
sam build && sam deploy --guided
```
Build가 성공하면 아래와 같은 메세지를 확인 할 수 있습니다.

![SAM BUILD SUCCESS](/images/mlops/build.png)

Build가 성공 했다면 배포를 위한 parameter 입력 문구가 나옵니다. 아래와 같이 입력합니다. 



Stack Name [sam-app]: `forecast-demo`

AWS Region [us-east-1]: `ap-northeast-2`

Parameter Email [youremail@yourprovider.com]: `본인 email` (Step function 실패시 notification 받을 email 주소 입니다.)

Parameter ParameterFile [params.json]: **(빈칸)**

#Shows you resources changes to be deployed and require a 'Y' to initiate deploy

Confirm changes before deploy [Y/n]: `y`
	
#SAM needs permission to be able to create roles to connect to the resources in your template
	
Allow SAM CLI IAM role creation [Y/n]: `y`

Save arguments to samconfig.toml [Y/n]: `y` 

![SAM deploy](/images/mlops/deploy.png)

**SAM**을 통해 배포되는 **CloudFormation stack**의 변경 사항을 체크하고 **y**를 입력하여 승인 합니다.
![SAM deploy check](/images/mlops/deploy_check.png)

정상적으로 배포 되었다면 다음과 같은 메세지가 출력 됩니다.
![SAM deploy result](/images/mlops/deploy_result1.png)

위 출력화면에서 붉은 테두리로 표시한 부분의 값이 이번 실습에서 사용 될 ForecastBucket 이름의 예시 입니다.

다음 명령어로 param.json 파일을 생성된 ForecastBucket으로 복사합니다.

명령어 실행 시 [본인의 ForecastBucket 이름] 부분을 출력화면의 ForecastBucketName의 Value로 변경합니다.

```bash
aws s3 cp amazon-forecast-samples/ml_ops/visualization_blog/testing-data/params.json s3://[본인의 ForecastBucket 이름]
```

root 디렉토리를 확인 해보면 아래와 같이 **samconfig.toml** 파일이 생성 된 것을 확인 하실 수 있습니다.

프로덕션 환경에서 매번 배포 할 때마다 parameter를 재 입력 할 수는 없으므로 **samconfig.toml** 에 정의 된 parameter로 배포를 합니다.

현재 생성된 **samconfig.toml**의 내용은 sam deploy --guided 명령어를 통해 입력한 parameter를 기반으로 생성 되었습니다.
![SAM config toml](/images/mlops/sam_config_toml.png)

배포가 완료 되었으면 입력했던 email 주소로 **AWS Notification - Subscription Confirmation** 라는 제목으로 AWS SNS Topic에 대한 구독 메일이 수신 되었을 것 입니다. 

해당 메일의 **Confirm subscription** 링크를 클릭하여 구독을 승인 합니다.
![Confirm subscription](/images/mlops/email_confirm.png)

***
### 4. 리소스 확인
**CloudFormation** 콘솔에 들어가서 배포 된 스택을 확인 하실 수 있습니다.
![CloudFormation stack](/images/mlops/cf-stack.png)
**CloudFormation** stack의 **resource** 탭을 선택하면 MLOps 파이프라인을 만들기 위해 배포 된 리소스들을 확인 하실 수 있습니다.
![CloudFormation resource](/images/mlops/cf-resource.png)
생성된 리소스 중 **s3**로 검색하여 **ForecastBucket** 의 이름(Physical ID)를 메모장에 저장 해둡니다. 뒤에 실습 몇가지 설정에 해당 bucket 이름을 넣어줘야 합니다.

**ForecastBucket** ID 링크를 클릭하여 해당 Bucket 콘솔로 이동 합니다.
![Forecast s3](/images/mlops/cf-resource-s3-name.png)