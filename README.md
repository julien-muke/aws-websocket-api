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


We're are going to set up some lambda role permissions in order for it to be able to put item into dynamodb table

* Click on "Configuration" on the left hand side choose "Permissions" then click the Role name link



![Screenshot 2024-01-25 at 13 49 56](https://github.com/julien-muke/aws-websocket-api/assets/110755734/0f8d1484-116b-4b21-ab31-335de23fc56b)


When whenever you create a lambda function it automatically comes with the default rule which allows to add cloudwatch logs


* Sroll down to "Permissions Policies" then click the Policy Name


![Screenshot 2024-01-25 at 13 50 39](https://github.com/julien-muke/aws-websocket-api/assets/110755734/89e70557-5346-4adf-bed0-5634f7b2cbbc)


* Let's edit the Policy click on "Edit"


![Screenshot 2024-01-25 at 13 52 19](https://github.com/julien-muke/aws-websocket-api/assets/110755734/842ed99d-b6ab-4aa1-91e9-a402e24d0ada)


* Let's Add some permissions click "Add more permissions"


![Screenshot 2024-01-25 at 13 53 46](https://github.com/julien-muke/aws-websocket-api/assets/110755734/c72f7f06-a5e6-4189-a7d5-ffae7b862275)


* Select service as "DynamoDB" permission


![Screenshot 2024-01-25 at 13 54 05](https://github.com/julien-muke/aws-websocket-api/assets/110755734/68534748-4ef4-44c7-8478-0c5185b6266f)


*  Select actoins as "Putitem"
*  As resources select "Specific"
*  "Add ARNs" to restric access


![Screenshot 2024-01-25 at 13 55 22](https://github.com/julien-muke/aws-websocket-api/assets/110755734/333aeec8-8cfc-444f-b6b9-21e76e8d0d5e)


* Enter the specific ARNs:
    * Enter your resource region mine is `us-east-1`
    * Enter the resource table name that we created `websocket-connections`
    * Click on "Add ARNs"


![Screenshot 2024-01-25 at 13 56 20](https://github.com/julien-muke/aws-websocket-api/assets/110755734/de9965b7-d541-44a9-b4b2-e42725917c84)



* Review the permissions, specify details, and tags then click "Save changes"


![Screenshot 2024-01-25 at 13 57 53](https://github.com/julien-muke/aws-websocket-api/assets/110755734/5464b831-d17c-45e0-b13b-e1146c5897ab)



NOTE: We are going to follow the same steps and create two more Lambda (from step 2)


👉  The second Lambda is going to be `websocket-disconnect` this will be invoked whenever there is a disconnection request to the websocket api, we are going select python 3.9 for Runtime


![Create-function-Lambda (9)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/b45414d0-b904-4340-ab72-cba11a137718)


* The code for this particular Lambda is going to invoke DynamoDB again but it's a delete item, we are going to delete the connection ID that's being disconnected


```python
import json
import boto3
import os

dynamodb = boto3.client('dynamodb')

def lambda_handler(event, context):
    connectionId = event['requestContext']['connectionId']

    dynamodb.delete_item(
        TableName=os.environ['WEBSOCKET_TABLE'],
        Key={'connectionId': {'S': connectionId}}
    )

    return {}
```

* Let's paste it into the Lambda code source and deploy


![Screenshot 2024-01-25 at 14 00 24](https://github.com/julien-muke/aws-websocket-api/assets/110755734/c1adfc1c-5983-4816-b006-8be9cda51be7)



* Repeat the two configuration steps as we did earlier:

1. We are going to create an "Environment variable" to store the DynamoDB table name so this way if you're going to change the table name then you need not touch the code it's just enough to change the environment variable

2. On the top bar click "Configuration" then on the left hand side choose "Environment variables", clikc "Edit", then click "Add Environment variables"

3. The key is going to be named `WEBSOCKET_TABLE` and the values is going to be `websocket-conntections` then click "Save"

We're are going to set up some lambda role permissions in order for it to be able to put item into dynamodb table.

4. Click on "Configuration" on the left hand side choose "Permissions" then click the Role name link

When whenever you create a lambda function it automatically comes with the default rule which allows to add cloudwatch logs

5. Sroll down to "Permissions Policies" then click the Policy Name

6. Let's edit the Policy click on "Edit"

7. Let's Add some permissions click "Add more permissions"

8. Select service as "DynamoDB" permission

* Select actoins as "Deleteitem"
* As resources select "Specific"
* "Add ARNs" to restric access



![Screenshot 2024-01-25 at 14 09 12](https://github.com/julien-muke/aws-websocket-api/assets/110755734/8fa56ab6-9adc-43ab-a4f6-0a710bc5552a)



👉  Moving on to our third and final Lambda so that's going to be the main one which is going to be used to send the messages so we're going to call it as `websocket-send` and again it's going to use python 3.9 ( follow to same steps as shown above )


* Let's quick look at the code, it's going to do a scan of the DynamoDB Table to get all the connection IDs and also going to get the API Gateway Endpoint, these endpoint URLs can be derived from the event message that is being passed into the Lambda by the API gateway and we are going to go through each of the connection and post messages to all the connections which are stored in DynamoDB Table, this way we can ensure that all the connection all the action active connections receive the message.


```python
import json
import boto3
import os

dynamodb = boto3.client('dynamodb')


def lambda_handler(event, context):
    message = json.loads(event['body'])['message']
    paginator = dynamodb.get_paginator('scan')
    connectionIds = []

    apigatewaymanagementapi = boto3.client(
        'apigatewaymanagementapi', 
        endpoint_url = "https://" + event["requestContext"]["domainName"] + "/" + event["requestContext"]["stage"]
    )

    for page in paginator.paginate(TableName=os.environ['WEBSOCKET_TABLE']):
        connectionIds.extend(page['Items'])

    for connectionId in connectionIds:
        apigatewaymanagementapi.post_to_connection(
            Data=message,
            ConnectionId=connectionId['connectionId']['S']
        )

    return {}
```

* Let's paste the code to the Lambda code source and deploy 


![Screenshot 2024-01-25 at 14 13 35](https://github.com/julien-muke/aws-websocket-api/assets/110755734/3b913c83-9569-4712-ac29-21d86df8809b)



* Under configuration we are going to repeat the same set up the environment variable for DynamoDB Table name

* Then we're going to update the permissions, We need 2 different permissions:

* We need the DynamoDB permission but this time it's going to be a `SCAN`, get to the role and into the policy and choose DynamoDB as the servers and under action it's going to be `SCAN` 

* And table it's a specific resource the region of the resource and the table name is going to be our first permission


![Screenshot 2024-01-25 at 14 20 18](https://github.com/julien-muke/aws-websocket-api/assets/110755734/e3abc613-8955-4741-8131-e73a66b85200)



* Second one is going to be specific to the connections this time the service is going to be `ExecuteAPI` and we are going to allow the action callers `Manageconnections` so this will allow the Lambda to post messages into all the connections and i'm selecting `ANY` of the API Gateways


![Edit-policy-IAM-Global (6)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/c8d40ef0-7d14-46de-9914-be600befb9dc)


👏 All right so we have all the three lambdas. Perfect!!



## Step 3️⃣ : Creating the API Gateway


To create the API Gateway, navigate to the console and search for "API Gateway" choose "Websocket API" then click "Build"


![Screenshot 2024-01-25 at 14 27 08](https://github.com/julien-muke/aws-websocket-api/assets/110755734/5ce2f320-9c2e-4744-b5d8-2c9e2ebcfb92)



* We are going to name it `websocket-api`

* And then a route selection expression as `request.body.action` this is how your api will know which route to redirect your request to



![Screenshot 2024-01-25 at 14 28 12](https://github.com/julien-muke/aws-websocket-api/assets/110755734/3a8e97c2-ee9d-496f-87b2-5658ff4401d5)



* You have to mention the route name within the action, we are going to define 3 different routes:
-  1 for connection
-  1 for disconnection
-  1 for sending the message (send message is going to be a custom route)


![API-Gateway-Create-WebSocket-API-Step-2 (1)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/6689c087-8701-42db-8545-c2537daef927)



*  We are attaching each route to the Lambda which we created so you have various options you can
either create with a Lambda or a Mock or a Mock API, in our case we are going to do all 3 routes to the 3 different lambdas that we created.


![API-Gateway-Create-WebSocket-API-Step-3 (1)](https://github.com/julien-muke/aws-websocket-api/assets/110755734/819957e4-29bc-48e7-a42c-e5929505dc0e)





 