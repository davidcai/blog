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

For people new to Lambda, the amount of concepts introduced by Serverless seems to be overwhelming -- CloudFormation templates, Serverless's S3 bucket, stages, Lambda function configurations, and Serverless' commands. At the early stage of JAWS (Serverless' previous brand name) development, I even found the framework is opinionated in some ways. So I tried to come up with my own solution -- [a collection of gulp tasks](https://github.com/davidcai/lambda-gulp) that automates CloudFormation and Lambda function deployment. The funny thing is that during the development of these Gulp scripts, I start to have better understanding of the Lambda development workflow, and appreciate more of what Serverless is trying to achieve. Understanding the aforementioned concepts are actually crucial to develop scalable Lambda projects. I stopped working on my own solution, just before JAWS had a major release which changed the brand name to Serverless, and brought in many needed features. And from there, I picked up Serverless again.

The great thing about Serverless framework is that it incorporates many best practices that might seem to be over complicate for simple projects, but will exponentially accelerate the development cycle for mid or large projects. Sure, there are many other tools and plugins that support Lambda development and deployment, such as Jenkins' Lambda plugin, and ThoughtWorks' node module [node-aws-lambda](https://github.com/ThoughtWorksStudios/node-aws-lambda). But so far, Serverless is still the only one framework that handles the full development cycle of Lambda, API Gateway, and other AWS resources. And the Serverless CLI can be integrated with CICD pipelines such as Jenkins (which we do in FE) that further automates the workflow. I found Serverless is especially useful in the following areas:

## Local Development



## Scaffolding


##

## Stages

## CloudFormation



<br />
