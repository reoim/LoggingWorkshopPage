---
title: "실습 환경 구축"
weight: 210
pre: "<b>5-2.a. </b>"
---

***

{{% notice info %}}
이 실습은 Ohio (us-east-2) 리전에서 진행 합니다.
{{% /notice %}}

## Workspace 생성하기
1. [Cloud9 콘솔](https://us-east-1.console.aws.amazon.com/cloud9/home?region=us-east-1)으로 
이동하여 **Create environment** 를 클릭 합니다.

![Create C9 envrionment](/images/settings/c9-create.png)


2. **Name** 과 **Description** 을 입력 후 **Next step**을 클릭 합니다. 
![Clou9 name](/images/settings/c9-name.png)

3. 다음과 같이 설정 후 (Platform - Amazon Linux 2) **Next step**을 클릭 합니다.
![Cloud9 config](/images/settings/c9-config.png)

4. **Review** 페이지에서 설정이 잘 되었는 지 확인 후 **Create environment**를 클릭 합니다.
![Cloud9 review](/images/settings/c9-review1.png)

몇분 간 로딩 후 다음과 같이 Workspace가 생성 됩니다.
![Cloud9 workspace](/images/settings/c9-browser.png)  

## Install and setup CDK
Cloud9 터미널 창에 다음의 명령어를 입력 합니다.
```bash
# Setting environment variable for CDK Version
echo 'export AWS_CDK_VERSION="1.78.0"' >> ~/.bashrc
source ~/.bashrc

# Install aws-cdk
npm install -g --force aws-cdk@$AWS_CDK_VERSION
```
정상적으로 설치 되었는지 `cdk --version` 명령어로 다음과 같이 CDK version을 확인 합니다.

```
$ cdk --version
1.78.0 (build c2f38e8)
```