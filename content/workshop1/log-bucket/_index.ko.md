---
title: 로그 버킷
weight: 200
pre: "<b>4-2. </b>"
---

## S3 bucket 생성
로그를 적재할 S3 버킷을 생성 합니다. 생성된 S3 버킷은 앞으로 작성할 여러 stack들 사이에서 공유되며 중앙 로그 버킷으로 사용될 것입니다.

Cloud9 IDE에서 **lib/log-bucket-stack.ts** 파일을 열어봅니다.

다음과 같이 기본 코드가 작성되어 있을 것입니다.

```typescript
import * as cdk from '@aws-cdk/core';

export class LogBucketStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

`import * as cdk from '@aws-cdk/core';` 라인 밑에 다음 코드를 추가하여 aws-s3 모듈을 import 해줍니다.
```typescript
import * as s3 from '@aws-cdk/aws-s3';
```

버킷을 공유하려면 버킷 object를 다른 stack에게 보내야 합니다.

`constructor` 위에 readonly 값을 public으로 정의합니다.
```typescript
    public readonly logBucket: s3.Bucket;
```

`constoructor`의 주석 `// The code that defines your stack goes here` 라인 밑에 다음 코드를 추가합니다.

s3 버킷을 생성하고 위에서 정의한 public 변수에 버킷 object를 지정 합니다.
```typescript
// Create s3 bucket to store logs
const bucket = new s3.Bucket(this, 'LogBucket', {
    bucketName: cdk.PhysicalName.GENERATE_IF_NEEDED
});

this.logBucket = bucket;
```

파일 하단에 다음 코드를 추가하여, 다른 stack에서 참조 할 interface를 export 합니다.
```typescript
export interface BucketProps extends cdk.StackProps {
  bucket: s3.Bucket;
}
```


완성된 코드는 다음과 같습니다.
```typescript
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

export class LogBucketStack extends cdk.Stack {
  public readonly logBucket: s3.Bucket;
  
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
    // Create s3 bucket to store logs
    const bucket = new s3.Bucket(this, 'LogBucket', {
      bucketName: cdk.PhysicalName.GENERATE_IF_NEEDED
    });
    
    this.logBucket = bucket;
  }
}

export interface BucketProps extends cdk.StackProps {
  bucket: s3.Bucket;
}
```

새로 열었던 터미널 창에 다음 명령어를 실행하여 CDK 코드가 정상적으로 CloudFormation template를 변환 되는지 확인 합니다.
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
      aws:cdk:path: LogBucketStack/LogBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.77.0,@aws-cdk/aws-events=1.77.0,@aws-cdk/aws-iam=1.77.0,@aws-cdk/aws-kms=1.77.0,@aws-cdk/aws-s3=1.77.0,@aws-cdk/cloud-assembly-schema=1.77.0,@aws-cdk/core=1.77.0,@aws-cdk/cx-api=1.77.0,@aws-cdk/region-info=1.77.0,jsii-runtime=node.js/v10.23.0
    Metadata:
      aws:cdk:path: LogBucketStack/CDKMetadata/Default
```


## 앤트리포인트에 스택 추가하기

**bin/centralized-logging-skeleton.ts** 파일을 열어 LogBucketStack을 추가할 것입니다.

다음 코드를 추가하여 LogBucketStack을 import 합니다.

```typescript
import { LogBucketStack } from '../lib/log-bucket-stack';
```

`const app = new cdk.App();` 코드 밑에 다음 코드를 추가합니다.

```typescript
new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
```

완성된 코드는 다음과 같습니다.
```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

new LogBucketStack(app, 'LogBucketStack', { env: envRegion });

```

## CDK deploy
이제 CDK로 정의한 **S3 Bucket**을 배포 합니다.

다음 명령어를 터미널 창에 입력하여 LogBucketStack을 배포 합니다.
```
cdk deploy
```

다음과 같이 Stack이 배포 됩니다.
```term
LogBucketStack: deploying...
LogBucketStack: creating CloudFormation changeset...
[██████████████████████████████████████████████████████████] (3/3)





 ✅  LogBucketStack

Stack ARN:
arn:aws:cloudformation:us-east-2:xxxxxxxxxxxx:stack/LogBucketStack/287f8780-3bc3-11eb-8b97-060f9727d0d0
```

&nbsp;
&nbsp;

[CloudFormation 콘솔](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=LogBucketStack&filteringStatus=active&viewNested=true&hideStacks=false)에 들어가면 정상적으로 리소스가 배포된 것을 확인할 수 있습니다.
![CloudFormation Console](/images/workshop1/cf-console.png)