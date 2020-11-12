---
title: "CloudTrail Log"
weight: 320
---
***
이 페이지에서는 **CloudTrail Log**를 활성화 하고 **CloudWatch Log**와 **S3 Bucket**에 적재하는 방법과

**Metric Filter**를 이용하여 3번 이상 Sign-in 실패시 알람을 발생하여 **Amazon SNS**로 notification을 받도록 구성하는 방법을 알아봅니다.

&nbsp;

&nbsp;

## CloudTrail Log를 S3 bucket과 CloudWatch Log에 적재
**lib/logging-wokrshop-stack.ts** 의 상단에 다음의 모듈들을 추가로 import 해줍니다.
```typescript
import * as logs from '@aws-cdk/aws-logs';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
import * as cloudwatch from '@aws-cdk/aws-cloudwatch';
import * as cw_actions from '@aws-cdk/aws-cloudwatch-actions';
import * as sns from '@aws-cdk/aws-sns';
import * as subs from '@aws-cdk/aws-sns-subscriptions';
```

S3 bucket을 정의하였던 코드 밑에 다음 코드를 추가 후 저장 합니다.

CloudTrail log를 적재하기 위한 LogGroup을 생성하고 CloudTrail 로그를 S3 bucket과 LogGroup으로 보내도록 하는 코드 입니다.
```typescript
// Create CloudtWatch LogGroup for ClouTrail log
const trailLogGroup = new logs.LogGroup(this, 'TrailLog', {
    logGroupName: 'TrailLog',
    retention: logs.RetentionDays.THREE_MONTHS
});

// Send CloudTrail log to logGroup and bucket
const trail = new cloudtrail.Trail(this, 'CloudTrail', {
    cloudWatchLogGroup:trailLogGroup,
    sendToCloudWatchLogs: true,
    bucket: logBucket,
    includeGlobalServiceEvents: true,
    isMultiRegionTrail: true
});
```
&nbsp;

&nbsp;

## Metric 생성 및 알람 설정
CloudWatch Log에 적재되는 로그 데이터에 Metric filter를 적용하여 원하는 Metric을 생성하고, 해당 Metric으로 CloudWatch Alarm을 생성 할 수 있습니다.



다음 코드를 추가하여 Sign-in 실패 (3번 실패)에 대한 Metric과 Alarm을 설정하고 Amazon SNS을 통하여 email로 notification을 받을 수 있도록 설정 합니다.

```typescript
// Create metric filter for console sign-in failure 
const signInFailureMetricFilter = new logs.MetricFilter(this, 'SignInFailMetricFilter', {
    logGroup: trailLogGroup,
    metricNamespace: 'CloudTrailMetrics',
    metricName: 'ConsoleSigninFailureCount',
    filterPattern: logs.FilterPattern.all(
    logs.FilterPattern.stringValue('$.eventName','=', 'ConsoleLogin'),
    logs.FilterPattern.stringValue('$.errorMessage','=', 'Failed authentication')),
    metricValue: '1'
});

// Create CloudWatch alarm for 3 times sign-in failure
const alarm = new cloudwatch.Alarm(this, 'TrailAlarm', {
    alarmName:'ConsoleSignInFailures',
    metric: signInFailureMetricFilter.metric(),
    threshold: 3,
    evaluationPeriods: 1,
    statistic: 'Sum'
});

// Create SNS topic ans subscription
const topic = new sns.Topic(this, 'TrailTopic');
const email = 'my-email@email.com';
topic.addSubscription(new subs.EmailSubscription(email));

// Add alarm action. (send it to SNS topic)
alarm.addAlarmAction(new cw_actions.SnsAction(topic));
```

위 코드에서 SNS subscription 에 사용 할 email주소가 아래와 같이 하드코딩 되어 있습니다.
```typescript
const email = 'my-email@email.com';
```

