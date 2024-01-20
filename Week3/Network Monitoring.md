---
tags:
  - 12weeksawsworkshopchallenge
---


In this section, you will utilize the three VPC’s with Internet Gateways, Transit Gateway, and EC2 instances that were created in the Multiple VPCs lab. You will set up VPC Flow logs for VPC A, generate some traffic, and then view the logs in CloudWatch.

If you are running this lab in **AWS Workshop Studio**, the region has been set by your facilitator. The region you see in screenshots may not match your environment. This will not cause any problems.

If you are running this lab in **your own AWS Account**, it is recommended for all lab resources to be created in us-east-1 region so that the screenshots match your environment. This is not mandatory.

## Prerequisites

If you have not completed the Multiple VPCs section...

1. Complete the [prerequisites](https://catalog.workshops.aws/networking/en-US/foundational/prereqs) section.
    
2. Download the following CloudFormation template to create three VPCs connected by a Transit Gateway:
    
    [CloudFormation template](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/cfn/multi-vpc-create-all-vpcs-and-tgw.yaml)
    
3. Navigate to [CloudFormation](https://console.aws.amazon.com/cloudformation/home)  section in the AWS console. Click **Create stack** button and select **With new resources (standard)**.
    
4. Under **Specify template**, select **Upload a template file**, click **Choose file** and select the CloudFormation template that you downloaded above. Click **Next**.
    
5. Enter the **Stack name** `NetworkingWorkshopMultiVPCandTGW`. Leave the parameter defaults unchanged if you are running in us-east-1 and click **Next**. If you are running in another region, update the availability zones. ![CFN Stack Details](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/multivpc/pre-reqs/cfn_stack_details.png)
    
6. Scroll to the bottom of the **Configure stack options** page and click **Next** again.
    
7. Select the checkbox to acknowledge the creation of IAM resources and click the **Submit** button. ![Create stack](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/prereqs/prereqs_cfn_create_stack.png)
    
8. The CloudFormation will begin deployment which you can follow by refreshing the **Events** and **Resources** tab.
    
9. Once the CloudFormation template finishes lets go take a look at what it created. Click on the resources tab and you will find all the resources that were built as part of the CloudFormation stack.
    

This stack will create 3 VPCs each with two public subnets and two private subnets, a Transit Gateway to provide private connectivity between the VPCs, as well as EC2 instances in the VPCs.

Wait for the stack to be created before proceeding. This can take 3-5 minutes. Verify that the status in CloudFormation Console show as `CREATE_COMPLETE`, indicating successful creation of the stack.

![Multiple VPCs created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/lab-states/multiple_vpcs_completed.png)

# Cloudwatch
## Review Automatic Dashboard
Amazon CloudWatch dashboards are customizable home pages in the CloudWatch console that you can use to monitor your resources in a single view, even those resources that are spread across different Regions. You can use CloudWatch dashboards to create customized views of the metrics and alarms for your AWS resources.

1. Navigate to the [CloudWatch console](https://console.aws.amazon.com/cloudwatch) 
    
2. Click **Dashboards**, then click **Automatic Dashboards** and **VPC NAT Gateways**  
    ![Click CloudWatch Automated Dashboards](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_automated_dashboards.png)
    
3. A dashboard will be shown that contains the key network metrics for NAT Gateways

![CloudWatch NAT Gateway Dashboard](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_natgw_dashboard.png)

1. Review the metrics

## Create Custom Dashboard
Amazon EC2 provides instance-level metrics that measure CPU, disk, and network performance. These metrics include bytes and packets in/out and collected by default and can be viewed in Amazon CloudWatch.  
Amazon EC2 has recently announced additional high-resolution instance level network performance metrics for Elastic Network Adapter (ENA). With these new metrics you can gain insights into traffic drops when network allowances are exceeded. For more information, please see [Amazon EC2 instance-level network performance metrics uncover new insights](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-ec2-instance-level-network-performance-metrics-uncover-new-insights/) 

In this section we will review the Network metrics in CloudWatch for the **VPC A Private AZ1 Server** instance in VPC A and create a custom dashboard with a widget showing the **NetworkIn** and **NetworkOut** metrics and another widget for the **NetworkPacketsIn** and **NetworkPacketsOut** metrics.

1. Navigate to the [CloudWatch console](https://console.aws.amazon.com/cloudwatch) 
    
2. Under the **All Metrics** tab in the main screen click on **EC2**
    
    ![CloudWatch Metrics Page](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_tab.png)
    
3. Click on **Per-Instance Metrics**.
    
    ![Per Instance Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_per_instance.png)
    
4. Filter the list for metrics with network in the name by entering **network** in the search
    
    ![Network Metrics Search](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_search_network.png)
    
5. Select the metric **NetworkIn** and **NetworkOut** for the `VPC A Private AZ1 Server`
    
    ![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_select_network.png)
    
6. Click on the pencil icon at the top left corner of the graph and enter a name of `VPC A Private AZ1 Server Network In/Out` ![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_name_graph.png)
    
7. Click on **Actions** and select **Add to dashboard** from the dropdown
    
    ![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_select_add_to_dashboard.png)
    
8. Click on **Create new**
    

![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_create_new_dashboard.png)

1. Enter `NetworkingWorkshopEC2` and click **Create**
    
    ![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_new_dashboard_click_create.png)
    
2. Click **Add to dashboard**
    

![EC2 Instance Network Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_add_to_dashboard.png)

1. The graph of Network In/Out statistics in bytes for the EC2 instance will be added to a new dashboard. To add Packets In/Out statistics click on the **+** in the top right corner to add another widget

![New dashboard created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_metrics_added_to_dashboard.png)

1. In the **Add Widget** screen select `Number`

![Widget Screen](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_dashboard_widget_number.png)

1. Click on **EC2**

![Widget Select EC2](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_dashboard_metrics_ec2.png)

1. Click on **Per-Instance Metrics**

![Widget Select Per Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_dashboard_metrics_per_instance.png)

1. Enter `NetworkPackets` in the Search box and select the two lines for `VPC A Private AZ1 Server` and click **Create Widget**

![Select Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_dashboard_instance_select.png)

1. The new widget will then be placed on your dashboard, click **Save** to persist the dashboard. ![Widget Added](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_dashboard_widget_added.png)

## Create Alarm
You can create a CloudWatch alarm that monitors CloudWatch metrics for a given AWS service. CloudWatch will automatically send you a notification when the metric reaches a threshold you specify.

In this section you will create an alarm that monitors the amount of traffic received by an EC2 instance and sends an email notification if a threshold is breached.

1. In the [CloudWatch console](https://console.aws.amazon.com/cloudwatch)  navigate to **All alarms** and click on **Create alarm**
    
    ![All Alarms](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_console.png)
    
2. Click **Select metric**
    
    ![Select Metric](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_select_metric.png)
    
3. Click on **EC2**
    
    ![EC2 Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_ec2_metrics.png)
    
4. Click on **Per-instance Metrics**
    

![EC2 Metrics](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_instance_metrics.png)

1. Enter `NetworkIn` in the search box and hit enter.
    
    ![Metric Search](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_metric_search.png)
    
2. Select the checkbox for `VPC B Private AZ1 Server` and click **Select metric**
    
    ![Metric select](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_metric_select.png)
    
3. In the resulting **Specify metric and conditions** screen change the **Statistic** and **Period** fields to `Maximum` and `1 minute` respectively.
    

![Metric select](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_metric_fields.png)

1. In the **Conditions** section below, enter a threshold value of `1000000` and click **Next**

![Metric threshold](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_metric_threshold.png)

1. In the **Configure actions** screen click on **Add notification**
    
    ![Add Notification](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_add_notification.png)
    
2. Select **Create new topic**, enter `NetworkingWorkshopAlarmTopic` as the name, enter your email and click **Create topic**
    

![Add Notification](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_configure_notification.png)

1. Scroll to the bottom of the page, click **Next** and on the resulting screen
    - Enter `NetworkingWorkshopAlarm` as the **Alarm name**
    - Click **Next**

![Enter alarm name](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_name_alarm.png)

1. Review the values on the **Preview and create** screen and click **Create alarm**

![Preview and create](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_preview_create.png)

1. The alarm will be created but the "State" of the alarm may show "Insufficient data". This will happen until enough data points are received by the alarm.

![Alarm Created](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_ec2_alarm_created.png)

1. Under **Actions** there will be a warning for "Pending confirmation" which means that you have not confirmed the email subscription to SNS yet.

![Alarm warning](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_alarm_warning.png)

1. Go into your email, open the new message from "AWS Notifications" and confirm the subscription. This will be required to ensure you receive the notification.

![Confirm subscription](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_confirm_subscription.png)

In the next section you will get the opportunity to trigger the alarm and test the notification that you configured.

>VPC Flow Logs is a feature that enables you to capture information (metadata) about the IP traffic going to and from network interfaces in your VPC. For example, if you have a content delivery platform, flow logs can profile, analyze, and predict customer patterns of the content access, and track down top talkers and malicious calls.

# VPC Flow Logs
## Create Flow Log
Create a flow log for all traffic in VPC A and save it to the destination log group **NetworkingWorkshopFlowLog**.

### Create CloudWatch Log Group
First, let’s create a CloudWatch Log group to store flow logs in:

1. Navigate to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch) 
    
2. In the menu on the left, click **Log groups** under **Logs** and click **Create log group** in the top right
    
    ![Create Log Group](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_create_log_group.png)
    
3. Enter `NetworkingWorkshopFlowLogsGroup` as Log group name and click **Create**
    
    ![Create Log Group](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_log_group_create.png)
    

### Create VPC Flow Log

1. In the VPC Dashboard navigate to [Your VPCs](https://console.aws.amazon.com/vpc/home?#vpcs:) 
    
2. Select `VPC A`, scroll down to the **Flow Logs** tab and click on **Create flow log**
    
    ![Create Flow Log](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_flow_log_create.png)
    
3. In the resulting **Flow log settings** section
    
    - Enter `NetworkingWorkshopFlowLog` in the **Name** field
    - Set **Filter** to `All`
    - Set **Maximum aggregation interval** to `1 minute`
    - Leave **Destination** as `Send to CloudWatch Logs`
    - Set **Destination log group** as `NetworkingWorkshopFlowsLogGroup`
    - Select `NetworkingWorkshopFlowLogsRole` from the **IAM role** dropdown (this IAM role was created by the workshop's base CloudFormation template)
    
    ![Flow Log Settings](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_flow_log_settings.png)
    
4. Click on **Create flow log**
    

You completed setup of the VPC flow logs.

## Generate Traffic
<mark style="background: #ADCCFFA6;">IP traffic going to and from network interfaces in VPC A is now being collected in CloudWatch</mark>.  
 Generate some traffic between the **VPC A Private AZ1 Server** Amazon EC2 instance in **VPC A**, and the **VPC B Private AZ1 Server** instance in **VPC B** using `iperf` (a widely used tool for network performance measurement and tuning).

After generating the traffic, continue to the next step to view the flow log in CloudWatch.

### Review Security Group for EC2 Instance in VPC B

A Security Group rule for the Amazon EC2 instance in VPC B has been created for you to allow the `iperf` server to receive incoming traffic.
1. In the EC2 Dashboard navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
2. Select the _check box_ next to the `VPC B Private AZ1 Server` instance, scroll down and click on the **Security** tab below and click on the **Security groups** _link_ for `sg-xxxxxxxx (VPC B Security Group)`

![Security Group Link](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_security_group_link.png)

1. In the **Security Group** screen that opens scroll down to the **Inbound rules** tab and <mark style="background: #CACFD9A6;">confirm that port 5201 is open for TCP traffic from 10.0.0.0/8</mark>

![Security Group Screen](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_security_group.png)


### Install and Run Iperf3 Server on EC2 Instance in VPC B
1. In the EC2 Dashboard navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _check box_ next to the `VPC B Private AZ1 Server` instance, click **Connect**

![VPC B Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_vpcb_instance_connect.png)

1. Click **Connect** again in the **Session Manager** tab to open a command prompt

![Session Manager](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_session_manager_connect.png)

1. Install and start the iperf server on the EC2 instance in VPC B:
    ```
    sudo dnf install iperf3 -y &amp;&amp; iperf3 -s
    ```

![Install iperf](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_iperf_install.png)

1. Leave the Session Manager browser tab open, switch back to the **Connect to instance** tab and click on the **Instances** link

![Instances Link](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_instances_link.png)

1. Select the _check box_ next to the `VPC A Private AZ1 Server` instance, and click **Connect**

![VPC A Instance](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_vpca_instance_connect.png)

1. Click **Connect** again in the **Session Manager** tab to open a command prompt

![Session Manager](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_session_manager_connect.png)

1. Install iperf and set up a TCP transfer with 2 parallel streams for 30 seconds to the EC2 instance in VPC B.
```
    sudo dnf install iperf3 -y &amp;&amp; iperf3 -c 10.1.1.100
    -P 2 -t 30
```
    

```bash
Connecting to host 10.1.1.100, port 5201 [ 4] local 10.0.1.100 port 37860 connected to 10.1.1.100 port 5201 [ 6] local 10.0.1.100 port 37862 connected to 10.1.1.100 port 5201 [ ID] Interval Transfer Bandwidth Retr Cwnd [ 4] 0.00-1.00 sec 64.9 MBytes 544 Mbits/sec 18 429 KBytes [ 6] 0.00-1.00 sec 57.5 MBytes 482 Mbits/sec 16 380 KBytes [SUM] 0.00-1.00 sec 122 MBytes 1.03 Gbits/sec 34 - - - - - - - - - - - - - - - - - - - - - - - - - [ 4] 1.00-2.00 sec 59.6 MBytes 500 Mbits/sec 15 322 KBytes [ 6] 1.00-2.00 sec 59.0 MBytes 495 Mbits/sec 25 256 KBytes [SUM] 1.00-2.00 sec 119 MBytes 994 Mbits/sec 40 - - - - - - - - - - - - - - - - - - - - - - - - - [ 4] 2.00-3.00 sec 64.6 MBytes 542 Mbits/sec 20 330 KBytes [ 6] 2.00-3.00 sec 55.4 MBytes 465 Mbits/sec 23 223 KBytes [SUM] 2.00-3.00 sec 120 MBytes 1.01 Gbits/sec 43 - - - - - - - - - - - - - - - - - - - - - - - - - [ 4] 3.00-4.00 sec 49.6 MBytes 416 Mbits/sec 25 280 KBytes [ 6] 3.00-4.00 sec 69.3 MBytes 581 Mbits/sec 15 272 KBytes [SUM] 3.00-4.00 sec 119 MBytes 998 Mbits/sec 40 - - - - - - - - - - - - - - - - - - - - - - - - - ... - - - - - - - - - - - - - - - - - - - - - - - - - [ 4] 28.00-29.00 sec 59.7 MBytes 501 Mbits/sec 13 396 KBytes [ 6] 28.00-29.00 sec 54.9 MBytes 460 Mbits/sec 14 330 KBytes [SUM] 28.00-29.00 sec 115 MBytes 961 Mbits/sec 27 - - - - - - - - - - - - - - - - - - - - - - - - - [ 4] 29.00-30.00 sec 62.9 MBytes 528 Mbits/sec 11 330 KBytes [ 6] 29.00-30.00 sec 51.4 MBytes 431 Mbits/sec 17 289 KBytes [SUM] 29.00-30.00 sec 114 MBytes 959 Mbits/sec 28 - - - - - - - - - - - - - - - - - - - - - - - - - [ ID] Interval Transfer Bandwidth Retr [ 4] 0.00-30.00 sec 1.78 GBytes 508 Mbits/sec 412 sender [ 4] 0.00-30.00 sec 1.77 GBytes 508 Mbits/sec receiver [ 6] 0.00-30.00 sec 1.66 GBytes 474 Mbits/sec 441 sender [ 6] 0.00-30.00 sec 1.66 GBytes 474 Mbits/sec receiver [SUM] 0.00-30.00 sec 3.43 GBytes 983 Mbits/sec 853 sender [SUM] 0.00-30.00 sec 3.43 GBytes 982 Mbits/sec receiver iperf Done.
```

1. When iperf completed with an _iperf Done._ message, terminate the Session Manager connection on the VPC A instance and switch to the Session Manager tab for the connection to the VPC B instance and terminate that session too.

<mark style="background: #ABF7F7A6;">You have successfully generated traffic between the two instances.</mark> The next step is to view the flow log in CloudWatch.

## View Flow Logs in CloudWatch
When publishing to CloudWatch, flow log data is published to a log group, and each network interface has a unique log stream in the log group. Log streams contain flow log records. You can create multiple flow logs that publish data to the same log group.

1. In the EC2 Dashboard, navigate to [Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:) 
    
2. Select the _checkbox_ next to `VPC A Private AZ1 Server`, scroll down to the **Networking** tab and make a note of the **Interface ID** under **Network Interfaces**
    
    ![Instance ENI](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_note_instance_eni.png)
    

VPC Flow logs can be sent to either an Amazon S3 bucket or CloudWatch. In this lab, you configured the flow logs from VPC A to be sent to CloudWatch.

1. Navigate to Log Groups in the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups/log-group/NetworkingWorkshopFlowLogsGroup)  and click on the `NetworkingWorkshopFlowLogsGroup` log group
    
    ![Log groups](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_log_groups.png)
    
2. Click on the log stream matching the interface ID noted in step (1) to see the flow records for that interface (make sure to select the ENI from VPC A EC2)
    
    ![Flow Log Streams](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_log_streams.png)
    
3. Click on any entry to expand the log line  
    ![Flow log ENI](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_eni_logs.png)
    

Anatomy of a flow log: ![Flow log anatomy](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_anatomy.png)

## Query Flow Log for Insights
<mark style="background: #FFF3A3A6;">CloudWatch Logs Insights enables you to interactively search and analyze log data in CloudWatch Logs, including VPC flow logs.</mark>  
You can perform queries to help you more efficiently and effectively respond to operational issues. In the section you will run a query to show the top 10 talkers based on bytes transferred.

1. In the [CloudWatch console](https://console.aws.amazon.com/cloudwatch)  click on **Logs Insights**
    
    ![CloudWatch Log Insights](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_cwlogs_insights.png)
    
2. Select `NetworkingWorkshopFlowLogsGroup` from the **Select log group(s)** dropdown and click the **Queries** folder on the right hand side
    
    ![CloudWatch Log Insights](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_cwlogs_queries.png)
    
3. Click **Top 10 byte transfers by source and destination IP addresses** under **VPC Flow Logs**, click **Apply** and then **Run query**
    
    ![CloudWatch Log Insights](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_query.png)
    
4. Review the query results. Do you recognize the top two IP addresses?
    
    ![CloudWatch Query Results](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/flow_logs_query_results.png)

## Check Alarm Notification
Generating traffic to the instance in VPC B via iperf should have triggered the alarm we created earlier.

Open the inbox for the email destination you configured for the alarm and check that an email notification has arrived.

![Alarm Email Received](https://static.us-east-1.prod.workshops.aws/public/21da23c2-688e-4ff4-a97e-a0f3134c68a1/static/foundational/images/monitoring/cloudwatch_alarms_notification_email.png)

## Clean up
If you are using your own AWS Account to conduct this workshop and you are finished, complete the follow steps to clean up.

You should only complete this clean up section if you do not plan on continuing with this workshop.

Make sure you terminate / delete the resources below to avoid unnecessary charges.

#### Delete CloudWatch Alarm
Step-by-step to delete CloudWatch Alarm

- Navigate to [CloudWatch Alarm console](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups) .
    
- Select the CloudWatch Alarm that you created for the lab. Click the **Actions** button, and select **Delete**.

#### Delete the FlowLog Cloudwatch Log Group

Step-by-step to delete CloudWatch Log Group

- Navigate to [CloudWatch Log Group console](https://console.aws.amazon.com/cloudwatch/home?#alarmsV2:) .
    
- Select the CloudWatch Log Group that you created for the lab (e.g. `NetworkingWorkshopFlowLogsGroup`). Click the **Actions** button, and select **Delete log group(s)**. Confirm by pressing the **Delete** button.

#### Delete CloudWatch Dashboard
Step-by-step to delete CloudWatch Dashboard

- Navigate to [CloudWatch Dashboard console](https://console.aws.amazon.com/cloudwatch/home?#dashboards:) .
    
- Select the CloudWatch Dashboard that you created for the lab.
    
- Click the **Delete** button.
    
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
    
