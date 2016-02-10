---
title: "Serverless Framework & AWS API Gateway CORS"
date: "2016-02-10"
description: "Enable AWS API Gateway CORS using the Serverless Framework"
categories:
  - "aws"
hero: "/img/serverless.png"
heroalt: "Serverless and CORS"
---

If you run into HTTP errors related to 'Access-Control-Allow-Origin' when calling a REST API endpoint through the AWS API Gateway, you probably need to enable CORS (Cross-Origin Resource Sharing) for your endpoint methods.
<!--more-->

AWS has a well written [document](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html) describing how to enable CORS in the API Gateway UI. However, if you'd like to leverage the Serverless framework to automatically create API endpoints for you, including CORS settings, this article has an example of the Serverless configuration (see below).

The Serverless configuration file you want to modify is at:

`<serverless project>/back/modules/<module>/<function>/s-function.json`

Take a look at the comment-annotated configurations from the snippet below. Please note if you copy the content to your s-function.json, remove all comments and empty lines; otherwise, the JSON probably will be invalid:

~~~js
{
  "functions": {
    "hello": {
      "custom": {
        "excludePatterns": [],
        "envVars": []
      },
      "handler": "modules/greetings/hello/handler.handler",
      "timeout": 6,
      "memorySize": 1024,
      "events": [
        {
          "name": "default",
          "eventSourceArn": "",
          "startingPosition": "TRIM_HORIZON",
          "batchSize": 100,
          "enabled": true
        }
      ],
      "endpoints": [
        {
          // Your API endpoint path
          "path": "greetings/hello",

          // Method could be POST, DELETE, etc, depending on your API schema
          "method": "GET",
          "authorizationType": "none",
          "apiKeyRequired": false,
          "requestParameters": {},
          "requestTemplates": {

            // Your endpoint method probably doesn't need this mapping
            "application/json": "{\"name\": \"$input.params('name')\"}"
          },
          "responses": {
            "400": {
              "statusCode": "400"
            },
            "default": {
              "statusCode": "200",

              // [start] Add the following response parameters for response 200. This will enable CORS for this GET method.
              "responseParameters": {
                "method.response.header.Access-Control-Allow-Headers": "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,Cache-Control'",
                "method.response.header.Access-Control-Allow-Methods": "'*'",
                "method.response.header.Access-Control-Allow-Origin": "'*'"
              },
              "responseModels": {},
              "responseTemplates": {
                "application/json": ""
              }
              // [end]
            }
          }
        },

        // [start] Add the OPTIONS method required by CORS.
        {
          "path": "greetings/hello",
          "method": "OPTIONS",
          "authorizationType": "none",
          "apiKeyRequired": false,
          "requestParameters": {},
          "requestTemplates": {
            "application/json": "\"statusCode\": 200"
          },
          "responses": {
            "default": {
              "statusCode": "200",
              "responseParameters": {
                "method.response.header.Access-Control-Allow-Headers": "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,Cache-Control'",
                "method.response.header.Access-Control-Allow-Methods": "'*'",
                "method.response.header.Access-Control-Allow-Origin": "'*'"
              },
              "responseModels": {},
              "responseTemplates": {
                "application/json": ""
              }
            }
          }
        }
        // [end]
      ]
    }
  }
}
~~~

Once you added required configuration to enable CORS, run the Serverless command to deploy the endpoint:

~~~bash
serverless function deploy
~~~

Go to AWS API Gateway, you should be able to see that CORS is now enabled for the endpoint method.

<br />
