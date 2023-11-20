---
tags:
  - cloud/12weeksworkshop
  - cloud/aws
---
# AWS General Immersion Day- Advanced > Advanced Modules- Web Application
> This workshop provides users unfamiliar to AWS services with the hands on labs of building high availability and scalability web applications. 

![](https://i.imgur.com/QiHyiW6.png)

![](https://i.imgur.com/2psbaqu.png)

![](https://i.imgur.com/StUmpzp.png)
## **[Network- Amazon VPC]()**
![](https://i.imgur.com/qv7Vot8.png)

[Create VPC ](#) 

- Navigate to [[Amazon VPC]] console. Choose VPC and more.
- Name the VPC `VPC-LAB`. Choose 2 Availability Zones, 2 public subnets & 2 private subnets.
- Customize Availability Zones. Create **2** subnet space and select **2a** and **2c** for **Customize AZs**. And set the CIDR value of the public subnet that can communicate directly with the Internet. 
- Customize subnets CIDR blocks
![](https://i.imgur.com/muxVoFv.png)
You can use a **NAT gateway** so that instances in your private subnets can connect to services outside your VPC, but external services cannot initiate direct connections to these instances. In this lab, we will create a NAT gateway in only one Availability Zone to save cost. 
VPC endpoints-choose none
Also, for DNS options, **enable** both **DNS hostnames** and **DNS resolution**. After confirming the setting value, click the **Create VPC** button.
![](https://i.imgur.com/a0J35ad.png)
![](https://i.imgur.com/qGOUTdC.png)
![](https://i.imgur.com/53VQ2p9.png)
![](https://i.imgur.com/oPUhTGb.png)
![](https://i.imgur.com/IYFz7Zz.png)
[Create VPC Endpoint](#)
- While still on the VPC dashboard, select *Endpoints*. Click Create endpoint
- Use `S3 endpoint` for name and select AWS services in the service category. In search bar type `s3`. 
- [*] For S3 VPC endpoints there are *gateway* types and *interface* types. In this case select the gateway type. 
![](https://i.imgur.com/YC5LxtY.png)
- Deployment location select the `VPC-lab-vpc` you just created.
- Choose a route table to reflect the endpoint. Select the **two private subnets**
![](https://i.imgur.com/opqGgbO.png)

- There is also the capability to configure policies to control access to endpoints. However we shall go with **Full access**. Then click "create endpoint"
![](https://i.imgur.com/3DwzduA.png)
- Go to route tables to Confirm that the route to access Amazon S3 through the gateway endpoint has been automatically added to the **private route table** 
![](https://i.imgur.com/BsOHups.png)
> VPC endpoints are communications with the AWS network and have the **security and compliance** advantage of being able to control traffic through the endpoints. 
> You can also optimize the *data processing cost* if you transfer your data through a VPC endpoint rather than a NAT gateway. 

## **[Compute-Amazon EC2 ](#)**
Final Architecture- This computer lab will use an Auto Scaling Group([[EC2 AUTO SCALING]]) to deploy web service instances to private subnets in your VPC that you created earlier in this network lab. This configures the highly available web services so that external users can access the sample web page through the web browser. 
![](https://i.imgur.com/D9Yqbam.png)
The following items are contained in this chapter.
- Launch web server instances and execute user data
- Set up a security group
- Create a custom Amazon Machine Image (AMI)
- Launch an Application Load Balancer (ALB)
- Configure a Launch Template
- Configure an Auto Scaling Group
- Test auto scaling and change manual settings
The order of this lab will be:
	- Launch a web server instance
	- Deploy auto scaling web service
	- Check web service and test. 
[Launch instance and connect to web service](#)
Go to [[Amazon EC2]] from you AWS console. And press the **Launch instance** button. 
![](https://i.imgur.com/0SGnSwX.png)
![](https://i.imgur.com/VqTrSfX.png)
![](https://i.imgur.com/9CgOPJ8.png)
- Click the **Edit** button on network settings. To specify the VPC,  a public subnet and enable "Auto-assign public IP"
- Create a security group that will apply rules to the EC2 instances that will be created. Add a security group rule and set _**HTTP**_ to **Type**. Also allow TCP/80 and source `0.0.0.0/0` for Web Service by specifying it.
 ![](https://i.imgur.com/r7K12Ju.png)
- Next click on **Advanced details** scroll to _Metadata version_  field. Select `v2 only(token required)`
- Enter the following bash script in the **User data field** and select **Launch instance**. [[Amazon EC2#^d8f476| Working with UserData]]
```bash
#!/bin/sh
​
#Install a LAMP stack
dnf install -y httpd wget php-fpm php-mysqli php-json php php-devel
dnf install -y mariadb105-server
dnf install -y httpd php-mbstring
​
#Start the web server
chkconfig httpd on
systemctl start httpd
​
#Install the web pages for our lab
if [ ! -f /var/www/html/immersion-day-app-php7.zip ]; then
   cd /var/www/html
   wget -O 'immersion-day-app-php7.zip' 'https://static.us-east-1.prod.workshops.aws/public/dd38a0a0-ae47-43f1-9065-f0bbcb15f684/assets/immersion-day-app-php7.zip'
   unzip immersion-day-app-php7.zip
fi
​
#Install the AWS SDK for PHP
if [ ! -f /var/www/html/aws.zip ]; then
   cd /var/www/html
   mkdir vendor
   cd vendor
   wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
   unzip aws.zip
fi
​
# Update existing packages
dnf update -y

```
[Access the web service](#)
- Select the instance and click the **Connect** button
![](https://i.imgur.com/dbI2LKm.png)
![](https://i.imgur.com/NTlfOZt.png)
[Connect to the Linux Instance using Session Manager](#)
In the database lab to be follow. You will need to connect to RDS using the IAM role granted to the web server. The role will be `SSMInstanceProfile` and the permissions attached will be the Amazon managed `AmazonSSMManagedInstanceCore` as well as customer managed `ReadSecrets` but that comes later. However to provide more detail it should help the web server read the user and password to the RDS database so as to connect to it. 
[Create a Custom AMI](#)
In the EC2 console. Select the instance we made. Click **Action>Image and Templates > Create Image** 
![](https://i.imgur.com/Y3YBEFb.png)
Use these details and leave everything else on default. Click **Create Image**
While still on the EC2 dashboard find the AMIs section on the left side. Confirm that the image status is "Available"
[Terminate the instance](#)
Now that the custom AMI is created. We don't need the EC2 instance running anymore. Autoscaling can be completed using the image of the EC2 instance. 
![](https://i.imgur.com/GuHReRp.png)
![](https://i.imgur.com/EF4SazX.png)


### **[Deploy auto scaling web service](#)**
[[EC2 AUTO SCALING]]
From the **EC2 Management Console** in the left navigation panel, click **Load Balancers** under **Load Balancing**. Then click **Create Load Balancer**
Click _create_ under the **Application Load Balancer**
![](https://i.imgur.com/MNW0pRh.png)

![](https://i.imgur.com/nItmL5o.png)
- Choose the `lab-vpc`. On _Mappings section_. Tick off both Availability Zones. Select one subnet per zone. In this case we will be selecting the public subnets in both Availability Zones.
![](https://i.imgur.com/QtuYCV2.png)
- In the **Security groups** section, click the **Create new security group hyperlink**. Enter `web-ALB-SG` as the security group name and check the VPC information. Click the **Add rule** button and select **HTTP** as the Type and **Anywhere-IPv4** as the Source. And create a security group. 
- One you click the hyperlink you should find yourself in a new tap with these values for your security group. 
![](https://i.imgur.com/klQk7vA.png)
- Return to the load balancer page again, click the refresh button, and select the **web-ALB-SG** you just created. **Remove the default security group.**
![](https://i.imgur.com/P56Jwzw.png)

**Listeners and routing** column, click **Create target group** hyperlink. Once you are in a new tab, put `Web-TG` for Target group name and check all settings same with the screen below. 
	- Basic configuration. "choose a target type" = "instances"
	- Target group name = "Web-TG"
- After that click **Next** button.
- This is where we would register our instances. However, as we mentioned earlier, there are not instances to register at this moment. Click **Create target group**. Once you create the target group go back to the _Load balancer tab_ and add the new target group. 
- Click **Create load balancer**. Check to see if the state changes.

[Configure launch template](#)
- Now that the ALB has been created, it's time to place the instances behind the load balancer. 
- To configure an Amazon EC2 instance to start with Auto Scaling Group, you can use **Launch Template**, **Launch Configuration**, or **EC2 Instance**.
- We will use the launch template to create an auto scaling group.
- <mark style="background: #BBFABBA6;">The launch template configures all parameters within a resource at once, reducing the number of steps required to create an instance. </mark>
- Launch templates make it easier to implement best practices with support for Auto Scaling and spot fleets, as well as spot and on-demand instances. This helps you manage costs more conveniently, improve security, and minimize the risk of deployment errors.
- <mark style="background: #BBFABBA6;">The launch template contains information that Amazon EC2 needs to start an instance, such as AMI and instance type. </mark>
- The Auto Scaling group refers to this and adds new instances when a scaling out event occurs. 
- If you need to change the configuration of the EC2 instance to start in the Auto Scaling group, you can create a new version of the launch template and assign it to the Auto Scaling group. You can also select a specific version of the launch template that you use to start an EC2 instance in the Auto Scaling group, if necessary. You can change this setting at any time.
	[Create Security Group](#)
- From the left navigation panel of the EC2 console select _Security Groups_ 
![[Pasted image 20231117231436.png]]
Add inbound rule. Select `HTTP` in the *Type*. For _Source_ type `ALB` in the search bar to search for the security group created earlier `Web-ALB-SG` This will **configure the security group to only receive HTTP traffic coming from ALB** . So this security group will be attached to the launch template. Which means every instance from launch template will follow this inbound rule. Allow HTTP traffic only coming from the ALB. 
![](https://i.imgur.com/qEsWiZy.png)

[Create Launch Template](#)
In the EC2 console. Select **Launch Templates**
![](https://i.imgur.com/Z35ZnGX.png)
![](https://i.imgur.com/g4n6skI.png)
![](https://i.imgur.com/eXjBUqb.png)
![](https://i.imgur.com/N7kZPgP.png)
 In the **Advanced details** tab, set the **IAM instance profile** to **SSMInstanceProfile**. Leave all other settings as default, and click the **Create launch template** button. 
![](https://i.imgur.com/L9kA5jw.png)

[Set Auto Scaling Group](#)
In the EC2 console and select **Auto Scaling Groups** at the bottom of the left navigation panel. Then click the **Create Auto Scaling group** button to create an _Auto Scaling Group_.
![](https://i.imgur.com/BOA6clY.png)
Set the network configuration with the Purging options and instance types as default. Choose `VPC-Lab-vpc` for **VPC**, select **Private subnet 1** and **Private subnet 2** for **Subnets**. When the setup is completed, click the **Next** button.
![](https://i.imgur.com/KucDD3o.png)
Next, proceed to set up load balancing. First, select `Attach to an existing load balancer`. Then in **Choose a target group for your load balancer**, select `Web-TG` created during in ALB creation. At the **Monitoring**, select `Check box` for **Enable group metrics collection within CloudWatch**. This allows CloudWatch to see the group metrics that can determine the status of Auto Scaling groups. Click the **Next** button at the bottom right.
<mark style="background: #BBFABBA6;">The load balancer is selected from the Application Load balancer that was attached to the target group we created. </mark>
![](https://i.imgur.com/DJ9LgSQ.png)
 In the **Group size** column, specify **Desired capacity**.In the Scaling column **Minimum capacity** as `2` and **Maximum capacity** as `4`. 
 ![](https://i.imgur.com/F5XIMKw.png)
 **Target tracking scaling policy** and type `30` in **Target value**. This is a scaling policy for adjusting the number of instances based on the CPU average utilization. In this case its lower to trigger auto scaling sooner for the purpose of demonstration. 
>[!highlight] Target value
>The target value represents the optimal average utilization or throughput for the Auto Scaling group. To use resources cost efficiently, set the target value as high as possible with a reasonable buffer for unexpected traffic increases. When your application is optimally scaled out for a normal traffic flow, the actual metric value should be at or just below the target value.
When a scaling policy is based on throughput, such as the request count per target for an Application Load Balancer, network I/O, or other count metrics, the target value represents the optimal average throughput from a single instance, for a one-minute period.
![](https://i.imgur.com/tqHdCLD.png)

Click next and stay on default values.Click Next on Add notification page & Tags. We shall not have notifications right now. 
Now we are in the final stage of review. After checking the all settings, click the **Create Auto Scaling Group** button at the bottom right.
![](https://i.imgur.com/sbHJIrO.png)

[Check web service and load balancer](#)
 To access through the Application Load Balancer configured for the web service, click the **Load Balancers** menu in the EC2 console and select the `Web-ALB` you created earlier. Copy _**DNS name**_ from the basic configuration.
 Open a new tab in your web browser and paste the **copied DNS name**.
 ![](https://i.imgur.com/NkyE0F2.png)
Currently, in the the Auto Scaling group, scaling policy's baseline has been set to 30% CPU utilization for each instance.
- If the average _**CPU utilization of an instance is less than 30%**_, Reduce the number of instances.
- If the average _**CPU utilization of an instance is over 30%**_, Additional instances will be deployed, load will be distributed, and adjusted to ensure that the average CPU utilization of the instances is 30%.
 Now, let's test load to see whether Auto Scaling works well. On the web page above, click the **LOAD TEST** menu.
 ![](https://i.imgur.com/8vg3VKo.png)
> The principle that causes CPU load is that when the CPU Idle value is over 50. The PHP code operates every five seconds to create, compress, and decompress arbitrary files. Traffic is distributed and operated by the ALB, so the load is applied to other instances continuously.

Enter **Auto Scaling Groups** from the left side menu of the EC2 console and click the **Monitoring** tab. Under **Enabled metrics**, click **EC2** and set the right time frame to **1 hour**. If you wait for a few seconds, you'll see the **CPU Utilization (Percent)** graph changes.
Wait for about 5 minutes (300 seconds) and click the **Activity** tab to see the additional EC2 instances deployed according to the scaling policy.
You will see you now have 4 instances running. 
![](https://i.imgur.com/M6Xj7zb.png)
![](https://i.imgur.com/Lr7ZmiQ.png)
> If the page that causes the CPU load is working, close the page to prevent additional load.


## **[Database- Amazon Aurora](#)**
Final architecture
![](https://i.imgur.com/pgk3Sx5.png)
The is workshop lab will follow this order:
	- Create VPC security group
	- Create RDS instance
	- Connect RDS with web app server
	- Access RDS from EC2
	- (option) RDS Management features
	- (option) Connect RDS Aurora
- In the previous lab we created web server EC2 instances using Launch Template and Auto Scaling Group. These instances use Launch Template to apply the security group **ASG-Web-Inst-SG** . Using this information, <mark style="background: #BBFABBA6;">we will create a security group so that only web server instances within the Auto Scaling Group can access RDS instances.</mark>

[Create VPC security Group](#)
On the left side of the VPC dashboard, select **Security Groups** and then select **Create Security Group**.
- Name the security group. Choose the VPC that was created previously. `VPC-Lab`
![](https://i.imgur.com/5rOYIOr.png)
- Scroll down to Inbound rules- Click Add rule to create a security group policy that allows access to RDS from the EC2 Web servers that you previously created through the Auto Scaling Group. 
	Under **Type**, select **MySQL/Aurora** The port range should default to **3306**. The protocol and port ranges are automatically specified. The **Source type** entry can specify the IP band (CIDR) that you want to allow access to, or other security groups that the EC2 instances to access are already using. Select the security group(named _**ASG-Web-Inst-SG**_ ) that is applied to the web instances of the Auto Scaling group
![](https://i.imgur.com/CSoSEiS.png)
This means traffic from the instance created from the launch template will be able to access the database. 
When settings are completed. Click **Create security group**

[Create RDS instance](#)
In the AWS Management console, go to the [RDS(Relational Database Service)](https://console.aws.amazon.com/rds). Select **Create Database** in dashboard to start creating a RDS instance.
- Select **Standard Create** in the choose a database creation method section. Set engine type to **Amazon Aurora(MySQL Compatible**
![](https://i.imgur.com/Q3x8ZaG.png)

- Select **Production** in _**Template**_. Under **Settings**, we want to specify administrator information for identifying the RDS instances.
![](https://i.imgur.com/SmWSRDM.png)
- Under **DB instance size** select **Memory Optimized class**. Under **Availability & durability** select **Create an Aurora Replica or reader node in a different AZ**. Select **db.r5.large** for instance type.
![](https://i.imgur.com/YDv6Iwm.png)
- Set up network and security on the **Connectivity** page. Select the VPC-Lab that you created earlier in the Virtual private cloud (VPC) and specify the subnet that the RDS instance will be placed in, public access, and security groups. Choose the `DB-SG` because it already has an inbound rule to accept traffic from the launch template SG `ASG-Web-Inst-SG`
![](https://i.imgur.com/Q8atk2V.png)
Scroll down and click **Additional configuration**. Set database options as shown below. Be aware of the uppercase and lowercase letters of **Initial database name**.
![](https://i.imgur.com/Zb0Oq6Q.png)
8. Subsequent items such as **Backup**, **Entry**, **Backtrack**, **Monitoring**, and **Log exports** all accept the default values, and press **Create database** to create a database.
 ![](https://i.imgur.com/clVf5vb.png)
- [*] Architecture Configured So far
![](https://i.imgur.com/4wIIS7K.png)
[Connect RDS with Web App server](#)
- In the console window, open [AWS Secrets Manager]([https://console.aws.amazon.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/) ) and click the **Store a new secret** button.
- Choose secret type- **Credentials for Amazon RDS database**
- Database- select the DB instance you just created. `rdscluster`
Click **Next**
- Name your **secret**. The Sample code is written to ask for the secret by this a specific name `mysecret`. 
	However you can change this specific name when you get to review choices section. Click Next. 
- Leave Secret rotation at default values. Click Next.
- Review your choices. Click **Store**.
![](https://i.imgur.com/Hg4AIlk.png)
You can check the list of secret values with the name of you secret name. Mine is **mydbsecret** 
Click **`mydbsecret`** hyperlink and find **Secret value** tab. And click **Retrieve secret value** button.
Click **Edit** button, and check whether there is **dbname** and **advancedimmersionday** in key/value section. If they were not, click **Add** button, fill out the value and click **save** button.
![](https://i.imgur.com/MV9fqFf.png)
[Access RDS from EC2 ](#)
Open the IAM console. In the navigation pane, choose **Policies** and then choose **Create Policy**
Select a service- choose or type **`Secrets Manager`**
Under **Access level**, click on the carat next to **Read** and then check the box by **GetSecretValue**.
![](https://i.imgur.com/V5rjC8d.png)
Click on the carat next to **Resources**. For this lab, select **All resources**. Click **Next: Tags**.
![](https://i.imgur.com/WOAp0zX.png)

> The purpose of this is we're allowing EC2 to access all secrets. With a real workload, you should consider allowing access to specific secrets.
![](https://i.imgur.com/bPEOFuM.png)

- In the navigation pane, choose **Roles** and type **SSMInstanceProfile**
Choose **Roles** and type **SSMInstanceProfile** into the search box. This is the role you created previously. Click **SSMInstanceProfile**.
Under **Permissions policies**, click **Attach policies**.
Search for the policy you created called **ReadSecrets**. Check the box and click **Attach policy**.
![](https://i.imgur.com/PZVoWu0.png)
[Try the address book](#)
Access the EC2 console. Click **load balancer** After copying the DNS name of the load balancer created earlier. Open a new tab in your browser. 

The Architecture Configured So far
![](https://i.imgur.com/L9rnIzJ.png)

[RDS Management Features](#)
In multiple AZ deployments, Amazon RDS automatically provisions and maintains synchronous spare replicas in different availability zone. 
- The default DB instance is synchronized from the availability zone to the spare replica to provide data redundancy.
**RDS Failover Tests**
- When multiple AZs are enabled, Amazon RDS automatically switches to a spare replica in another availability zone if the DB instance has a planned or unplanned outage. 
- The amount of time that failover takes to complete depends on the database activity and other conditions when the default DB instance becomes unavailable. The time required for failover is typically 60-120 seconds. However, if the transaction is large or the recovery process is complex, the time required for failover can be increased. 
When failover is complete, the RDS console UI takes additional time to reflect in the new availability zone.

- From the RDS management console, select **Databases**, select the _instance_ that you want to proceed with the failover, and click **Failover** in the task menu.
![](https://i.imgur.com/ahvrDSU.png)
![](https://i.imgur.com/S5GXMD4.png)
The **refresh** button changes the status of **rdscluster** in the DB identifier to **Failing-over**. In a few minutes, press the **Refresh** button to see **Reader and Writer roles changed**. The failover is complete.
![](https://i.imgur.com/N0RJAeV.png)

[Create RDS Snapshot](#)
![](https://i.imgur.com/wGqMjnX.png)
![](https://i.imgur.com/km52LRn.png)
From the left RDS menu, select **Snapshots** and check the creation status of the snapshot. Check the state of the snapshot. You can use that snapshot to restore the database when the state becomes available. 
To restore, select the **snapshot** and select **Actions** to see what you can do with that snapshot. **Restore Snapshot** allows you to create RDS instances with the same data based on snapshots taken.

[Change RDS Instance Type](#)
To change the RDS instance by selecting the instance you want to change and clicking *Modify*
You can select the specification of the instance that you want to change by selecting the list box of **instance classes**. Let's choose **db.r6g.large** here.
![](https://i.imgur.com/bCX9dyx.png)
- Click Continue and apply the changes. 
Select **Apply immediately**. In this case, RDS changes its instance immediately after perform a back up task. Then click **Modify DB Instance**.
![](https://i.imgur.com/PIEntqp.png)
- [!] RDS can change the size of the instance at any time. However the size of the database does not support shrink after scaling up. 

**[Connect RDS Aurora](#)**
1. Create an EC2 instance with the AMI created in Public Subnet within the `VPC-Lab`. The networking option should allow Public IP.
2. Changes the security group settings for RDS Aurora. Configure the newly created EC2 instance to accept security groups as sources.
![](https://i.imgur.com/cIxSMpF.png)
3. Log in to the EC2 instance you just created with SSH, and connect to RDS Aurora through the MySQL Client. The EC2 web server already has MySQL client installed during EC2 deployment.



## **[Amazon S3](#)**
Final Architecture.
![](https://i.imgur.com/R55UeSl.png)
The order of this lab follows this order:
- Create bucket on S3
- Adding objects to buckets
- View Objects
- Enable Static Web Site Hosting
- Move Objects
- Enable Bucket Versioning
- Deleting objects and buckets
[Create Bucket](#)
From the AWS Management Console, connect to S3. **Create bucket**
Enter a unique bucket name in the _**Bucket name**_ field. For this lab, type `immersion-day-user_name`, substituiting user-name with your name. In the **Region** drop-down box, specify the region to create the bucket. In this lab, select the region closest to you.
**Object Ownership** change to **ACLs enabled**. 
Bucket settings for Block Public Access use default values, and select **Create bucket** in the lower right corner.
[Adding objects to buckets](#)
This lab hosts static websites through S3. The static website serves as a redirect to an instance created by the VPC Lab when you click on a particular image.
Therefore, prepare one image file, one HTML file, and an ALB DNS name.
Write and save an `index.html`

![](https://i.imgur.com/m9Qzo1t.png)

```html
<html>
    <head>
        <meta charset="utf-8">
        <title> AWS General Immersion Day S3 HoL </title>
    </head>
    <body>
        <center>
        <br>
        <h2> Click image to be redirected to the EC2 instance that you created </h2>
        <img src="{{Replace with your S3 URL Address}}" onclick="window.location='DNS Name'"/>
        </center>
    </body>
</html>
```

Upload the `aws.png` file to S3. Click _**S3 Bucket**_ that you just created.
Click the **Upload** button. Then click the **Add files** button. Select the pre-downloaded `aws.png` file through File Explorer. Alternatively, place the file in Drag and Drop to the screen.
Check the URL information to fill in the image URL in `index.html` file. Select the uploaded `aws.png` file and copy the _**Object URL**_ information from the details on the right.
![](https://i.imgur.com/r8oSDQT.png)
- specify the _**ALB DNS Name**_ of the load balancer created by [Deploy auto scaling web service](https://catalog.us-east-1.prod.workshops.aws/workshops/869a0a06-1f98-4e19-b5ac-cbb1abdfc041/en-US/advanced-modules/compute/auto-scaling) to redirect to ALB when you click on the image.
![](https://i.imgur.com/jYPqF9U.png)
Upload the `index.html` file to S3 following the same instructions as you did to upload the image.
[View Objects](#)
In the Amazon S3 Console, please _**click the object**_ you want to see. You can see detailed information about the object as shown below.
![](https://i.imgur.com/ltyEKna.png)

>By default, all objects in the S3 bucket are owner-only(Private). To determine the object through a URL of the same format as _**https://{Bucket}.s3.{region}.amazonaws.com/{Object}**_, you must grant _**Read**_ permission for external users to read it. Alternatively, you can create a signature-based Signed URL that contains credentials for that object, allowing unauthorized users to access it temporarily.

Return to the previous page and select the **Permissions** tab in the bucket. To modify the application of **Block public access (bucket settings)**, press the right Edit button.
![](https://i.imgur.com/vZzogBU.png)
**Uncheck box** and press the **Save changes** button.
![](https://i.imgur.com/TEMevRW.png)
Click the **Objects** tab, select the uploaded **files**, click the **Action** drop-down button, and press the **Make public** button to set them to public.
![](https://i.imgur.com/n8uECcf.png)
![](https://i.imgur.com/gsra5Po.png)
Return to the bucket page, select index.html, and click the _**Object URL**_ link in the Show Details entry.
![](https://i.imgur.com/utjQzuv.png)
When you access the HTML object file object URL, the following screen is printed.
![](https://i.imgur.com/xPtmGPu.png)
When you click on an image, it is redirected to the instance's web page you created.
![](https://i.imgur.com/ZUrkdLU.png)

[Enable Static Web Site Hosting](#)
select the bucket you just created, and click the **Properties** tab. Scroll down and click the Edit button on **Static website**
![](https://i.imgur.com/bVRJcOT.png)
Click **Bucket website endpoint** created in the **Static website hosting** entry to access the static website.
![](https://i.imgur.com/xsY7wYx.png)
his allows you to host static websites using Amazon S3.
![](https://i.imgur.com/4p0XKNg.png)

[Move Objects](#)
Create a temporary bucket for moving objects between buckets (Bucket name: `immersion-day-myname-target`). Substitute **myname** with your name. Rememeber the naming rules for the bucket.
**Block all public access** _**Uncheckbox**_ for quick configuration.
![](https://i.imgur.com/dJFX9Qg.png)
![](https://i.imgur.com/renAX69.png)
![](https://i.imgur.com/FAlkLGr.png)
In the Amazon S3 Console, select the bucket that contains the object (the first bucket you created) and click the checkbox for the object you want to move. Select the **Actions** menu at the top to see the various functions you can perform on that object. Select **Move** from the listed features.
![](https://i.imgur.com/JnO6v7l.png)
![](https://i.imgur.com/kTGEWG2.png)
![](https://i.imgur.com/dyyaH0G.png)
![](https://i.imgur.com/lnv2Fyt.png)

[Enable Bucket Versioning](#)
In the Amazon S3 Console, select the first S3 bucket we created. Select the **Properties** menu. Click the Edit button in **Bucket Versioning**.
1. Make some changes to the **index.html** file. Then upload the modified file to the same S3 bucket.
2. When the changed file is completely uploaded, click the object in the S3 Console. You can view _**current version**_ information by clicking the **Versions** tab on the page that contains object details.

**[Cleaning up Resources](#)**
[Deleting objects and buckets](#)
In the Amazon S3 Console, select the **Bucket** that you want to delete. Then click **Delete**. A dialog box appears for deletion.

**Database**
[Delete an Amazon RDS cluster](#)
- After accessing to the Amazon RDS console, select **DB Instances**.
- By default, an **Amazon RDS cluster** has delete protection enabled to prevent accidental deletions. To disable it, select the **Cluster** and click the **Modify** button.
- Uncheck the **Enable deletion protection** button and click the **Continue** button.
![](https://i.imgur.com/9dM7kss.png)
![](https://i.imgur.com/6RHYP1u.png)
In order to delete a DB Cluster, you must first delete the DB instances included in the cluster. They can be deleted in any order, but we will delete the **Writer instance** first. Select the **Writer instance**, and click the **Delete** button on the **Actions** menu.
Lastly, we will delete the **DB Cluster**. Click the **Delete** button on the **Actions** menu.
1. Uncheck the **Take a final snapshot** button, check the **I acknowledge that automatic backups, including system snapshots and point-in-time recovery, are no longer available when I delete an instance** button, and type **delete me** in the blank. Click **Delete DB Cluster** and the DB cluster will be deleted.
![](https://i.imgur.com/5k4AmCW.png)
 To delete the snapshot of the DB Cluster created during the lab, select **immersionday-snapshot** and click the **Delete snapshot** button on the **Actions** menu.

[Delete a secret in AWS Secrets Manager](#)
We're going to delete the secret that stored a **RDS credential** during the lab. Type Secrets Manager in the AWS console search bar and then select it. 
![](https://i.imgur.com/GfrT9H1.png)
[Delete Autoscaling Group](#)
![](https://i.imgur.com/mdcqXDc.png)
[Delete Load Balancer](#)
![](https://i.imgur.com/HQK2RQy.png)
[Delete a Target Group](#)
![](https://i.imgur.com/WDdwMCg.png)
[Delete EC2 AMIs](#)
Select **AMIs** from the left menu. Select the AMI named **Web Server v1** that you created in the lab. Click the **Deregister AMI** button on the **Actions** menu.
![](https://i.imgur.com/jisJQwI.png)
You've just deleted an AMI, but this action doesn't automatically remove the associated snapshot. So you need to remove it manually. From the left menu, choose **Snapshots**. Be sure to note the snapshot's creation date. Then, select the snapshot you created in the lab, and click the **Delete snapshot** button on the **Actions** menu.
![](https://i.imgur.com/HnuFSM8.png)
[Delete Launch Templates](#)
![](https://i.imgur.com/DL5pdFO.png)
[Delete VPC endpoints](#)
![](https://i.imgur.com/IFOntbX.png)
[Delete NAT gateway](#)
![](https://i.imgur.com/3JHc616.png)

[Delete an Elastic IP](#)
You've just deleted the NAT gateway, but this action doesn't automatically delete the Elastic IP that the NAT gateway used, so you need to remove it manually. Select **Elastic IPs** from the left menu, and select **VPC-Lab-eip-ap-northeast-2a**. (The name after VPC-Lab-eip may vary depending on your region.) 
Click the **Release Elastic IP addresses** button on the **Actions** menu. If it says it is still associated with the NAT gateway and cannot be deleted, refresh the webpage and try again.
[Delete a Security Group](#)
Select **Security Groups** from the left menu. Select **Immersion Day - Web Server and DB-SG** first, and then click the **Delete security groups** button on the **Actions** menu. The reason for not deleting all security groups at once is that some security groups reference other security groups in their inbound rules. 
A security group that is being referenced cannot be deleted until the security group that is referencing it is deleted. Therefore, delete the security groups in the following order: **Immersion Day - Web Server, DB-SG -> ASG-Web-Inst-SG -> web-ALB-SG**.
[Delete VPC](#)
