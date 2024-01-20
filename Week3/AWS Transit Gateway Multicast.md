---
tags:
  - 12weeksawsworkshopchallenge
---


# Prerequisites
## [AWS Account](https://catalog.workshops.aws/networking/en-US/advanced/multicast/prereq/aws-account#aws-account)
You may complete these labs with your own AWS account. Make sure that you have the proper permissions to create CloudWatch, EC2, and IAM resources. No additional setup is required.

Please be aware that if you complete this workshop in your own AWS account, costs will be incurred.

### Create the Base Infrastructure
If you did not complete [Foundational Topics - Multiple VPCs](https://catalog.workshops.aws/networking/en-US/foundational/multivpc), you must:

1. Complete the [Foundational Topics - Prerequisites](https://catalog.workshops.aws/networking/en-US/foundational/prereqs/aws-account) section.
    
2. Download the following CloudFormation template to create three VPCs connected by a Transit Gateway:
    
    [CloudFormation template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/multi-vpc-create-all-vpcs-and-tgw.yaml)
    
3. Navigate to [CloudFormation](https://console.aws.amazon.com/cloudformation/home)  section in the AWS console. Click **Create stack** button and select **With new resources (standard)**.
    
4. Under **Specify template**, select **Upload a template file**, click **Choose file** and select the CloudFormation template that you downloaded above. Click **Next**.
    
5. Enter the **Stack name** `NetworkingWorkshopMultiVPCandTGW`. Leave the parameter defaults unchanged if you are running in us-east-1 and click **Next**. If you are running in another region, update the availability zones. ![CFN Stack Details](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/pre-reqs/cfn_stack_details.png)
    
6. Scroll to the bottom of the **Configure stack options** page and click **Next** again.
    
7. Scroll to the bottom of the **Review** page and click the **Submit** button.
    
8. The CloudFormation will begin deployment which you can follow by refreshing the **Events** and **Resources** tab.
    
9. Once the CloudFormation template finishes lets go take a look at what it created. Click on the resources tab and you will find all the resources that were built as part of the CloudFormation stack.
    

This stack will create 3 VPCs each with two public subnets and two private subnets, a Transit Gateway to provide private connectivity between the VPCs, as well as EC2 instances in the VPCs.

Wait for the stack to be created before proceeding. This can take 3-5 minutes. Verify that the status in CloudFormation Console show as `CREATE_COMPLETE`, indicating successful creation of the stack.

![Multiple VPCs created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/multiple_vpcs_completed.png)

If you do not complete this prerequisite, portions of the multicast lab that depend on unicast routing will not work.


# Lab 1: Multicast in AWS

### [Review Multicast Networking](https://catalog.workshops.aws/networking/en-US/advanced/multicast/lab1#review-multicast-networking)

**Multicast** is a<mark style="background: #FFB86CA6;"> communication protocol used for delivering a single stream of data to multiple receiving computers simultaneously.</mark>  
AWS Transit Gateway supports routing multicast traffic between hosts in the same subnet or between subnets of attached VPCs, and it serves as a multicast router for instances sending traffic destined for multiple receiving instances.  
While an AWS VPC by itself does not support multicast, Transit Gateway can provide this new capability.  
<mark style="background: #FFF3A3A6;">Transit Gateway will act as the rendezvous point, receiving the packets from the multicast source, replicate it, and send it to the multicast receiver.</mark>  
Transit Gateway supports routing multicast using the Internet Group Management Protocol (IGMP) protocol or static source and member configurations.

### Understanding the Lab Environment

In this lab, you will configure a multicast domain on the Transit Gateway created in [Foundational Topics - Multiple VPCs](https://catalog.workshops.aws/networking/en-US/foundational/multivpc). Recall at the end of that lab, you provisioned 3 VPCs and connected them using Transit Gateway with unicast networking. You had enabled multicast routing support when you created the Transit Gateway. The following diagram is the network topology for this lab.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/image1.png)

___

For this lab, please be aware of the following:
- You will create a transit gateway multicast domain on the existing Transit Gateway.
- Multicast group membership is managed using the Amazon VPC Console, the AWS CLI, or IGMP.
    - In this lab, IGMPv2 is automatically configured on the EC2 instances created by the CloudFormation stack. IGMPv3 is currently not supported.
    - To check the IGMP version run `cat /proc/net/igmp` on the EC2 instances.
- A subnet can only be in one multicast domain.

For additional considerations outside of those needed for this lab, please see the [Multicast on Transit Gateways](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-multicast-overview.html#limits)  documentation.

## Deploy Lab Resources

### Create the CloudFormation Stack

A CloudFormation template has been provided to create the EC2, and Cloudwatch dashboard resources used in this lab. The CloudFormation template also modifies the VPC A and VPC B inbound security groups to allow IGMP and the multicast test traffic.

| Type | Protocol | Port range | Source | Description |
| --- | --- | --- | --- | --- |
| Custom Protocol | IGMP(2) | All | 0.0.0.0/32 | IGMP query |
| Custom UDP Protocol | UDP | 8123 | 10.0.0.0/8 | Inbound multicast traffic |

[CloudFormation template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/cfn/TGW-Multicast-Lab1.yaml)

1. Open the **AWS CloudFormation** console [https://console.aws.amazon.com/cloudformation/](https://console.aws.amazon.com/cloudformation/) .
    
2. Click **Create stack** (With new resources).
    
3. Choose **Template is ready**, then choose **Upload a template file**, and click **Choose file**.
    
4. Select the file `TGW-Multicast-Lab1.yaml`, click **Open**, and then click **Next**.
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step1/image1.png)

1. Name the stack `ImmersionDay-Multicast-Lab1`. This stack will create three (3) EC2 instances, and a CloudWatch dashboard. The EC2 Instances automatically update, install required network tools for the exploratory section of this lab, and reboot after launch. The status check for each EC2 instance should display _2/2 checks passed_ when complete.
    
2. In the **EC2 Setup** section, select a subnet and corresponding security group for the source and member EC2 instances. The EC2 instances can be in the same or different VPCs and subnets, or any combination thereof. You will connect to the EC2 instances using Session Manager.
    

- Source Subnet: `VPC A Private Subnet AZ1`
- Source Security Group: `VPC A Security Group`
- Member Subnet:`VPC A Private Subnet AZ2`
- Member Security Group: `VPC A Security Group`
- Member Subnet: `VPC B Private Subnet AZ2`
- Member Security Group: `VPC B Security Group`
- EC2 Instance Type: Leave as default
- Latest Amazon Linux 2 AMI: Leave as default

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step1/image2.png)

1. Click **Next**, leave all options default, and click **Next** again.
    
2. Scroll to the bottom of the page and click **Submit**.
    

If you receive the error `You have specified two resources that belong to different networks` during the stack creation, your Security Group to VPC mapping is incorrect.

1. Wait for the stack to be created and then click the **Outputs** tab. Take note of the IPv4 Address for each member instance.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step1/image3.png)

| **EC2 Instance** | **IPv4 Address** |
| --- | --- |
| MulticastMember1 |  |
| MulticastMember2 |  |

## Set up Transit Gateway

### Setting Up Multicast on Transit Gateway
You may use the existing Transit Gateway created in the foundational workshop since multicast was enabled at creation time.

#### Create a Static Multicast Domain
<mark style="background: #BBFABBA6;">A multicast domain allows segmentation of a multicast network into different domains, and makes the Transit Gateway act as multiple multicast routers.</mark>  
You define multicast domain membership at the subnet level. For this domain, you will configure static sources support which allows the addition of multicast members as sources. Only multicast sources are allowed to send multicast data.

1. In the left hand pane of the VPC Dashboard scroll down and click on [Transit Gateway Multicast](https://console.aws.amazon.com/vpc/home#TransitGatewayMulticastDomains:) , and click **Create transit gateway multicast domain**.
2. Name the domain `TGW-Multicast-Static` and choose the multicast Transit Gateway (TGW) from the _Transit gateway ID_ dropdown.
3. Select **Static sources support** and click **Create transit gateway multicast domain**.
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step2/image4.png)

1. Select the static multicast domain (TGW-Multicast-Static), click the **Associations** tab, and click **Create association**.  
    ![[Pasted image 20240120101657.png]]
2. For each VPC, <mark style="background: #FFF3A3A6;">choose the two private subnets that will be allowed to send and receive multicast traffic</mark>, and click **Create association**.
    
![[Pasted image 20240120102656.png]]

<mark style="background: #FF5582A6;">Make sure you repeat the previous step for VPC B and VPC C</mark>  
![[Pasted image 20240120111908.png]]  
![[Pasted image 20240120112014.png]]

1. Select the static multicast domain (TGW-Multicast-Static), click the **Groups** tab, and click **Add source**. Enter `239.0.0.100` for the _Group IP address_, select the source network interface, and click **Add group sources**.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step2/image6.png)

1. Click the **Groups** tab and click **Add member**. Enter `239.0.0.100` for the _Group IP address_, select the member network interfaces, and click **Add Group Members**.

![[Pasted image 20240120113501.png]]
#### Create a Dynamic Multicast Domain

<mark style="background: #FFF3A3A6;">Transit Gateway supports Internet Group Management Protocol (IGMP) for simplified deployment and management of multicast applications. </mark>  
For this domain, you will configure IGMP support which enables Transit Gateway to dynamically add and delete multicast members based on IGMP protocol interactions.

This feature provides real-time visibility into the multicast network and enables you to accurately keep track of group membership changes over time. You cannot use IGMPv2 and Static Sources support at the same time.

1. In the navigation pane, choose **Transit Gateway Multicast**, and click **Create transit gateway multicast domain**.
    
2. Name the domain `TGW-Multicast-Dynamic` and choose the multicast Transit Gateway (TGW) from the _Transit gateway ID_ dropdown.
    
3. Select **IGMPv2 support** and click **Create transit gateway multicast domain**.
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step2/image8.png)

<mark style="background: #FFF3A3A6;">Since a subnet can only be in one multicast domain, you will associate subnets to this domain later in the lab.</mark>

## Explore Multicast

### Connecting to the Application Instances

<mark style="background: #D2B3FFA6;">Session Manager provides secure and auditable instance management without the need to open inbound ports, maintain bastion hosts, or manage SSH keys.</mark>  
Follow the steps below to connect to the application instances using Session Manager.

1. Open the **Amazon EC2** console
2. In the navigation pane, choose **Instances**.
3. Select the source instance (Multicast-Source) and choose **Connect**.
4. Click the **Session Manager** tab and choose **Connect**.

If you see the warning **Can't connect to your instance**, click the **SSH client** tab and then the **Session Manager** tab to refresh. ![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/warning1.png)

1. <mark style="background: #FFB86CA6;">Repeat these steps to connect to the member instances (Multicast-Member-1, Multicast-Member-2)</mark>.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image1.png)

