---
title: "Log destination 설정"
weight: 100
pre: "<b>5-1. </b>"
---
***

{{% notice info %}}
이번 실습을 진행하기 위해서는 [실습 1, CDK로 로깅 인프라 구축](../../workshop1)의 [베이스 프로젝트 클론](../../workshop1/base-project)과 [로그 버킷](../../workshop1/log-bucket) 실습이 선행되어야 합니다.
{{% /notice %}}

&nbsp;

이 페이지에서는 로그 버킷 계정(`111111111111`)에 Log destination을 생성합니다. 

로그 생성 계정(`222222222222`)으로부터 Log destination에 전송된 로그들을 kinesis firehose를 사용하여 로그 버킷으로 전송하는 구성을 만들어볼 것입니다.

&nbsp;

## 중앙 로그 버킷 공유

먼저 중앙 로그 버킷 object를 props로 전달 받기 위해 `lib/log-destination-stack.ts` 파일을 열어 다음과 같이 수정합니다.

`LogBucketStack`에서 정의한 `interface`를 `import` 합니다.

```typescript
import { BucketProps } from './log-bucket-stack';
```

`constructor`의 `props`를 수정합니다.
```typescript
constructor(scope: cdk.Construct, id: string, props: BucketProps)
```

수정된 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import { BucketProps } from './log-bucket-stack';

export class LogDestinationStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

&nbsp;

## Kinesis Firehose 설정

먼저 파일 상단에 다음 코드를 추가하여 필요한 모듈들을 import 합니다.

```typescript
import * as iam from '@aws-cdk/aws-iam';
import * as logs from '@aws-cdk/aws-logs';
import * as firehose from '@aws-cdk/aws-kinesisfirehose';
import * as log_ds from '@aws-cdk/aws-logs-destinations';
import * as ssm from '@aws-cdk/aws-ssm';
import fs = require('fs');
```

`constructor`에 다음 코드들을 추가합니다.

Firehose에 부여할 IAM role을 생성하고 로그 버킷에 대한 권한을 추가합니다.

```typescript
    // Create IAM role for firehose delivery stream
    const firehoseRole = new iam.Role(this, "FirehoseRole", {
      assumedBy: new iam.ServicePrincipal('firehose.amazonaws.com')
    });
    
    // Add S3(Log bucket) permission to the role
    firehoseRole.addToPolicy(new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: [props.bucket.bucketArn, props.bucket.bucketArn+'/*'],
        actions: ['s3:AbortMultipartUpload', 's3:GetBucketLocation', 's3:GetObject', 's3:ListBucket', 's3:ListBucketMultipartUploads', 's3:PutObject']
    }));
```

Firehose delivery stream을 생성하는 설정입니다.

기본적으로 5분마다 로그를 전송하며 buffer에 쌓인 로그가 5MB 이상일 경우 바로 로그를 전송하도록 설정 하였습니다.
```typescript
    // create firehose delivery stream
    const firehoseDliverySteram = new firehose.CfnDeliveryStream(this, "FireHoseStream", {
        deliveryStreamName: "FireHoseStream",
        deliveryStreamType: "DirectPut",
        s3DestinationConfiguration: {
            bucketArn: props.bucket.bucketArn,
            prefix: 'AWSLogs/SecondAccount/',
            bufferingHints: {
                intervalInSeconds: 300,
                sizeInMBs: 5,
            },
            roleArn: firehoseRole.roleArn
        },
    });
```

&nbsp;

## Log destination IAM role설정

위의 `Firehose` 설정 코드 밑에 다음 코드를 추가합니다.

`Log destination`에 부여할 `IAM role`을 생성하고 `Firehose`에 데이터를 전송할 수 있도록 권한을 부여합니다.

```typescript
    // Create IAM role for log destination
    const logDestinationRole = new iam.Role(this, "LogDestinationRole", {
      assumedBy: new iam.ServicePrincipal('logs.amazonaws.com')
    });
    
    // Add firehose permission to the role
    logDestinationRole.addToPolicy(
      new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: ['*'],
        actions: ['firehose:PutRecord']
      })
    );
```

&nbsp;

## Log destination access policy 설정
베이스 프로젝트에 제공하는 `resources/policies/log-access-policy.json` 파일을 열어봅니다.

```json
{
  "Version" : "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Principal" : {
        "AWS" : ["{ Second Account ID }"]
      },
      "Action" : "logs:PutSubscriptionFilter",
      "Resource" : "arn:aws:logs:{region}:{accountid}:destination:{log-destination-name}"
    }
  ]
}
```

위의 `{ Second Account ID }`를 로그를 생성할 두번째 계정, 로그 생성 계정 ID로 수정합니다. (예: `222222222222`) 

현재 Resource의 값은 Log destination의 ARN 형식 입니다. 

`{region}`을 `us-east-2`로, `{accountid}`를 현재 작업중인 첫번째 계정, 로그 버킷 계정 ID (예: `111111111111`)로 수정합니다.

`{log-destination-name}`은 `CentralLogDestination`으로 수정 후 저장합니다.

완성된 Access Policy 는 다음과 비슷할 것입니다.

```json
{
  "Version" : "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Principal" : {
        "AWS" : ["222222222222"]
      },
      "Action" : "logs:PutSubscriptionFilter",
      "Resource" : "arn:aws:logs:us-east-2:111111111111:destination:CentralLogDestination"
    }
  ]
}
```

다시 `lib/log-destination-stack.ts` 파일로 돌아와서 

다음 코드를 추가하여 수정한 Access Policy를 읽어옵니다.

