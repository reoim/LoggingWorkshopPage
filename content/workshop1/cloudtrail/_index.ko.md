---
title: CloudTrail 로그
weight: 300
pre: "<b>4-3. </b>"
---

`CloudTrail` 로그를 `CloudWatch Logs`와 `S3` 버킷에 수집하도록 활성화할 것입니다.

`CloudTrail`은 기본적으로 로그를 `S3`에 직접 내보낼수 있습니다.

## 중앙 로그 버킷 공유

먼저 중앙 로그 버킷 object를 props로 전달 받기 위해 `lib/cloudtrail-stack.ts` 파일을 열어 다음과 같이 수정합니다.

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

export class CloudtrailStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

## CloudTrail 로그 수집
`lib/cloudtrail-strack.ts` 파일 상단에 다음의 모듈들을 추가로 import 해줍니다.

```typescript
import * as logs from '@aws-cdk/aws-logs';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
```

`constructor` 안에 다음 코드를 추가 후 저장합니다.

`CloudWatch Logs` 로그 그룹을 생성하고 `CloudTrail` 로그를 중앙 로그 버킷과 로그 그룹으로 보내도록 하는 코드 입니다.

로그 그룹의 로그 보관기간은 1주일로 설정하였습니다.

```typescript
    // Create CloudtWatch LogGroup for ClouTrail log
    const trailLogGroup = new logs.LogGroup(this, 'TrailLog', {
      logGroupName: 'TrailLog',
      retention: logs.RetentionDays.ONE_WEEK
    });
    
    // Send CloudTrail log to logGroup and bucket
    const trail = new cloudtrail.Trail(this, 'CloudTrail', {
      cloudWatchLogGroup:trailLogGroup,
      sendToCloudWatchLogs: true,
      bucket: props.bucket,
      includeGlobalServiceEvents: true,
      isMultiRegionTrail: true
    });
```

완성된 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import * as logs from '@aws-cdk/aws-logs';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
import { BucketProps } from './log-bucket-stack';

export class CloudtrailStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
    // Create CloudtWatch LogGroup for ClouTrail log
    const trailLogGroup = new logs.LogGroup(this, 'TrailLog', {
      logGroupName: 'TrailLog',
      retention: logs.RetentionDays.ONE_WEEK
    });
    
    // Send CloudTrail log to logGroup and bucket
    const trail = new cloudtrail.Trail(this, 'CloudTrail', {
      cloudWatchLogGroup:trailLogGroup,
      sendToCloudWatchLogs: true,
      bucket: props.bucket,
      includeGlobalServiceEvents: true,
      isMultiRegionTrail: true
    });
  }
}
```

&nbsp;

## 엔트리포인트에 스택 추가하기
`bin/centralized-logging-skeleton.ts` 파일을 열어 스택을 추가할 것입니다.

다음과 같이 코드를 수정하여 CloudTrailStack을 추가하고 로그 버킷 object를 전달 합니다.

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';
import { CloudtrailStack } from '../lib/cloudtrail-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

const logBucketStack = new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
new CloudtrailStack(app, 'CloudtrailStack', { env: envRegion, bucket:logBucketStack.logBucket });
```
&nbsp;

## 배포하기
터미널 창에 다음 명령어를 입력하여 배포되는 인프라의 변경 사항을 확인 합니다.

CDK 코드가 정상적으로 컴파일 된다면 다음과 비슷한 결과를 볼수 있습니다.

![diff](/images/workshop1/diff.png)

로그 그룹과 로그 버킷에 `CloudTrail` 로그를 적재하기 위한 `IAM Role`과 `IAM Policy`가 자동으로 추가 된 것을 확인 할 수 있습니다.

다음 명령어로 CloudtrailStack을 배포합니다.

```
cdk deploy CloudtrailStack
```

각 stack 별로 다음과 같이 변경 사항에 대하여 배포를 진행할 것인지 승인을 요구할 것입니다. 내용을 살펴본 뒤 `y`를 입력하여 배포합니다.
```term
Do you wish to deploy these changes (y/n)? y
```

배포가 완료되면 [CloudFormation 콘솔](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=CloudtrailStack&filteringStatus=active&viewNested=true&hideStacks=false)에서 배포된 CloudtrailStack을 확인합니다.

&nbsp;

## 로그 확인
`CloudFormation 콘솔`에서 CloudtrailStack의 `Resources` 탭을 클릭하여 배포된 리소스들을 확인합니다.

![CloudtrailStack](/images/workshop1/trail-check.png)

`TrailLog` 로그 그룹을 클릭하여 CloudWatch Log에 정상적으로 로깅이 되는지 확인합니다.

![TrailLog](/images/workshop1/trail-loggroup-check.png)

`CloudFormation 콘솔`에서 LogBucketStack의 `Resources` 탭을 확인 후 로그 버킷으로 이동합니다.

![LogBucketStack](/images/workshop1/bucket-stack.png)

default prefix 인 AWSLogs/{YourAccountID}/ 경로에 CloudTrail 로그가 생성 되었는지 확인합니다.

![Log Bucket](/images/workshop1/trail-bucket.png)

&nbsp;

## AWS Explorer
`Cloud9` 의 `AWS Explorer`를 이용하면 편리하게 IDE에서 AWS 리소스 배포 상태를 확인 할 수 있습니다.

![AWS toolkit](/images/workshop1/aws-explore.png)