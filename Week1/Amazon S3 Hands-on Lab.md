---
tags:
  - cloud/12weeksworkshop
  - cloud/aws
Associations: "[[S3]]"
---
![](https://i.imgur.com/0RY5jlF.png)
We need a web host to interact with the S3 bucket in a real world environment. So this in this case we will use a CloudFormation Template to build the web host in EC2.`S3-General-ID-Lab.yaml`
```
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: This CloudFormation template will produce an EC2 Web Host
Parameters:
  AmiID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Description: The AMI ID - Leave as Default
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
  InstanceType:
    Description: Web Host EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - m5.large
  MyVPC:
    Description: Select Your VPC (Most Likely the Default VPC)
    Type: 'AWS::EC2::VPC::Id'
  MyIP:
    Description: Please enter your local IP address followed by a /32 to restrict HTTP(80) access. To find your IP use an internet search phrase "What is my IP".
    Type: String
    AllowedPattern: '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(32))$'
    ConstraintDescription: Must be a valid IP CIDR range of the form x.x.x.x/32
  PublicSubnet:
    Description: Select a Public Subnet from your VPC that has access to the internet
    Type: 'AWS::EC2::Subnet::Id'

Resources:
  WebhostSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      VpcId: !Ref MyVPC
      GroupName: !Sub ${AWS::StackName} - Website Security Group
      GroupDescription: Allow Access to the Webhost on Port 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: !Ref MyIP
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName} - Web Host Security Group
  WebServerInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: !Ref AmiID
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}
      NetworkInterfaces:
        - GroupSet:
            - !Ref WebhostSecurityGroup
          AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          DeleteOnTermination: 'true'
          SubnetId: !Ref PublicSubnet
      UserData: 
        Fn::Base64:
          !Sub | 
          #!/bin/sh
          yum -y update
          yum -y install httpd
          amazon-linux-extras install php7.2
          yum -y install php-mbstring
          yum -y install telnet
          case $(ps -p 1 -o comm | tail -1) in
          systemd) systemctl enable --now httpd ;;
          init) chkconfig httpd on; service httpd start ;;
          *) echo "Error starting httpd (OS not using init or systemd)." 2>&1
          esac
          if [ ! -f /var/www/html/s3-web-host.tar.gz ]; then
          cd /var/www/html
          wget https://workshop-objects.s3.amazonaws.com/general-id/s3_general_lab/s3-web-host.tar
          tar xvf s3-web-host.tar
          chown apache:root /var/www/html/labs/s3/s3.conf.php
          chown apache:root /var/www/html/labs/s3/reset_config/s3.conf.php
          fi
          yum -y update
Outputs:
  PublicIP:
    Value: !Join 
      - ''
      - - 'http://'
        - !GetAtt 
          - WebServerInstance
          - PublicIp
    Description: Newly created webhost Public IP
  PublicDNS:
    Value: !Join 
      - ''
      - - 'http://'
        - !GetAtt 
          - WebServerInstance
          - PublicDnsName
    Description: Newly created webhost Public DNS URL

```
- Once you have the yaml template upload it as a template file.
- Name you stack. For this case `[Your-initials]-S3-Web-Host`
- Select _t2.micro_ for instance type. You can choose _m5.large_ if you have a problem with t2.micro.
- Choose the default VPC or nay VPC you want to use that has a public subnet. Because the next option you need to choose a public subnet. 
- Click next once you done as well click next when configuring stack options. Leave Tags, Permissions and Advanced options as default.
- On the Review. Review your settings and click Submit.
- [!] Check on Events to view if you Web host has been created. As well as the WebhostSecurityGroup
![](https://i.imgur.com/QEegVnb.png)
![](https://i.imgur.com/cQsDRCT.png)
[Creating a bucket in S3](#)
Create a bucket in S3 within the same region as the webhost you set up with CloudFormation. `ng-workshop-s3-lab`
[Adding Objects to your S3 Bucket](#)
Upload some file. In this case i am uploading 7 photos. Leave the Permissions & Storage class setting on default and upload the photos. 
![](https://i.imgur.com/m4PHSJb.png)
[Working with objects on the S3 console](#)
- Move an object to a folder(prefix) in the same bucket to another bucket/prefix or to an access point. Create a new folder(prefix) and move the photo7.jpg object
You will find several actions that you can take on an object specific to S3:
- **Copy S3 URI:** The S3 URI acts as an internal address for access to buckets and objects by some AWS services. `s3://ng-workshop-s3-lab/photo1.jpg`
- **Copy URL:** Even though they are private unless you make them public, buckets and objects all have a URL. e.g. "photo1.jpg" in bucket  In this case it was `https://ng-workshop-s3-lab.s3.amazonaws.com/photo1.jpg`
- **Edit storage class:** This changes the [class of storage](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)  for the object, this is best done by a lifecycle policy that fits the use case.
- **Edit metadata:** Object metadata is a set of name-value pairs associated with the object. This action creates a new version of the object with updated settings and a new last-modified date.
- **Edit tags:** You can tag buckets or objects to track storage costs or other criteria.

[Accessing objects stored in S3](#)
- We start by creating an IAM role for my EC2 instance i created later in the lab. Setup to view our objects. 
- We will need three things to achieve this. 
	- Permission to access our new S3 bucket. Since our S3 bucket is private. We will do this by creating a policy and attaching it to a role with the IAM service. 
	- The name of our bucket
	- The region hosting our bucket.
- While specifying permissions, select the visual policy editor- Under select `S3`
- And the specific actions you want in this case is `GetObject` under _Read actions_
- Under _Resources_ - pick specific then specific _Add ARNs_. Paste your bucket name in the resource bucket name field. Select the check box next to "Any object name"
- Then select the on *Add ARNs* button
![](https://i.imgur.com/fkoEVIF.png)
After that click "Next" Name the policy and create policy.
- Navigate to the Roles Tab. Click *create role*. 
	- Select EC2 as the service or use case. For specific use case choose EC2 as well. Then click Next.
	- Add the permission we created in the last step. 
	- Name the role `NG-EC2-S3-ACCESS-Role` this role `Allows EC2 instances to read from a specific S3 bucket`
	- Lastly review and create the role.
- Attach the new role to your EC2 instance. Through the modify role action.
![](https://i.imgur.com/Y5yFByv.png)
**View your objects in a Web Browser**
Navigate to the EC2 instance pages. Select the instance connected to the bucket. Copy the IPV4DNS address into the clipboard. Paste the address into a new tab on your web browser. 
You should now see the "S3 Hands on lab page" where you can input your bucket information which you find on the S3 service page. 
You will be prompted for the bucket name(`ng-workshop-s3-lab`) and region(`us-east-1`). This was the result
![](https://i.imgur.com/uHa6Dx9.png)
If you are interested in what is happening behind the scenes: The images are read by your web browser from your private S3 bucket. This is done by creating a presigned URL for each object. (in this case the objects are the images) An object owner can share objects with others by creating a presigned URL, using their own security credentials, to grant time-limited permission to those objects. In this case the security credentials we passed to S3 is the role we created for our EC2 instance. If you directly open one of the images in your browser you will see the very long presigned URL.

![](https://i.imgur.com/d9vxo3p.png)
[Enabling bucket versioning](#)
1. In the [S3 Console,](https://s3.console.aws.amazon.com/s3/)  click on the **Buckets** link in the left-hand menu. Click on the name of the bucket you created earlier in the lab and then select the **Properties** tab. Under the "Bucket Versioning" heading select the **Edit** button.
2. Navigate back to the objects tab. In this case i will upload a second version on photo 2. Leave everything else on default.
3. If you go back to the web page you will be able to see all the photos again but this time with a new version of the first photo. 
4. ![](https://i.imgur.com/vZPKoVQ.png)

[Setting up a lifecycle policy](#)
You can use lifecycle policies to define actions you want Amazon S3 to take during an object's lifetime, e.g. transition objects to another storage class, archiving objects, or deleting objects after a specified period.

A versioning-enabled bucket can have many versions of the same object, one current version and zero or more noncurrent (previous) versions. Using a lifecycle policy, you can define actions specific to current and noncurrent object versions.

We are going to setup a lifecycle policy that will move noncurrent (previous) versions of your objects to the S3 Infrequent Access (IA) tier after 30 days and then delete them 30 days later.
1. In your bucket's overview page, select the **Management** tab.
2. Under "Lifecycle rules" select the **Create lifecycle rule** button. This should then open the "Create lifecycle rule" page.
3. Give your rule the name `[your initials] - S3 Lifecycle policy`
 and select the scope as **This rule applies to _all_ objects in the bucket** and put a **check** in the box acknowledging the warning. We could setup more fine-grained rules based on the prefix or on object tags, but for this lab we will apply it to the entire bucket.
4. Under "Lifecycle rule actions" put a check in the box next to **Move _noncurrent_ versions of objects between storage classes** & **Permanently delete _noncurrent_ versions of objects**. Selecting an action for a "noncurrent" version means these actions will take place on the older object version when it is replaced by a newer object version.![](https://i.imgur.com/jaae37U.png)

5. Under "Transition noncurrent versions of objects between storage classes" select **Standard-IA** for "Choose storage class transitions". Enter `30`for "Days after objects become noncurrent".
This part of the rule will move all objects from S3-Standard to S3-IA, 30 days after it becomes a previous version. This rule might be useful to save costs in S3 if the files being uploaded are frequently accessed within the first 30 days but only occasionally accessed after the first 30 days.
6. Under "Permanently delete noncurrent versions of objects" enter `60`
    . This will delete an object 60 days after it becomes previous versions. (30 days after it is moved to S3-IA.)![](https://i.imgur.com/LUyBS6S.png)

7. At the bottom you will get a timeline summary of the rule you just setup. Select **Create rule** when you have finished reviewing the summary.
8. ![](https://i.imgur.com/nBlFY7t.png)
