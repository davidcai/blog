---
title: "Serverless Framework at Financial Engines"
date: "2016-03-17"
description: "What we learned from adopting Serverless framework at Financial Engines?"
categories:
  - "aws"
hero: "/img/serverless-fe.png"
heroalt: "Serverless framework at FE"
---

When working with Amazon Lambda, my colleagues asked what benefits we will reap from using the Serverless framework. We can directly code Lambda functions in AWS' online editor, or upload a zip file to AWS if the Lambda function requires any 3rd-party node modules. The workflow might be tedious but quite easy to understand. So why Serverless?
<!--more-->

For people new to Lambda, the amount of concepts introduced by Serverless could be overwhelming -- CloudFormation templates, Serverless's S3 bucket, stages, Lambda function configurations, and Serverless' commands. At the early stage of JAWS (Serverless' previous brand name), I even found the framework is very opinionated in some way. So I tried to come up with my own solution -- [a collection of gulp tasks](https://github.com/davidcai/lambda-gulp) that automate CloudFormation and Lambda function deployment. The funny thing is that during the development of these Gulp scripts, I start to have better understanding of the Lambda development workflow, and appreciate more of what Serverless is trying to achieve. Understanding the aforementioned concepts are actually crucial to develop scalable Lambda projects. I stopped working on my own solution just before JAWS had a major release which re-branded the framework to Serverless, and brought in many needed features. And from then, I picked up Serverless again.

## What Serverless gave us

The great thing about Serverless framework is that it incorporates many good practices that might seem to be over-complicate for simple projects, but will exponentially accelerate the development cycle for mid and large projects. Sure, there are many other tools and plugins that support Lambda development and deployment, such as Jenkins' Lambda plugin, and ThoughtWorks' node module [node-aws-lambda](https://github.com/ThoughtWorksStudios/node-aws-lambda). But so far, Serverless is still one of the very few popular frameworks that handle the full development cycle of Lambda, API Gateway, and other AWS resources. The Serverless CLI can be integrated with CICD pipelines such as Jenkins (which we do at Financial Engines) that further automates the workflow. I found Serverless is especially useful in the following areas:

<br/>

### 1. Local Development

Coding directly in AWS Lambda editor might be OK for small projects, but it is not fitting for any project larger than hello world examples. Having Lambda functions developed in local is essential to have an efficient working setup:

* Developers are able to version control the local Lambda code. With Serverless framework, configurations including API Gateway endpoint definitions and CloudFormation templates can also be version controlled (more about CloudFormation later).

* Developers can run local unit test before deploying the Lambda function. The new features coming to Serverless even enables local test of REST API endpoints together with Lambda functions. Greatly increased the chance to catch bugs before the actual deployment. Also, made local debugging feasible.

* We can even pre-process the JavaScript code, e.g. linting. AWS Lambda doesn't support ES6 yet, but we can set up transpiler such as Babel in the local development environment to transpile ES6 down to ES5.

<br/>

### 2. Scaffolding

With Serverless CLI, creating projects is just a command away. The `serverless project create` command prepares the local project structures, generates default CloudFormation template, and creates IAM roles.

The command also sets up a S3 bucket to hoist the bundled Lambda source code, environment variables, and CloudFormation templates. All of them are timestamped and nicely organized in their corresponding stages.

Creating new Lambda functions can be easily done via `serverless function create`, which generates the function structure, sample Lambda code, event mockup for local test, and function-level configuration file where the API endpoint will be defined.

All of this made Lambda development much easy to kick off.

<br/>

### 3. CloudFormation

A CloudFormation template is a JSON file that declares AWS resources required by a project. The benefit of having a template is that we can use the same template to duplicate the creation of AWS resources in different stages and regions. The picture below illustrated the relationships among regions, stages, Lambda functions, API endpoints, and other AWS resources:

![Stages](/img/aws-stages.png)

(L = Lambda function, E = API endpoint)

For instance, our project needs to access a S3 bucket, and such bucket has its own bucket policy. Instead of manually setting up the S3 bucket and a bucket policy for each region and stage, we can declare S3 bucket and its policy in a CloudFormation template. Then, the AWS CloudFormation service will automate the resource creation by honoring the declarations in the template. This configuration driven approach is much easier to scale up.

CloudFormation is the core part of the Serverless framework. Serverless relies on CloudFormation to manage AWS resources required by both the Serverless framework and project. Resources used by Serverless itself includes IAM policies, roles, log access, and Serverless' S3 bucket. Resources required by projects could be DynamoDB, S3, etc.

Serverless provided `serverless resources deploy` command that will validate the CloudFormation template, check if the template is changed, update the CloudFormation stack if there is any change, or create the stack if the stack doesn't exist, and track the CloudFormation creation/updating status.

CloudFormation is a powerful service that needs to be understood not only for Lambda development but also other AWS resource management. The Serverless framework fully embraces this service and makes the usage really easy.

Admittedly, all these CloudFormation operations can be achieved using AWS CLI or SDK. Actually, I attempted duplicating Serverless features by creating my own Gulp tasks that leverage AWS SDK to perform CloudFormation validation, deployment, and status tracking. It is definitely doable. But I don't like to re-invent wheels.

<br/>

### 4. API Gateway Endpoint

As of today, CloudFormation still cannot be used to define AWS API Gateway endpoints. Serverless framework tries to close this gap. It allows API endpoints to be defined in JSON files. The `serverless function create` command generates a default configuration file, and the `serverless endpoint deploy` command will use the configuration to create and deploy the API endpoints in Gateway. (For more examples, check my other [blog post](/posts/serverless-and-cors/).)

What I'd love to see is the ability to use Swagger to define API endpoints instead of the current ad hoc syntax. Hope the Serverless team will pick this up in the future.

<br/>

## Things I'd love to have

<br/>

### 1. Multiple AWS account support

Just like other young software, Serverless certainly has room for improvement. One thing I'd really love to have is the support for deploying AWS resources in multiple AWS accounts. Currently, Serverless embraces Stages for promoting builds from development to test, and then from test to production. Each build environment is a 'stage', and they all live in one AWS account.

However, at Financial Engines, we have three AWS accounts -- dev, test, and prod. We'd like to promote builds from dev account to test account, and to prod account without bothering with stages. This three layers of accounts is actually quite common practice in IT. However, since Serverless doesn't support multiple AWS accounts yet, we have to rely on stages. We ended up with something like this in order to promote builds:

![Build promotion using stages](/img/serverless-what-we-have.png)

Basically, we have code and resources deployed in one AWS account -- prod; and inside this account, code is promoted from stage development to test, and then to production. We are not using the other two AWS accounts -- dev and test.

This practice seems to be working, however, it has one issue. AWS accounts often have different permissions. Dev account is usually set up for developers and has more generous access policies. With these generous policies, developers are able to login to AWS Dev account, and inspect or even change deployed resources (e.g., Lambda functions, and API endpoints). On the contrary, the Prod account will have more restricted access policies. Developers shouldn't be able to login to the Prod account and mess around. This access might be granted to System Admin or Release Engineers. Since we deploy all resources to the Prod account -- the one with strict policies, developers will not have credentials to login to that account, and thus lost the ability to inspect or modify the deployed resources. Certainly, we can grant developers the read permission. There is some workaround we can do. But still it seems that the stage promotion within one AWS account doesn't fit very well with the environment that leverages multiple AWS accounts to promote builds.

What we would love to have is something like this instead:

![Build promotion using accounts](/img/serverless-what-we-need.png)

No stage promotions but cross-account promotions.

I haven't played around the Project Variables -- the newly introduced feature. Not sure if this new feature can be harnessed to solve the multiple account problem. If anyone has experience with this, please let me know :)

<br/>

### 2. Session token support

The AWS SDK supports Session Token -- a token that will expire after a certain period, and will be used together with access ID and secrete access key for AWS authentication. This is extremely useful for maintaining large number of IDs and keys. Since Session Token will simply expire, there is much less need to actively delete unused/obsolete IDs or keys.

The last time I checked, Serverless hasn't incorporated the support for Session Token yet. Would love to have that.

<br/>

## Serverless framework alternatives

Like every framework, it might solve 80% of your problems, but to get the other 20%, things might turn to exponentially difficult, to a point, you even feel like fighting against the framework :)

If you dislike framework that 'dictates' your workflow, here are some alternatives to the Serverless framework.

I already mentioned [node-aws-lambda](https://github.com/ThoughtWorksStudios/node-aws-lambda) from ThoughtWorks studio. It is a Node module to automate the AWS Lambda function deployment. It doesn't include API endpoint deployment. But with [AWS API Gateway Importer](https://github.com/awslabs/aws-apigateway-importer) from AWS lab, you can create or update API endpoints from a Swagger or RAML document.

For editing CloudFormation templates, IntelliJ has a plugin -- [AWS CloudFormation support](https://plugins.jetbrains.com/plugin/7371?pr=idea).

My unfinished [Gulp script project](https://github.com/davidcai/lambda-gulp/blob/master/gulp/lambda.js) has some interesting Gulp tasks to validate and upload CloudFormation templates. Deleting, updating, describing stacks, and tracking stack update progress are also included. I haven't worked on this project for a while, but I believe the majority of the code still work.

<br/>

## Conclusion

Serverless framework is still young, yet it already brought us useful features that simplified the Lambda development. It certainly has room for improvement. The early product feels more limiting in how to organize and name your code assets. The later development has made some parts of this framework more configurable and thus flexible.

There are many times when I questioned myself if we should come up with something else rather than adopting this young and fast changing framework. Our current approach is to stay the course with Serverless. I just feel that it is too early to deviate away from 'the Serverless way'. Like I said, the framework has some good practices baked in.

If anything is missing, the framework authors and contributors are quite diligent to close the gap. There are 44 contributors after all. The other approach is to contribute to the framework by ourselves. This is what we did at Financial Engines -- fork the source, implement a new feature, then submit a pull request. It is a very open-source way to fix issues for ourselves and at the meanwhile evolve the framework.

In the worst scenario, even when we decide to 'stray away' from this Serverless framework, we've already learned a ton along the way. The journey itself is worth it. And at that time, we might have articulated enough intelligence to confidently develop our own solutions. So far, I feel this time is still far away. The framework is at pre-release stage, and constantly improving. I'd recommend checking out its new features, and involving the open source community. 

<br />
