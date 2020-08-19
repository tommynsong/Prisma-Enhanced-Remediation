# Setup Guide

## Overview
- Build the Lambda remediation infrastructure using a CloudFormation template in your Parent AWS account.
- (Optional) Create an IAM Role and Policy in target (Child) AWS accounts using a CloudFormation template.
- Integrate Prisma Cloud to send alerts to the Lambda remediation infrastructure.

---

## Step 1 - Create Prisma Cloud Remediation Stack
[![Launch Button](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=PrismaRemediation&templateURL=https://prisma-remediation-us-west-2.s3-us-west-2.amazonaws.com/templates/cloudformation_prisma_template.json)

This CloudFormation stack creates the following resources in the Oregon (us-west-2) region:
- AWS SQS Queue
- AWS IAM Role and Policy to be used by Lambda (See [iam_role_permission.json](../templates/iam_role_permission.json))
- AWS Lambda package containing the runbook scripts to remediate Prisma Cloud alerts.


## Step 2 - Setup IAM Permissions

### Single Account Setup
To remediate a single AWS account, you're done.  You can skip this step and move on to step 3, Prisma Cloud Integration.

### Multi Account Setup
The account where the stack is launched (in step 1) can be considered the **Parent** account.  All other accounts are called **Child** or **Target** accounts.

The Parent account will need permission to change/modify AWS resources in your Child account(s).  Next, we'll create a Role that allows the lambda function to perform the remediation.

When you created the CloudFormation stack in step 1, one of the parameters was called `CrossAccountRoleName` (The default name is **CrossAccountRemediationRole**).  Use the same name when using the CloudFormation template below to create the IAM Role in your Chlid account(s).

For Multi Account Setup, you will need to create this IAM Role in each Child/Target AWS account.

[![Launch Button](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=PrismaChlidRemediationRole&templateURL=https://prisma-remediation-us-west-2.s3-us-west-2.amazonaws.com/templates/cloudformation_role_template.json)

This CloudFormation stack creates the following resources in the Oregon (us-west-2) region:
- AWS IAM Role and Policy to be used by Lambda (See [iam_role_permission.json](../templates/iam_role_permission.json))


## Step 3 - Prisma Cloud Integration
You will need to get the **PrismaRemediationSQSQueue** URL from the Output section of the CloudFormation stack created in step 1.
- Go to the AWS CloudFormation Dashboard. https://us-west-2.console.aws.amazon.com/cloudformation
- Click on the Prisma Remediation stack name.
- Select the **Outputs** tab. 
- Take note of the **PrismaRemediationSQSQueue** URL.

Create a new Prisma Cloud Integration:

- Login to the Prisma Cloud console.
- From the left-side window pane, slect **Settings**.
- Choose **Integrations**.
- Near the top of the window, click **Add new**.
- Fill out the **Integration name** and **Queue URL** fields, then click Next.  Use the SQS Queue URL from the Output of your CloudFormation stack.
- Click **Test**, then Save.

Create a new Prisma Cloud Alert rule:

- Login to the Prisma Cloud console.
- From the left-side window pane, slect **Alerts**.
- Choose **Alert rules**.
- Near the top of the window, click **Add new**.
- Fill out the **Alert name** field, then click Next.
- Select the account group(s), and Next. These are the AWS accounts you're setting up for remediation.
- Choose the Policies you'd like to remediate, then Next.
- Enable **Amazon SQS** queue and select the integration you created above.
- Save your new Alert rule.