### Running a Multicast Application

As part of the CloudFormation stack, the EC2 instances download and configure a Python-based script ([mcast.py](https://svn.python.org/projects/python/trunk/Demo/sockets/mcast.py) ) that will be used to send and receive multicast traffic. The script is renamed to _mcast\_app_ and automatically configured to join the 239.0.0.100 group IP address.

If you get an error message trying to open _mcast\_app_ such as `command not found` or `Permission denied`, or the app closes immediately, run the command `sudo mcast_app_install.sh` to force a re-install.

1. <mark style="background: #FFB86CA6;">On the source instance, run the command</mark>  
   `mcast_app -s`
2. <mark style="background: #FFB86CA6;">On the member instances, run the command</mark> `mcast_app`
3. <mark style="background: #ABF7F7A6;">Observe that the member instances begin receiving multicast traffic from the source instance.</mark> The output will contain the source IPv4 address, source port, and timestamp.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image2.png)

1. Press `CTRL+C` on the member instances to stop the multicast application.
    
2. While still on the member instances, run the command `sudo tcpdump -i eth0 -n port 8123`.
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image3.png)

1. Observe that the member instances continue to receive multicast traffic. _Why is this the case?_
    
2. Press `CTRL+C` on the source and member instances to stop the multicast application and packet capture.
    
