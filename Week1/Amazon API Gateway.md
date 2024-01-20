---
tags:
  - cloud/aws
---
## Amazon API Gateway Lab.

[Amazon API Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)  is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. APIs act as the "front door" for applications to access data, business logic, or functionality from your backend services. Using API Gateway, you can create RESTful APIs, HTTP APIs, or WebSocket APIs (which enable real-time two-way communication applications). API Gateway supports containerized and serverless workloads, as well as web applications.

API Gateway handles all the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls, including traffic management, CORS support, authorization and access control, throttling, monitoring, and API version management. API Gateway has no minimum fees or startup costs. *You pay for the API calls you receive and the amount of data transferred out and, with the API Gateway tiered pricing model, you can reduce your cost as your API usage scales.*

![](https://i.imgur.com/wXNKbVs.png)  
![](https://i.imgur.com/Ow4EMXt.png)


Deployed lambda functions using AWS CLI  
	Using this command after download the template yaml file.

```yaml
aws cloudformation deploy --template-file /Users/mac/NickG/CloudDev/Workshops/General-Immersion-Workshop/packaged-template.yaml --stack-name serverless-immersion-day-stack --capabilities CAPABILITY_IAM
```
![](https://i.imgur.com/tXLf7Fv.png)

original command
```yaml
aws cloudformation deploy --template-file path-to-template-file/packaged-template.yaml --stack-name serverless-immersion-day-stack --capabilities CAPABILITY_IAM
```
Result  
![](https://i.imgur.com/HEMgNDb.png)


![](https://i.imgur.com/NCeLLOh.png)  
![](https://i.imgur.com/7jjTZib.png)  
![](https://i.imgur.com/cio8YD2.png)
#### Build a Model
The representation of the schema of an incoming request  
![](https://i.imgur.com/7F8oCBA.png)

![](https://i.imgur.com/g6G99xu.png)  
![](https://i.imgur.com/KCC0HLF.png)

[AWS Cognito](#)  
Start by creating user pools  
![](https://i.imgur.com/7llMGs3.png)

![](https://i.imgur.com/GXsFVE0.png)

![](https://i.imgur.com/NmM5NCa.png)  
![](https://i.imgur.com/ATmONm3.png)  
![](https://i.imgur.com/u2TLTWb.png)
> Configure sign-up experience leave to default values.  
![](https://i.imgur.com/TV6GIZR.png)  
![](https://i.imgur.com/Fk7mlqN.png)  
![](https://i.imgur.com/O6hM7IF.png)  
![](https://i.imgur.com/hGu79Jg.png)  
![](https://i.imgur.com/1e9PuEL.png)  
![](https://i.imgur.com/ue1OT5v.png)  
![](https://i.imgur.com/PB7jZD2.png)  
![](https://i.imgur.com/U2VUOdA.png)  
![](https://i.imgur.com/eHeuxx3.png)  
This part gave me some trouble. Because i was not using the right client id. I copied something else.  
Command used in terminal
```bash
 aws cognito-idp admin-initiate-auth --user-pool-id us-east-1_eUR6iim5J --client-id 2qsvlnafjn6oa5oavf6k55ghm5 --auth-flow ADMIN_NO_SRP_AUTH --auth-parameters 'USERNAME=example@gmail.com,PASSWORD="testUser123!"'
```

![](https://i.imgur.com/dvkp41N.png)  
command that worksi just need to replace password from the previous command. The session i got as a result of the the last command
```bash
aws cognito-idp admin-respond-to-auth-challenge --user-pool-id us-east-1_eUR6iim5J --client-id 2qsvlnafjn6oa5oavf6k55ghm5 --challenge-name NEW_PASSWORD_REQUIRED --challenge-response "USERNAME=nmaina55@gmail.com,NEW_PASSWORD=*G9#tvaXxvBN84n5" --session "AYABeFxSbWsudiYVI8DNYyxS3mwAHQABAAdTZXJ2aWNlABBDb2duaXRvVXNlclBvb2xzAAEAB2F3cy1rbXMAS2Fybjphd3M6a21zOnVzLWVhc3QtMTo3NDU2MjM0Njc1NTU6a2V5L2IxNTVhZmNhLWJmMjktNGVlZC1hZmQ4LWE5ZTA5MzY1M2RiZQC4AQIBAHgDHnKSW2nDRJSDSLf55TGFyX5On_wV32whMfiMxuCEIAHPpj63ufEDgk5M2DqYjcuXAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMDd9yQSznyGWRYwPbAgEQgDspfUCb9TmEzUJbee6aY0yv8oHOIPVBZAHd6NnDV3pj29teX60B4O4TZSOUi_HIZxlgcFi-wq-Q96siWwIAAAAADAAAEAAAAAAAAAAAAAAAAAAzZUdTFmOy8tB7NJqvOEhc_____wAAAAEAAAAAAAAAAAAAAAEAAADVwBiJugKaiRXQvaJljl8iRUMGbx0eM6SR7adwAEW2G_hzv_RExfUyzAqutL3zTer4xqDWP_NpsfONlmmvggDc-m0suO3q-6dOHUPlUyQ7DuJ6FBUj2qJDOHRbsr82qCF1IZoZzGf-4j3o967xaCTT5yqmbCoJsnMJQBYYcvsbKYOOy4ab4njVUKsXdnGcp0wB2j6e2aT0ZXA5GCpiiNAesDjJd8HDluQ_fwRjeOf7OTVG77LgGrcbR1KD1F9Vr54h1nYtJhjcIRnVJngRhAtlPNlC-9FT70mTl9S64pD4YU9dZ992dQ"
```
![](https://i.imgur.com/J61xO5d.png)  
The response contains three important parameters.  
The <mark style="background: #D2B3FFA6;">`IdToken`</mark> is the Open ID Connect-compatible identity token that API Gateway uses to authenticate calls - this is the token we will pass to the APIs in the Authorization header.  
The <mark style="background: #D2B3FFA6;">`AccessToken`</mark> is a JWT token that contains the user scopes and identity pool information.  
The last property is a <mark style="background: #D2B3FFA6;">`RefreshToken`</mark>, you can use this token with the User Pool APIs to fetch a new identity and access token.

[Postman](#)  
![](https://i.imgur.com/bG4EzZR.png)  
![](https://i.imgur.com/cqlPGsj.png)

