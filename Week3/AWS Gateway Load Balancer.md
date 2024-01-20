---
tags:
  - 12weeksawsworkshopchallenge
---
https://catalog.workshops.aws/networking/en-US/advanced/gwlb
# Introduction
## What is AWS Gateway Load Balancer
**Gateway Load Balancer (GWLB)** <mark style="background: #ADCCFFA6;">helps you easily deploy, scale, and manage your third-party virtual appliances.</mark>  
<mark style="background: #FFB8EBA6;">It gives you one gateway for distributing traffic across multiple virtual appliances while scaling them up or down, based on demand.</mark> This decreases potential points of failure in your network and increases availability.

You can find, test, and buy virtual appliances from third-party vendors directly in AWS Marketplace. This integrated experience streamlines the deployment process so you see value from your virtual appliances more quickly—whether you want to keep working with your current vendors or try something new.

Gateway Load Balancer provides both Layer 3 gateway and Layer 4 load balancing capabilities.  
It is a transparent bump-in-the-wire device that does not change any part of the packet.  
It is architected to handle millions of requests/second, volatile traffic patterns, and introduces extremely low latency. GWLB uses a new type of VPC endpoint to connect to sources and destinations of network traffic, which is called Gateway Load Balancer Endpoint (GWLBe).

You send traffic to GWLB by making configuration updates in your VPC route tables. With GWLB, customers can scale their virtual appliances elastically by load balancing traffic across a fleet of virtual appliances. GWLB improves availability by routing traffic flows through healthy virtual appliances, and reroutes flows when an appliance becomes unhealthy.

## How Does GWLB Send Traffic to the Right Appliance
<mark style="background: #FFB86CA6;">GWLB will pick a target based on a five-tuple hash, which means GWLB picks the backend target based on these five components</mark> contained in every IP packet:

- IP Protocol (for example TCP)
- Source IP address (for example 54.11.48.21)
- Source Port (for example 54001)
- Destination IP address (for example 10.1.2.113)
- Destination Port (for example 80 for HTTP)

By ensuring flow stickiness in both directions GWLB will send the packet to the same target for the return traffic as well. It would look like this:

- IP Protocol (for example TCP)
- Source IP address (for example 10.1.2.113)
- Source Port (for example 80)
- Destination IP address (for example 54.11.48.21)
- Destination Port (for example 54001)

Note that if the same two hosts are talking on two different sets of ports, the traffic could be routed through two different appliances.

## GENEVE Encapsulation

A _Gateway Load Balancer_ <mark style="background: #FFB86CA6;">encapsulates IP traffic with a GENEVE header and forwards it to the appliance.</mark>  
By encapsulating the traffic to the target, GWLB can provide separation and some additional information about which GWLB endpoint (GWLBe) the traffic came from. This allows the return traffic to go back to the correct GWLBe.

# Prerequisites
## Creating a Key Pair
Key Pairs are regional, so make note of the region you are using. This lab was designed to run in the North Virginia (us-east-1) Region. If you are using Event Engine for this lab, this section isn't necessary, as a Key Pair is already created for you and available via the Event Engine interface.

___

If you do not already have a Key Pair in the account and region your are using, proceed with the below instructions to create a new Key Pair.

___

## Steps
___

1. In the **AWS Console**, Navigate to **EC2**.
2. Then select **Key Pairs** in the left hand menu under Network & Security.
3. Now select **Create key pair** in the top right.
4. Go ahead and configure your key pair like shown in the image below, if you prefer to use Putty, you can configure your key pair to use Putty instead (Putty is meant for Windows users, other OS types should use .pem files).

___

![Create Key Pair](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/15-prerequisites/keypair/gwlblab-create-keypair.png)

___

1. Download your Key Pair and put it in a folder for future use. **Note** If you delete this Key Pair, you won't be able to access to your 3rd Party Firewall Instances, so keep it safe.
2. If you are on Mac or Linux, open up a terminal window and navigate to where your Key Pair is saved and then type in the following command, substituting the proper .pem filename.

```
chmod 400 your-key-pair-name.pem
```

___

