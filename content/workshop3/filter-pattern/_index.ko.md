---
title: "로그 필터로 지표 생성"
weight: 100
pre: "<b>6-1. </b>"
---
***

로그 `필터 패턴`을 사용하여 `지표 필터`를 생성하고 로그 이벤트에서 일치하는 단어, 구문 도는 값을 검색할 수 있습니다. `지표 필터`가 로그 이벤트에서 일치되는 값을 찾으면 `CloudWatch 지표`의 값을 증가시키도록 설정 할 수 있습니다. 예를 들어 로그 이벤트에서 ERROR 라는 단어를 검색하는 지표 필터를 생성하여 발생 횟수를 `CloudWatch 지표`에 게시하도록 할 수 있습니다.

이 페이지에서는 `CloudTrail` 로그 그룹에 지표 필터를 생성하여 누군가 `CloudTrail`을 생성, 수정, 삭제, 로깅 중지, 로깅 시작 등의 작업을 수행할 경우  `CloudWatch 알람`을 발생하도록 설정 할 것입니다.

## 지표 필터 및 지표 생성

`lib/cloudtrail-stack.ts` 파일을 열고 다음 코드를 추가하여 필요한 모듈들을 import 합니다.

```typescript
import * as sns from '@aws-cdk/aws-sns';
import * as subs from '@aws-cdk/aws-sns-subscriptions';
import * as cloudwatch from '@aws-cdk/aws-cloudwatch';
import * as cw_actions from '@aws-cdk/aws-cloudwatch-actions';
```

`constructor`의 하단부에 다음 코드를 추가하여 지표 필터를 추가합니다.

```typescript
    // Create metric filter for console sign-in failure 
    const trailMetricFilter = new logs.MetricFilter(this, 'CloudTrailMetricFilter', {
        logGroup: trailLogGroup,
        metricNamespace: 'CloudTrailMetrics',
        metricName: 'CloudTrailChanges',
        filterPattern: logs.FilterPattern.any(
        logs.FilterPattern.stringValue('$.eventName','=', 'CreateTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'UpdateTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'DeleteTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'StartLogging'),
        logs.FilterPattern.stringValue('$.eventName','=', 'StopLogging')),
        metricValue: '1'
    });
```

위 코드에서 로그 필터 패턴은 JSON 패턴을 사용하여 CloudTrail의 변경 이벤트를 필터링 하도록 설정하였으며 일치되는 값이 생길때 지표 값을 1씩 증가 하도록 설정하였습니다.

필터 패턴에서 지원하는 타입은 총 3가지로 Text, JSON, Space-delimitied table 패턴 입니다.

