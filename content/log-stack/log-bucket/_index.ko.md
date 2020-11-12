---
title: "Log Bucket 생성"
weight: 310
---
***



## S3 bucket 생성
제일 먼저, 로그를 적재 할 중앙 S3 Bucket을 생성 할 것 입니다.

Cloud9 IDE에서 **lib/logging-wokrshop-stack.ts** 파일을 열어 봅니다.

다음과 같이 기본 코드가 작성되어 있을 것 입니다.

```typescript
import * as cdk from '@aws-cdk/core';

export class LoggingWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

**lib/logging-workshop-stack.ts** 파일의 상단부에 다음 코드를 추가하여 aws-s3 모듈을 import 해줍니다.
```typescript
import * as s3 from '@aws-cdk/aws-s3';
```

**LoggingWorkshopStack** 클래스 안에 s3 bucket을 생성하는 다음의 코드를 추가 합니다.
```typescript
const logBucket = new s3.Bucket(this, 'LogBucket');
```

코드를 복사, 붙여넣기 하지말고 직접 타이핑을 해보시길 바랍니다.

Cloud9 과 같은 IDE를 이용하여 CDK 코드를 작성하면 언어 지원 기능을 통해 자동 완성, type 체크, inline 코드 문서 등의 도움을 받을 수 있어 좀 더 손쉽게 리소스를 정의 하실 수 있습니다.

![Language support](/images/log-stack/ide_support.png)

Stack을 배포할 때, CloudFormation 콘솔의 해당 Stack의 Output에서 Bucket 이름을 확인 할 수 있도록 다음의 코드를 추가합니다.
```typescript
new cdk.CfnOutput(this, 'LogBucketName', { value: logBucket.bucketName });
```

최종 코드는 다음과 같습니다.
```typescript
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

export class LoggingWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    
    // The code that defines your stack goes here
    
    // Create s3 bucket
    const logBucket = new s3.Bucket(this, 'LogBucket');
    
    // CloudFormation output for the bucket name
    new cdk.CfnOutput(this, 'LogBucketName', { value: logBucket.bucketName });
    
  }
}
```
&nbsp;
새로 열었던 터미널 창에 다음 명령어를 실행하여 CDK 코드가 어떻게 CloudFormation template를 정의하는지 확인 합니다.
```bash
cdk synth
```

다음과 비슷한 결과를 확인 할 수 있습니다.
```term
Resources:
  LogBucketCC3B17E8:
    Type: AWS::S3::Bucket
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: LoggingWorkshopStack/LogBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.72.0,@aws-cdk/aws-events=1.72.0,@aws-cdk/aws-iam=1.72.0,@aws-cdk/aws-kms=1.72.0,@aws-cdk/aws-s3=1.72.0,@aws-cdk/cloud-assembly-schema=1.72.0,@aws-cdk/core=1.72.0,@aws-cdk/cx-api=1.72.0,@aws-cdk/region-info=1.72.0,jsii-runtime=node.js/v10.23.0
    Metadata:
      aws:cdk:path: LoggingWorkshopStack/CDKMetadata/Default
    Condition: CDKMetadataAvailable
Conditions:
  CDKMetadataAvailable:
    Fn::Or:
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-2
          - Fn::Equals:
              - Ref: AWS::Region
    ...
    ...
```

한 줄의 코드가 Bucket을 정의하는 여러 줄의 CloudFormation template을 생성하는 것을 확인 할 수 있습니다.

생성 된 template는 **cdk.out/LoggingWorkshopStack.template.json** 에서도 확인 가능 합니다.

![CF template](/images/log-stack/cdkout.png)

&nbsp;

CDK 앱을 처음 배포한다면 CDK toolkit에 필요한 리소스들을 먼저 배포 해야 합니다. 다음 명령어로 CDK toolkit stack을 배포 합니다.
```bash
cdk bootstrap
```

다음과 같이 CDK Toolkit에 필요한 리소스가 배포 됩니다.
```term
 ⏳  Bootstrapping environment aws://99999999999/us-east-1...
CDKToolkit: creating CloudFormation changeset...
[██████████████████████████████████████████████████████████] (3/3)
```

## CDK deploy
이제 CDK로 정의한 **S3 Bucket**을 배포 합니다.

N.Virginia (us-east-1) 리전에서 진행하기 위해 **bin/logging-workshop.ts** 의 코드를 다음과 같이 수정 후 저장 합니다.

Region 정보를 변수에 담아서 Stack을 호출할 때 전달 합니다.
```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LoggingWorkshopStack } from '../lib/logging-workshop-stack';

const envRegion = { region: 'us-east-1' };
const app = new cdk.App();
new LoggingWorkshopStack(app, 'LoggingWorkshopStack', { env: envRegion });

```


`cdk synth` 명령어로 정상적으로 template 변환이 되는지 확인 후, 다음 명령어를 터미널 창에 입력하여 배포 합니다.
```bash
cdk deploy
```

다음과 같이 Stack이 배포 됩니다.
```term
LoggingWorkshopStack: deploying...
LoggingWorkshopStack: creating CloudFormation changeset...
[██████████████████████████████████████████████████████████] (3/3)





 ✅  LoggingWorkshopStack

Outputs:
LoggingWorkshopStack.LogBucketName = loggingworkshopstack-logbucketcc3b17e8-15ipdr71mx1kd

Stack ARN:
arn:aws:cloudformation:us-east-1:...
```

CloudFormation 콘솔
![CloudFormation Console](/images/log-stack/cf-console.png)