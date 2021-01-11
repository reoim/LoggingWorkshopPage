---
title: "Create Workspace"
weight: 300
pre: "<b>2-3. </b>"
---

***

{{% notice info %}}
In this workshop, use "Ohio (us-west-2a)" as your region
{{% /notice %}}

## Create Workspace
1. Open the [Cloud9 Console](https://us-east-2.console.aws.amazon.com/cloud9/home?region=us-east-2) and click the **Create environment** button.

![Create C9 envrionment](/images/settings/c9-create.png)


2. Enter the **Name** and **Description**. Click the **Next step**.
![Clou9 name](/images/settings/c9-name.png)

3. Configure as the following (Platform - Amazon Linux 2). Click the **Next step**.
![Cloud9 config](/images/settings/c9-config.png)

4. **Review** the settings. Click the **Create environment**.
![Cloud9 review](/images/settings/c9-review1.png)

After loading for a few minutes, the workspace will come up as follows.
![Cloud9 workspace](/images/settings/c9-browser.png)  

## Install and setup CDK
Enter the following command in the terminal tab to install CDK toolkit. The toolkit is a command-line utility which allows you to work with CDK apps.
```bash
# Setting environment variable for CDK Version
echo 'export AWS_CDK_VERSION="1.78.0"' >> ~/.bashrc
source ~/.bashrc

# Install aws-cdk
npm install -g --force aws-cdk@$AWS_CDK_VERSION
```
Enter the command `cdk --version` to check CDK toolkit version.
```
$ cdk --version
1.78.0 (build c2f38e8)
```