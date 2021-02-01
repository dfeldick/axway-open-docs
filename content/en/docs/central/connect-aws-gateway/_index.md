---
title: Connect AWS Gateway
linkTitle: Connect AWS Gateway
weight: 140
date: 2021-01-04
description: Understand why you would want a connected / managed environment for
  AWS API Gateway within AMPLIFY Central. Learn how you can govern and monitor
  the creation / deployment / publishing and subscriptions of the AWS API
  Gateway hosted APIs in one central location.
---

## Why do you want to connect AWS API Gateway and AMPLIFY Central?

Connecting AWS API Gateway to AMPLIFY Central will provide you with a global centralized view of your APIs and their related traffic.

Each AWS Gateway can be represented by an AMPLIFY Central environment allowing you to better filter APIs and their traffic. Supplied with the environment, two agents (Discovery and Traceability) interact with AWS API Gateway and AMPLIFY Central to:

* Detect changes to AWS API Gateway stages and deployments using the Discovery Agent. The Discovery Agent pushes the service configuration as an API service for the environment, which can then be published as a catalog item to be used by consumers to subscribe to the service.
* Filter the AWS Cloudwatch logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY Platform.

### Discovery Agent

The Discovery Agent is used to discover new deployments and stage updates to existing deployments for publishing related APIs in AMPLIFY Central (either as a catalog item or as an API service). As part of the deployment package, use the provided cloud formation scripts to set up the following agent-dependent AWS Services:

* AWS Config - Administers, audits and monitors resource configurations. Records and validates configuration changes.
* AWS CloudWatch - Monitors resources and AWS applications in real time. Receives and routes supported AWS Service events.
* AWS SQS - Decouples and scales microservices, distributed systems and serverless applications.

![Service Discovery](/Images/central/connect-aws-gateway/aws-discovery-agent_v2.png)

### Traceability Agent

The Traceability Agent is used to filter the AWS CloudWatch logs and prepare the transaction events that are sent to AMPLIFY Central and visible in the API Observer. Viewing your traffic helps you to identify the bottleneck and errors. The traffic can be filtered by environment in case multiples are involved in your topography. As part of the deployment package, you can use cloud formation scripts to set up the following agent-dependent AWS Services:

* AWS CloudWatch - Monitors resources and AWS applications in real time. Receives and routes supported AWS Service events.
* AWS Lamda - Runs code in response to events and automatically manages the computing resources required by that code.
* AWS SQS - Decouples and scales microservices, distributed systems and serverless applications.

The types of logging you can do with API Gateway to CloudWatch:

* Execution logging - API Gateway manages the CloudWatch logs.
* Access logging - Developer managed custom logging.

For additional logging information, see <https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-logging.html>.

![Service Discovery](/Images/central/connect-aws-gateway/aws-traceability-agent_v2.png)

## Minimum requirements

* An Axway AMPLIFY Central subscription in the AMPLIFY™ Platform. See [Get started with AMPLIFY Central](/docs/central/quickstart/).
* An AMPLIFY Central Service Account. See Create Service Account step below.
* API Key credentials on AWS. Allow for CLI access.
* Amazon CloudWatch Service
* Amazon Simple Queue Service (AWS SQS)
* AWS Lambda
* CloudFormation template. Download from [https://axway.jfrog.io/artifactory/ampc-public-generic-release/aws-agents/aws_apigw_agent_config/](<https://axway.jfrog.io/artifactory/ampc-public-generic-release/aws-agents/aws_apigw_agent_config/>).

## Connect AWS API Gateway to AMPLIFY Central using CLI

The following is a high-level overview of the required steps to connect an AWS API Gateway environment to AMPLIFY Central:

* Create a service account for the agent to communicate with AMPLIFY platform
* Create an environment to group the APIs
* Configure AWS services / roles (optional) / Infrastructure (oprional)
* Deploy the agent in the choosen infrastructure (EC2 / ECS-fargate / Docker only)

You will be guided through this procedure using AMPLIFY Central CLI. See [Deploy your agents with AMPLIFY CLI](/docs/central/connect-aws-gateway/deploy-your-agents-with-amplify-cli)
