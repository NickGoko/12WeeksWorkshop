---
tags:
  - 12weeksawsworkshopchallenge
---
you do not need to complete this section. The CloudFormation template will already be deployed for you.

1. Your account must have the ability to create new IAM roles and instance profiles.
    
2. Please be aware that if you complete this workshop in your own AWS account, costs will be incurred.
    

If you have existing resources consuming Elastic IP addresses deployed in the region that you are going to use in this workshop, you may hit the **Elastic IP addresses per Region** quota of 5 Elastic IP addresses.

Option 1. Request an **Elastic IP addresses per Region** quota increase from the [Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)  page.

Option 2. Delete existing resources consuming Elastic IP addresses if they are not longer required.

Option 3. If you have 1 existing Elastic IP address resource in Region, you can delete the NAT Gateway that is created in VPC C in the [Multiple VPCs](https://catalog.workshops.aws/networking/en-US/foundational/multivpc) section.

You may complete these labs with your own AWS account. Make sure that you have the proper IAM permissions to create new IAM roles and scope other needed IAM permissions.

In addition you must have spare capacity in your account to create new VPCs and Elastic IP addresses (the default limit is five VPCs per region and five Elastic IP addresses).

The introductory labs create four of each in total:

- **Fundamentals:** one VPC, one Elastic IP address
- **Multi-VPC:** two VPCs, two Elastic IP addresses
- **On-premises:** one VPC, one Elastic IP address

## Deploy Pre-requisites

In order to run this workshop the following resources will be created:

- IAM role for EC2 instances. The `AmazonS3FullAccess` <mark style="background: #FFF3A3A6;">is required to complete the VPC Endpoints section</mark> and `AmazonSSMManagedInstanceCore` <mark style="background: #FFB86CA6;">allows us to remotely connect to the instance using Session Manager instead of SSH</mark>
- <mark style="background: #FFF3A3A6;">EC2 instance profile to define the IAM role that EC2 instances should use</mark>.
- <mark style="background: #ABF7F7A6;">IAM role for VPC Flowlogs</mark>
- <mark style="background: #FFB8EBA6;">An S3 bucket with the name</mark> `networking-day-${AWS::Region}-${AWS::AccountId}` t<mark style="background: #FFB8EBA6;">hat will be used to test VPC Endpoints</mark>

If you are running this lab in **your own AWS Account**, it is recommended for all lab resources to be created in us-east-1 region so that the screenshots match your environment. This is not mandatory.

[CloudFormation Template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/pre-requisites.yaml)

1. To get started, navigate to [CloudFormation](https://console.aws.amazon.com/cloudformation/home)  section in the AWS console. Click **Create stack** button and select **With new resources (standard)**.
    
2. Under **Specify template**, select **Upload a template file**, click **Choose file** and select the `pre-requisites.yaml` CloudFormation template that you downloaded. Click **Next**.
    
3. Enter a stack name, e.g. `NetworkingWorkshopPrerequisites`. Click **Next**.
    
4. Leave everything on this screen as is. Scroll down and click **Next**.
    
5. Select the checkbox to acknowledge the creation of IAM resources and click the **Submit** button. ![Create stack](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/prereqs/prereqs_cfn_create_stack.png)
    
6. The stack will launch and have a status of in progress. ![In progress](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/prereqs/prereqs_cfn_stack_in_progress.png)
    
7. Waiting until the stack completes successfully before progressing to the workshop.