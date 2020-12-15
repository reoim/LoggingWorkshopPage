---
title: ECS 로그
weight: 600
pre: "<b>4-6. </b>"
---
***

이 페이지에서는 `CDK`를 이용하여 샘플 `ECS` 애플리케이션을 배포하고 `firelens` 로그 드라이버를 사용하여 `CloudWatch Log`에 로깅하는 설정을 알아볼 것입니다.

&nbsp;

## Cluster 생성

`ecs-stack.ts` 파일을 열어 다음 모듈들을 추가합니다.

```typescript
import * as logs from '@aws-cdk/aws-logs';
import * as ecs from '@aws-cdk/aws-ecs';
import * as ec2 from '@aws-cdk/aws-ec2';
import * as iam from '@aws-cdk/aws-iam';
import * as ecsPatterns from '@aws-cdk/aws-ecs-patterns';
import * as elbv2 from '@aws-cdk/aws-elasticloadbalancingv2';
```


`constructor`에 다음 코드를 추가하여  `ECS` 클러스터를 배포할 `VPC` 네트워크를 생성합니다.

```typescript
    // Create VPC
    const vpc = new ec2.Vpc(this, 'EcsDemoVpc', { maxAzs: 2 });
```

다음 코드를 추가하여 클러스터를 생성하고 컨테이너 모니터링을 위해 Container Insight도 활성화합니다.

```typescript
    // Create an ECS cluster
    const cluster = new ecs.Cluster(this, 'Cluster', { 
      vpc, 
      containerInsights: true
    });
```

다음 코드를 추가하여 autoscaling capacity를 추가합니다. 실습 목적이므로 autoscaling 상세 조건은 설정하지 않았습니다.
```typescript
    // Add capacity to it
    cluster.addCapacity('DefaultAutoScalingGroupCapacity', {
      instanceType: new ec2.InstanceType("t2.small"),
      desiredCapacity: 1
    });
```

&nbsp;

## CDK로 ECS 로깅 설정

다음 코드를 추가하여 ECS 로깅을 위한 로그 그룹을 생성합니다. 로그 보관기간은 1주일로 설정하였습니다.
```typescript
    // Create CloudtWatch LogGroup for ClouTrail log
    const ecsLogGroup = new logs.LogGroup(this, 'EcsLog', {
      logGroupName: 'EcsLog',
      retention: logs.RetentionDays.ONE_WEEK
    });
```

다음 코드를 추가하여 IAM role을 생성하고 위에서 생성한 로그 그룹에 대한 권한을 부여합니다.
```typescript
    // Add permissions of the log group to the role
    containerTaskRole.addToPolicy(new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: [ecsLogGroup.logGroupArn],
        actions: ['*']
    }));
```

ECS task definition에 해당 role을 task role으로 지정합니다.
```typescript
    // Define taskRole in task definition
    const taskDefinition = new ecs.Ec2TaskDefinition(this, 'TaskDef', {
      taskRole: containerTaskRole
    });
```

ECS 컨테이너를 정의합니다. firelens 로그드라이버를 사용하여 위에서 만들어둔 로그 그룹으로 컨테이너 로깅을 설정합니다.

