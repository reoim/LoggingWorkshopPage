---
title: "...on your own"
weight: 100
pre: "<b>2-1. </b>"
---

{{% notice warning %}}
Only complete this section if you are running the workshop on your own. If you are at an AWS hosted event (such as AWS Summit, Immersion Day, etc), go to [Start the workshop at an AWS event](../start-event-engine).
{{% /notice %}}

1. If you don’t already have an AWS account with Administrator access: [create one now by clicking here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/?nc1=h_ls)

2. Once you have an AWS account, ensure you are following the remaining workshop steps as an IAM user with administrator access to the AWS account: [Create a new IAM user to use for the workshop](https://console.aws.amazon.com/iam/home?#/users$new?step=details)

3. Enter the user details:

![Add IAM User](/images/settings/iam-1-create-user.png)

4. Attach the AdministratorAccess IAM Policy:

![Attatch IAM Policy](/images/settings/iam-2-attach-policy.png)

5. Click to create the new user:

![Create the new user](/images/settings/iam-3-create-user.png)

6. Take note of the login URL and save:

![Sign-in url](/images/settings/iam-4-save-url.png)

7. Sign-out from a root user and sign-in at the URL.