각 패턴별 CDK 문법은 다음을 참고하세요. [CDK API reference - log patterns](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-logs-readme.html#patterns)

필터 및 패턴구문에 대한 자세한 내용은 [User Guide - Filter and Pattern Syntax
](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html)를 참고하세요.


&nbsp;

## 알람 생성 및 SNS 연동

위에서 만든 지표 필터를 이용하여 `CloudWatch 지표`를 생성하고 해당 지표를 사용하여 `CloudWatch 알람`을 생성합니다.

다음 코드를 추가하세요. 5분(default period) 내에 1번 이상의 `CloudTrail` 변경이 일어날 경우 알람을 발생시키는 코드 입니다.

```typescript
    // Expose a metric form the metric filter
    const trailChangeMetric = trailMetricFilter.metric();
  
    // Create CloudWatch alarm for 3 times sign-in failure
    const alarm = new cloudwatch.Alarm(this, 'TrailAlarm', {
        alarmName:'CloudTrailChanged',
        metric: trailChangeMetric,
        threshold: 1,
        evaluationPeriods: 1,
        statistic: 'Sum'
    });
```

다음 코드를 추가하여 SNS 토픽과 구독을 생성하고 알람 액션에 토픽을 등록합니다.

```typescript
    // Create SNS topic and subscription
    const topic = new sns.Topic(this, 'TrailTopic');
    const email = this.node.tryGetContext('email');
    topic.addSubscription(new subs.EmailSubscription(email));
    
    // Add alarm action. (send it to SNS topic)
    alarm.addAlarmAction(new cw_actions.SnsAction(topic));
```

context의 `email` 값을 본인의 email로 변경합니다.

`cdk.json` 파일의 열어봅니다.

```json
{
  "app": "npx ts-node --prefer-ts-exts bin/centralized-logging-skeleton.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": "true",
    "aws-cdk:enableDiffNoFail": "true",
    "@aws-cdk/core:stackRelativeExports": "true",
    "@aws-cdk/aws-ecr-assets:dockerIgnoreSupport": true,
    "@aws-cdk/aws-secretsmanager:parseOwnedSecretName": true,
    "destination":"arn:aws:logs:{region}:{accountid}:destination:{log-destination-name}",
    "email": "example@email.com"
  }
}
```

위의 `example@email.com` 값을 SNS 구독에 사용할 본인의 email 주소로 변경 후 저장합니다.

&nbsp;

## 배포하기

완성된 `lib/cloudtrail-stack.ts` 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import * as logs from '@aws-cdk/aws-logs';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
import { BucketProps } from './log-bucket-stack';
import * as sns from '@aws-cdk/aws-sns';
import * as subs from '@aws-cdk/aws-sns-subscriptions';
import * as cloudwatch from '@aws-cdk/aws-cloudwatch';
import * as cw_actions from '@aws-cdk/aws-cloudwatch-actions';

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
    
    // Create metric filter for console sign-in failure 
    const trailMetricFilter = new logs.MetricFilter(this, 'CloudTrailMetricFilter', {
        logGroup: trailLogGroup,
        metricNamespace: 'CloudTrailMetrics',
        metricName: 'CloudTrailChanges',
        filterPattern: logs.FilterPattern.any(
        logs.FilterPattern.stringValue('$.eventName','=', 'CreateTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'UpdateTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'DeleteTrail'),
        logs.FilterPattern.stringValue('$.eventName','=', 'StartLogging'),
        logs.FilterPattern.stringValue('$.eventName','=', 'StopLogging')),
        metricValue: '1'
    });
    
    // Expose a metric form the metric filter
    const trailChangeMetric = trailMetricFilter.metric();
  
    // Create CloudWatch alarm for 3 times sign-in failure
    const alarm = new cloudwatch.Alarm(this, 'TrailAlarm', {
        alarmName:'CloudTrailChanged',
        metric: trailChangeMetric,
        threshold: 1,
        evaluationPeriods: 1,
        statistic: 'Sum'
    });
    
    // Create SNS topic and subscription
    const topic = new sns.Topic(this, 'TrailTopic');
    const email = this.node.tryGetContext('email');
    topic.addSubscription(new subs.EmailSubscription(email));
    
    // Add alarm action. (send it to SNS topic)
    alarm.addAlarmAction(new cw_actions.SnsAction(topic));
    

  }
}
```

터미널 창에서 해당 CDK 앱 폴더로 이동한 뒤 다음 명령어로 배포합니다.

```
cdk deploy CloudtrailStack
```

배포가 완료되면 등록한 email 주소로 구독 확인 메일이 올 것입니다.

![Subscription confirm email](/images/workshop3/sns1.png)

`Confirm subscription` 을 클릭하여 구독을 확인 합니다.

![Subscription confirm email](/images/workshop3/sns2.png)

&nbsp;

## 테스트

[CloudTrail 콘솔](https://us-east-2.console.aws.amazon.com/cloudtrail/home?region=us-east-2#/dashboard)로 이동하여 `CloudTrailStack`으로 생성했던 `CloudTrail`을 선택합니다.

![trail change](/images/workshop3/trail1.png)

&nbsp;

`Stop logging` 을 클릭하여 로깅을 중지합니다.

![trail change](/images/workshop3/trail2.png)

&nbsp;

`Start logging` 을 클릭하여 로깅을 다시 시작합니다.

![trail change](/images/workshop3/trail3.png)

&nbsp;

알람 발생 조건의 period 가 5분으로 설정되어 있기 때문에 약 5분 후에 알람이 발생한 것을 확인할 수 있습니다.

![trail change](/images/workshop3/trail4.png)

&nbsp;

이메일을 확인하여 SNS 메세지가 발송된 것을 확인합니다.

![trail change](/images/workshop3/trail5.png)
