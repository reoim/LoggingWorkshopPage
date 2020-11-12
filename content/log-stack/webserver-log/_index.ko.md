---
title: "VPC, Instance Log"
weight: 330
---
***

이 페이지에서는 **VPC Flow log**와 **CloudWatch log agent**를 인스턴스에 설치하여 각종 인스턴스 metric 수집 및 인스턴스 Log를 **CloudWatch Log**로 스트림 하는 구성을 만들어 볼 것 입니다.

## VPC Flow Log
**lib/logging-wokrshop-stack.ts** 의 상단에 다음의 모듈들을 추가로 import 해줍니다.
```typescript
import * as ec2 from '@aws-cdk/aws-ec2';
import * as iam from '@aws-cdk/aws-iam';

import fs = require('fs');
```

[CloudTrail Log 페이지](../cloudtrail-log)에서 정의한 코드 밑에 다음 코드를 추가하여 인스턴스를 배포 할 VPC와 Public Subnet을 생성 합니다.

```typescript
// Create VPC
const demoVpc = new ec2.Vpc(this, 'DemoVpc', {
    cidr: "192.168.0.0/16",
    subnetConfiguration: [
    {
        cidrMask: 24,
        name: 'webserver',
        subnetType: ec2.SubnetType.PUBLIC
    }]
});
```

다음 코드도 추가하여 VPC Flow Log를 활성화 하고 해당 로그를 CloudWatch Log와 S3 bucket으로 전송 합니다.
```typescript
// Create LogGroup for VPC Flow Log
const flowLogGroup = new logs.LogGroup(this, 'VpcFlowLogGroup', {
    logGroupName: 'VpcFlowLogGroup',
    retention: logs.RetentionDays.THREE_MONTHS
});

// Enalbe VPC flow log. Send it to the LogGroup
demoVpc.addFlowLog('FlowLogToLogGroup', {destination: ec2.FlowLogDestination.toCloudWatchLogs(flowLogGroup)});
// Send the flow log to s3 bucket
demoVpc.addFlowLog('FlowLogToS3', {destination: ec2.FlowLogDestination.toS3(logBucket)});
```