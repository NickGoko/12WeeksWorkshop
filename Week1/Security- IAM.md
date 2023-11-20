---
tags:
  - cloud/12weeksworkshop
---
## Security- AWS IAM
![](https://i.imgur.com/zm66dhI.png)
[AWS Identity and Access Management (IAM)](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)  is a web service that helps you securely control access to AWS resources. You use IAM to control who is **authenticated** (signed in) and **authorized** (has permissions) to use resources.
AWS IAM key components:
![](https://i.imgur.com/LbS9EYi.png)

- IAM Identities
    - **[IAM Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)** 
    - **[IAM User groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html)** 
    - **[IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)** 
- **[IAM Policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)** 

And to help secure your AWS resources, check the document for [security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) .
![](https://i.imgur.com/vkYayBS.png)
>[!highlight]- Summary
>- So i am going to have two instances. 
>- One prod-instance and dev-instance. 
>- And then i will create a user ("dev-user"), group(Dev, & policy("DevPolicy") that will be attached to a group and the dev-user will be part of that group. 
>- The policy written in json will allow the dev-user to be able to access the EC2 instance with the tag `"ec2:ResourceTag/Env": "dev"` from their own account. 
>- Which i will have create a username and password. But will still use the same alias as the account that create the user. 
>- I should then modify the prod-instance permission so that i can have the same user permission as the dev-instance. This will be done by attaching an IAM role to the prod-instance. 



![](https://i.imgur.com/7IaEY9K.png)
![](https://i.imgur.com/RaQuutl.png)
- Defining policy for the AWS permissions, you can create and edit in visual editor or JSON. In this lab, we will use **JSON** method. Briefly describe the permission below, this policy allows all actions for **EC2 tagged as `Env-dev`**. Also, it **allows describe-related actions** for all EC2 instances. But it **denies create and delete tags action** to prevent users from modifying tags arbitrarily. Note that deny effect takes precedence over allow effect.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:*",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/Env": "dev"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": "ec2:Describe*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": [
                "ec2:DeleteTags",
                "ec2:CreateTags"
            ],
            "Resource": "*"
        }
    ]
}

```
![](https://i.imgur.com/SBKH9dD.png)
∆ After creating a policy you create a user group then attach the policy to the user group. After that you create a user and add them to that user group. After logging in the `dev-user` account you can view that both prod and dev instance are running. Try and stop the pro-instance. 

![](https://i.imgur.com/YsAMJoj.png)
- dev-user is only allowed to perform the stop action on the EC2 instance with the resource Env(key)-dev(value)
- ![](https://i.imgur.com/YQAbdxR.png)
![](https://i.imgur.com/x05WGAG.png)
![](https://i.imgur.com/GWV44dX.png)

The image below shows that there is a statement in the policy that denies creating and deleting tags in EC2. While rest are implictly denied because there are no policies to allow or deny them. 
![](https://i.imgur.com/0piOril.png)

This section below says the DevPolicy and hence the Dev-group is not allowed to stop any instance. However it is allowed to stop any instances with the dev tag.  
![](https://i.imgur.com/N4i9xM2.png)

![](https://i.imgur.com/07iE71W.png)
![](https://i.imgur.com/UjggVRF.png)
![](https://i.imgur.com/DJUFIux.png)


[Next in the lab. ](#)
I create 2 s3 buckets. Then create a policy that named both s3 buckets. 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::iam-other-traveller55/*",
                "arn:aws:s3:::iam-test-traveller55"
            ]
        }
    ]
}
```
- Then after than i connected my prod-instance with EC2 connect. I tried to list out all the s3 buckets with `aws s3 ls`. However it didn't work. So i had modify the EC2 instance through clicking on action>security then >modify IAM role. Adding the policies that allows to list all the s3 buckets. 