Once you created your Key Pair and configured the necessary settings, you can move onto the next section.


# Lab 1: Deploying Infrastructure

## Lab 1.1: Architecture Overview


### Basic VPC Architecture

The image below shows what you will be deploying if you select **Infrastructure-Only** as the **Environment Type** in the CloudFormation template. If you select to deploy any of the instances, then an EC2 instance for your firewall will be deployed into the **SecurityVPC-Data-AZ** subnets.

![Basic TGW and VPCs Diagram](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-basic-architecture.png)

___

### Finished VPC Architecture

Once you completed the lab this is what the final configuration will look like.

![Finished TGW and VPCs Diagram](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-finished-architecture.png)

___

## Lab 1.2: CloudFormation Deployment
In order to deploy the lab environment, we will be using [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) . Depending upon which firewall appliance you would like to use in the lab, you can choose that in the CloudFormation template.

Once you have specified which firewall you would like to use, navigate accordingly in the following section.

This lab was designed to run in the North Virginia (us-east-1) Region. If there are any issues with deployment, please be sure you are using this region.

___

### Downloading CloudFormation Files and Creating Our Deployment Bucket

The CloudFormation templates need to be downloaded, unzipped, and uploaded to a S3 bucket in your AWS account. We can use CloudShell in order to do this as it will be much quicker, but feel free to also download the files and upload them to an S3 bucket yourself if you would like to.

___

1. In the AWS Console, navigate to **CloudShell** and open it, it should look like the following image:

___

![AWS CloudShell UI](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/deployment/AWS-Cloudshell.png)

___

1. Type in the following command in order to download the files to the Cloudshell instance:

```bash
curl 'https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb-files/gwlb-files.zip' --output gwlb-files.zip
```

If you are asked to paste in multiple lines, simply select paste and unselect "Ask before pasting multiline code" in order to avoid future prompts.  
3. Unzip the files from gwlb-files.zip and place it in a separate folder called gwlb-files:

```bash
 unzip gwlb-files.zip -d ./gwlb-files
```

1. Now lets look to <mark style="background: #ABF7F7A6;">create a new S3 bucket so that we can move the files into that bucket for our deployments. </mark>First, go ahead and grab the region which you are currently in, and export it to a separate variable (AWS\_REGION). And at the same time, lets define a username which you plan to use for the workshop, this must be unique.

```bash
AWS_REGION=`aws ec2 describe-availability-zones --output text --query 'AvailabilityZones[0].[RegionName]'`
```

```shell
AWS_REGION=`aws ec2 describe-availability-zones --region us-east-1 --output text --query 'AvailabilityZones[0].[RegionName]'`
```

```bash
YOUR_USERNAME="Enter-Username"
```

Remember that AWS S3 buckets need to be globally unique, hence if you are unable to create a bucket, try using a more unique name.  
5. Go ahead now and use the below command to create the S3 bucket.
```bash
aws s3api create-bucket --bucket $YOUR_USERNAME-gwlb-deployment-bucket --region $AWS_REGION
```
>Use the command as is not need to fill in YOUR_USERNAME & AWS_REGION  
![[Pasted image 20240120091650.png]]
1. Now we need to go ahead and sync the files for the lab, into the S3 bucket. Type in the following command in order to do so:
```bash
aws s3 sync ./gwlb-files/ s3://$YOUR_USERNAME-gwlb-deployment-bucket
```

2. Once you have done all that, in the AWS Console, navigate to **S3**, and <mark style="background: #CACFD9A6;">you should see the bucket we just created</mark> `yourusername-gwlb-deployment-bucket`, go into it and check that there are files there, if there are, you can proceed onto the deployment.
___

![S3 Bucket File Setup](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/deployment/Deployment-Bucket.png)

___

### Deploying the CloudFormation Template

