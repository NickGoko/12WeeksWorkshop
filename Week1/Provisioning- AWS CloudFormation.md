---
tags:
  - cloud/12weeksworkshop
  - cloud/aws
---
## Provisioning- AWS CloudFormation

![](https://i.imgur.com/rwljpqq.png)

![](https://i.imgur.com/9i5Drss.png)
[Create a VPC ](#)
1. Open a text editor and create an empty YAML File called `sfid-cfn-vpc.yaml`
2. Copy and paste the sample CloudFormation template below that defines a VPC and save the file.
```yaml
Resources:
  # Create a VPC
  MainVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

```
- Navigate to CloudFormation console. Choose "Template is ready"
- Upload the yaml file you created above. 
- Name the stack `SFID-CFN-VPC`
- Leave Configure stack options on default.Click Next.
- Submit the stack and refresh until the creation is complete on events. 
- Open you VPC console to check if the VPC was created using AWS CloudFormation. 
- Now that we created a simple VPC, we need to enable the DNS Options and name the VPC “VPC for SFID CFN
	- Add the following to the bottom of the YAML File called _sfid-cfn-vpc.yaml_ and save the file.
```yaml
      EnableDnsHostnames: 'true'
      EnableDnsSupport: 'true'
      Tags:
      - Key: Name
        Value: VPC for SFID CFN
```
- Select the "SFID-CFN-VPC" stack name in the stack list.
- Click on Update button & replace current template. Upload the update template file. 
- You can leave Parameters since nothing was defined and Click **Next**.
- You can leave Configure stack options default, click **Next**.
- Check the Changes list under the Change set preview; which shows how the changes can affect the running resources, for this case, it won’t affect our template.
- Click **Submit**.
- You can click the refresh button a few times until you see in the status _**UPDATE_COMPLETE**_.
- Navigate to the [AWS VPC Console](https://us-east-2.console.aws.amazon.com/vpc/)  to check the VPC Tag and DNS options enabled using AWS CloudFormation.
>[!highlight]
>Before the update the VPC didn't have a name just the CIDR block. 
>Now it has enabled DNS options
![](https://i.imgur.com/LUjMrzq.png)

![](https://i.imgur.com/zQ1xQZx.png)
[Create Internet Gateway](#)
- Add the following to the bottom of the YAML File called _sfid-cfn-vpc.yaml_ and save the file.
```yaml
  # Create and attach InternetGateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    DependsOn: MainVPC

  AttachIGW:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MainVPC
      InternetGatewayId: !Ref InternetGateway
```
Run the same steps you did when you last update the yaml file. 
![](https://i.imgur.com/Jdwjp4j.png)
![](https://i.imgur.com/HEb0mBm.png)
[Create First Subnet](#)
- Add the following to the bottom of the yaml file. Save the file.
```yaml
  # Create First Subnet
  FirstSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MainVPC
      CidrBlock: 10.0.10.0/24
      AvailabilityZone: "us-east-2a"
      Tags:
      - Key: Name
        Value: Public Subnet A - SFID

```
- Update the stack with new saved template. 
![](https://i.imgur.com/0AjcayX.png)
[Create Additional Subnet](#)
Add the following to the the bottom of the YAML file called sfid-cfn-vpc.yaml and save the file.
```yaml
  # Creating additional subnet
  SecondSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MainVPC
      CidrBlock: 10.0.20.0/24
      AvailabilityZone: "us-east-2b"
      Tags:
      - Key: Name
        Value: Public Subnet B - SFID

```
- Update the stack list. 
![](https://i.imgur.com/ZI8nnKq.png)
![](https://i.imgur.com/bys3F9T.png)
[Setting up routing table](#)
- Add the following to the bottom the YAML file again
```yaml
  # Create and Set Public Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:  !Ref MainVPC
      Tags:
      - Key: Name
        Value: Public Route Table

  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: AttachIGW
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # Associate Public Subnets to Public Route Table
  PublicSubnet1RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref FirstSubnet
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet2RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref SecondSubnet
      RouteTableId: !Ref PublicRouteTable

```
- Update the stack just as you have before. 
- View the changes.
- ![](https://i.imgur.com/an5tt8Y.png)

- Navigate to the VPC console and check the public routing table and association with 2 subnets. 
- ![](https://i.imgur.com/TP19GK8.png)

![](https://i.imgur.com/gIenxy2.png)
[Create Security Group](#)
- Add the following to the bottom of the YAML file
```yaml
  # Create Security Group for the following:
  MainSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security Group for Web Server
      VpcId: !Ref MainVPC
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      Tags:
      - Key: Name
        Value: Web Server Security Group - SFID

```
![](https://i.imgur.com/dwWkvUe.png)
- Navigate to security groups on the left side of the screen. 
![](https://i.imgur.com/F0lY7xo.png)
![](https://i.imgur.com/Z4opF1A.png)
- On the last step we are going to add a description to the CloudFormation template and Add outputs.
- Add the following to the top of the YAML file.
```yaml
Description: Introduction to CloudFormation SFID - Virtual Private Cloud (VPC)
```
- Add the following to the bottom of the YAML file.
```yaml
Outputs:
  MainSubnet:
    Value: !Ref FirstSubnet
    Description: Public Subnet ID with Direct Internet Route

  MainSecurityGroup:
    Value: !Ref MainSecurityGroup
    Description: Security Group ID for the Web Server

```
- Now update the “SFID-CFN-VPC” Stack Name in the Stack List
- The changes do not appear on change set preview but description does appear on the template section.
![](https://i.imgur.com/nn4qAk0.png)
Once you submit, refresh the stack & navigate to the output section. Same section you find the Events section 
![](https://i.imgur.com/xAfzNRM.png)
>[!summary] Summary Lab 1
>We divided our CloudFormation Template into the following and provided a recap to the CloudFormation Template:- Created a VPC, tagged it by providing a name and sat up the DNS Options
>- Created an Internet Gateway and attached it to the VPC
>- Created two Subnets in the VPC
>- Created a public Route Table and Associated the two subnets
>- Created a Security Group which allows Inbound Access on HTTP for Lab 2
>- Added a Description and Outputs to better understands the template.

**Now that we laid down for the networking portion for this lab, we will move to our next Lab to set up an EC2 Instance and act as web server using CloudFormation.**
![](https://i.imgur.com/I6dKsNs.png)
**Here is the complete yaml file.**
```yaml
Description: Introduction to CloudFormation SFID - Virtual Private Cloud (VPC)
Resources:

  # Create a VPC
  MainVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: 'true'
      EnableDnsSupport: 'true'
      Tags:
      - Key: Name
        Value: VPC for SFID CFN

# Create and attach InternetGateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    DependsOn: MainVPC

  AttachIGW:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MainVPC
      InternetGatewayId: !Ref InternetGateway

# Create First Subnet
  FirstSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MainVPC
      CidrBlock: 10.0.10.0/24
      AvailabilityZone: "us-east-2a"
      Tags:
      - Key: Name
        Value: Public Subnet A - SFID

# Creating additional subnet
  SecondSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MainVPC
      CidrBlock: 10.0.20.0/24
      AvailabilityZone: "us-east-2b"
      Tags:
      - Key: Name
        Value: Public Subnet B - SFID

# Create and Set Public Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:  !Ref MainVPC
      Tags:
      - Key: Name
        Value: Public Route Table

  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: AttachIGW
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  # Associate Public Subnets to Public Route Table
  PublicSubnet1RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref FirstSubnet
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet2RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref SecondSubnet
      RouteTableId: !Ref PublicRouteTable
      
  # Create Security Group for the following:
  MainSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security Group for Web Server
      VpcId: !Ref MainVPC
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      Tags:
      - Key: Name
        Value: Web Server Security Group - SFID

Outputs:
  MainSubnet:
    Value: !Ref FirstSubnet
    Description: Public Subnet ID with Direct Internet Route

  MainSecurityGroup:
    Value: !Ref MainSecurityGroup
    Description: Security Group ID for the Web Server

```
[Setting up an EC2 instance](#)
- Open a text editor and create an empty YAML file called `sfid-cfn-EC2.yaml`
- Copy and paste the following yaml code and save the file. Note the ImageId is copied from
```yaml
Resources:
# Create EC2 Linux
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: "ami-0331ebbf81138e4de"
      InstanceType: t3a.micro
```
- Open the AWS CloudFormation Console. Click on _Create stack_. Upload a template file. Select the yaml file you just created.
- Name the stack `SFID-CFN-EC2`. You can leave Configure stack options default. Submit. Refresh a few times until you see in the status _CREATE_COMPLETE_
![](https://i.imgur.com/c6bnpqN.png)

- Open the AWS EC2 Console to check the EC2 created using AWS CloudFormation.
[Tag and pass User Data to EC2 Instance](#)
- Add the following to the bottom of the YAML file called `sfid-cfn-EC2.yaml` and save the file.
```yaml
      Tags:
          - Key: Name
            Value: Web Server for IMD
      UserData: 
        Fn::Base64:
          !Sub |
          #!/bin/sh
          yum -y install httpd
          chkconfig httpd on
          systemctl start httpd
          echo '<html><center><text="#252F3E" style="font-family: Amazon Ember"><h1>AWS CloudFormation is Fun !!!</h1>' > /var/www/html/index.html
          echo '<h3><img src="https://d0.awsstatic.com/logos/powered-by-aws.png"></h3></html>' >> /var/www/html/index.html
```
- Select the "SFID-CFN-EC2" stack name in the stack list and update the stack.
- You can leave Parameters since nothing was defined and Click **Next**.
- You can leave Configure stack options default, click **Next**.
- Check the Changes list under the Change set preview; which shows how the changes can affect the running resources, for this case, it won’t affect our template. Click **Submit**.
- Navigate to the [AWS EC2 Console](https://us-east-2.console.aws.amazon.com/ec2)  to check the EC2 Name Tag and access to the Website other characteristics.

Paying attention to the EC2 Instance settings, you will notice that the Instance was launched in the default VPC, using a default Security Group which doesn’t allow traffic to the internet and User Data Script running only during Launch of the instance in our case, this is not the end goal for our lab.
[Terminate EC2 instance](#)
- Open the AWS CloudFormation Stacks Console
- We shall delete the `SFID-CFN-EC2` stack 
[Launch EC2 instance in the lab VPC ](#)
- Now that we have a better understanding on how the AWS CloudFormation Template works and setting needed for a web host to launch with the proper settings Let’s create an EC2 Instance in the proper VPC using CloudFormation
- Add the following to the top of the yaml file called `sfid-cfn-ec2.ymal`
```yaml
Parameters:
  PublicSubnet:
    Description: Select a Public Subnet created in the "VPC for SFID CFN" Lab (Hint - Search for "SFID")
    Type: 'AWS::EC2::Subnet::Id'
  SecurityGroup:
    Description: Select the Security Group created in the "VPC for SFID CFN" Lab (Hint - Search for "SFID")
    Type: 'AWS::EC2::SecurityGroup::Id'


```
Add the following to the bottom
```yaml
      NetworkInterfaces:
        - GroupSet:
            - !Ref SecurityGroup
          AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          DeleteOnTermination: 'true'
          SubnetId: !Ref PublicSubnet
```
- Create stack. choose _Template is ready_. Upload a _template file_.
- Recommend stack name `SFID-CFN-EC2`
- Select any of the _**Public Subnet A or B**_ created in the "VPC for SFID CFN" Lab _**(Hint - Search for "SFID")**_
- Select _**webserver-sg**_ created in the "VPC for SFID CFN" Lab _**(Hint - Search for "SFID")**
![](https://i.imgur.com/2Qw8ML6.png)
- Leave Configure stack options on default. Click _Next_. Submit. Click refresh until you see the status _CREATE_COMPLETE
- When you open the AWS EC2 Console. Check the security and Networking tab on the EC2 console and you will notice that instance was launched based on the information we selected in the "Parameters" prompt. 
![](https://i.imgur.com/mQPO5C4.png)
On the last step. You can add add-ons . Add a description to the CloudFormation Template and Add Outputs. 
Add the following to the top of the YAML file 
```yaml
Description: Introduction to CloudFormation SFID - Elastic Compute Cloud (EC2)
```
Add the following to the bottom of the YAML file
```yaml
Outputs:
  PublicDNS:
    Value: !Join 
      - ''
      - - 'http://'
        - !GetAtt 
          - WebServerInstance
          - PublicDnsName
    Description: Web Host Public URL

```
Update the stack and upload the update yaml file. 
>[!summary] Lab 2 Summary
>

**This is the complete yaml code for lab2**
```yaml
Description: Introduction to CloudFormation SFID - Elastic Compute Cloud (EC2)

Parameters:
  PublicSubnet:
    Description: Select a Public Subnet created in the "VPC for SFID CFN" Lab (Hint - Search for "SFID")
    Type: 'AWS::EC2::Subnet::Id'
  SecurityGroup:
    Description: Select the Security Group created in the "VPC for SFID CFN" Lab (Hint - Search for "SFID")
    Type: 'AWS::EC2::SecurityGroup::Id'

Resources:
# Create EC2 Linux
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: "ami-0331ebbf81138e4de"
      InstanceType: t3a.micro
      Tags:
          - Key: Name
            Value: Web Server for IMD
      UserData: 
        Fn::Base64:
          !Sub |
          #!/bin/sh
          yum -y install httpd
          chkconfig httpd on
          systemctl start httpd
          echo '<html><center><text="#252F3E" style="font-family: Amazon Ember"><h1>AWS CloudFormation is Fun !!!</h1>' > /var/www/html/index.html
          echo '<h3><img src="https://d0.awsstatic.com/logos/powered-by-aws.png"></h3></html>' >> /var/www/html/index.html
      NetworkInterfaces:
        - GroupSet:
            - !Ref SecurityGroup
          AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          DeleteOnTermination: 'true'
          SubnetId: !Ref PublicSubnet

Outputs:
  PublicDNS:
    Value: !Join 
      - ''
      - - 'http://'
        - !GetAtt 
          - WebServerInstance
          - PublicDnsName
    Description: Web Host Public URL

```
![](https://i.imgur.com/8RLrWN5.png)
