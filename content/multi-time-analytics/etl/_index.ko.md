---
date: 2020-09-2
title: "ETL 작업"
chapter: false
weight: 230
pre: "<b>4-3 </b>"
---


## 테스트 데이터 준비
{{% notice note %}}
테스트 데이터는 [3-2 노트북에서 Forecast 사용하기 실습](https://master.dsd6loerysixy.amplifyapp.com/ko/one-time-analytics/start-forecasting/)에서 다운 받은 데이터를 사용 합니다.
{{% /notice %}}
**ForecastBucket**에서 **raw_data**라는 폴더를 생성하고 다운받은 데이터(파일명: forecast_data.csv)를 **raw_data**폴더 안에 업로드 합니다.
![Data check](/images/mlops/s3-check.png) 




## AWS GLUE
***
{{% notice note %}}
AWS Glue는 완전 관리형 추출, 변환 및 로드(ETL) 서비스로, 효율적인 비용으로 간단하게 여러 데이터 스토어 및 데이터 스트림 간에 원하는 데이터를 분류, 정리, 보강, 이동합니다. 좀 더 자세한 설명은 [워크샵 소개](https://master.dsd6loerysixy.amplifyapp.com/ko/overview/#amazon-glue) 페이지를 참고 해주세요.
{{% /notice %}}
### Crawler 생성
[Glue콘솔의 Crawler 메뉴](https://ap-northeast-2.console.aws.amazon.com/glue/home?region=ap-northeast-2#catalog:tab=crawlers)로 이동 하여 **Add crawler** 버튼을 클릭 합니다.
![Add Crawler](/images/mlops/add-crawler.png)
Crawler name을 `forecast-demo-crawler`로 입력 하고 **Next** 버튼을 클릭 합니다.
![Crawler info](/images/mlops/crawler-info.png)
**Data sotres** 를 선택하고 **Next** 버튼을 클릭 합니다.
![Crawler source type ](/images/mlops/crawler-source-type.png)
**ForecastBucket**에 생성된 하위 폴더 **raw_data**를 path에 추가하고 **Next** 버튼을 클릭 합니다.
![Crawler datastore](/images/mlops/crawler-datastore.png)
**No**를 선택하고 **Next** 버튼을 클릭 합니다.
![Crawler datastore2](/images/mlops/crawler-datastore2.png)
**Create an IAM role**을 선택하고 **AWSGlueServiceRole-** 뒤 빈칸에 `forecast-demo`를 입력한 뒤 **Next** 버튼을 클릭 합니다.
![Crawler iam](/images/mlops/crawler-iam.png)
**Run on demand** 로 설정하고 **Next** 버튼을 클릭 합니다.
![Crawler schedule](/images/mlops/crawler-schedule.png)
데이터베이스 이름으로 `forecast-demo-db`를 입력하고 **Next** 버튼을 클릭 합니다.
![Crawler output](/images/mlops/crawler-output.png)
잘못 입력한 부분이 있는지 확인 후 **Finish** 버튼을 클릭하여 crawler를 생성 합니다.
![Crawler review](/images/mlops/crawler-review.png)

***
### Run Crawler 
**Run crawler** 버튼을 클릭하여 crawler를 실행합니다.
![Run Crawler](/images/mlops/run-crawler.png)
Crawler가 정상적으로 동작 완료 되면 다음과 같이 **forecast-demo-db** 데이터 베이스에 **raw_data** 테이블이 추가 된 것을 확인 할 수 있습니다.
![Table added](/images/mlops/glue-table.png)

***
### Glue job 생성
[Glue콘솔의 jobs메뉴](https://ap-northeast-2.console.aws.amazon.com/glue/home?region=ap-northeast-2#etl:tab=jobs)로 이동하여 **Add job** 버튼을 클릭합니다.

Job 이름에 `forecast-demo-job`을 입력 합니다.

IAM role은 crawler 생성시 만들었던 **AWSGlueServiceRole-forecast-demo**를 선택 합니다.

**This job runs** 항목에서 **A new script to be authored by you**를 선택 합니다.

나머지 설정은 default 상태로 남기고 다음 단계로 넘어 갑니다.
![configure job](/images/mlops/add-job.png)

**Save job and edit script**를 클릭하여 다음 단계로 넘어 갑니다.
![configure job](/images/mlops/add-job2.png)

다음의 스크립트는 원본 데이터를 로드한 뒤 timestamp 형식을 yyyy/MM/dd에서 yyyy-MM-dd로 변경하고 train 폴더에 csv 형식으로 파일을 저장하는 간단한 ETL 작업을 하는 코드 입니다. 해당 스크립트를 복사하여 에디터에 붙여 넣습니다.
```python
import sys
import boto3
import pyspark.sql.functions as F
from awsglue.transforms import *
from awsglue.dynamicframe import DynamicFrame
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job


glueContext = GlueContext(SparkContext.getOrCreate())
spark = glueContext.spark_session

session = boto3.Session(region_name='ap-northeast-2') 
client = boto3.client('s3')
glue_client = session.client(service_name='glue')

s3BucketName = #your bucket name
s3Folder = "train/"
s3BucketPath = "s3://"+s3BucketName+"/"+s3Folder

datasource = glueContext.create_dynamic_frame.from_catalog(database = "forecast-demo-db", table_name = "raw_data")

df1 = datasource.toDF()

df2 = df1.withColumn('timestamp',F.from_unixtime(F.unix_timestamp('timestamp', 'yyyy/MM/dd'),'yyyy-MM-dd'))
data_frame=DynamicFrame.fromDF(df2, glueContext, "data_frame")

glueContext.write_dynamic_frame.from_options(frame = data_frame, connection_type = "s3", connection_options = {"path":s3BucketPath}, format = "csv")


response = client.list_objects(
    Bucket=s3BucketName,
    Prefix=s3Folder
)
name = response['Contents'][0]['Key']
client.copy_object(Bucket=s3BucketName, CopySource=s3BucketName+"/"+name, Key=name+".csv")
client.delete_object(Bucket=s3BucketName, Key=name)
```

위 스크립트 내용 중 s3BucketName에 자신의 **ForecastBucket** 이름을 넣습니다.
```
s3BucketName = #your bucket name
```

아래 예시처럼 자신의 bucket명을 변수에 넣고 **Save** 버튼을 클릭하여 저장 한 후 우측 상단 **X** 버튼을 클릭 합니다.
![job script](/images/mlops/job-script.png)


***
### IAM policy 수정
생성한 ETL job이 동작하기 위해서 ForecastBucket의 다른 하위 폴더에 접근하고 object를 삭제할 수 있는 권한이 필요 합니다.

[IAM 콘솔](https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/roles)로 이동하여 `AWSGlueServiceRole-forecast-demo` role을 검색하여 선택 합니다.
![IAM role](/images/mlops/iam-role.png)
다음 화면과 같이 policy를 선택하고 **Edit policy** 버튼을 클릭 합니다.
![IAM policy](/images/mlops/iam-policy.png)
**JSON** 탭을 클릭하여 다음과 같은 형식으로 policy를 수정 합니다.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": [
                "arn:aws:s3:::[자신의 ForecastBucket]/*"
            ]
        }
    ]
}
```
다음 예시와 같이 자신의 **ForecastBucket** 이름을 입력하여 수정 후 저장 합니다.
![Modify policy](/images/mlops/modify-policy.png)

***
### Glue job 실행
[Glue콘솔 job메뉴](https://ap-northeast-2.console.aws.amazon.com/glue/home?region=ap-northeast-2#etl:tab=jobs)로 돌아와서 **forecast-demo-job**을 선택 후 **Action -> Run job**을 클릭 하여 job을 실행 합니다.
![Run job](/images/mlops/run-job.png)
다음과 같이 job status를 콘솔에서 확인 할 수 있습니다.
![Job completed](/images/mlops/job-done.png)
Job이 성공적으로 완료된 뒤, **ForecastBucket**에 **train**폴더가 생성 되어 있고 ETL 작업이 완료된 csv 파일이 해당 폴더에 생성 된 것을 확인 할 수 있습니다.
![ETL completed](/images/mlops/etl-done.png)