firelens option에 대한 내용은 [amazon-ccloudwatch-logs-for-fluent-bit](https://github.com/aws/amazon-cloudwatch-logs-for-fluent-bit)에서 확인할수 있습니다. 

```typescript
    // Define taskRole in task definition
    const taskDefinition = new ecs.Ec2TaskDefinition(this, 'TaskDef', {
      taskRole: containerTaskRole
    });
    
    const container = taskDefinition.addContainer('EcsSampleContainer', {
      image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample"),
      memoryLimitMiB: 512, 
      logging: ecs.LogDrivers.firelens({
        options: {
            Name: 'cloudwatch',
            region: 'us-east-2',
            log_group_name: 'EcsLog',
            log_stream_name: 'from-fluent-bit',
            auto_create_group: 'true'
        }
      })
    });
```

&nbsp;

## 로드밸런서 추가 및 외부 노출

다음 코드를 추가하여 호스트와 컨테이너의 port를 맵핑하고 ECS 서비스를 설정, 외부 노출 Application load balancer를 추가하여 ECS 서비스와 연결합니다.
```typescript
    // Container port mapping
    container.addPortMappings({
      containerPort: 80,
      hostPort: 8080,
      protocol: ecs.Protocol.TCP
    });
    
    // Instantiate an Amazon ECS Service
    const ecsService = new ecs.Ec2Service(this, 'Service', {
      cluster,
      taskDefinition
    });

    // Define application load balancer
    const lb = new elbv2.ApplicationLoadBalancer(this, 'LB', {
      vpc,
      internetFacing: true
      });
      
    const listener = lb.addListener('PublicListener', { port: 80, open: true });
      
    listener.addTargets('ECS', {
      port: 80,
      targets: [ecsService]
    });
```

&nbsp;

완성된 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import * as logs from '@aws-cdk/aws-logs';
import * as ecs from '@aws-cdk/aws-ecs';
import * as ec2 from '@aws-cdk/aws-ec2';
import * as iam from '@aws-cdk/aws-iam';
import * as ecsPatterns from '@aws-cdk/aws-ecs-patterns';
import * as elbv2 from '@aws-cdk/aws-elasticloadbalancingv2';

export class EcsStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    
    // Create VPC
    const vpc = new ec2.Vpc(this, 'EcsDemoVpc', { maxAzs: 2 });
    
    // Create an ECS cluster
    const cluster = new ecs.Cluster(this, 'Cluster', { 
      vpc, 
      containerInsights: true
    });
    
    // Add capacity to it
    cluster.addCapacity('DefaultAutoScalingGroupCapacity', {
      instanceType: new ec2.InstanceType("t2.small"),
      desiredCapacity: 1
    });
    
    // Create CloudtWatch LogGroup for ClouTrail log
    const ecsLogGroup = new logs.LogGroup(this, 'EcsLog', {
      logGroupName: 'EcsLog',
      retention: logs.RetentionDays.ONE_WEEK
    });
    
    // Create IAM role for ECS
    const containerTaskRole = new iam.Role(this, "TaskRole", {
      assumedBy: new iam.ServicePrincipal('ecs-tasks.amazonaws.com')
    });
    
    // Add permissions of the log group to the role
    containerTaskRole.addToPolicy(new iam.PolicyStatement({
        effect: iam.Effect.ALLOW,
        resources: [ecsLogGroup.logGroupArn],
        actions: ['*']
    }));
    
    // Define taskRole in task definition
    const taskDefinition = new ecs.Ec2TaskDefinition(this, 'TaskDef', {
      taskRole: containerTaskRole
    });
    
    const container = taskDefinition.addContainer('EcsSampleContainer', {
      image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample"),
      memoryLimitMiB: 512, 
      logging: ecs.LogDrivers.firelens({
        options: {
            Name: 'cloudwatch',
            region: 'us-east-2',
            log_group_name: 'EcsLog',
            log_stream_name: 'from-fluent-bit',
            auto_create_group: 'true'
        }
      })
    });
    
    // Container port mapping
    container.addPortMappings({
      containerPort: 80,
      hostPort: 8080,
      protocol: ecs.Protocol.TCP
    });
    
    // Instantiate an Amazon ECS Service
    const ecsService = new ecs.Ec2Service(this, 'Service', {
      cluster,
      taskDefinition
    });
    
    // Define application load balancer
    const lb = new elbv2.ApplicationLoadBalancer(this, 'LB', {
      vpc,
      internetFacing: true
      });
      
    const listener = lb.addListener('PublicListener', { port: 80, open: true });
      
    listener.addTargets('ECS', {
      port: 80,
      targets: [ecsService]
    });

  }
}
```

&nbsp;

## 앤트리포인트에 스택 추가하기
`bin/centralized-logging-skeleton.ts` 파일을 열어 EcsStack을 추가합니다.

다음 코드를 추가하여 EcsStack을 import 합니다.

```typescript
import { EcsStack } from '../lib/ecs-stack';
```

파일 하단에 다음 코드를 추가하여 스택을 추가합니다.

```typescript
new EcsStack(app, 'EcsStack', { env: envRegion });
```

완성된 코드는 다음과 같을 것입니다.

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';
import { CloudtrailStack } from '../lib/cloudtrail-stack';
import { WebServerStack } from '../lib/webserver-stack';
import { ServerlessStack } from '../lib/serverless-stack';
import { EcsStack } from '../lib/ecs-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

const logBucketStack = new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
new CloudtrailStack(app, 'CloudtrailStack', { env: envRegion, bucket:logBucketStack.logBucket });
new WebServerStack(app, 'WebServerStack', { env: envRegion, bucket:logBucketStack.logBucket });
new ServerlessStack(app, 'ServerlessStack', { env: envRegion });
new EcsStack(app, 'EcsStack', { env: envRegion });
```

&nbsp;

## 배포 및 테스트

터미널에 다음 명령어를 입력하여 EcsStack을 배포합니다.

```
cdk deploy EcsStack
```

배포가 완료되면 [ECS 콘솔](https://us-east-2.console.aws.amazon.com/ecs/home?region=us-east-2#/clusters)로 이동하여 클러스터 및 서비스가 정상적으로 배포되었는지 확인합니다.

![ECS 콘솔](/images/workshop1/ecs-console.png)

[ELB 콘솔](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#LoadBalancers:sort=loadBalancerName)로 이동하여 로드벨런서의 DNS 주소를 복사합니다.

![ELB 콘솔](/images/workshop1/elb.png)

브라우저에 복사한 DNS 주소를 입력하여 샘플 앱이 잘 구동하는지 확인합니다.

![ECS 테스트](/images/workshop1/ecs-test.png)

[CloudWatch Log 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#logsV2:log-groups)로 이동하여 로그 그룹이 잘 생성되었는지 확인합니다.

![ECS log groups](/images/workshop1/ecs-log.png)

직접 정의한 EcsLog 로그 그룹 외에도 firelens 로그 라우터의 기동 로그와 Container Insight 퍼포먼스 로그용 로그 그룹도 함께 추가된 것을 확인할수 있습니다.

각 로그 그룹을 클릭하여 로그들을 살펴보시길 바랍니다.

