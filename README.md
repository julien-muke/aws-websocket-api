# ![aws](https://github.com/julien-muke/Search-Engine-Website-using-AWS/assets/110755734/01cd6124-8014-4baa-a5fe-bd227844d263)     AWS WebSocket API (Real-time chat application using python)


## 🤖 Introduction

In today's video we will discuss about websocket APIs web circuit apis support two-way communication that is both the users and the back-end can send messages back and forth once connected so it's used in real-time applications like charts broadcast messages or real-time dashboards you can create websocket apis in aws using aws api gateway service.

## 	📝 Hands-on Overview

The sample use case that we are going to see today is a real-time char application there are few components:

* First is obviously the API gateway service itself which is used to set up the application 
* And will be connected to three different lambdas to perform three different actions to establish connection to send messages * * * Finally to disconnect then we have a dynamodb table which keeps track of all the current connections.

## 🚨 AWS Services Used

* Amazon API Gateway
* AWS Lambda
* Amazon DynamoDB



## 📐 Architecture Design


![Blank diagram-11](https://github.com/julien-muke/AWS-Control-Tower/assets/110755734/786ed493-5a4d-469d-ba5a-c0047744a0e6)



## ➡️ Set up a Landing Zone