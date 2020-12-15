---
title: VPC Flow 로그
weight: 410
pre: "<b>4-4.a. </b>"
---
***

## VPC 생성 

다음 코드를 추가하여 필요한 모듈들을 import 합니다.

```typescript
import * as ec2 from '@aws-cdk/aws-ec2';
import * as logs from '@aws-cdk/aws-logs';
import * as iam from '@aws-cdk/aws-iam';
import fs = require('fs');
```

`constructor`에 다음 코드를 추가하여 VPC를 생성하고 웹서버를 배포할 public subnet을 만듭니다.

```typescript
    // Create VPC
    const demoVpc = new ec2.Vpc(this, 'DemoVpc', {
        cidr: "192.168.0.0/16",
        subnetConfiguration: [
        {
            cidrMask: 24,
            name: 'demo-public-subnet',
            subnetType: ec2.SubnetType.PUBLIC
        }]
    });
```

위와 같이 `VPC`를 `CDK`로 생성하면 기본적으로 필요한 `Internet Gateway`, `NAT Gateway` 등이 자동으로 생성되며 라우팅 룰도 자동으로 정의 됩니다.

&nbsp;

## VPC Flow 로그 활성화

다음 코드를 추가하여 `VPC Flow Log`를 저장할 로그 그룹을 생성 합니다. 로그 보관 기간은 1주일로 지정하였습니다.

```typescript
    // Create LogGroup for VPC Flow Log
    const flowLogGroup = new logs.LogGroup(this, 'VpcFlowLogGroup', {
      logGroupName: 'VpcFlowLogGroup',
      retention: logs.RetentionDays.ONE_WEEK
    });
```

다음 코드를 추가하여 `VPC Flow Log`를 생성한 로그 그룹과 로그 버킷에 적재하도록 구성합니다.

트래픽은 Reject 된 트래픽만 수집하도록 설정하였습니다.

```typescript
    // Enalbe VPC flow log. Send it to the LogGroup
    demoVpc.addFlowLog('FlowLogToLogGroup', {
      trafficType: ec2.FlowLogTrafficType.REJECT,
      destination: ec2.FlowLogDestination.toCloudWatchLogs(flowLogGroup)
    });
    // Send the rejected flow log to the log bucket
    demoVpc.addFlowLog('FlowLogToS3', {
      trafficType: ec2.FlowLogTrafficType.REJECT,
      destination: ec2.FlowLogDestination.toS3(props.bucket)
    });
```

수정된 코드는 다음과 같습니다.
```typescript
import * as cdk from '@aws-cdk/core';
import * as ec2 from '@aws-cdk/aws-ec2';
import * as logs from '@aws-cdk/aws-logs';
import * as iam from '@aws-cdk/aws-iam';
import fs = require('fs');
import { BucketProps } from './log-bucket-stack';

export class WebServerStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
    // Create VPC
    const demoVpc = new ec2.Vpc(this, 'DemoVpc', {
        cidr: "192.168.0.0/16",
        subnetConfiguration: [
        {
            cidrMask: 24,
            name: 'demo-public-subnet',
            subnetType: ec2.SubnetType.PUBLIC
        }]
    });
    
    // Create LogGroup for VPC Flow Log
    const flowLogGroup = new logs.LogGroup(this, 'VpcFlowLogGroup', {
      logGroupName: 'VpcFlowLogGroup',
      retention: logs.RetentionDays.ONE_WEEK
    });
    
    // Enalbe VPC flow log. Send it to the LogGroup
    demoVpc.addFlowLog('FlowLogToLogGroup', {
      trafficType: ec2.FlowLogTrafficType.REJECT,
      destination: ec2.FlowLogDestination.toCloudWatchLogs(flowLogGroup)
    });
    // Send the rejected flow log to s3 bucket
    demoVpc.addFlowLog('FlowLogToS3', {
      trafficType: ec2.FlowLogTrafficType.REJECT,
      destination: ec2.FlowLogDestination.toS3(props.bucket)
    });

  }
}