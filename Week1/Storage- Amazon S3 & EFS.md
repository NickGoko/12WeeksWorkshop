---
tags:
  - cloud/12weeksworkshop
Associations: "[[S3]]"
---
## Storage- Amazon S3 & EFS
![](https://i.imgur.com/4Tk8ZuA.png)
### Amazon Elastic File System
Amazon Elastic File System (EFS) is designed to provide serverless, fully elastic file storage that lets you share file data without provisioning or managing storage capacity and performance. 
Amazon Elastic File System (EFS) automatically grows and shrinks as you add and remove files with no need for management or provisioning.
![](https://i.imgur.com/4WB3ZCb.png)
[Create a VPC with two public subnets](#)
- No private subnets. As well set none for VPC end points and NAT gateway
 [Create a Security Group for EC2 and EFS](#)
- **EC2-SSH-SecurityGroup**- should have inbound rule (Type:SSH, Source type anywhere-IPV4 Source: 0.0.0.0/0, Port range: 22)
- **EC2-EFS-SecurityGroup** -(Connection from EC2 to EFS) (TYPE:NFS, Choose Custom and then click in the box with the magnifying glass and select the **EC2-SSH-SecurityGroup**)
- [!] NFS means Network file System. And that is the connection that will be used to connect traffic from the EC2 instance to the EFS
After creating the security group we need to create another security group with type NFS and the source being the EC2-EFS-SecurityGroup itself. I presume this is to allow other EFS instances with the same SG to talk to each other. 
[Creating an Elastic File System](#)
- Now that we have our security groups, we need to create an Elastic File System that we can mount it to our EC2 instances that we will be creating later.![](https://i.imgur.com/C37q9eR.png)
- Pick the right VPC that will have the mount targets to be used. 
*Mount Targets*- the Availability Zones the EFS file system will run. In this case its `us-east-1a` and `us-east-1b`. Pick from the dropdown the EFS Security group we created earlier.
![](https://i.imgur.com/fIcSHcV.png)
- Keep File system policy at default values. click next and create after reviewing. 
[Create the first EC2 Instance and Mount our EFS drive](#)
- Name the instance `EFS-SERVER-1`. Choose the default configurations within the free tier. 
- Proceed without key-pair. 
- Select Existing Security Group and pick _BOTH_ Security Groups you created earlier, EC2 to EFS Security Group and the SSH Security Group.
The first one allows your _EC2 to connect to the EFS drive_ 
The SSH Security Group _allows you to connect through Instance Connect to the servers._
- [*] To find your security groups make sure you are in the right VPC to see them. 
- Enable auto-assign public IP.

![](https://i.imgur.com/egM3Ntv.png)


Uncheck automatically create and attach security groups. You already have security groups created. 
Leave  Advanced network configuration & advanced details as is. 
You can now *Launch the instance.*
[Creating the second EC2 instance and mount our EFS drive](#)
**The only difference in the second EC2 instance build is that we will be placing this EC2 instance in Public Subnet 2.** In this case `us-east-1b`
- Call it `EFS-SERVER-2`
- Make sure you configure storage and also check enable auto-assign IP addresses. 
![](https://i.imgur.com/HTANsLT.png)
Then check `automatically mount shared file system by attaching required user data script`
![](https://i.imgur.com/kEGVXD8.png)

>[!highlight] Summary
>So far we have 2 security groups, 2 EC2 instances and one EFS. Both security groups are used on the EFS and both instances. Note the EFS security group is also added on the EFS. The two instances are connected to the EFS through the security groups as well they are connected through mounting the `immersion-efs` file system to both EC2 instances. However in different Availability Zones `us-east-1 and us-east-2` which ensures reliabilty.
>

>[!bookmark]-
**Durability:** This refers to the ability of data to persist over time and resist corruption or loss. Amazon EFS itself is designed for durability. It automatically replicates data across multiple Availability Zones within a region to provide high durability. 
**Elasticity:** This refers to the ability of a system to dynamically adapt to changes in workload by provisioning and de-provisioning resources in an autonomic manner. While EFS supports elasticity by allowing you to grow and shrink the file system size based on your needs, connecting it to EC2 instances in different Availability Zones doesn't directly relate to elasticity.
**Reliability:** This refers to the ability of a system to consistently perform and deliver correct results. Connecting an EFS file system to instances in different Availability Zones enhances reliability because it ensures that your application can access the file system even if one Availability Zone becomes unavailable.
**Scalability:** The ability of a system to handle an increasing amount of workload or to be easily expanded.
**Availability:** The percentage of time that a system or service is operational and available for use.
**Resilience:** The ability of a system to recover quickly from failures or disruptions.
**Fault Tolerance:** The capability of a system to continue operating without interruption in the presence of hardware or software failures.
**High Availability (HA):** A design approach or system configuration that aims to ensure a high level of operational performance, usually by reducing downtime.
**Redundancy:** The inclusion of extra components or resources to improve system reliability and performance.
**Performance:** The measure of how well a system accomplishes its tasks, often related to response time, throughput, and efficiency.
    

[Connect to our EC2 instance using Instance Conenct](#)
Now that we have both instances running in our VPC and we have our EFS drive built and connected to both instances, it's time to verify that the Elastic File System works.
We will accomplish this by doing the following:
	- We will use Instance connect to SSH into both instances
	- We will verify the Elastic File System is mounted to both instances
	- Then we will use one instance to create a file on the Elastic File System
	- Verify that the file is there
	- Then switch to the second instance and perform a list command to see the file

First on separate tabs connect both instances to EC2 Instance Connect
- [!] Run into trouble because my second instance i hadn't enabled auto assign ip address. So it did not have a public IPV4 address. 
![](https://i.imgur.com/5DJ0fu7.png)
[Create a file on the EFS drive](#)
So now that we have connected to both instances, it's time to see the EFS drive in action. If you remember, both of these instances are in two different availability zones within our VPC. One of the benefits of the Elastic File System is that you can connect 100s of EC2 instances across multiple Availability Zones. This exercise will demonstrate that behavior.
When we mounted the EFS drive, we were given a mount point.
```
/mnt/efs/fs1
```
Input the command below in the command line interface and hit enter/return to change directories to the EFS mount
```
cd /mnt/efs/fs1
```
![](https://i.imgur.com/YToA2Nu.png)
Now that you have switched directories to the Elastic file system. It is time to create a file.
```
sudo touch newfile.txt
```
Confirm the file is created with `ls` command. 
[Demonstrate the EFS mount from instance #2](#)
We change directories just like in the first instance. And when you run the `ls` command you should see the files and folders on the mount. 
Once you have verified you can see the file  we created from the first instance is visible from second. Let's create a second file on this instance and check to see if it's available to view on the first instance. filename`SecondNewFile.txt`. You should be able to see both files. 
>[!highlight]- Summary
>- First we created a Virtual Private Cloud spanned across two availability zones
>- We created EC2 instances in each availability zone
>- We created a Elastic File System in order for us to connect both instances to share it.
>- We created a file on the EFS drive from one of our instance- And we were able to see it when we connected to the second instance
>- This demonstrates the ability to have a share drive common to both instances.
> ![](https://i.imgur.com/BG9cRIt.png)


