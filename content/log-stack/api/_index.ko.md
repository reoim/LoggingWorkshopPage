---
title: "(Optional) API gateway log"
weight: 350
---
***
이 페이지에서는 다른 AWS 계정에 API Gateway - Lambda로 구성 된 서버리스 샘플 앱을 배포하고 API Gateway Log를 CloudWatch Log에 적재 합니다.

적재 된 Log는 Subscription filter 기능을 이용하여 Kinesis Data Firehose가 구독하고 Log 관리 계정의 S3 Bucket으로 전송 됩니다.

## 실습 환경 구성

두번째 계정으로 로그인 한 뒤 [실습 환경 구축](../../settings/create-workspace/) 페이지를 참고하여 실습 환경을 구축 합니다.

Cloud9 환경이 준비 완료 되면 터미널 창에 다음의 명령어를 입력하여 프로젝트 생성 및 필요한 패키지를 다운 받습니다.

```bash
# Setting environment variable for CDK Version
echo 'export AWS_CDK_VERSION="1.72.0"' >> ~/.bashrc
source ~/.bashrc

# Install aws-cdk
npm install -g --force aws-cdk@$AWS_CDK_VERSION

# Create project folder 
mkdir second-account && cd second-account

# Initiate CDK app
cdk init app --language typescript

# Install npm packages that we need for the workshop
npm i @aws-cdk/aws-logs@$AWS_CDK_VERSION @aws-cdk/aws-lambda@$AWS_CDK_VERSION @aws-cdk/aws-apigateway@$AWS_CDK_VERSION
```

패키지 설치가 완료 되면 터미널 창에 다음 명령어를 실행하여 TypeScript를 자동으로 컴파일 하게 합니다. 
```
npm run watch
```
결과 화면 예시
```terminal
[2:29:53 AM] File change detected. Starting incremental compilation...

[2:29:53 AM] Found 0 errors. Watching for file changes.
``` 
npm run watch 명령어가 실행 된 터미널 창은 열어둔 채로 유지하고 새로운 터미널 창을 하나 생성 해둡니다.


## Lambda 생성

새로 생성한 터미널 창에서 다음 명령어를 입력하여 Lambda 함수를 저장 할 폴더를 만듭니다.
```
cd second-account
mkdir -p resources/lambda
```

**resources/lambda** 폴더에 **sample.py** 파일을 생성하고 다음 코드를 추가 후 저장 합니다.
```python
import json

def handler(event, context):
    print('request: {}'.format(json.dumps(event)))
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'text/plain'
        },
        'body': 'AWS CDK sample lambda function'
    }
```
![lambda](/images/log-stack/lambda.png)

다음은 위의 샘플 함수를 실제 AWS Lambda로 정의 하는 작업 입니다.

Cloud9 좌측 네비게이션에서 **lib/second-account-stack.ts** 파일을 더블 클릭 합니다.

![second-account-stack](/images/log-stack/second-stack.png)

`import * as cdk from '@aws-cdk/core';` 아래에 다음 코드를 추가하여 필요한 패키지들을 import 합니다.

```typescript
import * as lambda from '@aws-cdk/aws-lambda';
import * as apigw from '@aws-cdk/aws-apigateway';
import * as logs from '@aws-cdk/aws-logs';
```

주석 `// The code that defines your stack goes here` 아래에 다음 코드를 추가 합니다.

미리 정의한 sample.py 코드를 기반으로 런타임 Python 3.7의 Lambda를 생성하는 코드 입니다.

```typescript
// Defines an AWS Lambda resource
const sample = new lambda.Function(this, 'SampleHandler', {
    runtime: lambda.Runtime.PYTHON_3_7,    // execution environment
    code: lambda.Code.fromAsset('resources/lambda'),  // code loaded from "resources/lambda" directory
    handler: 'sample.handler'                // file is "sample", function is "handler"
});
```

다음 코드를 추가하여 API Gateway에 Lambda를 연동하고 API Gateway의 Log를 CloudWatch Log에 적재 합니다. 

추후 Cross-account/Cross-region Monitoring Dashbaord 실습을 위해서 간단한 Client Error 지표도 생성 합니다.

```typescript
// Defines a log group for API Gateway
const apigwLogGroup = new logs.LogGroup(this, 'APIgatewayLogs', {
    logGroupName: 'APIgatewayLogs',
    retention: logs.RetentionDays.THREE_MONTHS
});

// Defines an API Gateway REST API resource backed by our "sample" function.
const api = new apigw.LambdaRestApi(this, 'Endpoint', {
    handler: sample,
    deployOptions: {
    accessLogDestination: new apigw.LogGroupLogDestination(apigwLogGroup),
    accessLogFormat: apigw.AccessLogFormat.clf()
    }
});

//Metric for the number of client-side errors captured in a given period.
//@default - sum over 5 minutes
const clientErrorMetric = api.metricClientError();
```