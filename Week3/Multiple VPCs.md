---
tags:
  - 12weeksawsworkshopchallenge
---

### VPC Peering & Transit Gateway Overview

<mark style="background: #ADCCFFA6;">A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them using private IPv4 addresses or IPv6 addresses. Instances in either VPC can communicate with each other as if they are within the same network.</mark>  
You can create a VPC peering connection between your own VPCs, or with a VPC in another AWS account. The VPCs can be in different regions (also known as an inter-region VPC peering connection).

VPC Peering is a useful approach to connect a small number of pairs of VPCs however managing point-to-point connectivity across many VPCs without the ability to centrally manage the connectivity policies, can be operationally costly and cumbersome. For on-premises connectivity, you need to attach your AWS VPN to each individual VPC. This solution can be time consuming to build and hard to manage when the number of VPCs grows into the hundreds.

AWS Transit Gateway is a service that enables you to connect VPCs and on-premises networks to a single gateway. As you grow the number of workloads running on AWS, you need to be able to scale your networks across multiple accounts and Amazon VPCs to keep up with the growth.

In this lab, you will learn how to peer VPCs, and also create a Transit Gateway, attach VPCs, and configure routing with the Transit Gateway route tables.

If you are running this lab in **AWS Workshop Studio**, the region has been set by your facilitator. The region you see in screenshots may not match your environment. This will not cause any problems.

If you are running this lab in **your own AWS Account**, it is recommended for all lab resources to be created in us-east-1 region so that the screenshots match your environment. This is not mandatory.

For this lab we will use CloudFormation to automatically build VPCs.

## Prerequisites

If you have not completed the VPC Fundamentals section...