1. Within the S3 bucket, navigate to the **cloudformation/** folder and search for the **gwlblab-main.yaml** file and select the checkbox on the left hand side of it.

___

![Copy gwlblab-main.yaml File Link in S3](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/deployment/Deployment-CopyURL.png)

___

> Note that we are using nested stacks for this lab, and the gwlb-main.yaml file simply kicks off all of the separate cloudformation templates within the S3 bucket.

1. Select **Copy URL** at the top. This URL will be used for the CloudFormation deployment in the next step.
2. In the AWS Console, navigate to **CloudFormation**.
3. Select **Create Stack**

___

![CloudFormation Home 1](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/cloudformation/CloudFormation-Homepage-1.png)

If you have already deployed something with **CloudFormation**, then select **Create Stack** in the top right, followed by **With new resources (standard)**

___

1. On the **Create Stack** page, paste in the S3 URL which we copied earlier with the **gwlblab-main.yaml** file into the section **Amazon S3 URL** and then select **Next**.

___

![CloudFormation Amazon S3 URL](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/cloudformation/CloudFormation-AmazonS3URL.png)

___

1. Specify the stack details:
    - **Stack name:** `gwlblab-stack`
    - **Global Stack Name:** `gwlblab` . This name will be prepended to the resources deployed by this CloudFormation stack.
    - **Environment Type:** Select **Infrastructure-Only**.
        - Note: Later in the lab, you can choose a Firewall stack to deploy and redeploy the stack.
    - **S3 Bootstrap Bucket Name:** The name of your S3 bucket, for example **_yourusername_\-gwlblab-deployment-bucket**.
    - **EC2 Key Pair:** This will be the EC2 Key Pair used for your Firewall instances. If you don't have a Key Pair, then please create one by following this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) .
    - **Remote Management CIDR Range:** If you would like to restrict what IP addresses can connect to your firewall instances, specify an appropriate CIDR range here. This only applies to the firewall instances, as the EC2 instances behind the NLB and in the VPCs are private and can only be accessed via Systems Manager. You can set this to **0.0.0.0/0** meaning that any IP address can have network access to your firewall instances.
    - **VPC CIDRs:** Leave as the default.
    - **TGW Regional ASN:** Leave as the default. This is the ASN specified for the TGW.
    - **Firewall Instance Settings:** Leave as the default for now. The third-party firewall settings (Fortinet, Palo Alto, Open Source) will be used in a later part of the lab.

___

![CloudFormation Stack Details](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/cloudformation/CloudFormation-StackDetails.png)

___

For the labs, do not deploy any template with the +GWLB at the end. This is just meant for individuals who wish to deploy the complete infrastructure or who need to troubleshoot their configuration and would like to see an example of the configuration.

1. Once done with the specifying the stack details, select **Next**.
2. On the Configure stack options page, select **Next** in the bottom right hand corner.
3. On the reviewing your stack page, select the two checkboxes at the bottom in the **capabilities** section. Then select, **Submit**.

___

![CloudFormation Create Stack](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/cloudformation/gwlb-create-stack.png)

___

1. This will initiate deployment of the stack. It will take around 10 minutes for everything to be deployed. When deployment is complete, you will see a status of **"CREATE\_COMPLETE"** in green.

___

![CloudFormation Finished Deployment](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/cloudformation/CloudFormation-DeployedStack.png)

___

The lab utilizes nested stacks for different components of the deployment. This will be important in terms of deploying the different Firewall instances while keeping our base architecture the same.

- **VPCStack:** VPC components such as the Transit Gateway, the VPCs, the Systems Manager Endpoints and Security Groups
- **IAMStack:** IAM resources needed in order to deploy resources from our S3 bucket
- **EC2Stack:** EC2 instances and Network Load Balancers in VPC1 to host a website. Gateway Load Balancer Endpoints will be in front of the Network Load Balancer to inspect incoming traffic.

___

Once the main Stack and all the nested stacks show "CREATE\_COMPLETE", you can move onto the next section.

## Lab 1.3 Gateway Load Balancer Setup
In this section, we will be deploying Gateway Load Balancer (GWLB) in the **Security VPC**.

Once you have completed this section, your architecture for the Security VPC will look like the below image.

Note: The other AWS infrastructure (NAT Gateways, Transit Gateway, and ENIs) have are already provisioned from the CloudFormation template.

![GWLB Security VPC Architecture](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-SecurityVPC.png)

Note that we still won't have any of the EC2 instances for inspection at this point in the lab. Those will be deployed in later steps

___

Let's begin by deploying the Gateway Load Balancer and Target Group. We will register the firewall instances to the Target Group in the later sections.

___

### Creating Gateway Load Balancer
___

1. In the AWS Console, Navigate to **EC2**.
2. In the left column, find and click on **Load Balancers** under Load Balancing.

___

![GWLBs](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/Create-ELB.png)

___

1. You should see the NLB which was created by the CloudFormation stack. Click on **Create Load Balancer**.
2. Now select **Create** under **Gateway Load Balancer**.

___

![GWLB Load Balancer Options](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-Create.png)

___

1. Fill out the form:
    - **Load balancer name:** `gwlblab-gwlb-firewall`
    - **IP address type:** Select **IPv4** if it's not already selected.
    - **VPC:** Select the VPC we deployed **gwlblab-SecurityVPC**.
    - **Mappings:** You should only see two Availability Zones (for example, if you deployed in North Virginia, **us-east-1a** and **us-east-1b**) If you have more than two Availability Zones, you have selected the wrong VPC.
        - Click the checkmark for both availability zones:
            - **us-east-1a**
            - **us-east-1b**
        - Select the **"Data"** subnet for each AZ:
            - **gwlblab-SecurityVPC-_Data_\-AZ1**
            - **gwlblab-SecurityVPC-_Data_\-AZ2**

___

![GWLB Initial Configuration](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-Config-1.png)

___

1. Create a Target Group, by selecting the **Create target group** link under IP listener routing. It will open in a new browser tab.

___

![GWLB Create Target Group](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-Config-2.png)

___

1. When specifying the Target Group settings, we can use the following settings:
    - **Choose a target type:** Select **Instances**. This can change during other firewall setups, as some firewalls will work with the Instance selection, whereas others can only work with specific IP addresses. If you are planning to deploy the **lab 2d. Cisco FTD**, you will have to select IP Addresses for the target type in this step.
    - **Target group name:** `gwlblab-gwlb-tg`
    - **Protocol / Port:** Ensure that **GENEVE** is selected
    - **VPC:** Select **gwlblab-SecurityVPC**

___

![GWLB Target Group Configuration Part 1](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-Config-4.png)

___

1. For the health check configurations, be sure to select the following:
    - Ensure **TCP** is selected
    - Select **Advanced health check settings**
    - Select **Override**
    - Set the port number to: **22**

___

![GWLB Target Group Configuration Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-Config-5.png)

___

1. Once you are done with that, click **Next** in the bottom right.
2. Do not register any targets. That will be done in the following labs. Choose **Create target group** in the bottom right hand corner.
3. Navigate back to the browser tab for creating of your Gateway Load Balancer. Select the small **Refresh** button to the right of the "Select a target group" box.
    - Select the **gwlblab-gwlb-tg** target group
    - Click **Create load balancer** in the bottom right.

___

Once the Gateway Load Balancer is created, you can move onto the next section.

___

If you used something other than **gwlblab** for **Global Stack Name** when deploying the infrastructure with CloudFormation, your resource names will be different. Ensure you select the AWS resources with the correct names.


## Lab 1.4: Gateway Load Balancer Endpoints Setup
In this section, you will be creating Gateway Load Balancer endpoints for both **VPC1** and **Security** VPCs.  
These endpoints are a requirement before you are able to setup the necessary routing.

This section has multiple parts:
- [Gateway Load Balancer Endpoint Services Setup](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/34-deploy-gwlbe#gateway-load-balancer-endpoint-services-setup)
- [Copy VPC Endpoint Service Name](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/34-deploy-gwlbe#copy-vpc-endpoint-service-name)
- [Gateway Load Balancer Endpoints for VPC1](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/34-deploy-gwlbe#gateway-load-balancer-endpoint-setup-for-vpc1)
- [Gateway Load Balancer Endpoints for the Security VPC](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/34-deploy-gwlbe#gateway-load-balancer-endpoints-for-the-security-vpc)
- [Copy VPC Endpoint IDs](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/34-deploy-gwlbe#copy-vpc-endpoint-ids)

___

### Gateway Load Balancer Endpoint Services Setup
___

Start by configuring the Gateway Load Balancer endpoint service, which will be necessary to create the GWLB endpoints.

1. In the AWS VPC Console, navigate to **VPC**
2. In the menu on the left hand side select **Endpoint services** under the "Virtual private cloud" heading
3. Click on **Create endpoint service** at the top right corner
4. Configure:
    - **Name:** `gwlblab-endpoint-service`
    - **Load balancer type:** Select **Gateway**
    - **Available load balancers:** Select **gwlblab-gwlb-firewall**
        - Note that if you don't see your load balancer yet, it could still be provisioning. Wait for it to complete, and then refresh the list by selecting the refresh icon next to "Create new load balancer".
    - **Additional Settings:** Uncheck **Acceptance required**.
        - If this was enabled, it would require an extra approval, which for this lab we won't need, but would be valuable when sharing GWLB Endpoints across accounts.

___

![GWLB Endpoint Service Creation](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlb/GWLB-EndpointService-Creation.png)

___

1. Click **Create** in the bottom right and move onto the next section.

___

### Copy VPC Endpoint Service Name
___

The VPC Endpoint Service Name is needed in order to create the VPC Endpoints in each VPC

1. Check that you are still on **Endpoint Services** of the VPC Console
2. Select the Endpoint Service you just created **(gwlblab-endpoint-service)** and find the **Service name** column or at the bottom section of the web page.
3. Copy the **Service name** to a text editor
    - This will be used later when creating the endpoints.
    - For the sake of this lab, the service name has been hidden, though it will look something like **com.amazonaws.vpce.us-east-1.vpce-svc-#########**.

___

![Endpoint Service Name](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-service-name.png)

___

### Gateway Load Balancer Endpoint Setup for VPC1
___

For this section, we will be creating all of the necessary endpoints used for both VPC1 and the Security VPC. Take a note in terms of which subnets we deploy the Endpoints in each VPC.

Here is a table to make it easy to refer to the values to use for each VPC:

| **Create Endpoint** | **VPC 1** |
| --- | --- |
| **Name Tag** | `gwlblab-vpc1-endpoint-1a` |
| **Service category** | **Other endpoint services** |
| **Service settings** | _Paste the service name copied earlier_ |
| **Click** | **Verify Service** |
| **VPC** | **gwlblab-VPC1** |
| **Subnets: AZ** | **AZ A** - For example: us-east-1a |
| **Subnets: Subnet ID** | **gwlblab-VPC1-_Firewall_\-Subnet-A** |
| **IP address type** | Select **IPv4** |
| **Click** | **Create Endpoint** |
|  |  |
|  |  |
| **Name Tag** | `gwlblab-vpc1-endpoint-1b` |
| **Service category** | **Other endpoint services** |
| **Service settings** | _Paste the service name copied earlier_ |
| **Click** | **Verify Service** |
| **VPC** | **gwlblab-VPC1** |
| **Subnets: AZ** | **AZ B** - For example: us-east-1b |
| **Subnets: Subnet ID** | **gwlblab-VPC1-_Firewall_\-Subnet-B** |
| **IP address type** | Select **IPv4** |
| **Click** | **Create Endpoint** |

___

In order to double check your configurations, you can have a look at the images below.

___

![GWLBE VPC1 Creation Part 1](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-vpc1-1a-part1.png) ![GWLBE VPC1 Creation Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-vpc1-1a-part2.png)

___

Once you have gone through the table for VPC1, you should have two GWLB endpoints for VPC1, both in a different AZ, and it should look like this in the console.

___

![GWLBE VPC1 Both Endpoints](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-vpc1-endpoints.png)

___

Now our architecture will look like this for VPC1, keep this in mind as the how we route our traffic is dependent upon this architecture.

___

![VPC1 GWLBE Architecture](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-VPC1-Architecture.png)

___

### Gateway Load Balancer Endpoints for the Security VPC
___

1. Create the endpoints **gwlblab-securityvpc-endpoint-1a** and **gwlblab-securityvpc-endpoint-1b** for the **Security VPC**.
    - Follow the same instructions as above, but substitute the values for the **Security VPC**
    - Refer to the table below for the values to use:

| **Create Endpoint** | **Security VPC** |
| --- | --- |
| **Name Tag** | `gwlblab-securityvpc-endpoint-1a` |
| **Service category** | **Other endpoint services** |
| **Service settings** | _Paste the service name copied earlier_ |
| **Click** | **Verify Service** |
| **VPC** | **gwlblab-SecurityVPC** |
| **Subnets: AZ** | **AZ A** - For example: us-east-1a |
| **Subnets: Subnet ID** | **gwlblab-SecurityVPC-_GWLBE_\-AZ1** |
| **IP address type** | Select **IPv4** |
| **Click** | **Create Endpoint** |
|  |  |
|  |  |
| **Name Tag** | `gwlblab-securityvpc-endpoint-1b` |
| **Service category** | **Other endpoint services** |
| **Service settings** | _Paste the service name copied earlier_ |
| **Click** | **Verify Service** |
| **VPC** | **gwlblab-SecurityVPC** |
| **Subnets: AZ** | **AZ B** - For example: us-east-1b |
| **Subnets: Subnet ID** | **gwlblab-SecurityVPC-_GWLBE_\-AZ2** |
| **IP address type** | Select **IPv4** |
| **Click** | **Create Endpoint** |

___

Once you have completed these steps, you will have two GWLB endpoints for the Security VPC, and it will look like this.

___

![GWLBE Security VPC Endpoints Console](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-securityvpc-endpoints-console.png)

___

Our architecture for the Security VPC will now look as follows:

___

![GWLBE Security VPC Architecture](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/gwlbe/GWLBE-securityvpc-endpoints.png)

___

### Copy VPC Endpoint IDs
___

You will need your VPC endpoint IDs in the next section in order to setup routing properly.

1. In the VPC Console, navigate to **Endpoints** on the left side
2. Copy the **Endpoint Name** and **VPC Endpoint ID** for all four VPC Endpoints you created. Paste these values into a text editor. It should look something like this:

```text
Name VPC endpoint ID gwlblab-vpc1-endpoint-1a vpce-000XXXXXXXXXX8029 gwlblab-vpc1-endpoint-1b vpce-000XXXXXXXXXX11d7 gwlblab-securityvpc-endpoint-1a vpce-000XXXXXXXXXXf6ac gwlblab-securityvpc-endpoint-1b vpce-000XXXXXXXXXX953a
```

___

Once you have completed this section, you can then move onto configuring the route tables for this setup.

## Lab 1.5 Configuring Route Tables
In this section, you will configure the routing of our application, arguably the most important part of the initial configuration. After this, any troubleshooting or diagnosing of traffic flows will be performed in the following sections depending upon which third party firewall you plan to use.

You will configure routes for both **VPC1**, and **SecurityVPC**, but the configurations will be different, so be careful when adding the routes to follow the instructions exactly.

This lab has multiple sections to go through:
- [Configuring Routing Tables on VPC1](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/35-rt-configuration#configuring-routing-tables-on-vpc-1)
- [Configuring Routing Tables on the Security VPC](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/30-deploy-infra/35-rt-configuration#configuring-routing-tables-on-the-security-vpc)

Once you are done with this section, your architecture will look like the images below.

___

![VPC1 Routing Tables](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-Full-Architecture.png)

![Security VPC Routing Tables](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-Full-Architecture.png)

___

### Configuring Routing Tables on VPC 1
___

Tip: to make things simpler, at the top left screen of the VPC console, select **"Filter by VPC"** then select **gwlblab-VPC1**
___

![VPC Filter](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC-Filter2.png)

___

Below is a table that you can use in order to go through all the steps of the configuration. You can complete the steps below, and then scroll further down to see the correlating images if you require further help with the configuration. After completing the table, you will be finished with VPC1, and can then move onto the Security VPC.

___

| **Step** | **VPC** | **gwlblab-VPC 1** |  |
| --- | --- | --- | --- |
| **1** | **Route Table** | **gwlblab-VPC1-Public-A-Route-Table** |  |
| **2** | **Click** | "Routes" Tab |  |
| **3** | **Click** | "Edit Routes" Button |  |
| **4** | **Edit Route** | **`0.0.0.0/0`** |  |
| **5** | **Route Target** | GWLB **gwlblab-vpc1-endpoint-1a** |  |
| **6** | **Click** | "Save Changes" |  |
|  |  |  |  |
| **7** | **Route Table** | **gwlblab-VPC1-Public-B-Route-Table** |  |
| **8** | **Click** | "Routes" Tab |  |
| **9** | **Click** | "Edit Routes" Button |  |
| **10** | **Edit Route** | **`0.0.0.0/0`** |  |
| **11** | **Route Target** | GWLB **gwlblab-vpc1-endpoint-1b** |  |
| **12** | **Click** | "Save Changes" |  |
|  |  |  |  |
| **13** | **Click** | "Create route table" Button |  |
| **14** | **Name** | `gwlblab-VPC1-Edge-RT` |  |
| **15** | **VPC** | **gwlblab-VPC1** |  |
| **16** | **Click** | "Create route table" |  |
| **17** | **Click** | "Edge associations" Tab |  |
| **18** | **Click** | "Edit edge associations" Button |  |
| **19** | **Select** | IGW **gwlblab-VPC1-IGW** |  |
| **20** | **Click** | "Save changes" Button |  |
| **21** | **Click** | "Routes" Tab |  |
| **22** | **Click** | "Edit routes" Button |  |
| **23** | **Click** | "Add route" |  |
| **24** | **Destination** | **`10.1.0.0/24`** (gwlblab-VPC1-Subnet-A) |  |
| **25** | **Target** | GWLB **gwlblab-vpc1-endpoint-1a** |  |
| **26** | **Destination** | **`10.1.1.0/24`** (gwlblab-VPC1-Subnet-B) |  |
| **27** | **Target** | GWLB **gwlblab-vpc1-endpoint-1b** |  |
| **28** | **Click** | "Save changes" button |  |
|  |  |  |  |

___

**Step 3** - Editing routes button

___

![VPC1 Public A Route Table](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-PublicRT-A.png)

___

**Step 5** - Adding routes to the gwlblab-VPC1-Public-A-Route-Table

___

![VPC1 Public A Route Table](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-PublicRT-A-EditRoutes-1a.png)

___

**Step 11** - Adding routes to the gwlblab-VPC1-Public-B-Route-Table

___

![VPC1 Public A Route Table](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-PublicRT-A-EditRoutes-1b.png)

___

**Step 13** - Creating a route table for our Edge route table

___

![VPC1 Create Edge Route Table](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-CreateEdgeRT.png)

___

**Step 16** - Finishing creation of the Edge route table

___

![VPC1 Create Edge Route Table Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-CreateEdgeRT-2.png)

___

**Step 17** - Editing edge route table associations

___

![VPC1 Create Edge Route Table Part 3](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-CreateEdgeRT-3.png)

___

**Step 18** - Adding the IGW to the Edge route table

___

![VPC1 Create Edge Route Table Part 4](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-CreateEdgeRT-4.png)

___

**Step 23-28** - Configuring edge routes back to the GWLB endpoint IDs

___

![VPC1 Configure Edge Route Table Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-EditEdgeRT-1.png)

___

Once you have completed all of the steps, your architecture in VPC1 should look like the image below. After this, you can then move onto configuring the Security VPC.

___

![VPC1 Routing Tables](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-VPC1-Full-Architecture.png)

___

## Configuring Routing Tables on the Security VPC

___

As with the VPC 1 configuration, let's start by navigating to the **VPC** section in the console, and ensure we select the **gwlblab-SecurityVPC** within the filter, to make sure we only see the route tables of the Security VPC.
___

![Security VPC Filter](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-Filter.png)

___

| **Step** | **VPC** | **gwlblab-SecurityVPC** | **gwlblab-SecurityVPC** |
| --- | --- | --- | --- |
| **1** | **Route Table** | **gwlblab-SecurityVPC-_TGW_\-AZ1** | **gwlblab-SecurityVPC-_TGW_\-AZ2** |
| **2** | **Click** | "Routes", "Edit routes", "Add Route" | "Routes", "Edit routes", "Add Route" |
| **3** | **Destination** | **`0.0.0.0/0`** | **`0.0.0.0/0`** |
| **4** | **Route Target** | GWLB **gwlblab-securityvpc-endpoint-1a** | GWLB **gwlblab-securityvpc-endpoint-1b** |
| **5** | **Click** | "Save Changes" | "Save Changes" |
|  |  |  |  |
| **6** | **Route Table** | **gwlblab-SecurityVPC-_NATGW_\-AZ1** | **gwlblab-SecurityVPC-_NATGW_\-AZ2** |
| **7** | **Click** | "Routes", "Edit routes", "Add Route" | "Routes", "Edit routes", "Add Route" |
| **8** | **Destination** | **`10.0.0.0/8`** | **`10.0.0.0/8`** |
| **9** | **Route Target** | GWLB **gwlblab-securityvpc-endpoint-1a** | GWLB **gwlblab-securityvpc-endpoint-1b** |
| **10** | **Click** | "Save Changes" | "Save Changes" |
|  |  |  |  |

___

**Step 2** - Selecting the TGW-AZ1 and TGW-AZ2 route tables

___

![Security VPC TGW AZ1 RT](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-TGWAZ1.png)

___

**Step 3-4** - Adding the GWLB endpoint routes for the TGW-AZ1 and TGW-AZ2 route tables

___

![Security VPC TGW AZ1 RT Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-TGWAZ1-2.png) ![Security VPC TGW AZ1 RT Part 3](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-TGWAZ1-3.png)

___

**Step 5** - Results from your configuration to the two TGW route tables

___

![Security VPC TGW AZ1 RT Part 4](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-TGWAZ1-4.png) ![Security VPC TGW AZ1 RT Part 5](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-TGW-AZ2.png)

___

**Step 7** - Selecting the NATGW-AZ1 and NATGW-AZ2 route tables
___

![Security VPC NATGW AZ1](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-NATGWAZ1-1.png)

___

**Step 8-9** - Adding the GWLB endpoint routes for the TGW-AZ1 and TGW-AZ2 route tables
___

![Security VPC NATGW AZ1 Part 2](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-NATGWAZ1-2.png) ![Security VPC NATGW AZ1 Part 3](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-NATGWAZ1-3.png)

___

**Step 10** - Results from configuring the two NATGW route tables
___

![Security VPC NATGW AZ1 Part 4](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-NATGWAZ1-4.png) ![Security VPC NATGW AZ1 Part 5](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-NATGWAZ1-5.png)

Note that on all of the Security VPC route tables, you will see a pl-XXXXXXXX Destination targeting another endpoint, this is the S3 Gateway endpoint that we use in order to bootstrap the Firewall instances and have private access to our S3 buckets.

___

Great, now we are done with this section, if you have done everything correctly, your routing table configuration should now look like the below image. It is very important for you to understand how the routing works within your VPC, in order for you to troubleshoot your environment in the future if needed.

___

![Security VPC Routing Tables](https://static.us-east-1.prod.workshops.aws/public/f851c967-3238-43e9-9ce2-92b5e4a5ab82/static/gwlb/30-infra/rt/RT-SecurityVPC-Full-Architecture.png)

___

Now you can move onto the third-party appliances, go ahead and select from the menu in the left hand side, what third-party software you would now like to use with this lab.

---
Once you have completed deploying all of the configurations, now we can move onto choosing which one of our 3rd party Firewall applications we would like to use.

You can choose which Firewall instance you want to deploy from the list below, or navigate in the menu on the left hand side for the vendor you want to use.
___
- [Open Source Suricata](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/50-opensource-suricata)
- [Fortinet](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/150-fortinet)
- [Palo Alto](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/200-paloalto)
- [Cisco FTD](https://catalog.us-east-1.prod.workshops.aws/workshops/ae291640-10fe-4c0b-982f-9b9a61dbad26/en-US/250-cisco)