```typescript
    // Read access policy from the asset
    const logPolicy = fs.readFileSync('resources/policies/log-access-policy.json', 'utf8');
```

&nbsp;

## Log destination 설정

다음 코드를 추가하여 `Log destination`을 생성합니다.

```typescript
    // Create log destination
    const logDestination = new logs.CfnDestination(this, 'LogDestination', {
      destinationName: 'CentralLogDestination',
      roleArn: logDestinationRole.roleArn,
      targetArn: firehoseDliverySteram.attrArn,
      destinationPolicy: logPolicy
    });
```

다음 코드를 추가하여 `Log destination`이 생성되기 전에 반드시 `Firehose`가 준비되도록 설정 합니다.

```typescript
    // Make sure the Firehose delivery stream is provisioned when deploy the log destination.
    logDestination.addDependsOn(firehoseDliverySteram);
```

다음 코드를 추가하여 생성된 `Log destination`의 ARN을 `CloudFormation` Output으로 출력합니다.

```typescript
new cdk.CfnOutput(this, 'Log destination arn', { value: logDestination.attrArn });
```


완성된 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import * as iam from '@aws-cdk/aws-iam';
import * as logs from '@aws-cdk/aws-logs';
import * as firehose from '@aws-cdk/aws-kinesisfirehose';
import * as log_ds from '@aws-cdk/aws-logs-destinations';
import * as ssm from '@aws-cdk/aws-ssm';
import fs = require('fs');
import { BucketProps } from './log-bucket-stack';

export class LogDestinationStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
    // Create IAM role for firehose delivery stream
    const firehoseRole = new iam.Role(this, "FirehoseRole", {
      assumedBy: new iam.ServicePrincipal('firehose.amazonaws.com')
    });
    
    // Add S3(Log bucket) permission to the role
    firehoseRole.addToPolicy(
      new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: [props.bucket.bucketArn, props.bucket.bucketArn+'/*'],
        actions: ['s3:AbortMultipartUpload', 
                  's3:GetBucketLocation', 
                  's3:GetObject', 
                  's3:ListBucket', 
                  's3:ListBucketMultipartUploads', 
                  's3:PutObject']
      })
    );
 
    // create firehose delivery stream
    const firehoseDliverySteram = new firehose.CfnDeliveryStream(this, "FireHoseStream", {
        deliveryStreamName: "FireHoseStream",
        deliveryStreamType: "DirectPut",
        s3DestinationConfiguration: {
            bucketArn: props.bucket.bucketArn,
            prefix: 'AWSLogs/SecondAccount/',
            bufferingHints: {
                intervalInSeconds: 300,
                sizeInMBs: 5,
            },
            roleArn: firehoseRole.roleArn
        },
    });
  
    // Create IAM role for log destination
    const logDestinationRole = new iam.Role(this, "LogDestinationRole", {
      assumedBy: new iam.ServicePrincipal('logs.amazonaws.com')
    });
    
    // Add firehose permission to the role
    logDestinationRole.addToPolicy(
      new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: ['*'],
        actions: ['firehose:PutRecord']
      })
    );
    
    // Read access policy from the asset
    const logPolicy = fs.readFileSync('resources/policies/log-access-policy.json', 'utf8');
    
    // Create log destination
    const logDestination = new logs.CfnDestination(this, 'LogDestination', {
      destinationName: 'CentralLogDestination',
      roleArn: logDestinationRole.roleArn,
      targetArn: firehoseDliverySteram.attrArn,
      destinationPolicy: logPolicy
    });
    
    // Make sure the Firehose delivery stream is provisioned when deploy the log destination.
    logDestination.addDependsOn(firehoseDliverySteram);
    
  
    new cdk.CfnOutput(this, 'Log destination arn', { value: logDestination.attrArn });

  }
}
```

&nbsp;

## 앤트리포인트에 스택 추가하기
`bin/centralized-logging-skeleton.ts` 파일을 열어 LogDestinationStack을 추가합니다.

다음 코드를 추가하여 LogDestinationStack import 합니다.

```typescript
import { LogDestinationStack } from '../lib/log-destination-stack';
```

파일 하단에 다음 코드를 추가하여 스택을 추가합니다.

```typescript
new LogDestinationStack(app, 'LogDestinationStack', { env: envRegion, bucket:logBucketStack.logBucket });
```

완성된 코드는 다음과 같을 것입니다.

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';
import { CloudtrailStack } from '../lib/cloudtrail-stack';
import { ServerlessStack } from '../lib/serverless-stack';
import { WebServerStack } from '../lib/webserver-stack';
import { EcsStack } from '../lib/ecs-stack';
import { LogDestinationStack } from '../lib/log-destination-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

const logBucketStack = new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
new CloudtrailStack(app, 'CloudtrailStack', { env: envRegion, bucket:logBucketStack.logBucket });
new WebServerStack(app, 'WebServerStack', { env: envRegion, bucket:logBucketStack.logBucket });
new ServerlessStack(app, 'ServerlessStack', { env: envRegion });
new EcsStack(app, 'EcsStack', { env: envRegion });
new LogDestinationStack(app, 'LogDestinationStack', { env: envRegion, bucket:logBucketStack.logBucket });
```

&nbsp;

## 배포하기

터미널에 다음 명령어를 입력하여 LogDestinationStack을 배포합니다.

```
cdk deploy LogDestinationStack
```

배포가 완료되면 [CloudFormation 콘솔](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false)로 이동하여 스택이 잘 배포되었는지 확인하고 

![CF console](/images/workshop2/log-cf1.png)

Output에 출력된 Log destination의 ARN을 복사하여 메모 해둡니다.

![CF console](/images/workshop2/log-cf2.png)