1. If you **have not** completed the previous labs AND you are running this lab in **your own AWS Account**, you must complete the [prerequisites](https://catalog.workshops.aws/networking/en-US/foundational/prereqs/aws-account) section. If you are running this lab in **AWS Workshop Studio**, you may move to the next step.
    
2. If you **have not** completed the previous labs and you are starting at this step then load this CloudFormation template to create all three VPCs:
    
    [Create all three VPCs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/multi-vpc-create-all-vpcs.yaml)
    

\-or-

If you have completed the VPC Fundamentals section...

1. If you **have** completed the VPC Fundamentals lab and already have one VPC set up then load this CloudFormation template to create two additional VPCs:
    
    [Create two additional VPCs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/multi-vpc-add-extra-vpcs.yaml)
    
2. Navigate to [CloudFormation](https://console.aws.amazon.com/cloudformation/home)  section in the AWS console. Click **Create stack** button and select **With new resources (standard)**.
    
3. Under **Specify template**, select **Upload a template file**, click **Choose file** and select the CloudFormation template that you downloaded above. Click **Next**.
    
4. Enter the **Stack name** `NetworkingWorkshopMultiVPC`. Leave the parameter defaults unchanged if you are running in us-east-1 and click **Next**. If you are running in another region, update the availability zones. ![CFN Stack Details](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/pre-reqs/cfn_stack_details.png)
    
5. Scroll to the bottom of the **Configure stack options** page and click **Next** again.
    
6. Scroll to the bottom of the **Review** page and click the **Submit** button.
    
7. The CloudFormation will begin deployment which you can follow by refreshing the **Events** and **Resources** tab.
    
8. Once the CloudFormation template finishes lets go take a look at what it created. Click on the resources tab and you will find all the resources that were built as part of the CloudFormation stack.
    

If the deployment of the CloudFormation template fails with the error that starts with `Value (NetworkingWorkshopInstanceProfile) for parameter iamInstanceProfile.name is invalid.`, please make sure you have completed the [prerequisites](https://catalog.workshops.aws/networking/en-US/foundational/prereqs) section

If the deployment of the CloudFormation template fails due to the **Elastic IP address** limit being reached and you are using your own AWS account, please refer to the **Elastic IP Quota** section in [Prerequisites - Your own AWS Account](https://catalog.workshops.aws/networking/en-US/foundational/prereqs/aws-account).

Now go to [Your VPCs](https://console.aws.amazon.com/vpc/home?#vpcs:)  and take a look.

<mark style="background: #ABF7F7A6;">You will find two new VPCs</mark> **VPC A**, **VPC B** and **VPC C** <mark style="background: #ABF7F7A6;">with CIDR blocks of 10.0.0.0/16, 10.1.0.0/16 and 10.2.0.0/16. Each VPC has two public and two private subnets.</mark>

![CFN VPCs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/pre-reqs/cfn_created_vpcs.png)

Each VPC has an EC2 instance in the private subnet located in the us-east-1a availability zone.

![CFN VPCs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/pre-reqs/cfn_created_instances.png)

The environment now looks something like the following. You may notice that **VPC A** is missing the TGW subnets. You will be creating these in the Transit Gateway section. ![Multi-VPC setup](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_cloudformation.png)

# VPC Peering
Please note, this section is optional and is not required to be completed to proceed with this workshop. You may skip to [Transit Gateway](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/transit-gw)

A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them using private IPv4 addresses or IPv6 addresses. 

In this lab, we will establish VPC peering connections between _VPC A_ and _VPC B_, as well as between _VPC A_ and _VPC C_ and show that traffic flows between only those VPCs with direct peering links.

![VPC Peering](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_vpc_peering.png)

> Note that all three VPCs have non-overlapping CIDRs. You cannot create a VPC peering connection between VPCs with matching or overlapping IPv4 CIDR blocks.

[

## Setup VPC A and VPC B Peering

### Create the Peering Connection Between VPCs A & B

1. In the VPC Dashboard click on [Peering Connections](https://console.aws.amazon.com/vpc/home?#PeeringConnections:sort=vpcPeeringConnectionId) 
    
2. Click on **Create peering connection** in the right hand corner
    
    ![Peering Button](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_create_button.png)
    
3. Specify the Peering connection name as `VPC A <> VPC B`
    
4. Under **Select a local VPC to peer with** select `VPC A` as **VPC ID (Requester)**
    
    ![Select Requester](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_select_requester.png)
    
5. Under **Select another VPC to peer with** ensure that `My Account` is selected for **Account**
    
6. For **Region** select the region for this workshop `This Region (us-east-1)`.
    
7. For **VPC ID (Accepter)** select `VPC B`
    
    ![Select Accepter](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_select_accepter.png)
    
8. Click on **Create peering connection**
    
    ![Create Connection](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_create_connection.png)
    
9. The newly created peering connection will be in _Pending Acceptance_ state.
    
10. On the resulting screen, navigate under **Actions** and click **Accept request**
    
![Select Accept](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_accept_request.png)
    
1. On the following pop-select, click **Accept request**
    
![Accept Request](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_popup_accept.png)
    
1. Click on **Modify my route tables now** in the resulting screen
    
![Modify Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_modify_route_tables.png)
    
### Update Route Table in VPC A

1. Select the _check box_ for the `VPC A Private Route Table`
    
2. Scroll down and click on **Routes** tab
    
3. Click **Edit routes** ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_route_table_vpc_a.png)
    
4. Add route entry for "VPC B" using the CIDR range `10.1.0.0/16` and selecting Peering Connection `VPC A <> VPC B` for the target
    
    ![Add VPC B Route](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_route_select_vpc_b.png)
    
5. Click **Save changes**
    
6. Confirm that the new route appears in the **Routes** tab of the resulting screen
    
    ![Routes Updated](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_routes_updated_vpc_a.png)
    

### Update Route Table in VPC B

1. Click on **Route tables**
    
2. Select _check box_ for `VPC B Private Route Table`
    
3. Click on **Routes** tab
    
4. Click **Edit routes**
    
    ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_route_table_vpc_b.png)
    
5. Add a route entry for **VPC A** using CIDR range `10.0.0.0/16` as the Destination and `VPC A <> VPC B` as the target
    
    ![Select Target](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_route_select_vpc_a.png)
    
6. Click **Save changes**
    
7. The route table will be updated with routes for the peering connection
    
    ![Route Table Updated](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_routes_updated_vpc_b.png)
    


## Setup VPC A and VPC C Peering

### Create the Peering Connection Between VPCs A & C

1. In the VPC Dashboard click on [Peering Connections](https://console.aws.amazon.com/vpc/home?#PeeringConnections:sort=vpcPeeringConnectionId) 
    
2. Click on **Create peering connection** in the right hand corner
    
    ![Create Connection](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_button.png)
    
3. Specify the Peering connection name as `VPC A <> VPC C`
    
4. Under **Select a local VPC to peer with** select `VPC A` as **VPC ID (Requester)**
    
5. Under **Select another VPC to peer with** ensure that `My Account` is selected for **Account**
    
6. For **Region** select the region for this workshop `This Region (us-east-1)`.
    
7. For **VPC ID (Accepter)** select `VPC C`
    
    ![Select VPC C](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_settings.png)
    
8. Click on **Create peering connection** ![Create](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_create.png)
    
9. The newly created peering connection will be in _Pending Acceptance_ state.
    
10. On the resulting screen, navigate under **Actions** and click **Accept request**
    
    ![Accept](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_actions_accept.png)
    
11. On the following pop-select, click **Accept request**
    
12. Click on **Modify my route tables now** in the resulting screen
    
    ![Modify Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_modify_rt_button.png)
    

### Update Route Table in VPC A

1. Select _check box_ for `VPC A Private Route Table`
    
2. Scroll down and click on **Routes** tab
    
3. Click **Edit routes**  
    ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_route_table_vpc_a.png)
    
4. <mark style="background: #ADCCFFA6;">Add route entry for "VPC C" using the CIDR range</mark> `10.2.0.0/16` <mark style="background: #ADCCFFA6;">and selecting Peering Connection</mark> `VPC A <> VPC C` <mark style="background: #ADCCFFA6;">for the target</mark>
    
5. Click **Save changes**  
    ![Save](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_route_select_vpc_c.png)
    
6. Confirm that the new route appears in the **Routes** tab of the resulting screen
    
    ![Routes Tab](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_routes_updated_vpc_a.png)
    

[

### Update Route Table in VPC C

1. Navigate back to **Route Tables** and select _check box_ for `VPC C Private Route Table`
    
2. Click on **Routes** tab
    
3. Click **Edit routes**
    
    ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_route_table_vpc_c.png)
    
4. Add a route entry for **VPC A** using CIDR range `10.0.0.0/16` as the Destination and `VPC A <> VPC C` as the Target
    
    ![Add Route](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_route_select_vpc_a.png)
    
5. Click **Save changes**
    
6. The route table will be updated with routes for the peering connection
    
    ![Route Table Updated](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ac_routes_updated_vpc_c.png)
    

## Check Connectivity

### Check Connectivity from VPC A

1. Proceed to [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running) .
    
2. Select the `VPC A Private AZ1 Server` EC2 instance and click the **Connect** button above
    
    ![Select Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_select_vpc_a_instance.png)
    
3. Click **Connect** in the **Session Manager** tab
    
4. Try pinging EC2 instances in **VPC B** and **VPC C** using the private addresses of the instances
    

```bash
ping 10.1.1.100 -c 5 
ping 10.2.1.100 -c 5
```

<mark style="background: #FFB86CA6;">If peering and routing are configured correctly, you should be able to ping both instances.</mark>  
 ![Ping success](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_ping_from_vpc_a.png)

### Check Connectivity from VPC B
1. Terminate the Session Manager connection and in the resulting screen click on [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running) .
    
2. Select **VPC B Private AZ1 Server** EC2 instance and connect using Session Manager.  
    ![Session Manager](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_select_vpc_b_instance.png)
    
3. Ping the EC2 instance in VPC A using the IP address `10.0.1.100`  
    ![Ping A from B](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_ping_from_vpc_b_to_a.png)
    
4. Can you ping the instance in VPC C using the IP address `10.2.1.100`?  
```
ping 10.2.1.100 -c 5
```
![Ping C from B](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_ping_from_vpc_b_to_c.png)
    
<mark style="background: #BBFABBA6;">There is no direct peering between VPC B and VPC C. VPC B and VPC C cannot communicate via VPC A because VPC peering does not permit transitive routing.</mark>
    
1. Terminate the Session Manager connection and close the browser tab.
    

_**Congratulations**_ you've set up a peering architecture that connects VPC A to VPC B and VPC C but prevents VPC B and VPC C communicating.

While this approach can be used to interconnect many VPCs, managing many point-to-point connections can be cumbersome at scale.  
<mark style="background: #FFF3A3A6;">A more scalable approach is to utilize AWS Transit Gateway so we will now remove the point-to-point peering connections between VPCs in preparation for setting up Transit Gateway (TGW) to interconnect the three VPCs</mark>

## Delete VPC Peering Connections

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/vpc-peering#delete-vpc-peering-connections)

1. In the VPC Dashboard navigate to [Peering Connections](https://console.aws.amazon.com/vpc/home?#PeeringConnections:sort=vpcPeeringConnectionId) 
    
2. Select the `VPC A <> VPC B` peering connectoin and delete it by clicking **Actions** and selecting **Delete peering connection**
    
    ![Select Connection](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_delete.png)
    
3. Select the checkbox to **Delete related route table entries** to avoid traffic blackholing scenario.
    
    ![Select Checkbox](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/vpc-peering/peering_vpcs_ab_delete_confirm.png)
    
4. Type `delete` in the text box and click **Delete**
    
5. Repeat deletion of VPC peering for the `VPC A <> VPC C` connection.
    

_**Congratulations you now have now completed this section of the lab.**_

# Transit Gateway
<mark style="background: #BBFABBA6;">AWS Transit Gateway connects your Amazon Virtual Private Clouds (VPCs) and on-premises networks through a central hub.</mark>  
This simplifies your network and puts an end to complex peering relationships. It <mark style="background: #FFF3A3A6;">acts as a cloud router – each new connection is only made once.</mark>

## Create the Transit Gateway

1. In the left hand pane of the VPC Dashboard scroll down and click on [Transit Gateways](https://console.aws.amazon.com/vpc/home?#TransitGateways:) 
    
2. Click on **Create Transit Gateway**  
    ![Create TGW](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_create.png)
    
3. Add a name for the new Transit Gateway as `TGW` and add a description of `TGW for us-east-1`.
    
    ![TGW Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_name_description.png)
    
4. Select `Multicast support` and keep the remaining settings at the defaults. You will need this option enabled if you progress to the advanced multicast lab. Click on **Create transit gateway**.  
    ![Click Create TGW](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_settings.png)
    
5. The state of the new transit gateway will show as _**pending**_ for a few minutes.
    
    ![TGW Created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_created.png)
    

## Attach VPCs to Transit Gateway

<mark style="background: #D2B3FFA6;">The best practice for connecting VPCs to Transit Gateway is to use a dedicated /28 subnet in each availability zone</mark> and the CloudFormation run earlier created these for VPC B and VPC C alongside two private and public /24 subnets for hosting workloads.

However the "VPC Fundamentals" lab only created the two public and two private /24 subnets for VPC A and our AWS environment currently looks like this:

![VPC A after CFN](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_cloudformation.png)

Therefore before we create the transit gateway we need to add a dedicated /28 subnet in each availability zone in **VPC A** for the transit gateway attachments.

### Create Transit Gateway Subnets in VPC A

1. Within the VPC Dashboard click on [Subnets](https://console.aws.amazon.com/vpc/home?#subnets:)  and click the **Create subnet** button  
    ![Subnets](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_create_subnet.png)
    
2. Create a subnet under VPC A with a name of `VPC A TGW Subnet AZ1` in `us-east-1a` with a CIDR block of `10.0.5.0/28`  
    ![Create Subnet](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_subnet_az1.png)
    
3. Create another subnet under VPC A with a name of `VPC A TGW Subnet AZ2` in `us-east-1b` with a CIDR block of `10.0.5.16/28`
    
    ![Create Subnet](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_subnet_az2.png)
    

<mark style="background: #ADCCFFA6;">Now that we have subnets to place the transit gateway attachments into we will attach VPC A, VPC B, and VPC C to the transit gateway and test connectivity between our EC2 instances in each VPC.</mark>

![Aligned VPCs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_aligned_vpcs.png)


### Create Transit Gateway Attachment for VPC A
1. On the left navigation pane go to [Transit Gateway Attachments](https://console.aws.amazon.com/vpc/home?#TransitGatewayAttachments:) 
    
2. Click on **Create Transit Gateway Attachment**.  
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_create.png)
    
3. Enter `VPC A Attachment` as the **Name tag**.
    
4. Select the transit gateway from the dropdown for **Transit Gateway ID**.
    
5. Leave **Attachment Type** as `VPC`
    
    ![TGW Attachment Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_name_vpc_a.png)
    
6. Select **VPC A** from the **VPC ID** drop down.
    
7. <mark style="background: #D2B3FFA6;">Select</mark> `VPC A TGW Subnet AZ1` and `VPC A TGW Subnet AZ2` for the **Subnet IDs**.  
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_settings_vpc_a.png)
    

Note that the TGW subnets will not be selected by default, double check the subnets are the TGW ones.

1. Click **Create transit gateway attachment** on the bottom right corner.
    
2. VPC attachment should create successfully and will be in a pending state initially  
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_success_vpc_a.png)
    
### Create Transit Gateway Attachment for VPC B

1. Click **Create Transit Gateway Attachment**.
    
2. Enter `VPC B Attachment` as the **Name tag**.
    
3. Select the transit gateway from the dropdown for **Transit Gateway ID**.
    
4. Leave **Attachment Type** as `VPC`
    
    ![TGW Attachment Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_name_vpc_b.png)
    
5. Select **VPC B** from the **VPC ID** drop down
    
6. Select `VPC B TGW Subnet AZ1` and `VPC B TGW Subnet AZ2` for the **Subnet IDs**.
    
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_settings_vpc_b.png)
    

Note that the TGW subnets will not be selected by default, double check the subnets are the TGW ones.

1. Click **Create transit gateway attachment** on the bottom right corner.
    
2. VPC attachment should create successfully and will be in a pending state initially
    
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_success_vpc_b.png)
    
### Create Transit Gateway Attachment for VPC C

1. Click **Create Transit Gateway Attachment**.
    
2. Name the attachment as `VPC C attachment`.
    
3. Select the transit gateway from the dropdown for **Transit Gateway ID**.
    
4. Leave **Attachment Type** as `VPC`
    
    ![TGW Attachment Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_name_vpc_c.png)
    
5. Select `VPC C` from the **VPC ID** drop dowm.
    
6. Select `VPC C TGW Subnet AZ1` and `VPC C TGW Subnet AZ2` for the **Subnet IDs**.
    
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_settings_vpc_c.png)
    

Note that the TGW subnets will not be selected by default, double check the subnets are the TGW ones.

1. Click **Create transit gateway attachment** on the bottom right corner.
    
2. VPC attachment should create successfully and will be in a pending state initially
    
    ![TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_attachment_success_vpc_c.png)
    

Let’s go take a look at how the Transit Gateway attached to the VPC.

![Attachments](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_attachments.png)

On the left in the EC2 Dashboard click on [Network Interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC:) 

![Network Interfaces](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_network_interfaces.png)

There are now six interfaces with a Description beginning `Network Interface for Transit Gateway Attachment...` representing the Elastic Network Interfaces that have been placed in each of the two Transit Gateway subnets in each of the three VPCs.

Now that we have attachments in all three VPCs, we need to add routes to their route tables to point traffic to the interfaces.

## Add Routes to TGW in the VPC Route Tables

1. On the left in the VPC Dashboard click on [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:sort=tag:Name) 
    
2. Select the _check box_ for `VPC A Private Route Table`, scroll down select the **Routes** tab and click on **Edit routes**. ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_select_vpc_a.png)
    
3. <mark style="background: #BBFABBA6;">Add a route to VPC B in the</mark> **VPC A Private Route Table** <mark style="background: #BBFABBA6;">using a Destination of</mark> `10.1.0.0/16` <mark style="background: #BBFABBA6;">with a Target as the transit gateway.</mark>
    
4. <mark style="background: #BBFABBA6;">Add a route to VPC C in the</mark> **VPC A Private Route Table** <mark style="background: #BBFABBA6;">using a Destination of</mark> `10.2.0.0/16` <mark style="background: #BBFABBA6;">with a Target as the transit gateway.</mark>
    
5. Click **Save routes**.  
    ![Select Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_edit_vpc_a.png)
    
6. Confirm that the routes have been added to the route table.
    
    ![Routes Added](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_updated_vpc_a.png)
    
7. Click on **Route tables**
    
8. Select the _check box_ for `VPC B Private Route Table`, scroll down select the **Routes** tab and click on **Edit routes**.
    
    ![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_select_vpc_b.png)
    
9. Add an aggregate route with a Destination of `10.0.0.0/8` with a Target as the transit gateway. Click **Save routes**.
    
    ![Select Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_edit_vpc_b.png)
    
10. Confirm that the route has been added to the **VPC B Private Route Table**.
    
![Routes Added](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_updated_vpc_b.png)
    
1. Click on **Route tables**
    
2. Select the _check box_ for `VPC C Private Route Table`, scroll down select the **Routes** tab and click on **Edit routes**.
    
![Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_select_vpc_c.png)
    
1. Add an aggregate route with a Destination of `10.0.0.0/8` with a Target as the transit gateway. Click Save routes.
    
![Select Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_edit_vpc_c.png)
    
1. Confirm that the route has been added to the **VPC C Private Route Table**.
    
![Routes Added](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_private_route_table_updated_vpc_c.png)
    

[

## Test Connectivity

Now let’s test the connectivity between the instances in the private subnets in VPC A, B & C.

1. Navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:instanceState=running)  in the EC2 Dashboard
2. Select the _check box_ for `VPC A Private AZ1 Server` and click **Connect** to use Session Manager to connect ![SSM Connection](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_connect_ec2.png)
3. Confirm connectivity between the VPCs by pinging the IP address on the instances in VPC B and VPC C with the following commands:
    
    ```bash
    ping 10.1.1.100 -c 5 ping 10.2.1.100 -c 5
    ```
    
4. You should see a response from both EC2 instances ![Ping](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/transit-gw/tgw_ping_response.png)

_**Congratulations you now have a multi-VPC architecture with connectivity between the VPCs provided by Transit Gateway.**_

![Attachments Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_vpc_routing.png)

# TGW Route Tables
<mark style="background: #D2B3FFA6;">Your transit gateway routes IPv4 and IPv6 packets between attachments using transit gateway route tables.</mark>  
You can configure these route tables to propagate routes from the route tables for the attached VPCs, VPN connections, and Direct Connect gateways. You can also add static routes to the transit gateway route tables. When a packet comes from one attachment, it is routed to another attachment using the route that matches the destination IP address.

In this section we will explore using transit gateway route tables to provide network segmentation.

### Inspect the Transit Gateway Default Route Table

1. In the VPC Dashboard scroll down and click [Transit Gateway Route Tables](https://console.aws.amazon.com/vpc/home?#TransitGatewayRouteTables:) 
2. Select the Transit Gateway route table, scroll down and click on the **Associations Tab**. You will see the three attachments for the three VPCs. ![TGW Route Table Associations](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_tab_associations.png)
3. Next click on the **Propagations Tab**. You will see all three attachments are propagating the CIDRs for the VPCs. ![TGW Propagations](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_tab_propagations.png)
4. Finally click on the **Routes Tab**. You will see the three routes for the three VPCs. ![TGW Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_tab_routes.png)

As we have seen the Transit Gateway automatically associates newly attached VPCs to the default route table and propagates routes from the VPCs into the Transit Gateway Route Table. This makes it very easy for VPCs to have connectivity to other VPCs. ![TGW Route Table Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_attachments.png)

But there are often times when we do not want VPCs to have connectivity to other VPCs except for a Shared Services VPC. For this lab we will use **VPC A** as our Shared Services VPC and modify the default configurations on the Transit Gateway route table and make it so that **VPC B** and **VPC C** cannot talk to each other but both are able to communicate with any shared services in **VPC A**. ![Shared Services Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_shared_services.png)

This is a typical ‘shared services’ VPC configuration in which **VPC A** would host services like LDAP, DNS, or other shared resources.

[

### Delete VPC A Attachment from Transit Gateway Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/tgw-routing#delete-vpc-a-attachment-from-transit-gateway-route-table)

The first thing we need to do is delete the association for **VPC A** from the original Transit Gateway Route Table that you created. You need to reference the VPC ‘Resource ID’ to ensure you delete the correct VPC

1. Navigate to [Your VPCs](https://console.aws.amazon.com/vpc/home?#vpcs:)  and make a note of the VPC ID for **VPC A**. ![Note VPC A ID](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_vpc_a_id.png)
2. Navigate back to [Transit Gateway Route Tables](https://console.aws.amazon.com/vpc/home?#TransitGatewayRouteTables:) 
3. Select the _check box_ for the TGW route table and scroll down to the **Associations** tab ![Delete VPC C Association](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_association_delete.png)
4. Select the association with the **Resource ID** that matches the VPC ID for **VPC A** noted down earlier and click **Delete association**
5. Confirm the deletion by clicking **Delete Association** on the following screen ![Confirm Deletion](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_association_delete_confirm.png)
6. The association will move into _disassociating_ state. ![VPC C disassociating](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_association_disassociating.png)

[

### Delete Propagations for VPC B & C from Transit Gateway Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/tgw-routing#delete-propagations-for-vpc-b-and-c-from-transit-gateway-route-table)

We will now delete the propagations that were automatically created for both **VPC B** and **VPC C** one at a time from the route table so that the only one remaining is that for the VPC ID of **VPC A**. This will remove the routes for **VPC B** and **VPC C** so they cannot reach each other via this route table.

1. Navigate to the **Propagations** tab.
2. Select one of the propagations where the **Resource ID** does NOT match the VPC ID noted down for **VPC A** and click **Delete Propagation** ![Delete Propagation 1](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_delete_propagation_1.png)
3. Confirm the deletion by clicking **Delete propagation** on the pop-up. ![Confirm Propagation Delete 1](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_delete_propagation_1_confirm.png)
4. Select the other propagation where the **Resource ID** does NOT match the VPC ID for **VPC A** and click **Delete Propagation** ![Delete Propagation 2](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_delete_propagation_2.png)
5. Confirm the deletion by clicking **Delete propagation** on the pop-up. ![Confirm Propagation Delete 2](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_delete_propagation_2_confirm.png)
6. Navigate to the **Routes tab** to check the result (it may take a few seconds to update) ![Route Table Associations](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_table_after_deletes.png)

The only route in the original Transit Gateway Route Table should be a propagated route to 10.0.0.0/16 to **VPC A**.

The attachments for both **VPC B** and **VPC C** are still associated to the original route table and this means that both **VPC B** and **VPC C** are able to reach **VPC A** via the route table as there is a route for 10.0.0.0/16 being propagated into it by **VPC A**.

However the attachment for **VPC A** is no longer associated to a route table in the TGW and therefore there is no routing information that would allow traffic to reach **VPC B** and **VPC C** from **VPC A**. To add those routes we will now create another route table for **VPC A** to use.

[

### Create Shared Services Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/tgw-routing#create-shared-services-route-table)

To create a return path from **VPC A** we will create a new Shared Services Route Table.

1. Click **Create Transit Gateway Route Table** ![Create New Route Table](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_table_create.png)
2. Enter `Shared Services TGW Route Table` as the **Name tag** and select the transit gateway from the dropdown for **Transit Gateway ID** ![New Route Table Settings](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_table_settings.png)
3. Click **Create transit gateway route table**\* and wait for the new route table to change state to _available_ ![Shared Service Route Table Available](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_table_available.png)
4. Scroll down to the **Associations** tab and click **Create association** ![Associate VPC A to Shared Services RT](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_association_create.png)
5. Associate the `VPC A attachment` with the Shared Services route table by selecting it from the dropdown and clicking **Create association**. ![New Route Table Association](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_association_vpc_a.png)

[

### Create Propagations for VPC B & VPC C

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/tgw-routing#create-propagations-for-vpc-b-and-vpc-c)

Now that **VPC A** is associated with the new Shared Services Route Table in the Transit Gateway we need to create propagations for **VPC B** and **VPC C** so that this route table knows how to access 10.1.0.0/16 and 10.2.0.0/16. This enables **VPC A** to have a return path to both **VPC B** and **VPC C**.

1. Navigate to **Propagations** tab and click **Create Propagation**. ![New Route Table Propagation](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_propagations_tab.png)
2. Select the `VPC B attachment` from the dropdown and click **Create propagation** ![Create VPC B Propagation](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_propagate_vpc_b.png)
3. Repeat the process to add a propagation for **VPC C** so there are two propagations in the **Propagations** tab ![Create VPC C Propagation](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_propagate_vpc_c.png)
4. There should now be two propagations in the **Propagations** tab ![Shared Services Propagations](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_updated_propagations.png)
5. Take a look at the **Routes** tab for the shared services route table. ![Shared Services Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/tgw-routing/tgw_routing_shared_services_updated_routes.png)

There should be routes to both **VPC B** (10.1.0.0/16) and **VPC C** (10.2.0.0/16).

![Shared Services Routing Domains Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_routing_domains.png)

[

### Test Connectivity

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/tgw-routing#test-connectivity)

Now let’s use Session Manager to connect to the EC2 instance in **VPC B** and test connectivity to the instances in **VPC A** and **VPC C**.

1. In the EC2 Dashboard click on [EC2 Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running) 
2. Select `VPC B Private AZ1 Server` and Click **Connect**
3. In the **Session Manager** tab, click **Connect**
4. Ping the servers in VPC A `10.0.1.100` and VPC C `10.2.1.100`

```bash
ping 10.0.1.100 -c 5 ping 10.2.1.100 -c 5
```

```bash
sh-4.2$ ping 10.0.1.100 -c 5 PING 10.0.1.100 (10.0.1.100) 56(84) bytes of data. 64 bytes from 10.0.1.100: icmp_seq=1 ttl=254 time=1.002 ms 64 bytes from 10.0.1.100: icmp_seq=2 ttl=254 time=0.909 ms 64 bytes from 10.0.1.100: icmp_seq=3 ttl=254 time=0.908 ms 64 bytes from 10.0.1.100: icmp_seq=4 ttl=254 time=0.896 ms 64 bytes from 10.0.1.100: icmp_seq=5 ttl=254 time=0.903 ms --- 10.0.1.100 ping statistics --- 5 packets transmitted, 5 received, 0% packet loss, time 4004ms rtt min/avg/max/mdev = 0.896/1.128/2.024/0.448 ms sh-4.2$ ping 10.2.1.100 -c 5 PING 10.2.1.100 (10.2.1.100) 56(84) bytes of data. --- 10.2.1.100 ping statistics --- 5 packets transmitted, 0 received, 100% packet loss, time 4072ms sh-4.2$
```

Now because we deleted the association in the main route table **VPC B** can talk to **VPC A**, but **VPC B** cannot talk to **VPC C**.

_**Congratulations you've established network segmentation using the Transit Gateway route tables and completed the lab.**_

# Clean up
If you are using your own AWS Account to conduct this workshop and you are finished, complete the follow steps to clean up.

You should only complete this clean up section if you do not plan on continuing with this workshop.

Make sure you terminate / delete the resources below to avoid unnecessary charges.

[

#### Delete Transit Gateway and Attachments

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/cleanup#delete-transit-gateway-and-attachments)

Step-by-step to delete TGW and the three VPC attachments

Navigate to [Transit Gateway Attachments](https://console.aws.amazon.com/vpc/home?#TransitGatewayAttachments:) 

- Select the "VPC A Attachment" and from the **Actions** menu select **Delete**
    
- Confirm, by clicking the **Delete** button in the "Delete" Dialog box.
    
- Repeat the above steps for "VPC B Attachment" and again for "VPC C Attachment".
    

_Attachments will be in a "deleting" state for a few minutes. Click the refresh icon, in the upper right to check on the status after a few minutes._

- Once all three Transit Gateway attachments show state of "deleted", navigate to [Transit Gateways console](https://console.aws.amazon.com/vpc/home?#TransitGateways) 
    
- Select the Transit Gateway you created, and click on **Delete** from the **Actions** button.
    

[

#### Delete the Remaining Resources

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/cleanup#delete-the-remaining-resources)

Option 1: You started this lab at the start

Option 2: You started the lab in the Multi VPCs section by deploying the CloudFormation template

[

#### Delete the MultiVPC and Prerequisites CloudFormation Stack

](https://catalog.workshops.aws/networking/en-US/foundational/multivpc/cleanup#delete-the-multivpc-and-prerequisites-cloudformation-stack)

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPC`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
