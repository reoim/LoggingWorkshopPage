---
title: Unified CloudWatch Agent
weight: 420
pre: "<b>4-4.b. </b>"
---
***

이 페이지에서는 인스턴스에 Apache 웹서버를 설치, 배포하고

 [통합 CloudWatch agent](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)를 설치하여 시스템 수준 지표 및 Apache 웹서버의 access/error 로그를 수집 하도록 설정 합니다.

&nbsp;

## 통합 CloudWatch agent 설정 파일

`resources/cfn-init/amazon-cloudwatch-agent.json` 파일을 열어 미리 작성된 `CloudWatch agent 설정 파일`을 살펴봅니다.

```json
{
  "agent": {
     "metrics_collection_interval": 60,
     "region": "us-east-2",
     "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log"
  },
  
  "metrics": {
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ]
      },
      "swap": {
        "measurement": [
          "swap_used_percent"
        ]
      },
      "processes": {
        "measurement": [
          "running",
          "sleeping",
          "dead"
        ]
      }
    },
    "append_dimensions": {
      "ImageId": "${aws:ImageId}",
      "InstanceId": "${aws:InstanceId}",
      "InstanceType": "${aws:InstanceType}",
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}"
    },
    "aggregation_dimensions" : [["AutoScalingGroupName"],["InstanceId", "InstanceType"],[]]
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
            "log_group_name": "WebServerLogGroup",
            "log_stream_name": "amazon_cloudwatch_agent_log"
          },
          {
            "file_path":"/var/log/httpd/access_log",
            "log_group_name":"WebServerLogGroup",
            "log_stream_name": "apache_access_log"
          },
          {
            "file_path":"/var/log/httpd/error_log",
            "log_group_name":"WebServerLogGroup",
            "log_stream_name": "apache_error_log"
          }
        ]
      }
    }
  }
}
```
위 구성은 총 3가지 시스템 지표(memory, swap, process)와 CldouWatch Agent의 로그, Apache 웹서버에서 생성되는 access 로그와 error 로그를 수집하도록 설정 하였습니다.

CloudWatch Agent 에 대한 보다 자세한 설정은 다음 링크를 참고 하시길 바랍니다.

[CloudWatch Agent 구성 파일의 작성법 참고](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)

[CloudWatch Agent로 수집할 수 있는 지표들](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html)

&nbsp;

## CDK로 통합 CloudWatch Logs Agent 설치하기

**lib/webserver-stack.ts** 파일에서 [VPC Flow 로그](../vpc) 코드 밑에 다음의 코드를 추가 합니다.

[cfn-init](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/cfn-init.html)을 사용하여 CloudWatch Agent를 설치, 기동하는 설정을 정의 합니다.
```typescript
// (Config) Install the unified CloudWatch agent by cfn-init
const configInstallAgent = new ec2.InitConfig([
    ec2.InitPackage.rpm('https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm')
]);
    
// (Config) Copy the CloudWatch agent config file to EC2 instance
const configAgent = new ec2.InitConfig([
    ec2.InitFile.fromAsset(
        '/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json', // TargetFile path
        'resources/cfn-init/amazon-cloudwatch-agent.json' // Path of the asset
        )
]);
    
// (Config) Start the unified CloudWatch agent
const configStartAgent = new ec2.InitConfig([
    ec2.InitCommand.shellCommand('/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json')
]);

// Define config set with configs. When instance start, the cfninit will init the default config set. 
const cfnInit = ec2.CloudFormationInit.fromConfigSets({
    configSets: {
    // Applies the configs below in this order
    default: [
        'installCwAgent',
        'configCwAgent', 
        'startAgent']
    },
    configs: {
    installCwAgent: configInstallAgent,
    configCwAgent: configAgent,
    startAgent: configStartAgent
    }
});
```

&nbsp;

## 인스턴스 구성하기

아래의 코드를 추가하여 웹서버용 `IAM role`을 생성하고  `CloudWatch agent` Policy 를 추가합니다.

```typescript
// IAM role
const webServerRole = new iam.Role(this, 'WebServerRole', {
    assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),
});
// Add policy for cloudwatch log agent to the role
webServerRole.addManagedPolicy(iam.ManagedPolicy.fromAwsManagedPolicyName('CloudWatchAgentServerPolicy'));
```

다음 코드를 추가하여 인스턴스의 Machine Image를 설정하고 인스턴스 타입, VPC, 인스턴스 이름, IAM role, init 설정을 추가합니다.

```typescript
// Machine Image
const amznLinux = ec2.MachineImage.latestAmazonLinux({
    generation: ec2.AmazonLinuxGeneration.AMAZON_LINUX_2
});

// Create ec2 instance
const demo_instance = new ec2.Instance(this, 'DemoInstance', {
    instanceType: ec2.InstanceType.of(ec2.InstanceClass.T2, ec2.InstanceSize.MICRO),
    vpc: demoVpc,
    machineImage: amznLinux,
    instanceName: 'webserver',
    role: webServerRole,
    init: cfnInit
});
```

다음 코드를 추가하여 미리 정의한 bootstrap 스크립트를 인스턴스 user data에 추가합니다.

이 스크립트는 Apache 웹서버를 설치, 기동하는 작업을 하며 `resources/userdata/bootstrap.sh` 에 위치합니다.

```typescript
// Add userdata (bootstrap)
const bootstrap = fs.readFileSync('resources/userdata/bootstrap.sh', 'utf8');
demo_instance.addUserData(bootstrap);  
```

다음 코드를 추가하여 웹서버의 보안그룹을 생성하고 http와 ssh 포트를 허용하는 인바운드 트래픽 룰을 설정합니다.

