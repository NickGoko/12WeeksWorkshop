---
tags:
  - cloud/aws
Associations: "[[Amazon EC2]]"
---
### [EC2 Auto Scaling](https://catalog.workshops.aws/general-immersionday/en-US/basic-modules/10-ec2/ec2-auto-scaling/ec2-auto-scaling)
We start by creating an Amazon Machine Image (AMI) from a web host created with CloudFormation. 
Then continue to creating a launch template and setting up the web host within an auto scaling group behind an Application Load Balancer (ALB). 
The end result will be an auto scaling group behind a load balancer that scales based on CPU utilization of the hosts.
![](https://i.imgur.com/xo9y4jN.png)
#### [Prerequisites ](#)

- Navigate to AWS Console and search for `CloudFormation`
![](https://i.imgur.com/oO4gyKY.png)
![](https://i.imgur.com/CJHhihy.png)
1. Create Stack 
- Under template source select *Upload a template file*
![](https://i.imgur.com/UdzAV3p.png)
- Upload this EC2-Auto-Scaling-Lab.yaml file then click Next. 
>[!code]- Read through and use this yaml code to create a *EC2-Auto-Scaling-Lab.yaml* file


```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: This CloudFormation template will produce the web host to build your AMI
Parameters:
  AmiID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Description: The AMI ID - Leave as Default
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
  InstanceType:
    Description: Web Host EC2 instance type
    Type: String
    Default: m5.large
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
          #!/bin/bash -xe
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
          if [ ! -f /var/www/html/ec2-web-host.tar.gz ]; then
          cd /var/www/html
          wget https://workshop-objects.s3.amazonaws.com/general-id/ec2_auto_scaling/ec2-web-host.tar
          tar xvf ec2-web-host.tar
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
2. Specify Stack Details
- After that you will need to specify stack details.
![](https://i.imgur.com/xd423em.png)
To continue on to the next section you must first:
	- Edit you stack name with your initials `[Your initials]-EC2-Web-Host`
	- Leave AmiID to the default
	- Under instance type select m5.large to demonstrate real world performance for production workloads. Should you encounter any problem with the m5 instances load the CloudFormation Template again and select t2.micro. 
- Under MyIp, enter your local address followed by /32. Here is a link to lookup you [IP address](https://whatismyipaddress.com/)
	![](https://i.imgur.com/zkvZ7NA.png)

- Under MyVPC select the default VPC or any VPC you may want. 
- Under "PublicSubnet" **select** a subnet within your VPC that has internet access. A public subnet is defined by a subnet having a route to the internet gateway within it's route table. By default, the Default VPC subnets are all public. In the event you want to view all your subnets you can navigate to VPC on your AWS management console and click subnets. 
![](https://i.imgur.com/OjCEPGI.png)

- Once you are done entering the details above, click on **Next**.
3. Configure Stack Options. You can leave "Tags", "Permissions", and "Advanced options" as default and select **Next**. 
![](https://i.imgur.com/yaiNXaP.png)
4. Review [Your Initials]-EC2-Web-Host page, review your settings and click on **Create stack** to start building your web server.
	- Wait till the "Logical ID" "[Your Initials]-EC2-Web-Host" shows a status of "CREATE_COMPLETE".

Once the CloudFormation Stack is complete. Navigate to the EC2 service using you management console. 
Select **Instances** from the left hand menu. On the "Instances" page, select your instance "[Your Initials]-EC2-Web-Host" and copy the "Public IPv4 DNS" address. Paste this address into a new tab on your web browser.
You should be greeted by this web page with the title "EC2 Instance Metadata".
![](https://i.imgur.com/KFPay0I.png)
![](https://i.imgur.com/fpg30XO.png)


##### [Generate a Custom AMI of the web server.](#)
Now that we have our instance setup to host our website, we will generate a custom machine image for our auto scaling group. 
This will create an image of our web host that will be used by our Auto Scaling group to spin up multiple instances based on server load.
1. In the EC2 Console under Instances, you can create Amazon Machine Images (AMIs) from either running or stopped instances. 
2. Or create an Amazon Machine Image by using your instance name [your initials] - Webserver **Right-click** your webhost instance under "Image and templates" choose **Create image** from the context menu. ![](https://i.imgur.com/LZgNiW9.png)
3. On the "Create Image" page, put in the Image name `[Your Initials]_Auto_Scaling_Webhost` and a description. You can leave the Instance volumes as default and then choose **Create Image**.
![](https://i.imgur.com/eKoQGIa.png)
4. It may take a few minutes for the AMI to be created. In the EC2 console under "Images" in the left hand menu select **AMIs**. You should see the AMI you just created, it may be in a pending state, but after a few moments it will transition to an available state.

We are done with our new Amazon Machine Image for now, we can now move on to setting up our auto scaling security group.
![](https://i.imgur.com/Ly3kShk.png)

##### [Create a new Security Group for the Auto Scaling Group](#)
 A [security group](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) provides instance (virtual machine) level protection.
>[!article]-  A security group controls the traffic that is allowed to reach and leave the resources that it is associated with. For example, after you associate a security group with an EC2 instance, it controls the inbound and outbound traffic for the instance.
 ![](https://i.imgur.com/lGWo0RD.png)
![](https://i.imgur.com/u9R4skm.png)
1. Within the console Under "Services" select **EC2** or search `EC2` in the search bar. On the EC2 page under the "Network & Security" heading in the left-hand menu select **Security Groups**. 
 You should see other security groups, including the security group for your web server named "[Your Initials]-EC2-Web-Host - Website Security Group". 
 To start the creation of a new security group, click on the **Create security group** button.
2. Name your security group `[Your Initials] - Auto Scaling SG` and you can use the same name for the description as well and make sure you have the correct VPC select. (Most likely the **Default** VPC unless you setup a new one for this lab)
3. Under "Inbound rules" we will leave it empty for now. We will be creating a rule later but we need our load balancer security group to exist first.
4. "Outbound rules" currently allow all traffic out so there is no need for any additional rules. Add rule and select the CIDR range `0.0.0.0/0` setting the destination to allow all ip address to access your instance.
![](https://i.imgur.com/PRPJSpb.png)

> You are now finished with the prerequisites need. 

Below is a representation of the end state

![](https://i.imgur.com/RXwZBtA.png)


#### [Three Main Components to EC2 Auto Scaling on AWS](#)
 **1. Launch Template:** A Launch Template is a feature of EC2 Auto Scaling that allows a way to templatize your launch requests. It enables you to store launch parameters so that you do not have to specify them every time you launch an instance. 

**2. Auto Scaling Groups:** For auto scaling your EC2 instances are organized into groups so that they can be treated as a logical unit for the purposes of scaling and management. When you create a group, you can specify its minimum, maximum, and desired number of EC2 instances.

**3. Scaling Policies:** A Scaling Policy tells Auto Scaling when and how to scale. Scaling can occur manually, on a schedule, on demand, or you can use Auto Scaling to maintain a specific number of instances.

- Auto Scaling is well suited for applications that have unpredictable demand patterns that can experience hourly, daily, or weekly variability in usage. <mark style="background: #FFB86CA6;">This helps you to manage your cost and eliminate over-provisioning of capacity during times when it is not needed</mark>. Auto Scaling can also find an unhealthy instance, terminate that instance, and launch a new one based on the scaling plan.
- The number of EC2 instances can be scaled in or out as Auto Scaling responds to the metrics you define when creating these groups
	- You can specify the minimum number of instances in each Auto Scaling Group, so that your group never goes below this size. (Even if the instances are determined to be unhealthy)
	- You can specify the maximum number of instances in each Auto Scaling Group, so that your group never goes above this size.
	- You can specify a desired capacity to specify the number of healthy instances your auto scaling group should have at all times.️

[Creating a Launch Template](#)

1. Select **EC2** using your management console
2. In the left navigation pane, find "Instances" and select **Launch Templates**.
3. Now select **Create launch template**.
4. You should find yourself on Create launch template" page, starting with the "Launch template name and description":
	 a. _Launch template name_: `[Your Initials]-scaling-template`
	b. _Template version description_: This is optional
	c. _Auto scaling guidance_: **Check the box** to provide guidance.

![](https://i.imgur.com/1G7UCja.png)

5. "Launch Template Contents" defines the parameters for the instances in the Auto Scaling group:

a. Amazon machine image (AMI): Select "My AMIs" and "Owned by me". In the drop down search by typing your initials and select the custom AMI you just created [Your Initials]Auto_Scaling_Webhost. (The new AMI you just created may already be selected)
![](https://i.imgur.com/cRetsTb.png)

b. Instance type_: t2.micro
![](https://i.imgur.com/rNN9wUn.png)

c. Key pair (Login): **Select** the Key Pair you created in the past, it is most likely call [Your Initials]-KeyPair. If not here is a guide to create [one](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)
![](https://i.imgur.com/pnX5dnF.png)

d. Networking Settings:

- Subnet: **Don't include in launch template**
- Firewall (security groups): Select "Select existing security group" and then select the security group you created in the first part of this lab named **[Your Initials] - Auto Scaling SG**
![](https://i.imgur.com/a6UFdSN.png)

e. Configure storage: Leave as default
f. Resource tags: None
g. Advanced Details: **IMPORTANT**: Select the arrow to expand "Advanced details" and under "Detailed Cloudwatch monitoring" select **Enable**. Leave everything else as the default.

![](https://i.imgur.com/0N7q9fL.png)

By enabling CloudWatch Detailed monitoring. CloudWatch will monitor the instances in your auto scaling group in 1-minute intervals. This will allow the auto scaling group to respond quicker to changes in the group.

By default, your instance has basic monitoring in 5-minute intervals for the instances. 

[Create Auto Scaling Group](#)
1. Select **EC2** using your management console
2. In the left navigation pane find "Auto Scaling" and Select Auto Scaling Groups.
3. Click Create an Auto Scaling group.
4. Give the Auto Scaling group a name: `[Your Initials]-Lab-AutoScaling-Group`
![](https://i.imgur.com/r04sPZt.png)
5. From the Launch Template drop down choose the launch template named [Your Initials]-scaling-template you created in the previous section and select Next.
6. Configure settings page, configure the following and select **Next**:
    Network:   
    - VPC: **Select** your VPC (most likely Default) 
    - Subnets: **Select** the subnets where you would like the auto scaling group to use when spinning up the hosts. (If you are using the default VPC, this will most likely be four subnets)
    ![](https://i.imgur.com/vazKvTf.png)
    

A best practice for your Auto Scaling Group would be to select only private subnets. The instances will be sitting behind a load balancer and will not need public IP addresses.

7. Specify load balancing and health checks:

    a. _Load balancing_: **Attach to a new load balancer**

    b. _Load balancer type_: **Application Load Balancer**

    c. _Load balancer name_: `[Your Initials]-Application-Load-Balancer`

    d. _Load balancer scheme_: **Internet-facing**

    e. _Networking mapping_: You should see all the Availability Zones and subnets you selected in the previous step. (If you had multiple subnets per AZ, this were it would let you choose between them)
	![](https://i.imgur.com/14l4eKe.png)

    f. _Listeners and routing_: Keep the Port as 80 and select **Create a target group** from the "Default routing (forward to)" dropdown.
	![](https://i.imgur.com/Wdws1Jt.png)

    g. _New target group name_: `[Your Initials]-Target-Group`

    - The target group is where your load balancer is going to look for instances to distribute traffic. We are setting our auto scaling group to automatically register instances into this group and it will also be associated to our load balancer.

    h. _Health checks & Additional settings_: Leave as default and select **Next**.
8.  Configure the group size and scaling policies below and then select **Next**
    a. _Group Size_: The settings below will keep our group size to one EC2 instance unless a scaling policy is triggered.

    - Desired capacity: `1`
 
    - Minimum capacity: `1`

    - Maximum Capacity: `5`

    b. _Scaling policies_: Select **Target tracking scaling policy**
    
    - Metric type: Average CPU utilization
        
    - Target Value: `25`
        
	![](https://i.imgur.com/lyOp5tZ.png)

We are going to set our target CPU utilization low to speed up the results.

9. Add Notifications:
	You can configure your Auto Scaling Group to send notifications to an endpoint that you choose, such as an _email address_. You can receive notifications whenever a specified event takes place, including the successful launch of an instance, failed instance launch, instance termination, and failed instance termination.

10. Add Tags: Add a single tag and then select **Next**.
    a. Select the **Add tag** button and configure the following:
    - Key: `Name`
    - Value: `[Your Initials] - Auto Scaling Group`

11. Then select **Create Auto Scaling group**. You have now created your Auto Scaling Group, target group and load balancer.    
	a. You will soon see a new instance created by the Auto Scaling group in the EC2 console with the name tag "[Your Initials] - Auto Scaling Group". (You may need to refresh the screen to see the instance)
	    
	b. If you select **Load Balancers** under "Load Balancing" in the left hand menu, you will see your load balancer provisioning.
	    
In the next step we will create an additional security group and update the security settings to allow traffic to flow between the ALB and our web hosts.


#### Penultimate things we must do before we come to the end is:
#### [Configuring Security Groups](#)
[Creating a Load Balancer Security Group](#)

When our load balancer was provisioned it was setup with the default security group in our VPC. To allow access to the load balancer via the public DNS, we will need to create and attach a security group to allow inbound traffic on port 80 from the internet.

We will also create an outbound rule that allows outgoing traffic from the load balancer to only be sent to hosts within the Auto Scaling Security Group.
On the EC2 page under the "Network & Security" heading in the left-hand menu select **Security Groups**. You should see other security groups, including the security group for your web server named [Your Initials]-EC2-Web-Host - Website Security Group. 
Click on the **Create security group** button.

1. Basic details:
    a. _Security group name_: `[Your Initials]-SG-Load-Balancer`
    b. _Description_: `[Your Initials]-SG-Load-Balancer`
    c. _VPC_: Select your VPC (Most likely the Default VPC)
2. Inbound rules:
    a. Click on the **Add rule** button
    b. _Type_: `HTTP`
	![](https://i.imgur.com/ka3Cvy6.png)

    c. _Source_: Custom: `[Input your public IP address followed by a /32]`
    
     (You can find your local IP by searching [What is my IP.](https://www.google.com/search?q=What+is+my+ip%3F&rlz) )
    
3. Outbound rules:
    a. Find the "All traffic" rule and click on **Delete** to remove the rule. (All Outbound rules should now be removed)
    b. Click on the **Add rule** button
    c. _Type_: HTTP
    d. Under "Destination" select **Custom** and in the field select your **[Your Initials]-Auto Scaling SG** as the "Destination". Hint: start by typing `sg`
     to get the Security Group list.
    f. You security group configuration should look similar to the image below. Select **Create security group** when finished.
	  ![](https://i.imgur.com/FNK72qy.png)
	  
4. Attach your new Load Balancer Security group to your Load Balancer:
    a. On the EC2 service page left side menu find "Load Balancing" and select **Load Balancers**. Select the load balancer you created. Make sure the State is "Active".
    b. Under the "Description" tab scroll down to the "Security" section and click on **Edit security groups**.
    c. Select the box to the left of your new load balancer sg named `[Your Initials]-SG-Load-Balancer`
    d. Make sure you also **un-select** any other security group and then click on the **Save** button.
	![](https://i.imgur.com/eXmD19m.png)

#### [Add Inbound Rule to the Auto Scaling Security Group](#)
1. We will need to setup a rule to only allow traffic from the new Load Balancer Security Group to the Auto Scaling Security Group. This will be one of the layers of protection that will prevent our webhosts from being directly accessed from the internet.
    a. On the EC2 service page left side menu under "Network & Security" select **Security Groups**.
    b. Select your Auto Scaling Security Group: **[Your Initials] - Auto Scaling SG**
    c. Select the **Inbound Rules** tab and click on the **Edit inbound rules** button and then the **Add rule** button.
    d. From the "Type" drop down select **HTTP**. Under "Source" select **Custom** and in the field specify your **[Your Initials]-SG-Load-Balancer** as the "Source". Hint: start by typing `sg` to get the Security Group list. Now click on **Save rules**.
![](https://i.imgur.com/QM2NCIW.png)

2. We will now test to make sure your load balancer is working. There is currently only one instance (or target) running in the auto scaling group, but you should be able to access the website.
    
    Return to your the load balancers page by selecting **Load Balancers** from the left hand menu. Under the "Description" tab **copy** the DNS name and **paste** it into a web browser. You should now see the website being loaded from your auto scaling group. Leave this page open, you will need it in the next step.

#### [Testing the AutoScaling Group](#)
Now that you have created your Auto Scaling Group and load balancer, let's test it to ensure that everything is working correctly.
1. Make sure you are on the website accessed through the Load Balancer DNS address in the previous step.
2. At the bottom of the front page click on the **Start CPU Load Generation** link: Once the CPU load goes above 25% for a sustained period the Auto Scaling policy will begin spinning up the instances specified in the launch template to meet demand. _(You may have to do this twice if the first time doesn't generate enough load)_
3. In the "Instances" section of the EC2 Console you can watch for the new instances created by Auto Scaling, this might take a couple of minutes. Refresh the EC2 instances page and you should soon see a new instance spinning up automatically. You can select the instance named [Your Initials] - Auto Scaling Group and click on the **Monitoring** tab below to keep an eye on the "CPU Utilization".
4. 4. You can also see this by going to the Auto Scaling Groups page. [https://console.aws.amazon.com/ec2autoscaling](https://console.aws.amazon.com/ec2autoscaling)  Then select your auto scaling group **[Your Initials]-Lab-AutoScaling-Group**. If you look at the details under the Instance management tab, you can see if new instances are spinning up. You can look at the Instance management tab to see how many instances there are in your group currently. The monitoring tab shows you different metrics like group size, pending instances, total instances, and much more.
> Your architecture now looks like this
> 
![](https://i.imgur.com/BBVYhff.png)

5. Once a number of new instances have successfully started (probably 3 or 4), repeatedly **refresh** your web-browser on you web host. You should now see the Instance ID, Availability Zone and Private IP change as the load balancer distributes the requests across the Auto Scaling group.
> Your have successfully created an EC2 Auto Scaling Group behind an Application Load Balancer.



