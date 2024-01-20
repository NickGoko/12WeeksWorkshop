---
tags:
  - 12weeksawsworkshopchallenge
---

![[Pasted image 20240119075650.png]]

Please note, this module is optional and is not required to be completed to proceed with this workshop. You may skip to [Connecting to On-Premises](https://catalog.workshops.aws/networking/en-US/foundational/on-premises)

If you are running this lab in **AWS Workshop Studio**, the region has been set by your facilitator. The region you see in screenshots may not match your environment. This will not cause any problems.

If you are running this lab in **your own AWS Account**, it is recommended for all lab resources to be created in us-east-1 region so that the screenshots match your environment. This is not mandatory.

## Prerequisites
If you have not completed the Multiple VPCs section...

If you **have not** completed the previous labs you must:

1. Complete the [prerequisites](https://catalog.workshops.aws/networking/en-US/foundational/prereqs) section.
    
2. Download the following CloudFormation template to create three VPCs connected by a Transit Gateway:
    
    [CloudFormation template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/multi-vpc-create-all-vpcs-and-tgw.yaml)
    
3. Navigate to [CloudFormation](https://console.aws.amazon.com/cloudformation/home)  section in the AWS console. Click **Create stack** button and select **With new resources (standard)**.
    
4. Under **Specify template**, select **Upload a template file**, click **Choose file** and select the CloudFormation template that you downloaded above. Click **Next**.
    
5. Enter the **Stack name** `NetworkingWorkshopMultiVPCandTGW`. Leave the parameter defaults unchanged if you are running in us-east-1 and click **Next**. If you are running in another region, update the availability zones. ![CFN Stack Details](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/pre-reqs/cfn_stack_details.png)
    
6. Scroll to the bottom of the **Configure stack options** page and click **Next** again.
    
7. Scroll to the bottom of the page and click the **Submit** button.
    
8. The CloudFormation will begin deployment which you can follow by refreshing the **Events** and **Resources** tab.
    
9. Once the CloudFormation template finishes lets go take a look at what it created. Click on the resources tab and you will find all the resources that were built as part of the CloudFormation stack.
    

<mark style="background: #D2B3FFA6;">This stack will create 3 VPCs each with two public subnets and two private subnets, a Transit Gateway to provide private connectivity between the VPCs, as well as EC2 instances in the VPCs.</mark>

Wait for the stack to be created before proceeding. This can take 3-5 minutes. Verify that the status in CloudFormation Console show as `CREATE_COMPLETE`, indicating successful creation of the stack.

![Multiple VPCs created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/multiple_vpcs_completed.png)

## Network ACLs
**Network ACLs** are <mark style="background: #FFF3A3A6;">stateless access controls you configure at a subnet level, to allow or block a CIDR block on a particular port or range of ports.</mark>  
Network ACL rules are numbered list and evaluated top down, with a DENY ALL at the end. <mark style="background: #FFF3A3A6;">If a rule is matched, subsequent rules are not evaluated</mark>.

Both inbound and outbound traffic can be controlled with these rules.  
By default when you create subnets, they will be attached to the default Network ACL which has an ALLOW ALL rule for both inbound and outbound traffic.

In this section, <mark style="background: #BBFABBA6;">we will modify the Network ACL associated with the workload subnets in VPC A to only ICMP traffic from VPC B's CIDR</mark>; and <mark style="background: #FFB8EBA6;">test connectivity from VPC A to VPC C, and test other connectivity from VPC B to VPC C as well.</mark>

![NACLs Architecture Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_nacls.png)


### Default NACL Configuration VPC A

1. In the VPC Dashboard click on [Network ACLs](https://console.aws.amazon.com/vpc/home?#acls:) 
2. Select the _check box_ for `VPC A Workload Subnets NACL`
3. Click on the **Inbound Rules** tab below to view existing inbound rules ![View Inbound Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_workload_subnets_inbound_rules_vpc_a.png)

All traffic is evaluated for Protocol, Port and Source IP match. In this Network ACL, all traffic is allowed into the VPC A Public and Private Subnets by the first rule. The second rule which is a DENY ALL is not evaluated.

We will now modify the first rule (100) to allow only ICMP traffic from VPC B's CIDR.

1. Click on **Edit inbound rules** button in the **Inbound rules** tab.
2. For rule number 100 select `ALL ICMP - IPv4` as Type and enter VPC B's CIDR of `10.1.0.0/16` for Source
3. Click on **Save** ![Workloads NACL Save Changes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_workload_subnets_inbound_rules_edit_vpc_a.png)
4. The screen should return to the Network ACL page and the updated rule will display on the **Inbound rules** tab like below
5. Verify the rule for Type, Protocol, Port and Source for the 'ALLOW' rule 100. ![Workloads NACL Result](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_workload_subnets_inbound_rules_result_vpc_a.png)

<mark style="background: #FFB86CA6;">We have now completed modifying the default Network ACL of VPC A to allow ONLY ICMP traffic from VPC B's CIDR and all other traffic will be denied by the catch-all DENY rule.</mark>  
Let us now test this from VPC B for ALLOW, and VPC C for DENY.

<mark style="background: #CACFD9A6;">Note that we have not modified outbound rules, and the default outbound rule allows ALL traffic to flow out of the subnet.</mark>

### Test Connectivity from VPC B to VPC A
Here, we will login to the EC2 instance in VPC B using EC2 using Session Manager, and verify reachability to the EC2 instance in VPC A over ICMP (ping)

1. Click on [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:)  in the EC2 Dashboard
2. Select the _check box_ for `VPC B Private AZ1 Server` and click **Connect** button on the top right ![Select VPC B Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_select_instance_vpc_b.png)
3. Click **Connect** in the **Session Manager** tab
4. A new browser window opens with SSH connection to the instance in VPC B established and showing a command line prompt.
5. Type this following command in the prompt:
```
ping 10.0.1.100 -c 5
```
6. The ICMP traffic should flow through and return as shown below. ![Ping Response](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_ping_vpc_b.png)

<mark style="background: #D2B3FFA6;">We have now verified that the Network ACL on VPC A is allowing ICMP traffic to flow in and out from VPC B.</mark>


### Test Connectivity from VPC C to VPC A

Now we will login to the EC2 instance in VPC C using EC2 using Session Manager, and check reachability to the EC2 instance in VPC A over ICMP (ping)

1. Terminate the Session Manager connection to the EC2 Instance in VPC B
2. In the top left corner click [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
3. Select the _check box_ for `VPC C Private AZ1 Server` and click **Connect** button on the top right ![Select VPC C Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_select_instance_vpc_c.png)
4. Click **Connect** in the **Session Manager** tab
5. A new browser window opens with SSH connection to the instance in VPC C established and showing a command line prompt.
6. Type this following command in the prompt:
```
ping 10.0.1.100 -c 5
```
7. The ICMP traffic should not flow through and not responses should be returned as shown below. ![No Ping Response](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_ping_vpc_c.png)

<mark style="background: #FFB8EBA6;">The ping command will not generate a response because the Network ACL in VPC A is DENYING all ICMP traffic that does not originate from VPC B.  
</mark>  
8. Terminate the Session Manager connection

We modified the default Network ACL in VPC A to allow ICMP traffic only from VPC B; the only other rule is a DENY ALL. 
>We verified that ICMP traffic flows through from `VPC B Private AZ1 Server` to `VPC A Private AZ1 Server` but DID NOT flow through from `VPC C Private AZ1 Server`.

![NACLs Architecture Diagram](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_nacls_enforced.png)

[

### Revert Changes

1. In the VPC Dashboard click on [Network ACLs](https://console.aws.amazon.com/vpc/home?#acls:) 
2. Select the _check box_ for `VPC A Workload Subnets NACL` ![Inbound Rules Result](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_workload_subnets_inbound_rules_revert_vpc_a.png)
3. Click on **Edit inbound rules** button in the above screen.
4. Edit Rule 100 and select `All traffic` for Type enter a CIDR of `0.0.0.0/0` for Source
5. Click on **Save** ![Revert Rule](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_revert_rule.png)
6. The screen should return to the Network ACL page and the updated rule will display like below in the **Inbound rules** tab
7. Verify the rule for Type, Protocol, Port and Source for the 'ALLOW' rule 100. ![Verify](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/nacls/nacls_workload_subnets_inbound_rules_vpc_a.png)

_**Congratulations, you've completed this section on Network ACLs**_

## Security Groups
Security Groups are virtual, stateful firewalls attached to an instance or network interface. Both inbound and outbound rules can be defined to allow specific protocols, ports and source/destination CIDR. A DENY is not possible with security groups.

With Security Groups, all rules are evaluated before a network packet is allowed or blocked, unlike Network ACLs where the rules are evaluated in order of rule number and once a rule matches subsequent rules are not evaluated.

![Security Groups](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_security_groups.png)

In this exercise, we will modify the security group attached to `VPC A Private AZ1 Server` to allow only ICMP traffic inbound from VPC C's CIDR only. We will verify that the EC2 instance in VPC C is able to ping the EC2 instance in VPC A, but that the EC2 instance in VPC B is not able to ping the same EC2 instance in VPC A.

**It is NOT best practice** to have open Security Groups that allow everything (0.0.0.0/0).

Limit access to what is required.


### Modifying the Security Group in VPC A


1. In the EC2 Dashboard click on [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running;sort=tag:Name) 
2. In the EC2 instance list, select the _check box_ for `VPC A Private AZ1 Server`
3. Scroll down to the **Security** tab and click on the Security Group link named something like `sg-xxxxxxx (VPC A Security Group)` ![Select VPC A Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_select_instance_vpc_a.png)
4. In the Security Group page, click on the **Inbound rules** tab, and then click on **Edit inbound rules** button ![Edit Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_inbound_rules_edit.png)
5. In the **Edit inbound rules** page, <mark style="background: #ADCCFFA6;">update the rule that is currently allowing ICMP from 0.0.0.0/0 to allow only from VPC C's CIDR</mark> `10.2.0.0/16` ![Update Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_inbound_rules_update.png)
6. Click **Save rules**

Now we have modified the Security Group on the EC2 instance in VPC A allow ICMP traffic (ping traffic) only from sources in VPC C, and from nowhere else.  
<mark style="background: #CACFD9A6;">We will now test to verify that we are NOT able to ping this instance from VPC B, and we are ABLE to ping from VPC C.</mark>


### Test Connectivity from VPC B to VPC A

1. In the EC2 Dashboard click on [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running;sort=tag:Name) 
2. Select the _check box_ for `VPC B Private AZ1 Server` from the list of instances, and click **Connect** button ![Select VPC B Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_select_instance_vpc_b.png)
3. In the ‘Connect to instance’ dialogue click **Connect** in the **Session Manager** tab. This will open up a session in a browser window.
4. In the command line, try pinging the private IPv4 address of the EC2 instance in VPC A using the command
```
ping 10.0.1.100 -c 5
```
<mark style="background: #FFB86CA6;">It will freeze and make no progress. This is because the Security Group on the EC2 instance in VPC A has inbound rules to only allow ICMP from VPC C. There are no rules for allowing VPC B, and an implicit DENY occurs. </mark>  
![Ping VPC A Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_ping_vpc_b.png)

Now connect to the EC2 instance in VPC C and try to ping the EC2 instance in VPC A.

### Test Connectivity from VPC C to VPC A

1. Terminate the Session Manager connection and click [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running;sort=tag:Name) 
2. Select the _check box_ for `VPC C Private AZ1 Server` from the list of instances, and click **Connect** button
3. In the ‘Connect to instance’ dialog, click **Connect** in the **Session Manager** tab.
4. In the command line, try pinging the private IPv4 address of the EC2 instance in VPC A using the command

The ping will succeed and traffic will flow through. This is because the Security Group on the EC2 instance in VPC A is allowing ICMP traffic from VPC C's CIDR range.

![Ping VPC A Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_ping_vpc_c.png)

1. Terminate the Session Manager connection.

_**We have modified the Security Group on the EC2 instance in VPC A to allow only ICMP traffic from VPC C.  
We tested and verified that we cannot ping this instance from VPC B, but we are able to ping it from VPC C confirming the behavior of the Security Group.**_ 

![Security Groups](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab2_security_group_enforced.png)


### Reverting Changes to the Security Group in VPC A
1. In the EC2 Dashboard click on [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:instanceState=running;sort=tag:Name) 
2. In the EC2 instance list, select the _check box_ for `VPC A Private AZ1 Server`
3. Scroll down to the **Security** tab and click on the Security Group link named something like `sg-xxxxxxx (VPC A Security Group)` ![VPC A Security Group](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_select_instance_vpc_a.png)
4. In the Security Group page, click on the **Inbound rules** tab, and then click on **Edit inbound rules** button ![Revert Inbound Rules](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_inbound_rules_revert.png)
5. In the **Edit inbound rules** page, update the rule that is currently allowing ICMP only from VPC C's CIDR of 10.2.0.0/16 back to `0.0.0.0/0` ![Rules Reverted](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/security/security-groups/security_groups_inbound_rules_reverted.png)
6. Click **Save rules**

_**Congratulations you have completed this section on Security Groups**_

## Endpoint Policies
<mark style="background: #D2B3FFA6;">Endpoint policies are IAM policies attached to VPC endpoints to restrict/grant permission to the service's API calls.</mark>  
For example, with an S3 endpoint, a policy can be attached to limit only read access to one or more S3 buckets from the VPC.

#### Verify the Permissions for the Gateway Endpoint
Now let us now verify access to S3 and what actions are permitted from `VPC A Private AZ1 Server`.
1. In the EC2 console navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _check box_ for instance `VPC A Private AZ1 Server`
    
3. Click **Connect** and use "Session Manager" to open a command prompt
    
4. Issue the command
    
    ```bash
    aws s3 ls | grep networking-day
    ```
    ![[Pasted image 20240119141740.png]]
5. Copy the name of the bucket beginning `networking-day`
    
6. Issue the command `aws s3 ls s3://<your-bucket-name>` (<mark style="background: #BBFABBA6;">replace with name of bucket you created) to check if we can list the contents of the bucket</mark>.
    
7. Check that the command was successful and no errors returned. The bucket is empty and hence it did not return any listing.
    
8. Let us try to create a test file and upload to the S3 bucket.
    - Issue the command below to create a new file
```bash
sudo touch /tmp/test.txt
```
- Then attempt to copy to S3 by typing (replace with name of your bucket)
```bash
aws s3 cp /tmp/test.txt s3://<your-bucket-name>
```
        
1. You will see that the command succeeds and the file is uploaded to the S3 bucket.
    
2. Confirm the bucket contains the new file `aws s3 ls s3://<your-bucket-name>` (replace with name of bucket you created)  
    ![[Pasted image 20240119142730.png]]  
    ![[Pasted image 20240119142543.png]]
#### Update VPC Endpoint Policy Document to Remove Permissions

<mark style="background: #FFF3A3A6;">Let us now remove the permissions in the VPC Endpoint policy document that allow creating objects in the S3 bucket, and test it by attempting to upload a file to the bucket.</mark>

1. Navigate to VPC Dashboard and click on [Endpoints](https://console.aws.amazon.com/vpc/home?#Endpoints:) 
2. Select the Gateway Endpoint for S3 we created in the above steps.

>If you did not begin this lab at VPC Fundamentals, your VPC Endpoints will not have Name tags set, unlike the screenshot below that shows the names. 

Select the Gateway Endpoint for S3 based on the Endpoint with the Service Name ending in `s3` and with the VPC ID ending in `VPC A`.

1. In the bottom frame, click on 'Policy' tab, then click on the Edit Policy button ![Edit Policy](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/vpc-endpoints/gateway_endpoint_policy_edit.png)
    
2. In the policy editing screen, click on _custom_, enter the below policy document and click save. <mark style="background: #FFF3A3A6;">This policy removes</mark> `s3:Put*` <mark style="background: #FFF3A3A6;">permissions to the VPC Endpoint, effectively preventing instances from VPC A to upload objects to S3 buckets in the account.</mark>
    
    ```json
    { "Version": "2008-10-17",
     "Statement": [ 
     { 
	     "Sid": "ReadWriteAccess", 
	     "Effect": "Allow", 
	     "Principal": "*", 
	     "Action": [ 
		     "s3:Get*", 
		     "s3:List*"
		      ], 
		  "Resource": "*" 
			  } 
		  ] 
	  }
    ```
    
3. Let us now<mark style="background: #CACFD9A6;"> try uploading the test file again by issuing the command</mark> `aws s3 cp /tmp/test.txt s3://<your-bucket-name>`  
    (replace with name of bucket you created) to upload the file to the S3 bucket.  
![[Pasted image 20240119161716.png]]  
<mark style="background: #FFF3A3A6;">Note that the command returned an 'Access Denied' error message when trying to upload a file to the S3 bucket. </mark>  
This is <mark style="background: #FFB86CA6;">because the Endpoint policy allows only Get\* and List\* actions, effectively making the S3 bucket read only, and all other actions are denied.</mark>

## Conclusion

We tested access to an S3 bucket through Gateway VPC Endpoint to S3, then we set the endpoint policy to allow only read actions; we tested that we can only list and get objects from the S3 bucket, and verified that put objects are denied by the endpoint policy showing that the VPC <mark style="background: #BBFABBA6;">Endpoint policy is an effective mechanism to control allow/deny to service API calls from the VPC.</mark>

## Clean up

Make sure you terminate / delete the resources below to avoid unnecessary charges.
#### Delete the MultiVPC and Prerequisites CloudFormation Stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPC`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
Option 3: You started the lab in the Basic Security section by deploying the CloudFormation template

#### Delete the MultiVPC and Prerequisites CloudFormation Stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPCandTGW`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
