---
title: "Serverless Framework at FE"
date: "2016-03-17"
description: "Why do we use Serverless framework at FE?"
categories:
  - "aws"
hero: "/img/serverless.png"
heroalt: "Serverless framework"
draft: true
---

When working with Amazon Lambda, my colleagues asked what benefits we will reap from using the Serverless framework. We can directly code Lambda functions in AWS' online editor, or upload a zip file to AWS if the Lambda function requires any 3rd-party node modules. The workflow might be tedious but quite easy to understand. So why Serverless?
<!--more-->

For people new to Lambda, the amount of concepts introduced by Serverless seems to be overwhelming -- CloudFormation templates, Serverless's S3 bucket, stages, Lambda function configurations, and Serverless' commands. At the early stage of JAWS (Serverless' previous brand name), I even found the framework is very opinionated in some ways. So I tried to come up with my own solution -- [a collection of gulp tasks](https://github.com/davidcai/lambda-gulp) that automates CloudFormation and Lambda function deployment. The funny thing is that during the development of these Gulp scripts, I start to have better understanding of the Lambda development workflow, and appreciate more of what Serverless is trying to achieve. Understanding the aforementioned concepts are actually crucial to develop scalable Lambda projects. I stopped working on my own solution just before JAWS had a major release which changed the brand name to Serverless, and brought in many needed features. And from there, I picked up Serverless again.

The great thing about Serverless framework is that it incorporates many best practices that might seem to be over complicate for simple projects, but will exponentially accelerate the development cycle for mid or large projects. Sure, there are many other tools and plugins that support Lambda development and deployment, such as Jenkins' Lambda plugin, and ThoughtWorks' node module [node-aws-lambda](https://github.com/ThoughtWorksStudios/node-aws-lambda). But so far, Serverless is still the only one framework that handles the full development cycle of Lambda, API Gateway, and other AWS resources. The Serverless CLI can be integrated with CICD pipelines such as Jenkins (which we do in FE) that further automates the workflow. I found Serverless is especially useful in the following areas:

## Local Development

Coding directly in AWS Lambda editor might be OK for small projects, but it is not fitting for any project larger than hello world examples. Having Lambda functions developed in local is essential to have an efficient working setup:

1. Developers are able to version control the local Lambda code. With Serverless framework, configurations including API Gateway endpoint definitions and CloudFormation templates can also be version controlled (more about this later).

2. Developers can perform local unit test before deploying the Lambda function. The new features coming to Serverless even enables local testing of API endpoints together with Lambda functions. Greatly increased the chance to catch bugs before the actual deployment. Also, made local debugging feasible.

3. We can even pre-process the JavaScript code, e.g. linting. AWS Lambda doesn't support ES6 yet, but we can set up transpiler such as Babel in the local development environment to transpile ES6 to ES5.

## Scaffolding

With Serverless CLI, creating projects is just a command away. The `serverless project create` command prepares the local project structures, generates default CloudFormation template, and creates IAM roles.

The command also sets up a S3 bucket to hoist the bundled Lambda source code, environment variables, and CloudFormation templates. All of them are timestamped and nicely organized in their corresponding stages.

Creating new Lambda functions can be easily done via `serverless function create`, which generates the function structure, sample Lambda code, fake event JSON for local test, and function-level configuration file where the API endpoint will be defined.

All of this made Lambda development much easy to kick off.

## CloudFormation

A CloudFormation template is a JSON object that declares AWS resources required by a project. The benefit of having a template is that we can use the same template to duplicate the creation of AWS resources in different stages and regions.

For instance, our project needs to access a S3 bucket, and such bucket has its own bucket policy. Instead of manually setting up the S3 bucket and a bucket policy for each region and stage, we can declare S3 bucket and its policy in a CloudFormation template. Then, the AWS CloudFormation service will automate the resource creation by honoring the declarations in the template. This configuration driven approach is much easier to scale up, and friendly to version control.

CloudFormation is the core part of the Serverless framework. Serverless relies on CloudFormation to manage AWS resources required by both the Serverless framework and project. Resources used by Serverless itself includes IAM policies, roles, log access, and Serverless' S3 bucket. Resources required by projects could be DynamoDB, S3, etc.

Serverless provided `serverless resources deploy` command that will validate the CloudFormation template, check if the template is changed, update the CloudFormation stack if there is any change, or create the stack if the stack doesn't exist, and track the CloudFormation creation/updating status.

CloudFormation is a powerful service that needs to be understood not only for Lambda development but also other AWS resource management. The Serverless framework fully embraces this service and makes the usage really easy.

Admittedly, all these CloudFormation operations can be achieved using AWS CLI. Actually, I attempted duplicating Serverless features by creating my own Gulp tasks that leverage AWS CLI to perform CloudFormation validation, deployment, and status tracking. It is definitely doable. But I don't like to re-invent wheels.

## API Gateway Endpoint

As of today, CloudFormation still cannot be used to define AWS API Gateway endpoints. Serverless tries to close this gap. It allows API endpoints to be defined in JSON files. The `serverless function create` command will generate a default configuration file, and the `serverless endpoint deploy` command will use the configuration to create and deploy the API endpoints in Gateway.

What I'd love to see is that we will be able to use Swagger definitions to define API endpoints instead of the current ad hoc syntax. Hope the Serverless team will pick this up in the future.


<br />