### Using IGMP for Group Membership

IGMP support on Transit Gateway makes it easier to scale up multicast workloads, while also simplifying the management of multicast group membership and network deployment. You do not need to configure static multicast groups, sources, and receivers when building a multicast network in AWS. Transit Gateway dynamically adds and deletes multicast members based on IGMP protocol interactions.

#### Configure the Multicast Domain

1. Open the **Amazon VPC** console [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/) .
2. In the navigation pane, choose **Transit Gateway Multicast**
3. <mark style="background: #ADCCFFA6;">Disassociate all of the subnets from the static multicast domain (TGW-Multicast-Static)</mark>.
4. <mark style="background: #FFF3A3A6;">Reassociate the subnets to the dynamic multicast domain (TGW-Multicast-Dynamic).</mark>
    

#### Run the Multicast Application

1. On the source instance, run the command `mcast_app -s`.
    
2. On the member instances, run the command `mcast_app`.
    
3. Navigate to the **TGW-Multicast-Dynamic** multicast domain and select Groups.
    
4. Observe that the 239.0.0.100 multicast group was automatically created.
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image4.png)

Transit Gateway automatically joins the 224.0.0.1 (All Systems) multicast group and sends membership query packets to all the IGMP members so that it can track multicast group membership.

1. After a few moments, confirm that the member instances are receiving multicast traffic from the source instance.

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image5.png)

1. Press `CTRL+C` on one of the member instances to stop the multicast application.
    
2. On the same member instance, run the command `sudo tcpdump -i eth0 -n port 8123`.
    
3. Observe that this member instance is no longer receiving multicast traffic; however, the other member instance is still receiving multicast traffic. _Why is this the case?_
    
4. Press `CTRL+C` on the source and member instances to stop the multicast application and packet capture.
    

