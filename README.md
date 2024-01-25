# ![aws](https://github.com/julien-muke/Search-Engine-Website-using-AWS/assets/110755734/01cd6124-8014-4baa-a5fe-bd227844d263)     AWS WebSocket API (Real-time chat application using python)


## 🤖 Introduction

In today's video we will discuss about websocket APIs web circuit apis support two-way communication that is both the users and the back-end can send messages back and forth once connected so it's used in real-time applications like charts broadcast messages or real-time dashboards you can create websocket apis in aws using aws api gateway service.

## 	📝 Hands-on Overview

The sample use case that we are going to see today is a real-time chat application with few components:

* First is obviously the API gateway service itself which is used to set up the application 
* And will be connected to three different lambdas to perform three different actions to establish connection to send messages  
* Finally to disconnect then we have a dynamodb table which keeps track of all the current connections.

## 🚨 AWS Services Used

* Amazon API Gateway
* AWS Lambda
* Amazon DynamoDB



## 📐 Architecture Design


![Blank diagram-12](https://github.com/julien-muke/aws-websocket-api/assets/110755734/6fa322ac-40a7-43f9-bfcd-b1d09d5a3cf8)



##  Step 1️⃣ : Create a DynamoDB Table


Navigate to the console, seach "DynamoDB", then click "Create Table"

*  We are going to name it `websocket-connections` this is going to be used to store all the active connections

* For the "Partition Key" we can name it `connectionId` and these connection IDs are generated values which are generated by the api gateway

![Create-table-Amazon-DynamoDB-Management-Console-DynamoDB-us-east-1 (3)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/a622f731-a675-4ba5-9553-8fafeda2fe98)



## Step 2️⃣ : Create a Lambda Function


We are going to create 3 Lambda Functions

👉 The first one is going to be named `websockt-connect` this is going to be invoked whenever there is a new connection to our api gateway, i'm going to use `python 3.9` that's the latest version supported by lambda.


![Create-function-Lambda (8)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/91cc9797-23f9-48f1-8a6c-45e8850081f0)



* Let's copy and paste our code for our websocket connect to the Lambda Function then click "Deploy":


```python
import json
import boto3
import os

dynamodb = boto3.client('dynamodb')

def lambda_handler(event, context):
    connectionId = event['requestContext']['connectionId']

    dynamodb.put_item(
        TableName=os.environ['WEBSOCKET_TABLE'],
        Item={'connectionId': {'S': connectionId}}
    )

    return {}
```


![Screenshot 2024-01-25 at 13 47 31](https://github.com/julien-muke/aws-websocket-api/assets/110755734/0729f326-c952-4869-bd12-c702865f1c74)


* We are going to create an "environment variable" to store the dynamodb table name so this way if you're going to change the table name then you need not touch the code it's just enough to change the environment variable

    * On the top bar click "Configuration" then on the left hand side choose "Environment variables", clikc "Edit", then click "Add Environment variables" 


![Screenshot 2024-01-25 at 13 48 24](https://github.com/julien-muke/aws-websocket-api/assets/110755734/9999e494-3178-4f9c-9766-2d35c50fa1b9)



     * The key is going to be named `WEBSOCKET_TABLE` and the values is going to be `websocket-conntections` then click "Save"


![Screenshot 2024-01-25 at 13 49 15](https://github.com/julien-muke/aws-websocket-api/assets/110755734/a26f8c9a-062f-47c0-9951-c4e015639ce6)




