---
title: "VPC Flow Log"
weight: 330
---
***

이 페이지에서는 웹서버 인스턴스를 배포 할 **VPC**와 **Public Subnet**을 생성 합니다.

그리고 VPC flow log를 생성하여 **Cloud Watch Log**와 **S3 Bucket**으로 보냅니다.

## VPC 생성
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

## VPC Flow Log
다음 코드도 추가하여 **VPC Flow Log**를 활성화 하고 해당 로그를 **CloudWatch Log**와 **S3 Bucket**으로 전송 합니다.

**S3 Bucket** 에는 **Reject** 된 트래픽만 전송 하도록 설정 하였습니다.
```typescript
// Create LogGroup for VPC Flow Log
const flowLogGroup = new logs.LogGroup(this, 'VpcFlowLogGroup', {
    logGroupName: 'VpcFlowLogGroup',
    retention: logs.RetentionDays.THREE_MONTHS
});

// Enalbe VPC flow log. Send it to the LogGroup
demoVpc.addFlowLog('FlowLogToLogGroup', {
    trafficType: ec2.FlowLogTrafficType.ALL,
    destination: ec2.FlowLogDestination.toCloudWatchLogs(flowLogGroup)
});
// Send the rejected flow log to s3 bucket
demoVpc.addFlowLog('FlowLogToS3', {
    trafficType: ec2.FlowLogTrafficType.REJECT,
    destination: ec2.FlowLogDestination.toS3(logBucket)});
```

