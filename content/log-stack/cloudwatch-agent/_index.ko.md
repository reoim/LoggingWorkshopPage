---
title: "Unified CloudWatch Agent"
weight: 340
---
***
이 페이지에서는 인스턴스에 Apache Web Server를 설치, 기동하고

 [통합 CloudWatch agent](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)를 설치하여 시스템 수준 지표 및 Apache Web Server의 Access Log를 수집 하도록 설정 합니다.

 ## Bootstrap
인스턴스의 **User Data**에 추가 할 bootstrap 스크립트를 생성 합니다.

터미널 창 프로젝트 폴더 위치에서 다음의 명령어로 resources 폴더와 userdata 폴더를 생성 합니다.
```bash
mkdir -p resources/userdata
```

생성 된 폴더에 `bootstrap.sh` 이라는 새 파일을 생성 합니다.
![bootstrap.sh](/images/log-stack/bootstrap.png)

bootstrap.sh 파일에 다음과 같이 코드를 추가 후 저장 합니다.
```bash
#!/bin/bash
yum update -y
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
yum install -y httpd mariadb-server
systemctl start httpd
systemctl enable httpd
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
echo "<?php echo '<h1>AWS CDK sample PHP application</h1>'; ?>" > /var/www/html/index.php
```
위 코드는 간단한 Apache Web Server를 설치하고 기동 시키는 스크립트 입니다.

 ## CloudWatch Agent Configuration File
 이제 [CloudWatch Agent 구성 파일을 생성](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file.html) 합니다.

 먼저 터미널 창에서 다음 명령어로 CloudWatch Agent 구성 파일을 저장 할 폴더를 생성 합니다.
 ```bash
 mkdir resources/cfn-init
 ``` 

 생성 된 폴더에 `amazon-cloudwatch-agent.json` 이라는 이름으로 구성 파일을 생성 합니다.
![amazon-cloudwatch-agent.json](/images/log-stack/cfn-init.png)

amazon-cloudwatch-agent.json 파일에 다음의 코드를 추가 후 저장 합니다.
```json
{
  "agent": {
     "metrics_collection_interval": 60,
     "region": "us-east-1",
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
            "log_group_name": "CwAgentLog",
            "log_stream_name": "amazon-cloudwatch-agent.log",
            "timezone": "UTC"
          },
          {
            "file_path":"/var/log/httpd/access_log",
            "log_group_name":"WebServerLogGroup",
            "log_stream_name": "apache_access_log",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```
위 구성은 총 3가지 시스템 지표(memory, swap, process)와 CldouWatch Agent의 log, Apache Web Server에서 생성되는 access log를 수집하도록 설정 하였습니다.

CloudWatch Agent 에 대한 보다 자세한 설정은 다음 링크를 참고 하시길 바랍니다.

[CloudWatch Agent 구성 파일의 작성법 참고](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)

[CloudWatch Agent로 수집할 수 있는 지표들](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html)


## CDK로 통합 CloudWatch Log Agent 설치하기

**lib/logging-wokrshop-stack.ts** 파일에서 [VPC Flow Log 정의](../vpc-flow-log) 코드 밑에 다음의 코드를 추가 합니다.

[CloudFormation Init](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-init.html)을 사용하여 CloudWatch Agent를 설치, 기동하는 설정을 정의 합니다.
```typescript
// (Config) Install the unified CloudWatch agent by cfn-init
const configInstallAgent = new ec2.InitConfig([
    ec2.InitPackage.rpm('https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm')
    ]);
    
// (Config) Create file on the EC2 instance
const configAgent = new ec2.InitConfig([
    ec2.InitFile.fromAsset(
        '/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json', // TargetFile path
        'resources/cfn-init/amazon-cloudwatch-agent.json' // Path of the asset
        )
    ]);
    
// (Config) Start the unified CloudWatch agent
const configStartAgent = new ec2.InitConfig([
    ec2.InitCommand.shellCommand('/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json')
    ])

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
})
```

## 인스턴스 구성하기
아래의 코드를 추가하여 인스턴스를 구성 합니다.
```typescript
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
const demo_instance = new ec2.Instance(this, 'DemoInstance', {
    instanceType: ec2.InstanceType.of(ec2.InstanceClass.T2, ec2.InstanceSize.MICRO),
    vpc: demoVpc,
    machineImage: amznLinux,
    instanceName: 'webserver',
    role: webServerRole,
    init: cfnInit
});

// Add userdata (bootstrap)
const bootstrap = fs.readFileSync('resources/userdata/bootstrap.sh', 'utf8');
demo_instance.addUserData(bootstrap);  

// Define security group
const instance_sg = new ec2.SecurityGroup(this, 'webserver', {
    vpc: demoVpc,
    allowAllOutbound:true,
    description: 'Webserver Security Group'
})

// Allow inbound traffic for SSH (port 22), and HTTP (port 80).
instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'HTTP from anywhere');
instance_sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(22), 'SSH from anywhere');

// Add the security group to the instance
demo_instance.addSecurityGroup(instance_sg);
```

위 코드는 인스턴스를 구성할 때 필요한 Machine Image 선택, User Data 추가, CloudFromation init 설정, Security Group 설정, IAM role 추가 등의 작업을 합니다.

## 인스턴스 Log에 사용자 지표 추가
CloudWatch Agent로 수집 된 인스턴스 로그에 대해서도 사용자 지표를 추가 할 수 있습니다.

다음의 코드를 추가하여 Apache Web Server Log에 대해 사용자 지표를 추가 합니다.

```typescript
// Create log group for cloudwatch agent
const webServerLogGroup = new logs.LogGroup(this, 'WebServerLogGroup', {
    logGroupName: 'WebServerLogGroup',
    retention: logs.RetentionDays.ONE_MONTH
});

// Create custom metric 
const webServerMetricFilter = new logs.MetricFilter(this, 'WebServerMetricFilter', {
    logGroup: webServerLogGroup,
    metricNamespace: 'WebServerMetric',
    metricName: 'BytesTransferred',
    filterPattern: logs.FilterPattern.literal('[ip, id, user, timestamp, request, status_code, size]'),
    metricValue: '$size',
    defaultValue: 0
});

//expose a metric from the metric filter
const webServerMetric = webServerMetricFilter.metric();   
```

터미널 창에 `cdk deploy` 명령을 입력하여 Stack을 배포 합니다.


## 배포 확인
[EC2 콘솔](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:instanceState=running)로 이동하여 인스턴스가 잘 배포 되었는지 확인 합니다.
![EC2 콘솔](/images/log-stack/ec2.png)

인스턴스의 Public IPv4 Address를 복사하여 웹브라우저에 입력 해봅니다.

다음과 같이 Apache Web Server가 정상적으로 기동 되었음을 확인 할 수 있습니다.

![샘플 페이지](/images/log-stack/apache.png)

[CloudWatch Log 콘솔](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups) 에 LogGroup들이 정상적으로 생성 되었는지도 확인 합니다.
![Log Groups](/images/log-stack/log-group.png)