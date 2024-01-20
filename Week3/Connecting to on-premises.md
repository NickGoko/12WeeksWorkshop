---
tags:
  - 12weeksawsworkshopchallenge
---
## Prerequisites

### Deploy VPC to Simulate 'On-Premises'

Now we are ready to deploy a provided CloudFormation template a the simulated on-premises environment.  
This stack will create a VPC that will act as our on-premises environment with a public and private subnet as well as the three EC2 instances for a Customer Gateway Server, DNS Server, and Application Server.

1. Load the following CloudFormation template:  
    [CloudFormation Template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/on-premises-create-on-premises.yaml)
2. Enter the **Stack name** `NetworkingWorkshopOnPremisesVPC`
3. Accept the default values in **Configure stack options**
4. Click on **Next**
5. Click on **Create stack**
    
Wait for the stack to be created. This can take up to 10 minutes.

If the deployment of the CloudFormation template fails due to the **Elastic IP address** limit being reached and you are using your own AWS account, please refer to the **Elastic IP Quota** section in [Prerequisites - Your own AWS Account](https://catalog.workshops.aws/networking/en-US/foundational/prereqs/aws-account).

_**Now we are ready to create the VPN connection between AWS and the on-premise Customer Gateway**_

![Target State](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab3_cfn_deployed.png)

## Establish VPN Connection
In the earlier Multiple VPCs lab we created a Transit Gateway to interconnect our the VPCs. To integrate the simulated datacenter environment, we will establish a VPN connection between the Transit Gateway and a customer gateway device at the datacenter. Since this is a simulated environment, we will use OpenSWAN running on an EC2 host as the Customer Gateway.

![VPN target state](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab3_vpn_established.png)

[

### Create TGW VPN Attachment

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#create-tgw-vpn-attachment)

First, we need to capture the Public IP of the Customer Gateway that we will need in a later step.

1. In the EC2 Dashboard navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _check box_ beside the `On Premise Customer Gateway` EC2 instance and copy down the **Public IPv4 address**
    
    ![Save Customer Gateway IP](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/save_customergateway_ip.png)
    

Now we can create the VPN attachment on the Transit Gateway.

1. In the VPC dashboard navigate to [Transit Gateway Attachments](https://console.aws.amazon.com/vpc/home?#TransitGatewayAttachments) 
    
2. Click **Create transit gateway attachment** ![Create TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/create_tgw_attachment_1.png)
    
3. Under **Transit gateway ID** select the Immersion Day TGW
    
4. Change **Attachment type** to **VPN**
    
5. Under **Customer Gateway** select **New**
    
6. For IP address, enter the Public IP of the `On Premise Customer Gateway` EC2 instance that you captured in a previous step.
    
7. Change the **Routing Options** to **Static** routing.
    
8. Leave all other settings at their defaults. Click **Create transit gateway attachment**
    

![Create TGW Attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/create_tgw_attachment_2.png)

[

### Create Site-to-Site VPN Connection

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#create-site-to-site-vpn-connection)

1. Remain in the [Transit Gateway Attachments](https://console.aws.amazon.com/vpc/home?#TransitGatewayAttachments)  dashboard and wait for the newly created VPN connection to transition to the available status. Scroll down to the details tab and click on the **Resource ID** for the Site-to-Site VPN (starting with vpn-)
    
    ![VPN attachment](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_attachment.png)
    
2. In the resulting screen, select the check box next to the VPN and click **Download configuration**
    
    ![Download VPN Config](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_download_config.png)
    
3. Select `Openswan` for the vendor.
    
    ![Select Openswan](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_select_openswan.png)
    
4. Click **Download** and save the downloaded file for later.
    
5. Click **Cancel** to close the window.
    

[

### Create a New Transit Gateway Route Table for the VPN

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#create-a-new-transit-gateway-route-table-for-the-vpn)

The new VPN connection needs to be associated to a transit gateway route table. Since none of the existing ones are suitable (they do not have all the VPCs routes), we will create a new one.

1. In the VPC console navigate to [Transit Gateway Route Tables](https://console.aws.amazon.com/vpc/home?#TransitGatewayRouteTables:sort=transitGatewayRouteTableId) 
    
2. Click **Create transit gateway route table**
    
    ![Create TGW route table](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_create.png)
    
3. Enter the name `VPN Route Table` and select the Transit gateway ID.
    
4. Click **Create transit gateway route table**
    
    ![TGW RT Detail](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_create_detail.png)
    

[

### Delete VPN Attachment from Transit Gateway Default Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#delete-vpn-attachment-from-transit-gateway-default-route-table)

When we created the transit gateway, we left the option `Default route table association` enabled. This means that when we created the VPN attachment, it was automatically associated with the transit gateways default route table. Before we associate the VPN attachment to the new VPN route table, we have to disassociate it from the default route table.

If you started this workshop after the **Multiple VPCs** section, you do not need to complete this step. `Default route table association` **is not** enabled in the `NetworkingWorkshopMultiVPCandTGW` CloudFormation template deployed in the Security Controls or Connecting to On-Premises prerequisites sections.

1. Navigate back to [Transit Gateway Route Tables](https://console.aws.amazon.com/vpc/home?#TransitGatewayRouteTables:)  or remove the **Transit gateway route table ID:** filter.
2. Select the _check box_ for the TGW default route table and scroll down to the **Associations** tab ![Delete VPN Association](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_routing_association_delete.png)
3. Select the association with the resource type **VPN** and click **Delete association**
4. Confirm the deletion by clicking **Delete Association** on the following screen
5. The association will move into _disassociating_ state.

[

### Associate the VPN Attachment to the New Transit Gateway Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#associate-the-vpn-attachment-to-the-new-transit-gateway-route-table)

1. Select the new `VPN Route Table` and click **Associations** tab. Click **Create association**
    
    ![TGW RT Association overview](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_association.png)
    
2. Select the VPN attachment from the list and click **Create association**
    
    ![TGW RT Association](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_create_association.png)
    
3. Click **Propagations** tab and click **Create propagation**
    
4. Select the VPC A attachment and click **Create propagation**. Repeat this step for VPCB and VPC C.
    
    ![TGW RT VPN](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_create_propagation.png)
    

_The VPN connection is now associated to this route table, and has the ability to route to VPC A, B, and C._

![TGW RT Propagation](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/tgw_rt_vpn.png)

[

### Update Route Tables with On-Premise CIDR

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#update-route-tables-with-on-premise-cidr)[

#### Update Transit Gateway Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#update-transit-gateway-route-table)

Since we are using static routing for our VPN connection, we need to manually create a route in the Transit Gateway route table to route traffic to the on-premise CIDR block via the new VPN attachment.

1. In the VPC console navigate to [Transit Gateway Route Tables](https://console.aws.amazon.com/vpc/home?#TransitGatewayRouteTables:sort=transitGatewayRouteTableId) 
    
2. Select the _check box_ for the Transit Gateway's `Shared Services Route Table` route table and then select the **Routes** tab in the lower pane.
    
    ![Select TGW RT routes](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_tgw_route_table.png)
    
3. Click **Create static route** to add a new static route.
    
4. Enter the CIDR block for the simulated data center environment `172.16.0.0/16` and select the new VPN attachment. There should be 3 attachments of type VPC and one of type VPN.
    
5. Click **Create static route**
    
    ![Add Static Route](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_tgw_add_route.png)
    
6. It will initially be created with a route state of 'Blackhole' because we have yet to establish the VPN connection to the on-premise environment.
    
7. Repeat these steps to update the `Default Route Table` so that it also has a route to the simulated data center environment.
    

[

### Update VPC A Route Table

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#update-vpc-a-route-table)

Now that we have a Transit Gateway Attachment through which to send traffic to our on-premises network we need to add routes from our AWS VPCs to the Transit Gateway for the on-premise CIDR block. For the purposes of this lab you're only going to add an entry to the private route table for VPC A.

1. In the VPC Dashboard navigate to [Route Tables](https://console.aws.amazon.com/vpc/home?#RouteTables:) 
2. Select the _check box_ next to `VPC A Private Route Table` and scroll down to the **Routes** tab and click on **Edit routes** ![VPC A RT Edit](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_vpc_a_private_route_table.png)
3. Add a route for `172.16.0.0/16` toward Transit Gateway `VPC A Attachment`. Under **Target** select `Transit Gateway` and then choose the `VPC A Attachment`. ![VPC A RT Update](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_update_vpc_a_route_table.png)
4. Click **Save changes**

_VPC A will now have the ability to route to the simulated on-premesis environment_

**Please repeat this step for the VPC B and VPC C Private Route Tables.**

There is no need to configure the routing in the on-premesis VPC. This was configured for you in the on-premesis CloudFormation template.

[

## Configure OpenSWAN and Bring up the Tunnel

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#configure-openswan-and-bring-up-the-tunnel)

Now that we have configured the simulated datacenter VPC and created the VPN connection to the Transit Gateway, we are ready to configure OpenSWAN on the Customer Gateway and bring up the tunnel. OpenSWAN has already been installed on the host during deploymennt and we will use the configuration file downloaded previously to configure the VPN. Since OpenSWAN does not provide built-in tunnel failover capability, we will only be configuring one of the tunnels.

1. Open the configuration file that you downloaded from the VPC console for the customer gateway in a text editor. We will be following this file to configure OpenSWAN for Tunnel 1.
2. In the EC2 Dashboard navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
3. Select the _check box_ beside the `On Premise Customer Gateway` EC2 instance and click **Connect** ![Connect to CGW](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_cgw_select.png)
4. Click **Connect** on the **Session manager** tab to open a shell.
5. Create an aws.conf file in `/etc/ipsec.d`:

```bash
sudo vim /etc/ipsec.d/aws.conf
```

Enter `i` to edit in vim.

Previous way of configuring OpenSWAN

New way of configuring OpenSWAN

1. Copy and paste the following configuration. Modify `X.X.X.X` with your specific gateway IP address and `Y.Y.Y.Y` with your specific tunnel 1 IP addresses. You can get these IP addresses from the configuration file that you downloaded in [Create Site-to-Site VPN Connection](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/create-vpn#create-site-to-site-vpn-connection).
    
    ```bash
    conn Tunnel1 authby=secret auto=start left=%defaultroute leftid=X.X.X.X right=Y.Y.Y.Y type=tunnel ikelifetime=8h keylife=1h phase2alg=aes_gcm ike=aes256-sha2_256;dh14 keyingtries=%forever keyexchange=ike leftsubnet=172.16.0.0/16 rightsubnet=10.0.0.0/14 dpddelay=10 dpdtimeout=30 dpdaction=restart_by_peer
    ```
    

2. Enter `esc` and `:wq!` to save the updates and exit vim.
3. Create an aws.secrets file in `/etc/ipsec.d`
    
    ```bash
    sudo vim /etc/ipsec.d/aws.secrets
    ```
    
4. Copy and paste the pre-shared secret line from `Step 5` of the instructions into the secrets file. ![AWS Secret](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_aws_secret.png)
5. Enter `esc` and `:wq!` to save the update to aws.secrets and exit vim.
6. Enable and start OpenSWAN:

```bash
sudo systemctl enable ipsec.service sudo systemctl start ipsec.service sudo ipsec status
```

1. The output should end with lines indicating an authenticate IPsec SA and three lines indicating the tunnel details ![VPN Status](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_status.png)

AWS VPN connections are only brought up from the customer-side when using IKEv1, so we will need to send some traffic to one of our AWS VPCs from the simulated OnPrem environment.

1. From the Customer Gateway ping the private IP address of the EC2 instance in the private subnet in VPC A:
    
    Note that it may take up to 30 seconds before the tunnel comes up and you start seeing ping responses.
    
    ![VPN Ping](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/vpn_ping.png)
    
2. Enter `CTL C` to kill the ping command and terminate the Session Manager connection.
    

_**Congratulations you've just connected the on-premises environment to AWS via a Site-to-Site VPN attached to the Transit Gateway**_

![VPN deployed state](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab3_vpn_ping.png)


## Explore Hybrid Environment
Now we are ready to explore the simulated on-premises environment. ![VPN established state](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab3_vpn_established.png)

[

### Explore the Simulated Datacenter Environment

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/explore#explore-the-simulated-datacenter-environment)

We've now launched a simulated datacenter environment that consists of a customer gateway, a web application server, and a DNS server. Let's make sure that the components are working before we move on to connecting this to our AWS environment.

1. On the Customer Gateway instance check which DNS server the host is using by examining the `/etc/resolv.conf` file.
2. Take note of the "nameserver" line, which should point to 172.16.1.200 which is the IP address of the on premise DNS Server created by CloudFormation template rather than the default VPC name server on 172.16.0.2.
    
    ```bash
    sh-4.2$ cat /etc/resolv.conf ; generated by /usr/sbin/dhclient-script search example.corp options timeout:2 attempts:5 nameserver 172.16.1.200
    ```
    
3. Test the application server. In our simulated datacenter environment, we use the internal domain name _example.corp_ and the application server has a hostname entry for _myapp.example.corp_ created in the DNS server. We can test that the application server is up and running by using the curl command:
    
    ```bash
    curl http://myapp.example.corp
    ```
    
4. If your application server is working properly, you will see a response of "Hello, world."
    
    ```bash
    sh-4.2$ curl http://myapp.example.corp Hello, world.
    ```
    
5. Terminate the Session Manager connection.

[

### Test the VPN Connection from AWS to the Simulated Datacenter Network.

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/explore#test-the-vpn-connection-from-aws-to-the-simulated-datacenter-network.)

Now that our VPN tunnel is up, we can test connectivity from the AWS environment, via the Transit Gateway, to our simulated on-premises environment. Recall that our on-premises environment contains an application server and a DNS server. The application server has a DNS A record in the on-premises DNS server for the **myapp.example.corp** hostname. In this section, we will verify connectivity between the AWS and simulated datacenter environments and test that we can connect to the application server.

1. In the EC2 Dashboard select [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _check box_ next to `VPC A Private AZ1 Server` and click **Connect** to start a Session Manager console
    
3. Test connectivity to the on premise application server's private IP address of 172.16.1.100
    
    ```bash
    sh-4.2$ ping 172.16.1.100 -c 5 PING 172.16.1.100 (172.16.1.100) 56(84) bytes of data. 64 bytes from 172.16.1.100: icmp_seq=1 ttl=253 time=2.90 ms 64 bytes from 172.16.1.100: icmp_seq=2 ttl=253 time=2.28 ms 64 bytes from 172.16.1.100: icmp_seq=3 ttl=253 time=2.07 ms 64 bytes from 172.16.1.100: icmp_seq=4 ttl=253 time=2.17 ms 64 bytes from 172.16.1.100: icmp_seq=5 ttl=253 time=2.28 ms --- 172.16.1.100 ping statistics --- 5 packets transmitted, 5 received, 0% packet loss, time 4005ms rtt min/avg/max/mdev = 2.077/2.345/2.909/0.293 ms
    ```
    
4. Test the application server using curl:
    
    If your VPN connection is functioning correctly, this command will return "Hello, world." from the application server.
    
    ```bash
    sh-4.2$ curl http://172.16.1.100 Hello, world.
    ```
    
5. Next let's verify that the AWS EC2 instances can reach the on-premises DNS server if we query it directly by using the IP address and the dig command to query for myapp.example.corp:
    
    ```bash
    dig @172.16.1.200 myapp.example.corp
    ```
    
    Your query should return an A record for the on-premises application server, similar to the following:
    
```bash
    ; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.5.2 <<>> @172.16.1.200 myapp.example.corp ; (1 server found) ;; global options: +cmd ;; Got answer: ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2272 ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2 ;; OPT PSEUDOSECTION: ; EDNS: version: 0, flags:; udp: 4096 ;; QUESTION SECTION: ;myapp.example.corp. IN A ;; ANSWER SECTION: myapp.example.corp. 60 IN A 172.16.1.100 ;; AUTHORITY SECTION: example.corp. 86400 IN NS ns1.example.corp. ;; ADDITIONAL SECTION: ns1.example.corp. 60 IN A 172.16.1.200 ;; Query time: 7 msec ;; SERVER: 172.16.1.200#53(172.16.1.200) ;; WHEN: Tue Sep 21 05:01:56 UTC 2021 ;; MSG SIZE rcvd: 97
```
    
1. Test the application server by using the hostname in the curl request:
    
    ```bash
    curl http://myapp.example.corp
    ```
    
    **This should fail**. We will receive an error message "Could not resolve host" because DNS has not yet been configured to allow the AWS instances to resolve names hosted on the datacenter's DNS server.
    
    ```bash
    sh-4.2$ curl http://myapp.example.corp curl: (6) Could not resolve host: myapp.example.corp
    ```
    
2. Terminate the Session Manager connection.
    

_**Congratulations you've shown that there is connectivity in place from AWS to both the application server and DNS instance running in the on-premise environment. We will address hybrid-DNS in the next section that will allow DNS resolution of on-premise hosts from AWS.**_

![VPN traffic from AWS](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/lab3_traffic_from_aws.png)

## Hybrid DNS
Route 53 Resolver makes hybrid cloud easier for enterprise customers by enabling seamless DNS query resolution across your entire hybrid cloud. You can create DNS endpoints and conditional forwarding rules to allow resolution of DNS namespaces between your on-premises data center and AWS VPCs.

Recall that our simulated on-premises datacenter has a DNS server, providing authoritative name service for the example.corp domain where all internal application hosts are registered. In order to provide a complete hybrid-connectivity solution, we want to enable hosts in our AWS VPCs to resolve names of hosts in the datacenter environment. This can be achieved using Route 53 resolvers and conditional forwarding rules for the example.corp domain, while allowing the AWS instances to continue to take advantage of the highly available Amazon DNS service for all other name resolution inside the VPC and the internet.

_For this exercise, we will focus on establishing DNS resolution from the AWS environment to the simulated datacenter, but it's important to note that the reverse is possible as well. Route 53 Resolvers supports inbound DNS queries that are conditionally forwarded from an on-premises DNS server. You can learn more about inbound DNS resolution in the [AWS documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html#resolver-overview-forward-network-to-vpc) ._

[

### Configure a Route 53 Resolver Outbound Endpoint

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/hybrid-dns#configure-a-route-53-resolver-outbound-endpoint)

Route 53 Resolver uses endpoints to communicate with external DNS servers. An endpoint is an Elastic Network Interface (ENI) placed inside of a VPC which has connectivity to the existing DNS server. This may be a DNS server running on an EC2 instance, or a DNS server running on-premises accessible via Direct Connect or VPN. Since all three VPCs in our AWS environment have connectivity to the simulated datacenter via the Transit Gateway, we can use any of them for our endpoint. The endpoint will create interfaces in a minimum of two availability zones in your chosen VPC for high availability.

1. Navigate to [Route 53](https://console.aws.amazon.com/route53resolver/home?#/outbound-endpoints) ,
2. Expand the hamburger on the top left and Under **Resolver** select **Outbound endpoints** ![Select Outbound Endpoints](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_outbound_endpoints_screen.png)

If you have not been to the Route 53 Resolver console before, it will take you to a splash page that you see in the screenshot above. Under **Resolver** on the left menu, select **Outbound endpoints** again to take you to the screen you see in the screenshot below. You want to see the button **Create outbound endpoint** not **Configure endpoints**

1. Click **Create outbound endpoint**. ![Create Outbound Endpoint](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_outbound_endpoint_create.png)
    
2. Configure the settings for the outbound endpoint as follows:
    
    - **Endpoint name**: `NetworkingDayOutbound`
    - **VPC in the Region**: Select `VPC A`
    - **Security group for this endpoint**: Select the `default` Security Group for your VPC A, which already contains a rule permitting outbound connectivity.
    - **Endpoint type**: Select `IPv4`
    - **IP address #1**:
        - **Availability zone**: Choose `us-east-1a`
        - **Subnet**: `VPC A Private Subnet AZ1`.
        - **IP address**: `Use an IP address that is selected automatically`
    - **IP address #2**:
        - **Availability zone**: `us-east-1b`
        - **Subnet**: `VPC A Private Subnet AZ2`.
        - **IP address**: `Use an IP address that is selected automatically`
    
    ![Outbound endpoint config](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_outbound_resolver_config.png)
    
3. Click **Submit** to create the endpoint.
    
4. Wait for the endpoint status to change to _Operational_. Then proceed to the next step.
    
    ![Outbound Endpoint Operational](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_outbound_endpoint_operational.png)
    

[

### Create a Route 53 Resolver Rule for example.corp.

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/hybrid-dns#create-a-route-53-resolver-rule-for-example.corp.)

Now that we have created an outbound endpoint this provides the Route 53 Resolver with a path to the on premise DNS server via VPC A and the Transit Gateway's VPN connection. Next we need to configure a _Route 53 Resolver Rule_ to direct queries for **example.corp** to the on premise DNS server via that endpoint. We will associate the rule with VPC's A, B, and C and this will cause Route 53 Resolver to use this rule whenever the VPC DNS resolver processes queries for _example.corp_ from instances in any of those three VPCs.

1. While still in the Route 53 console, navigate to the **Rules** tab under **Resolver**
2. Select **Create rule** ![Create Resolver Rule](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_resolver_rule_create.png)
3. Provide configuration for the rule:
    - Set **Name** as `NetworkingDayRule`
    - Leave **Rule type** as `Forward`
    - Specify the **Domain name** as `example.corp`.
    - Associate the rule with all three of your AWS VPCs: `VPC A`, `VPC B`, and `VPC C`.
    - For **Outbound endpoint** choose the endpoint you created in the previous step: `NetworkingDayOutbound`
    - Under **Target IP addresses** enter the on-premise DNS server IP address: `172.16.1.200` ![Resolver Rule Config](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_resolver_rule_config.png)
4. Click **Submit** to create the rule.

[

### Testing the Route 53 Resolver Rule

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/hybrid-dns#testing-the-route-53-resolver-rule)

![DNS query flow](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/on-premises/dns_query_flow.png)

At this point, we have configured Route 53 resolver to forward queries to the datacenter's DNS server for the example.corp domain from any of the AWS VPC's A, B, or C. We can test name resolution by attempting to connect to the on-premises app server from one of the EC2 instances in our AWS VPCs.

1. In the EC2 console navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _check box_ next to `VPC B Private AZ1 Server` and click the **Connect** button above.
    
3. On the **Session manager** tab click **Connect** to open a command prompt
    
4. Check setting of the DNS server on the instance by examining `resolv.conf`:
    
    Note that the instance is using the AWS provided DNS server (e.g. 10.1.0.2) and not your on-premises DNS server for name resolution:
    
    ```bash
    nameserver 10.1.0.2 search ec2.internal
    ```
    
5. Check that the hostname for myapp.example.corp resolves to an IP address:
    
    ```bash
    nslookup myapp.example.corp
    ```
    
    Note that the instance is using the AWS provided DNS server (e.g. 10.1.0.2) and not your on-premises DNS server for name resolution:
    
    ```bash
    nslookup myapp.example.corp Server: 10.1.0.2 Address: 10.1.0.2#53 Non-authoritative answer: Name: myapp.example.corp Address: 172.16.1.100
    ```
    
6. Use _curl_ to query the on-premises application server by its hostname, myapp.example.corp
    
    ```bash
    curl http://myapp.example.corp
    ```
    
    If name resolution and your application server are functioning, you will receive back a "Hello, world" response.
    
    ```bash
    curl http://myapp.example.corp Hello, world.
    ```
    
7. Terminate the Session Manager connection.
    

_**Congratulations, you've established hybrid DNS that allows hostnames managed by the on-premises DNS server to be resolved from AWS hosts**_

## Clean Up
If you are using your own AWS Account to conduct this workshop and you are finished, complete the follow steps to clean up.

You should only complete this clean up section if you do not plan on continuing with this workshop.

Make sure you terminate / delete the resources below to avoid unnecessary charges.

[

#### Delete Site-to-Site VPN

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/cleanup#delete-site-to-site-vpn)

Step-by-step to delete Site-to-Site VPN

- Navigate to [VPC Site-to-Site console](https://console.aws.amazon.com/vpc/home?#VpnConnections:) .
    
- Select the VPN Connection that you created for the lab. From the **Actions** button, select **Delete**.
    
- Confirm the VPN is **deleted**
    

[

#### Delete the On-Premises VPC

](https://catalog.workshops.aws/networking/en-US/foundational/on-premises/cleanup#delete-the-on-premises-vpc)

Step-by-step to delete data center VPC using Cloudformation

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the On-Premises stack you created at the start of this section
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
#### Delete Route53 Resolver Endpoints

Step-by-step to delete Route53 resolver rules and endpoints

- Navigate to [Route 53 console](https://console.aws.amazon.com/route53resolver/home#/rules) .
    
- Click on the rule **Name** you created to forward request to the datacenter DNS server _it will have **Type** set to **Forward**_
    
- Click the radio button to the right of the first **VPC ID** and click the **Disassociate** button in the upper right.
    
- Repeat for all three VPCs. This will take a few minutes to complete.
    
- Again, navigate to [Route 53 rules console](https://console.aws.amazon.com/route53resolver/home#/rules) . Highlight the same rule again, and click the **Delete** button.
    
- Confirm the deletion by entering **delete** in the input field and click the **Delete** button.
    
- Navigate to [Route 53 Outbound Endpoint console](https://console.aws.amazon.com/route53resolver/home#/outbound-endpoints) .
    
- Click on the **ID** of the Route 53 Endpoint created in lab 2. _it will have a name similar to **rslvr-out-xxxxxxx**_.
    
- At the **Outbound endpoint** page, click the **Delete** button in the upper right. Confirm the deletion by entering **delete** in the input field and click the **Delete** button.
    
#### Continue Deleting Resources. Please Make a Selection:

Option 1: You started this lab at the start

Option 2: You started the lab in the Multi VPCs section by deploying the CloudFormation template

#### Delete the MultiVPC and Prerequisites CloudFormation Stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPC`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    

Option 3: You started the lab in the Basic Security or On-Premises section by deploying the CloudFormation template

#### Delete the MultiVPC and Prerequisites CloudFormation Stack

Navigate to [CloudFormation Stacks](https://console.aws.amazon.com/cloudformation/home)  in the AWS console.

- Select the stack you created in the Multi VPCs section `NetworkingWorkshopMultiVPCandTGW`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
- Select the stack you created in the prerequisites section `NetworkingWorkshopPrerequisites`
    
- Click **Delete**
    
- From the **Delete stack** dialog box, click the **Yes, Delete** button.
    