```typescript
// Define security group
const instance_sg = new ec2.SecurityGroup(this, 'webserver', {
    vpc: demoVpc,
    allowAllOutbound:true,
    description: 'Webserver Security Group'
});

// Allow inbound traffic for SSH (port 22), and HTTP (port 80).
instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'HTTP from anywhere');
instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(22), 'SSH from anywhere');

// Add the security group to the instance
demo_instance.addSecurityGroup(instance_sg);
```

&nbsp;

완성된 코드는 다음과 같습니다.

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
    
    
    // (Config) Install the unified CloudWatch agent by cfn-init
    const configInstallAgent = new ec2.InitConfig([
      ec2.InitPackage.rpm('https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm')
    ]);
      
    // (Config) Copy the CloudWatch agent config file to EC2 instance
    const configAgent = new ec2.InitConfig([
        ec2.InitFile.fromAsset(
          '/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json', // TargetFile path
          'resources/cfn-init/amazon-cloudwatch-agent.json' // Path of the asset
          )
    ]);
      
    // (Config) Start the unified CloudWatch agent
    const configStartAgent = new ec2.InitConfig([
        ec2.InitCommand.shellCommand('/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json')
    ]);
    
    // Define config set with configs. When instance start, the cfninit will init the default config set. 
    const cfnInit = ec2.CloudFormationInit.fromConfigSets({
      configSets: {
        // Applies the configs below in this order
        default: [
          'installCwAgent',
          'configCwAgent', 
          'startAgent']
      },
      configs: {
        installCwAgent: configInstallAgent,
        configCwAgent: configAgent,
        startAgent: configStartAgent
      }
    });
    
    // IAM role
    const webServerRole = new iam.Role(this, 'WebServerRole', {
      assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),
    });
    // Add policy for cloudwatch log agent to the role
    webServerRole.addManagedPolicy(iam.ManagedPolicy.fromAwsManagedPolicyName('CloudWatchAgentServerPolicy'));
    
    // Machine Image
    const amznLinux = ec2.MachineImage.latestAmazonLinux({
      generation: ec2.AmazonLinuxGeneration.AMAZON_LINUX_2
    });
    
    // Create ec2 instance
    const demo_instance = new ec2.Instance(this, 'WebServerInstance', {
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T2, ec2.InstanceSize.MICRO),
      vpc: demoVpc,
      machineImage: amznLinux,
      instanceName: 'WebServer',
      role: webServerRole,
      init: cfnInit
    });
    
    // Add userdata (bootstrap)
    const bootstrap = fs.readFileSync('resources/userdata/bootstrap.sh', 'utf8');
    demo_instance.addUserData(bootstrap);  
    
    // Define security group
    const instance_sg = new ec2.SecurityGroup(this, 'WebServer', {
      vpc: demoVpc,
      allowAllOutbound:true,
      description: 'Webserver Security Group'
    });
    
    // Allow inbound traffic for SSH (port 22), and HTTP (port 80).
    instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'HTTP from anywhere');
    instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(22), 'SSH from anywhere');
    
    // Add the security group to the instance
    demo_instance.addSecurityGroup(instance_sg);
    
      // Create log group for cloudwatch agent
    const webServerLogGroup = new logs.LogGroup(this, 'WebServerLogGroup', {
      logGroupName: 'WebServerLogGroup',
      retention: logs.RetentionDays.ONE_MONTH
    });
  }
}
```

&nbsp;

## 엔트리포인트에 스택 추가하기
`bin/centralized-logging-skeleton.ts` 파일을 열어 스택을 추가할 것입니다.

다음 코드를 추가하여 스택을 import 합니다.

```typescript
import { WebServerStack } from '../lib/webserver-stack';
```

`CloudtrailStack` 밑에 다음 코드를 추가합니다.

```typescript
new WebServerStack(app, 'WebServerStack', { env: envRegion, bucket:logBucketStack.logBucket });
```

수정된 코드는 다음과 같습니다.
```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { LogBucketStack } from '../lib/log-bucket-stack';
import { CloudtrailStack } from '../lib/cloudtrail-stack';
import { WebServerStack } from '../lib/webserver-stack';

const envRegion = { region: 'us-east-2' };
const app = new cdk.App();

const logBucketStack = new LogBucketStack(app, 'LogBucketStack', { env: envRegion });
new CloudtrailStack(app, 'CloudtrailStack', { env: envRegion, bucket:logBucketStack.logBucket });
new WebServerStack(app, 'WebServerStack', { env: envRegion, bucket:logBucketStack.logBucket });
```

&nbsp;

## 배포하기

터미널 창에 다음 명령어를 입력하여 정의한 리소스들을 배포합니다. 배포 작업에 수분의 시간이 걸립니다.

```
cdk deploy WebServerStack
```



[EC2 콘솔](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:instanceState=running)로 이동하여 인스턴스가 잘 배포 되었는지 확인 합니다.
![EC2 콘솔](/images/workshop1/webserver-check.png)

인스턴스의 Public IPv4 Address를 복사하여 웹브라우저에 입력 해봅니다.

다음과 같이 Apache Web Server가 정상적으로 기동 되었음을 확인할수 있습니다.

![샘플 페이지](/images/workshop1/website.png)

[CloudWatch Log 콘솔](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups) 에 LogGroup들이 정상적으로 생성 되었는지도 확인합니다.
![Log Groups](/images/workshop1/webserver-log.png)

로그 버킷에도 정상적으로 로깅이 되는지 확인합니다.
![Log bucket](/images/workshop1/vpc-s3.png)