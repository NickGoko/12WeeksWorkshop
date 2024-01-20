---
tags:
  - 12weeksawsworkshopchallenge
---
# Network ACLs
A network access control list (ACL) is an optional layer of security for your VPC for <mark style="background: #FFB86CA6;">controlling traffic in and out of one or more subnets.</mark>

![NACLs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_nacls.png)

Select any of the subnets, and scroll down to the _**Network ACL**_ tab to look at the default NACL rules.  
<mark style="background: #ADCCFFA6;">Rules are evaluated in order from lowest to highest.</mark> <mark style="background: #FFF3A3A6;">If the traffic doesn’t match any rules, the \* rule is applied</mark>, and the traffic is denied.  
Default NACLs allow all inbound and outbound traffic, as shown below, unless customized.

![NACLs](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_default_rules.png)

### Create a New Network ACL for Workload Subnets in VPC A

1. On the VPC Dashboard click on [Network ACLs](https://console.aws.amazon.com/vpc/home?#acls:) 
    
2. Click **Create network ACL**
    
    ![NACL Dashboard](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_dashboard_default.png)
    
3. In the **Network ACL settings** screen
    
    - Enter a name of `VPC A Workload Subnets NACL`
    - Select `VPC A` from the dropdown
    - Click **Create network ACL**
    
    ![Create Workload Subnets NACL](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_create.png)
    

The result will be a new NACL for VPC A alongside the default NACL created when the VPC was created.

1. In the resulting **Network ACLs** screen
    
    - Select the checkbox for `VPC A Workload Subnets NACL`
    - Scroll down to the **Subnet associations** tab
    - Click **Edit subnet associations**
    
    ![Edit NACL Associations](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_edit.png)
    
2. In the **Edit subnet associations** screen
    - Select all four VPC A subnets to associate them with the NACL
    - Click **Save changes**.
    
    ![Select Subnets](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_select.png)
    

<mark style="background: #ADCCFFA6;">The NACL should now be associated with four subnets on the following screen but because NACLs are created with only a DENY rule for inbound and outbound we will now change the default NACL rules to allow all traffic in both directions.</mark>

1. In the **Network ACLs** screen
    
    - Select the check box for `VPC A Workload Subnets NACL` for VPC A
    - Scroll down and select the **Inbound Rules** tab below
    - Notice that we have only `DENY` all rule
    - Click **Edit inbound rules**
    
    ![Edit Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_inbound_rules.png)
    
2. In **Edit inbound rules** screen
    
    - Click **Add new rule**
    - Input `100` in **Rule number**
    - Choose `All traffic` in **Type**
    - Leave **Source** as `0.0.0.0/0`
    - Click **Save changes**
    
    ![Save Changes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_inbound_rules_edit.png)
    
3. In the resulting screen you should have a success banner and a new `Allow` rule under the **Inbound rules** tab:
    
    ![Success Screen](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_inbound_rules_result.png)
    

Now follow the same steps described above for Inbound, but work on **Outbound Rules** tab of NACLs

1. On the **Outbound Rules** tab
    
    - Note that we have only `DENY` all rule
    - Click **Edit outbound rules**
    
    ![Outbound Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_outbound_rules.png)
    
2. In the **Edit outbound rules** screen
    
    - Click **Add new rule**
    - Input `100` in **Rule number**
    - Choose `All traffic` in **Type**
    - Leave **Destination** as `0.0.0.0/0`
    - Click **Save changes**
    
    ![Edit Outbound Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_outbound_rules_edit.png)
    
3. On the resulting screen check that the rule has been added under the **Outbound rules** tab
    

![Save Changes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/nacl_workload_subnets_outbound_rules_result.png)

Allowing all traffic in and out of your subnets is not a good security posture. You can use NACLs to set broad rules and/or DENY rules, and then use _**Security Groups**_ to create fine grained rules. For example, you can deny traffic from specific IPs with NACLs but not with Security Groups.

We will explore Network ACLs and Security Groups more in the **Basic Security** section.
# Route Tables
<mark style="background: #BBFABBA6;">A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same subnet route table.</mark>


![Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_route_tables.png)

### Create Route Table for Public Subnets

1. In left hand panel of the VPC Dashboard click on [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:sort=routeTableId)  ![Default Route Table](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_tables_vpc_a_default.png)

You will see the default route table that was created as part of the VPC creation, and in the **Subnet Associations** tab below the four subnets created earlier. We will now create a new public route table for the public subnets with a route to the internet via the Internet Gateway.

1. Add new public route table by clicking on **Create route table** in the right hand corner
    
2. Enter `VPC A Public Route Table` as the name and select `VPC A` from the VPC dropdown
    
    ![Create Public RT](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_create.png)
    
3. Click **Create route table** and a new route table will be created
    
    ![RT Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_routes_tab.png)
    

As you can see there is only a local route, so we're going to enable internet access by adding a route to an Internet Gateway in a later step. For now we need to associate this public route table with the public subnets we created earlier.

1. Scroll down and click on the **Subnet Associations** tab
    
    ![RT Subnets](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_subnets.png)
    
2. Click on **Edit subnet associations**
    
3. Select `VPC A Public Subnet AZ1` and `VPC A Public Subnet AZ2` and click **Save association** ![RT Associate](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_associate.png)
    
4. <mark style="background: #CACFD9A6;">The two public subnets will now be associated with the public route table under</mark> **Explicit Subnet Associations** within the **Subnet associations** tab.
    
    ![RT Subnets](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_new_subnets.png)
    

### Create Route Table for Private Subnets

1. In the left hand panel of the VPC Dashboard click on [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:sort=routeTableId)  and click on the **Create route table** button in the top right corner
    
2. In the **Create route table** screen \* Enter `VPC A Private Route Table` as the **Name** \* Select `VPC A` from the dropdown for **VPC ID** \* Click on **Create route table**
    

![Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_vpc_a.png)

1. A new route table will be created with a local route

![Route Table](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_created.png)

We're going to enable outbound internet access by adding a route to the Internet via a NAT Gateway in the next step. For now we need to associate the private subnets to the route table.

1. In the **Subnet Associations** tab click on **Edit subnet associations**

![Associate Private Subnet](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_associate.png)

1. Select the two private subnets `VPC A Private Subnet AZ1` and `VPC A Private Subnet AZ2` and click **Save associations**

![Associate Private Subnet](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_associate_subnets.png)

1. In the resulting screen click on **Route tables** and confirm that there are three route tables under VPC A: main/default, Public and Private.

![VPC A Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_tables_vpc_a.png)
# Internet Connectivity
An <mark style="background: #BBFABBA6;">Internet Gateway establishes outside connectivity for EC2 instances that will be deployed into the VPC and provides both inbound and outbound connectivity to workloads running in public subnets</mark> whereas a <mark style="background: #FFB86CA6;">NAT Gateway provides outbound connectivity for workloads running in private subnets.</mark>

In this section, we will deploy an Internet Gateway (IGW) and NAT Gateway into our VPC.

An Internet Gateway establishes outside connectivity for EC2 instances that will be deployed into the VPC and provides both inbound and outbound connectivity to workloads running in public subnets whereas a NAT Gateway provides outbound connectivity for workloads running in private subnets.

![VPC Networking](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_internet_connectivity.png)

[

### Deploy an Internet Gateway

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/internet-connectivity#deploy-an-internet-gateway)

1. In the left hand panel click on [Internet Gateways](https://console.aws.amazon.com/vpc/home?#igws:)  and click on **Create internet gateway**
    
    ![Create IGW Button](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/igw_create_button.png)
    
2. Enter `VPC A IGW` as the name and click **Create internet gateway** in the bottom right corner
    
    ![IGW Settings](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/igw_create_settings.png)
    
3. On the success screen for the newly created IGW and click on **Attach to VPC**:
    
    ![Create IGW Result](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/igw_create_result.png)
    
4. Select `VPC A` from the dropdown list for Available VPCs and click **Attach internet gateway**
    
    ![Attach IGW](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/igw_attach_to_vpc.png)
    
5. The Internet Gateway should attach successfully.
    
    ![Attach IGW Result](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/igw_attach_success.png)
    

We now have an internet access point for our VPC, but in order to utilize the newly created Internet Gateway, we need to update VPC routing tables to point the default routes for our public subnets to this Internet Gateway.

[

### Update Route Table for Public Subnets

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/internet-connectivity#update-route-table-for-public-subnets)

1. In left hand panel of the VPC Dashboard click on [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:sort=routeTableId)  and select `VPC A Public Route Table`
    
    ![Select VPC A Route Table](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_tables_select_vpc_a.png)
    
2. Scroll down to the **Routes** tab
    
    ![VPC A Public RT Edit Routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_routes_edit.png)
    

As you can see there is only a local route, so we're going to enable internet access by adding a route to the Internet Gateway

1. Click on **Edit Routes**
    
2. In the resulting screen
    
    - Click on **Add route**
    - Enter `0.0.0.0/0` in the **Destination**
    - Select `Internet Gateway` from the **Target** dropdown
    
    ![Add Route Target dropdown](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_add_route_igw_dropdown.png)
    
3. Choose `VPC A IGW`
    

![Add Route Select IGW](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_add_route_igw_select.png)

1. Click **Save changes** and confirm that a new route has been added to the **Routes** tab

![Add Route IGW Success](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_public_add_route_igw_success.png)

Next we will add outbound connectivity from the private subnets by deploying a NAT Gateway in a public subnet for use by workloads that should not be directly exposed to the internet.

[

### Create NAT Gateway

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/internet-connectivity#create-nat-gateway)

1. In the left hand panel of the VPC Dashboard click on [NAT Gateways](https://console.aws.amazon.com/vpc/home?#NatGateways)  and click on **Create NAT gateway**

![NAT GW Button](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/natgw_create_button.png)

1. In the **Create NAT gateway** screen \* Enter `VPC A NATGW` as the name \* Choose `VPC A Public Subnet AZ1` \* Click **Allocate Elastic IP** \* Click **Create NAT gateway**

![NAT GW Settings](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/natgw_create_settings.png)

1. Upon creation the NAT Gateway details are displayed

![NAT GW Success](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/natgw_create_success.png)

In this workshop, we only created one NAT Gateway in AZ1. It is best practice to create a NAT Gateway in each AZ that is utilized.

[

### Update Route Table for Private Subnets

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/internet-connectivity#update-route-table-for-private-subnets)

Now that we have a NAT Gateway in a public subnet we need to create a route to it from the private subnets and we will do that by adding an entry to the Route Table for the private subnets.

1. In the left hand panel of the VPC Dashboard click on [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:sort=routeTableId) 
    
2. Select `VPC A Private Route Table`, scroll down to the **Routes** tab and click on **Edit routes**
    

![Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_tables_private_routes_edit.png)

1. In the **Edit routes** screen \* Click on **Add route** \* Enter `0.0.0.0/0` in the **Destination** \* Select `NAT Gateway` from the **Target** dropdown

![Add Route NATGW dropdown](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_add_route_natgw_dropdown.png)

1. Choose `VPC A NATGW` and click on **Save changes**

![Add Route NATGW Select](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_add_route_natgw_select.png)

1. Confirm the new route appears in the **Routes** tab of the resulting screen

![Add Route NATGW Success](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/route_table_private_natgw.png)

_**We have now gone through the bread and butter of AWS networking and built a networking foundation of public and private subnets across two availability zones with internet access.**_

# VPC Endpoints

<mark style="background: #FFB86CA6;">VPC Endpoints are private links to supported AWS services from a VPC, instead of reaching the service's public endpoints through the internet.</mark>  
_Two types of VPC endpoints_ exist, <mark style="background: #FF5582A6;">Gateway endpoints </mark>and <mark style="background: #FF5582A6;">Interface endpoints</mark>.

<mark style="background: #ABF7F7A6;">Gateway endpoints support only S3 and DynamoDB, and reach these services through a gateway from the VPC.</mark>

<mark style="background: #BBFABBA6;">Interface endpoints create a network interface in the VPC's subnets, and all traffic to the service flows through this interface to the service.</mark>

Please see [What are VPC endpoints?](https://docs.aws.amazon.com/whitepapers/latest/aws-privatelink/what-are-vpc-endpoints.html)  in the _Securely Access Services Over AWS PrivateLink_ whitepaper if you would like to learn more on the differences.

![VPC Networking](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_networking_built.png)


#### Create an Interface Endpoint for KMS

1. Navigate to [Endpoints](https://console.aws.amazon.com/vpc/home?#Endpoints:)  with the VPC console and click on **Create Endpoint** to start creating a VPC Endpoint
    
    ![Create Endpoint](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_create.png)
    
2. In the **Endpoint settings** screen
    
    - Enter `VPC A KMS Endpoint` as the **Name tag**
    - Search for 'kms' under **Services**
    
    ![Search](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_search.png)
    
3. From the results select the KMS service name without the'-fips' suffix
    
    ![Search](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_select_kms.png)
    
4. In the **VPC** section
    
    - Select `VPC A` from the dropdown
    - Expand the **Additional settings** section
    - Ensure that **Enable DNS name** is checked
    - Select the **IPv4** radio button
    
    ![Select VPC](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_select_vpc.png)
    
5. Select `VPC A Private Subnet AZ1` and `VPC A Private Subnet AZ2` from the subnets and check the **IPv4** radio button.
    
    ![Endpoint Subnets](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_subnets.png)
    
6. Select the default security group and leave the Policy as `Full Access`
    
    ![Endpoint Policy](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_full_access.png)
    
7. Click on **Create endpoint** button to create the VPC Endpoint for KMS in VPC A.
    
8. Click on **Close** to return to the Endpoints screen.
    

[

#### Create a Gateway Endpoint for S3

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/vpc-endpoints#create-a-gateway-endpoint-for-s3)

1. Click 'Create Endpoint' to start creating another VPC Endpoint
    
    ![Endpoints Screen](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_created.png)
    
2. In the **Create endpoint** screen, enter `VPC A S3 Endpoint` search for 'S3' by service name
    
    ![Service Search](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_search.png)
    
3. Select the endpoint that has a "Type" listed as "Gateway" and in the drop down box for VPC
    
    ![Select VPC](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_select.png)
    
4. Select `VPC A` as the **VPC** and check the checkbox for all the route tables
    
    ![Route Tables](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_route_tables.png)
    
5. Leave the Policy as `Full Access`
    
    ![Full Access](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_full_access.png)
    
6. Click on **Create endpoint** button to create the VPC Endpoint for S3 attached to VPC A
    
    ![Endpoint Created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_created.png)
    

_**We have now gone through the bread and butter of AWS networking and built a networking foundation of public and private subnets across two availability zones with internet access and private connectivity to AWS service endpoints.**_

_**In the next section we will launch an EC2 instance into both a public subnet and a private subnet to verify the connectivity.**_

# EC2 Instances
In this section, you will spin up EC2 instances in your VPC and protect them with a security group only allowing ICMP traffic to reach the hosts.

![Foundation and Instances](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_ec2_instances.png)

[

### Launch an EC2 Instance into a Public Subnet

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/ec2-instances#launch-an-ec2-instance-into-a-public-subnet)

In this section, you will create an EC2 instance in the **Public** Subnet of **AZ2** (Availability Zone B). You will create the instance in AZ1 in the next section.

1. In the [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:)  section of the EC2 console click **Launch Instances**
    
    ![EC2 Dashboard](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_dashboard.png)
    
2. In the resulting **Launch an instance** screen
    
    - Enter `VPC A Public AZ2 Server` for the **Name**
    - Ensure that Amazon Linux 2023 AMI will be selected, and the instance type is t2.micro.

![EC2 Server Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_1.png)

1. Under **Key pair (login)** select `Proceed without a key pair`. A key pair is not needed since we will be using Systems Manager to connect to the instances.

![Key Pair](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_keypair.png)

1. Under **Network settings** click **Edit** and
    - Select `VPC A` from the dropdown for the **VPC** field
    - Select `VPC A Public subnet AZ2` from the dropdown for the **Subnet** field
    - Select `Enable` for the **Auto-assign Public IP** field

![Key pair and Networking](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_2.png)

1. Select **Create security group** with the name `VPC A Security Group`, description of `Open-up ports for ICMP`
    
2. In **Inbound security groups rules** under **Type** select `All ICMP - IPv4` and enter 0.0.0.0/0 as the Source
    

![Security Groups](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_3.png)

Since security groups are stateful, you don’t need to edit the outbound rules. The security group will allow the instance to respond to the ping since it saw the ping arrive at the instance.

1. Expand **Advanced network configuration** and under **Primary IP** enter `10.0.2.100`.

![IP Address](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_ip_address.png)

1. At the bottom of the section
    - Expand **Advanced details**
    - Under **IAM Instance profile** select `NetworkWorkshopInstanceProfile` which was created in the pre-requisites section.
    - Click **Launch instance**

![Advanced detail](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_4.png)

1. Click **View all instances**
    
    ![EC2 Server Name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_instance_running.png)
    

If you just finished the last step, your EC2 instance might still be spinning up. You can tell by looking at the Instance State and Status Checks columns. If you see _Pending_ state or status _Initializing_, the instance is not ready yet. After few minutes, you should have an EC2 instance in the "running" state.

_**Congratulations! You have just launched a virtual server in your public subnet in AZ2.**_

[

### Launch Instance in Private Subnet

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/ec2-instances#launch-instance-in-private-subnet)

You could follow the same process in the last two sections in order to deploy an EC2 instance into a private subnet, however it is also possible to launch a new instance using the same settings as previously.

1. In the [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:)  section of the EC2 console

- Select the running public instance `VPC A Public AZ2 Server`
- Click **Actions** then **Image and templates** then **Launch more like this**

![Launch Private EC2](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_more_like.png)

1. In the settings screen

- Update the **Name** to `VPC A Private AZ1 Server`
- Under **Key pair (login)** select `Proceed without a key pair`.
- Update the **Subnet** to be `VPC A Private Subnet AZ1`
- Set the **Auto-Assign Public IP** setting to `Disable`.
- Expand **Advanced network configuration** and under **Primary IP** enter `10.0.1.100`.
- Click **Launch instance**

![Private EC2 Server](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_launch_private_1.png)

1. Click **View all instances**. There should now be two EC2 instances running in the VPC

![EC2 Private Server Created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/ec2_private_created.png)

_**Congratulations, you now have an EC2 instance running in both a public and private subnet.**_

# Test Connectivity
In this section, you will use the EC2 instances in the public and private subnets of your VPC to test the connectivity for the network foundations you created in the previous section.

1. In the [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:)  section of the EC2 console
    - Select the `VPC A Public AZ2 Server` instance
    - Scroll down to the **Details** tab
    - Copy the **Public IPv4 address** field by clicking the copy icon to the left of it

![EC2 Details](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_select_public_ec2.png)

1. <mark style="background: #CACFD9A6;">To ping the instance, you need to open your CLI. On Windows, open the Command Prompt. On Mac, open the Terminal.</mark>
    
2. **Type `ping` then a space, then paste the Elastic IP from above, then space then `-c 5` and enter.**
    

If the instance is reachable, we expect to see lines appearing such as

![EC2 Public ping response](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_public_ping_success.png)

_**Good job! You have successfully confirmed connectivity between the public EC2 instance and the internet.**_

![Public Instance Connectivity](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_public_connectivity.png)

[

### Test Private Instance Connectivity

](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/test-connectivity#test-private-instance-connectivity)

1. In the [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:)  section of the EC2 console
    - Select the `VPC A Private AZ1 Server` instance
    - Click on the **Connect** button

![Select Private EC2](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_select_private_ec2.png)

1. In the following screen, select the Session Manager tab and click **Connect**

![Session Manager](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_session_manager_connect.png)

This will open a terminal from which you can test connectivity to both the public instance at `10.0.2.100` and external connectivity to `example.com` via the NAT Gateway.

1. Copy the following ping commands and paste in the Session Manager console

```bash
ping 10.0.2.100 -c 5 ping example.com -c 5
```

1. You should receive responses both from the public EC2 instance and example.com

![EC2 private ping](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_private_ping_success.png)

If the ping to 10.0.2.100 is not successful, please ensure you correctly configured the IP address in the [EC2 Instances](https://catalog.workshops.aws/networking/en-US/foundational/fundamentals/ec2-instances) section

_**Congratulations you have now confirmed outbound connectivity from both instances**_

![Private Instance Connectivity](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab1_private_connectivity.png)

[

### Compare Gateway and Interface Endpoint Approaches

#### Interface Endpoint DNS
fundamentals/test-connectivity#interface-endpoint-dns)

1. Type the following to check the DNS for the KMS service from the VPC A instance:

```bash
dig kms.us-east-1.amazonaws.com
```

1. The response should point to two local IP addresses within the VPC A private subnet CIDR blocks of `10.0.1.0/24` and `10.0.3.0/24`

![Interface Endpoint Dig](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/interface_endpoint_dig_vpc_a.png)

<mark style="background: #D2B3FFA6;">Because the VPC DNS is returning the IP addresses for the Elastic Network Interfaces placed into the private subnets by the Interface Endpoint rather than the public IP address for KMS, the traffic to KMS from VPC A will be routed to the VPC Endpoint rather than traversing the internet to reach KMS.</mark>

![Interface Endpoint Path](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/interface_endpoint_path.png)

1. Terminate the Session Manager session.

#### Gateway Endpoint Routing


1. Navigate to the VPC console, and select [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:) 
    
2. Select the _check box_ next to `VPC A Private Route Table` and scroll down to the **Routes** tab. It should show an entry pointing to the VPC Endpoint ID for destinations in the prefix list.
    

![GW Endpoint RT](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/fundamentals/testing_gw_endpoint_route_table.png)

<mark style="background: #FF5582A6;">Because the S3 prefix list is a more specific route to S3 than the default route to the internet the traffic to S3 from VPC A will be routed to the VPC Endpoint rather than traversing the internet to the public S3 endpoint.</mark>

![GW Endpoint Path](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/gateway_endpoint_path.png)

<mark style="background: #FF5582A6;">We have created VPC Endpoints for S3 and KMS and confirmed that the local route table pointed to the VPC endpoint for S3 whereas the local DNS entry for KMS now pointed to ENIs in the private subnets rather than the public interface.</mark>

# Clean up
If you are using your own AWS Account to conduct this workshop and you are finished, complete the follow steps to clean up.

You should only complete this clean up section if you do not plan on continuing with this workshop.

Make sure you terminate / delete the resources below to avoid unnecessary charges.

#### Terminate EC2 Instances

Step-by-step to delete \`VPC A Public AZ2 Server\` and \`VPC A Private AZ1 Server\`

Navigate to [EC2 Dashboard Services](https://console.aws.amazon.com/ec2/v2/home?#Instances:) .

- Select an EC2 instance.
    
- Navigate under "Instance state" and click **Terminate**.
    
- From the "Instance state settings" dialog box, select **Terminate**.
    
- Click the **Terminate** button.
    
- Repeat the steps for the other EC2 instance.

#### Delete the VPC Endpoints

Step-by-step to delete the KMS and S3 VPC Endpoints

- Navigate to [VPC Dashboard Services](https://console.aws.amazon.com/vpc/home?#Endpoints:sort=vpcEndpointId) .
    
- Select the KMS VPC Endpoint you created for VPC A. (You can verify it is for VPC A by looking that the VPC ID from the _Details_ tab)
    
- Navigate under **Actions** and click **Delete Endpoint**.
    
- From the **Delete Endpoint** dialog box, click the **Yes, Delete** button.
    
- Repeat the steps of the S3 VPC Endpoint
    
#### Delete the VPC

Step-by-step to delete VPC A

Navigate to [Your VPC](https://console.aws.amazon.com/vpc/home?#subnets:) 

- Select "VPC A" from the list of VPC and from the **Actions** button select **Delete**.
    
- At the bottom of the "Delete VPC" dialog box, type `delete` in the field and Click the **Delete** button.
    
#### Delete the Prerequisites CloudFormation Stack

Step-by-step to delete the CloudFormation stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
