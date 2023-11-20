---
tags:
  - cloud/12weeksworkshop
---
## Monitoring- Amazon CloudWatch
![](https://i.imgur.com/L89g5vs.png)
![](https://i.imgur.com/V3XKB2S.png)
- Started by setting up an SNS topic `NickGoko-topic` followed by creating a subscription using my email. Confirmed the subscription worked. Proceeding to creating an EC2 instance with the stress tool UserData script. 
![](https://i.imgur.com/18bCHnf.png)

Now we will add a script that will create a test stress script to simulate hits on your instance. The test script will run a workload generator tool **stress** designed to subject a system to a configurable measure of CPU. We configure **stress** to spawn 1 worker with a timeput of 300000000 microseconds or 5 mins for 30 minutes.
```bash
#!/bin/sh 
yum -y update
amazon-linux-extras install epel -y
yum install stress -y
stress -c 1 --backoff 300000000 -t 30m
```
- Configure a cloud watch alarm 
![](https://i.imgur.com/m5OVf44.png)
![](https://i.imgur.com/5YCQ1Q7.png)

![](https://i.imgur.com/gRy3arh.png)

![](https://i.imgur.com/hZHqA8e.png)
![](https://i.imgur.com/eRtaTFP.png)
![](https://i.imgur.com/X48CmUl.png)
![](https://i.imgur.com/Cp1ZrQn.png)
![](https://i.imgur.com/GyHyI4S.png)


>[!highlight]- Summary
>The objective was first to setup an SNS topic and subscription creating a way an alarm could be sent. 
>Secondly created an EC2 instance loaded with a stress tool script that would run up the CPU utilization above 80% which was the limit set. When enabling detailed monitoring and creating an alarm. Which could be viewed on AWS CloudWatch. I could also get detailed metrics by copying the instance ID and pasting on the metrics panel. Configuring the custom duration i would like to view. 