### Testing Bandwidth Conservation

As part of the CloudFormation stack, the EC2 instances download and install iPerf, an open-source bandwidth measurement tool. You will use iPerf to confirm that multicast conserves bandwidth by delivering a single stream of data to multiple receiving computers simultaneously.

#### Unicast

1. On the member instances, run the command `iperf -s -p 8123 -u`.
    
2. On the source instance, run the following commands. Replace **<MEMBER\_1\_IP>** and **<MEMBER\_2\_IP>** with the IPv4 Addresses that you noted in step 9 of the [Deploy Lab Resources](https://catalog.workshops.aws/networking/en-US/multicast/lab1/step1) section of this lab.
    
    `iperf -c <MEMBER_1_IP> -p 8123 -u -b 1M -t 600 &`
    
    `iperf -c <MEMBER_2_IP> -p 8123 -u -b 1M -t 600 &`
    
3. Open the Amazon CloudWatch console at [https://console.aws.amazon.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/) .
    
4. In the navigation pane, choose **Dashboards**.
    
5. Choose the network dashboard (Network-Throughput).
    
6. In the refresh dropdown, choose **10s** and allow 2 to 3 minutes for the graph to populate.
    
7. Observe that **NetworkOut** on the source instance is at twice the level as **NetworkIn** on the member instances. _Why is this the case?_
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image6.png)

1. Press `CTRL+C` on the source and member instances to stop the unicast throughput test.

[

#### Multicast
1. On the member instances, run the following command: `iperf -s -B 239.0.0.100 -p 8123 -u`
    
2. On the source instance, run the following command: `iperf -c 239.0.0.100 -p 8123 -u -b 1M -T 32 -t 600`
    
3. Navigate back to the network dashboard (Network-Throughput) and allow 2 to 3 minutes for the graph to populate.
    
4. Observe that **NetworkOut** on the source instance is at the same level as **NetworkIn** on both the member instances. _Why is this the case?_
    

![](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/multicast/lab1/step3/image7.png)

1. Press `CTRL+C` on the source and member instances to stop the multicast throughput test.


# Clean up
[

### Review

](https://catalog.workshops.aws/networking/en-US/advanced/multicast/lab1/step4#review)

In this lab, you deployed a separate Transit Gateway with multicast support for static and dynamic domains. Next, you verified multicast connectivity across multiple VPCs and subnets using a sample Python script. You performed a packet capture to understand how multicast traffic is routed differently in a static versus dynamic domains. Lastly, using iPerf and CloudWatch you confirmed that Transit Gateway delivered a single stream of data to multiple receiving instances simultaneously.

[

### Clean Up

If you are using your own AWS Account to conduct this workshop and you are finished, complete the follow steps to clean up.

Make sure you terminate / delete the resources below to avoid unnecessary charges.

#### Disassociate the Subnets from the Multicast Domain.

Step-by-Step Instructions

1. Open the **Amazon VPC** console at [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/) .
    
2. On the navigation pane, choose **Transit Gateway Multicast**.
    
3. Select the dynamic multicast domain.
    
    > TGW-Multicast-Dynamic
    
4. Click the **Associations** tab.
    
5. Select one subnet at a time, choose **Delete association** from the **Actions** dropdown, and click **Delete association** to confirm.


#### Delete Each Multicast Domain.

Step-by-Step Instructions

1. Open the **Amazon VPC** console at [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/) .
    
2. On the navigation pane, choose **Transit Gateway Multicast**.
    
3. Select one multicast domain at a time.
    
    > TGW-Multicast-Dynamic
    > 
    > TGW-Multicast-Static
    
4. Choose **Delete multicast domain** from the **Actions** dropdown, enter `delete`, and click **Delete** to confirm.
    

#### Delete the CloudFormation Stack.

Step-by-Step Instructions

1. Open the **AWS CloudFormation** console at [https://console.aws.amazon.com/cloudformation](https://console.aws.amazon.com/cloudformation) .
    
2. On the navigation pane, choose **Stacks**.
    
3. Select the multicast CloudFormation stack and click **Delete**.
    
    > ImmersionDay-Multicast-Lab1
    
4. Click **Delete stack** to confirm.
    

The stack deletion operation can't be stopped once the stack deletion has started. The stack proceeds to the `DELETE_IN_PROGRESS` state. After the stack deletion is complete, the stack will be in the `DELETE_COMPLETE` state.

#### Delete the MultiVPC and Prerequisites CloudFormation Stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPC`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    

Option 3: You started the lab in the Basic Security or Multicast sections by deploying the CloudFormation template

#### Delete the MultiVPC and Prerequisites CloudFormation Stack
Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPCandTGW`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