배포 관리의 편의성을 위해 email 주소를 context에 저장하고 [context value를 호출](https://docs.aws.amazon.com/cdk/latest/guide/get_context_var.html)하는 방식으로 변경 하겠습니다.

**cdk.json** 파일을 열어서 다음과 같이 **"email"** 필드를 추가한 뒤 본인이 sns notification을 받아 볼 email  주소를 입력하고 저장 합니다.
```json
{
  "app": "npx ts-node bin/logging-workshop.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": "true",
    "aws-cdk:enableDiffNoFail": "true",
    "@aws-cdk/core:stackRelativeExports": "true",
    "email": "my-email@email.com"
  }
}
```
다시 **lib/logging-wokrshop-stack.ts** 에 하드 코딩된 email 변수 부분을 다음과 같이 수정 합니다.
```typescript
const email = this.node.tryGetContext('email');
```

지금까지 수정 된 전체 코드는 다음과 같습니다.
```typescript
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';
import * as logs from '@aws-cdk/aws-logs';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
import * as cloudwatch from '@aws-cdk/aws-cloudwatch';
import * as cw_actions from '@aws-cdk/aws-cloudwatch-actions';
import * as sns from '@aws-cdk/aws-sns';
import * as subs from '@aws-cdk/aws-sns-subscriptions';

export class LoggingWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    
    // The code that defines your stack goes here
    
    // Create s3 bucket
    const logBucket = new s3.Bucket(this, 'LogBucket');
    
    // CloudFormation output for the bucket name
    new cdk.CfnOutput(this, 'LogBucketName', { value: logBucket.bucketName });
    
    // Create CloudtWatch LogGroup for ClouTrail log
    const trailLogGroup = new logs.LogGroup(this, 'TrailLog', {
      logGroupName: 'TrailLog',
      retention: logs.RetentionDays.THREE_MONTHS
    });
    
    // Send CloudTrail log to logGroup and bucket
    const trail = new cloudtrail.Trail(this, 'CloudTrail', {
      cloudWatchLogGroup:trailLogGroup,
      sendToCloudWatchLogs: true,
      bucket: logBucket,
      includeGlobalServiceEvents: true,
      isMultiRegionTrail: true
    });
    
    // Create metric filter for console sign-in failure 
    const signInFailureMetricFilter = new logs.MetricFilter(this, 'SignInFailMetricFilter', {
        logGroup: trailLogGroup,
        metricNamespace: 'CloudTrailMetrics',
        metricName: 'ConsoleSigninFailureCount',
        filterPattern: logs.FilterPattern.all(
        logs.FilterPattern.stringValue('$.eventName','=', 'ConsoleLogin'),
        logs.FilterPattern.stringValue('$.errorMessage','=', 'Failed authentication')),
        metricValue: '1'
    });
  
    // Create CloudWatch alarm for 3 times sign-in failure
    const alarm = new cloudwatch.Alarm(this, 'TrailAlarm', {
        alarmName:'ConsoleSignInFailures',
        metric: signInFailureMetricFilter.metric(),
        threshold: 3,
        evaluationPeriods: 1,
        statistic: 'Sum'
    });
    
    // Create SNS topic ans subscription
    const topic = new sns.Topic(this, 'TrailTopic');
    const email = this.node.tryGetContext('email');
    topic.addSubscription(new subs.EmailSubscription(email));
    
    // Add alarm action. (send it to SNS topic)
    alarm.addAlarmAction(new cw_actions.SnsAction(topic));
  }
}

```

터미널 창에 `cdk diff` 명령어를 입력하여 이미 배포된 Stack과 어떤 변경점이 있는지 확인 할 수 있습니다.
![cdk diff](/images/log-stack/diff.png)

&nbsp;

## Deploy
```bash
cdk deploy
```
터미널 창에 위 명령어를 입력하여 stack을 배포 합니다.

변경 된 내용을 배포 할것인지 묻는 질문에 **y** 를 입력하여 배포를 진행 합니다.

정상적으로 배포 된 것을 아래와 같이 확인 할 수 있습니다.
![deploy result](/images/log-stack/deploy_result.png)

CloudFormation 콘솔
![deploy result](/images/log-stack/console-result.png)

[CloudWatch Log 콘솔](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups) 에 TrailLog의 Log Stream이 생성 된 것을 확인 할 수 있습니다.

![CloudTrail Log](/images/log-stack/lg-trail.png)

**S3 Bucket**에도 CloudTrail Log가 생성 된 것을 확인 합니다.

s3 prefix key를 따로 설정하지 않았기 때문에 기본적으로 `AWSlogs/AccountID/`를 prefix key로 생성 됩니다.
![S3 TrailLog](/images/log-stack/s3-trail.png)

&nbsp;

## Amazon SNS Email 구독 확인
**cdk.json**에 입력한 이메일 주소로 다음과 같은 구독 확인 메일이 수분내로 발송 될 것입니다.

이메일 본문의 Confirm subscription 을 클릭하여 구독을 확인 합니다.
![email subscription](/images/log-stack/email.png)

정상적으로 구독 확인이 되면 웹 브라우저가 오픈되면서 다음과 같은 화면이 나타날 것 입니다.
![email confirm](/images/log-stack/email-confirm.png)