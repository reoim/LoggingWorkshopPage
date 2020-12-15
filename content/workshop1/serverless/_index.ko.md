---
title: 서버리스 로그
weight: 500
pre: "<b>4-5. </b>"
---
***

이 페이지에서는 람다 로깅과 API gateway 로깅에 대해서 알아봅니다.

API gateway 로깅에는 실행 로깅과 액세스 로깅이라는 두 가지 유형이 있습니다. 

실행 로깅의 경우 API 배포시 API gateway에서 자동으로 `API-Gateway-Execution-Logs_{rest-api-id}/{stage_name}`의 형식으로 `CloudWatch` 로그 그룹을 생성 합니다.

액세스 로깅은 누가 API에 액세스했고 호출자가 API에 어떻게 액세스했는지 로깅하는 것입니다. 

이 페이지에서는 샘플 람다 함수를 API gateway에 REST API로 배포하고 API gateway의 액세스 로그를 CloudWatch Log에 로깅하는 구조를 만들어볼 것입니다.

## API gateway, 람다 로깅

`lib/serverless-stack.ts` 파일에 다음 코드를 추가하여 필요한 모듈들을 import 합니다.

```typescript
import * as lambda from '@aws-cdk/aws-lambda';
import * as apigw from '@aws-cdk/aws-apigateway';
import * as logs from '@aws-cdk/aws-logs';
```

미리 작성한 샘플 람다 코드는 `resources/lambda/sample.py` 에 있습니다.

`contstructor` 안에 아래 코드를 추가하여 샘플 코드를 람다로 정의합니다.

기본적으로 람다는 자동으로 `/aws/lambda/<function-name>`의 형식으로 로그 그룹을 생성합니다.

다만 이렇게 생성된 로그 그룹의 default 보관기간은 `Never Expire`로 설정되어 있습니다.

아래 코드에서는 로그 보관 기간을 1주일로 설정 하였습니다.
```typescript
    // Defines an AWS Lambda resource
    const sample = new lambda.Function(this, 'SampleHandler', {
      runtime: lambda.Runtime.PYTHON_3_7,               // execution environment
      code: lambda.Code.fromAsset('resources/lambda'),  // code loaded from "resources/lambda" directory
      handler: 'sample.handler',                        // file is "sample", function is "handler"
      logRetention: logs.RetentionDays.ONE_WEEK         // default: Never Expire
    });
```

아래 코드를 추가하여 API gateway 액세스 로깅을 위한 로그 그룹을 생성합니다.

```typescript
    // Defines a log group for API Gateway
    const apigwLogGroup = new logs.LogGroup(this, 'APIgatewayLogs', {
      logGroupName: 'APIgatewayLogs',
      retention: logs.RetentionDays.ONE_WEEK
    });
```

아래 코드를 추가하여 위에서 정의한 람다 함수를 REST API로 등록 합니다.

```typescript
    // Defines an API Gateway REST API resource backed by our "sample" function.
    const api = new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: sample,
      deployOptions: {
        accessLogDestination: new apigw.LogGroupLogDestination(apigwLogGroup),
        accessLogFormat: apigw.AccessLogFormat.jsonWithStandardFields()
      }
    });
```

완성된 코드는 다음과 같을 것입니다.

```typescript
import * as cdk from '@aws-cdk/core';
import * as lambda from '@aws-cdk/aws-lambda';
import * as apigw from '@aws-cdk/aws-apigateway';
import * as logs from '@aws-cdk/aws-logs';

export class ServerlessStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
        // Defines an AWS Lambda resource
    const sample = new lambda.Function(this, 'SampleHandler', {
      runtime: lambda.Runtime.PYTHON_3_7,               // execution environment
      code: lambda.Code.fromAsset('resources/lambda'),  // code loaded from "resources/lambda" directory
      handler: 'sample.handler',                        // file is "sample", function is "handler"
      logRetention: logs.RetentionDays.ONE_WEEK         // default: Never Expire
    });
    
    // Defines a log group for API Gateway
    const apigwLogGroup = new logs.LogGroup(this, 'APIgatewayLogs', {
      logGroupName: 'APIgatewayLogs',
      retention: logs.RetentionDays.ONE_WEEK
    });
    
    // Defines an API Gateway REST API resource backed by our "sample" function.
    const api = new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: sample,
      deployOptions: {
        accessLogDestination: new apigw.LogGroupLogDestination(apigwLogGroup),
        accessLogFormat: apigw.AccessLogFormat.jsonWithStandardFields()
      }
    });
  }
}
```

## 엔트리포인트에 스택 추가하기

`bin/centralized-logging-skeleton.ts` 파일을 열어 스택을 추가할 것입니다.

다음 코드를 추가하여 스택을 import 합니다.

```typescript
import { ServerlessStack } from '../lib/serverless-stack';
```

다음 코드를 추가하여 ServerlessStack을 엔트리 포인트에 추가합니다.

```typescript
new ServerlessStack(app, 'ServerlessStack', { env: envRegion });
```

완성된 코드는 다음과 같습니다.

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';
import { CloudtrailStack } from '../lib/cloudtrail-stack';
import { WebServerStack } from '../lib/webserver-stack';
import { ServerlessStack } from '../lib/serverless-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

const logBucketStack = new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
new CloudtrailStack(app, 'CloudtrailStack', { env: envRegion, bucket:logBucketStack.logBucket });
new WebServerStack(app, 'WebServerStack', { env: envRegion, bucket:logBucketStack.logBucket });
new ServerlessStack(app, 'ServerlessStack', { env: envRegion });
```

&nbsp;

## 배포하기

터미널 창에 다음 명령어를 입력하여 ServerlessStack을 배포합니다.

```
cdk deploy ServerlessStack
```

&nbsp;

## API gateway 테스트

배포가 완료되면 [API gateway 콘솔](https://us-east-2.console.aws.amazon.com/apigateway/main/apis?region=us-east-2)에 접속하여 API가 잘 배포되었는지 확인합니다. 

![API gateway](/images/workshop1/api-gateway.png)
 
배포된 `API id`를 이용해 API를 호출 해봅니다.

API 호출 url 형식은 다음과 같습니다.

```
https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/
```

`{restapi_id}` 부분을 본인의 `API id`로 교체하고 `{region}`은 `us-east-2`로 수정, `{stage_name}`은 `prod`로 수정 합니다.

다음은 예시 url 입니다.

```
https://j9qdpbgbt9.execute-api.us-east-2.amazonaws.com/prod
```

웹브라우저에서 해당 url을 호출하여 API를 테스트 해봅니다.

![API test](/images/workshop1/api-url.png)

&nbsp;

## 로깅 확인

로그 그룹에 정상적으로 로깅이 되는지 확인 합니다.

![API test](/images/workshop1/api-loggroup.png)