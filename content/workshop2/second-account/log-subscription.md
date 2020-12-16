---
title: 로그 구독 필터
weight: 230
pre: "<b>5-2.c. </b>"
---
***

이 페이지에서는 로그 생성을 위해 실습 1에서 배포한것과 동일한 간단한 서버리스 워크로드를 배포할 것입니다.

그리고 로그 구독 필터 기능을 이용해 `CloudWatch Logs` 로그 그룹에 수집되는 로그를 Log destination으로 내보내는 설정을 알아볼 것입니다.

## API gateway, 람다 로깅

`lib/serverless-stack.ts` 파일에 다음 코드를 추가하여 필요한 모듈들을 import 합니다.

```typescript
import * as lambda from '@aws-cdk/aws-lambda';
import * as apigw from '@aws-cdk/aws-apigateway';
import * as logs from '@aws-cdk/aws-logs';
```

`contstructor` 안에 다음 코드를 추가하여 람다와 API gateway를 정의합니다. 실습 1에서 정의한 것과 동일한 코드입니다.

```typescript
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
      retention: logs.RetentionDays.ONE_DAY
    });

    // Defines an API Gateway REST API resource backed by our "sample" function.
    const api = new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: sample,
      deployOptions: {
        accessLogDestination: new apigw.LogGroupLogDestination(apigwLogGroup),
        accessLogFormat: apigw.AccessLogFormat.jsonWithStandardFields()
      }
    });
```

&nbsp;

## Context value 수정

`cdk.json` 파일을 열어봅니다.

```json
{
  "app": "npx ts-node --prefer-ts-exts bin/centralized-logging-skeleton.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": "true",
    "aws-cdk:enableDiffNoFail": "true",
    "@aws-cdk/core:stackRelativeExports": "true",
    "@aws-cdk/aws-ecr-assets:dockerIgnoreSupport": true,
    "@aws-cdk/aws-secretsmanager:parseOwnedSecretName": true,
    "destination": "arn:aws:logs:{region}:{accountid}:destination:{log-destination-name}",
    "email": "example@email.com"
  }
}
```

`"destination"`의  `"arn:aws:logs:{region}:{accountid}:destination:{log-destination-name}",` 값을 
[Log destination 설정 실습](../../log-destination)에서 메모해둔 Log destination의 ARN으로 수정합니다.

수정된 cdk.json 파일은 다음과 같을 것입니다.
```json
{
  "app": "npx ts-node --prefer-ts-exts bin/centralized-logging-skeleton.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": "true",
    "aws-cdk:enableDiffNoFail": "true",
    "@aws-cdk/core:stackRelativeExports": "true",
    "@aws-cdk/aws-ecr-assets:dockerIgnoreSupport": true,
    "@aws-cdk/aws-secretsmanager:parseOwnedSecretName": true,
    "destination":"arn:aws:logs:us-east-2:111111111111:destination:CentralLogDestination",
    "email": "example@email.com"
  }
}

```

&nbsp;

## 로그 구독 필터 설정

다시 `lib/serverless-stack.ts` 파일의 `constructor` 하단부에 다음 코드를 추가합니다.

```typescript
    // Define subscription filter for cloudwatch log.
    const subscriptionFilter = new logs.CfnSubscriptionFilter(this, 'SubscriptionFilter', {
      destinationArn: this.node.tryGetContext('destination'),
      filterPattern: "",
      logGroupName: apigwLogGroup.logGroupName
    })
```

로그 구독 필터를 사용하여 Log destination으로 로그를 내보내는 설정 입니다.

`this.node.tryGetContext('destination')` 코드로 cdk.json 에 추가한 destination context value를 로드할 수 있습니다.

&nbsp;

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
      retention: logs.RetentionDays.ONE_DAY
    });
    
    // Defines an API Gateway REST API resource backed by our "sample" function.
    const api = new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: sample,
      deployOptions: {
        accessLogDestination: new apigw.LogGroupLogDestination(apigwLogGroup),
        accessLogFormat: apigw.AccessLogFormat.jsonWithStandardFields()
      }
    });

    // Define subscription filter for cloudwatch log.
    const subscriptionFilter = new logs.CfnSubscriptionFilter(this, 'SubscriptionFilter', {
      destinationArn: this.node.tryGetContext('destination'),
      filterPattern: "",
      logGroupName: apigwLogGroup.logGroupName
    });
  }
}
```

## 엔트리포인트에 스택 추가하기

`bin/centralized-logging-skeleton.ts` 파일을 열어 ServerlessStack을 import 합니다.

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
import { ServerlessStack } from '../lib/serverless-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

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

로그 버킷 계정에 정상적으로 로그를 내보내는지 확인 합니다.

{{% notice info %}}
AWS 콘솔에서 로그아웃 한 뒤, 로그 버킷이 있는 첫번째 계정으로 로그인 합니다.
{{% /notice %}}

로그 버킷에 prefix `AWSLogs/SecondAccount`로 로깅이 잘 되었는지 확인합니다.

![s3 logging](/images/workshop2/log-s3.png)