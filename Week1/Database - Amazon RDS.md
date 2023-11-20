---
tags:
  - cloud/12weeksworkshop
---
## Database
### Amazon RDS MySQL 
[[Amazon RDS]]
![](https://i.imgur.com/Y3RInJi.png)
![](https://i.imgur.com/rLNCVtG.png)

![](https://i.imgur.com/kKxQdHJ.png)

- <mark style="background: #ABF7F7A6;">Start by creating a security group. </mark>
![](https://i.imgur.com/9BgAEI3.png)
- <mark style="background: #ABF7F7A6;">Launch an RDS instance. </mark>
Here are the configurations. 
1. Database creation method - standard create
2. Engine options - MySQL
3. Templates- free tier
	1. ![](https://i.imgur.com/0MLU1SM.png)
4. Availability and durability- will be turned of since its on free tier. Not dev/test or production
5. Database instance identifier - `awsdb`
	1. master username- `awsuser`
	2. Master password- `password`
6. Instance configuration - burstable classes -`db.t2.micro`
7. Storage- `General Purpose SSD`
8. Storage autoscaling - enable storage autoscaling
9. Connectivity - default VPC and default subnet group
10. Existing VPC security groups - `immersion day - db tier`
	1. Availability Zone- `us-east-1`
	2. Database port- `3306`
11. Additional configuration
	1. initial database name- `immersionday`
	2. DB parameter group- default
12. Backup- enable
	1. backup retention period- 7 days

- <mark style="background: #ABF7F7A6;"> Save RDS Credentials. </mark>
![](https://i.imgur.com/9wIUyAH.png)
![](https://i.imgur.com/JkQJKu8.png)
![](https://i.imgur.com/qnxkqyj.png)
![](https://i.imgur.com/eaTlwQG.png)

- <mark style="background: #ABF7F7A6;">Access RDS from EC2</mark>
Now that you have created a secret, you must give your web server permission to use it. To do this, we will create a **Policy** that allows the web server to read a secret. We will add this policy to the **Role** you previously assigned to the web server.
![](https://i.imgur.com/kZvCMbw.png)

FIRST i need an IAM [instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html). Instance profiles allow us to pass an IAM role to an EC2 instance. They act as containers for an IAM role. 
	`If you use the AWS Management Console to create a role for Amazon EC2, the console automatically creates an instance profile and gives it the same name as the role. When you then use the Amazon EC2 console to launch an instance with an IAM role, you can select a role to associate with the instance. In the console, the list that's displayed is actually a list of instance profile names. The console does not create an instance profile for a role that is not associated with Amazon EC2.`
Create a role then attach the **AmazonSSMManagedInstanceCore** policy. 
Service roles can allow EC2 instances to call AWS services on your behalf.

<mark style="background: #ABF7F7A6;">[Allow Web Server to access the secret.](#) </mark>
	We start by creating a policy called ReadSecrets as for permissions select the service *Secrets Manager*
	Enable to GetSecretValue
![](https://i.imgur.com/IpltJCR.png)

![](https://i.imgur.com/Xx6crkH.png)

![](https://i.imgur.com/Qg61Ak0.png)
Go to Roles and choose SSMInstanceProfile that gets permissions from **AmazonSSMManagedInstanceCore** policy
![](https://i.imgur.com/8ODcHxE.png)
Under **Permissions policies**, click **Attach policies**.

![](https://i.imgur.com/Akk6eLI.png)
![](https://i.imgur.com/dHrXAcc.png)
![](https://i.imgur.com/QhZdR4q.png)
![](https://i.imgur.com/wv5RA9I.png)


<mark style="background: #ABF7F7A6;">Create an RDS Snapshot</mark>
- Find your RDS instance or database. Select it then click on actions and take a snapshot. Name the snapshot. In this case i called it `immersion-day-snapshot`. Create the snapshot
- Find the snapshot on the snapshots page on the left side of the page. 
	![](https://i.imgur.com/rw5z5uD.png)

<mark style="background: #ABF7F7A6;">Modify RDS instance</mark>
- You can modify the RDS instance with a few clicks. Starting from the databases pages. 
- [*] Please note you cannot reduce the allocated storage to a lower amount than previously chosen.  
- Don’t forget to click “**Apply Immediately**” – otherwise changes will be queued for the next maintenance window. Then, click on **Modify DB Instance**.

>[!HIGHLIGHT]- SUMMARY
> 1. I started off by creating DB security group. With inbound rules for type MySQL/Aurora on port 3306 and outbound rules for all traffic, all port range and destination 0.0.0.0/0
> 2. I created an RDS instance with the suitable configurations. Ranging from passwords to Availability Zone to backup
> 3. I then saved the RDS credentials to AWS secrets manager.
> 	- I named the secret name `mysecret` because the code sample had secretName variable name `mysecret`
> 4. I then had to create a way to access the RDS from an EC2 instance. Through an instance profile 
> 	- So i need to first have an instance role([[AWS Identity and Access Management(IAM)#^85ea08|check service role]]), i created a role(**SSMMInstanceProfile**) attached a policy(AmazonSSMManagedInstanceCore) to it. This policy for Amazon EC2 Role to enable AWS Systems Manager service core functionality.
> 	- Then to enable the web server/ EC2 instance to access the secret. We start by creating a policy choosing the service which will be **Secrets Manager**.
> 	- We allow the only action we need for this case which is GetSecretValue. Call the policy `ReadSecrets`.
> 5. We shall then go back to the role SSMMInstanceProfile. And attach the ReadSecrets policy. So now it has two policies. **AmazonSSMManagedInstanceCore & ReadSecrets**. Which are both contained in the **SSMMInstanceProfile**. Which we will attach to the EC2 instance. 
> 6. Finally i created a snapshot and modified the RDS instance

### Amazon RDS SQL Server.
Amazon RDS is a web service that makes it easy to set up, operate, and scale a relational database in the cloud. It provides cost-efficient and resizable capacity while managing time-consuming database administration tasks, freeing you up to focus on your applications and business.
![](https://i.imgur.com/MKIt0DM.png)

- Started by creating an EC2 windows instance. This lab will make use of the web server previously created in EC2 lab to access the DB instance using the Microsoft SQL Server Management Studio client via Microsoft Remote Desktop Protocal(RDP).
	![](https://i.imgur.com/ny3qkqh.png)
- The step for creating this RDS instance is generally the same as the last lab. But in this case we connect this RDS instance with a EC2 windows instance and not through an instance profile.
![](https://i.imgur.com/2G3k5pE.png)

![](https://i.imgur.com/1p2GJUo.png)
![](https://i.imgur.com/k56Pf4G.png)
![](https://i.imgur.com/uA7X2OC.png)
![](https://i.imgur.com/AxfH3jY.png)

![](https://i.imgur.com/pbr7P0l.png)
![](https://i.imgur.com/YmOzMKf.png)
```
$downloadUrl = "https://aka.ms/ssmsfullsetup“
$savePath = "C:\Users\Administrator\Downloads\SSMS-Setup-ENU.exe"
$webClient = New-Object System.Net.WebClient
$webClient.DownloadFile($downloadUrl, $savePath)
Write-Host "SQL Server Management Studio downloaded successfully."
Start-Process -FilePath $savePath -Wait
Write-Host "SQL Server Management Studio installation completed."

```
![](https://i.imgur.com/s55Xqk4.png)
![](https://i.imgur.com/gtRdHWG.png)
![](https://i.imgur.com/BItIukL.png)
![](https://i.imgur.com/EqLeV1S.png)
![](https://i.imgur.com/ivxAYDC.png)
![](https://i.imgur.com/y9edLPb.png)
![](https://i.imgur.com/Fhu6AK1.png)
