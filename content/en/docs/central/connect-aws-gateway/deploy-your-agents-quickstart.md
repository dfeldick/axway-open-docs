---
title: Deploy agents - quickstart
linkTitle: Deploy your agents - quickstart
draft: true
weight: null
description: >-
  Learn how to deploy your Discovery Agent and Traceability Agent using Docker
  containers so that you can manage your AWS API Gateway environment within
  AMPLIFY Central.

  Once agents are correctly deployed, they can collect the data from the AWS API Gateway and send it securely to AMPLIFY Central.
---
## Before you start

* Read [AMPLIFY Central AWS API Gateway connected overview](/docs/central/connect-aws-gateway/)

## Objectives

Learn the basics to create your Discovery Agent and Traceability Agent configuration files.  Then deploy and run your agents using an EC2 instance.

## Connect AWS API Gateway to AMPLIFY Central quickstart (With EC2 instance)

### 1. Set up AMPLIFY Central service account

* The agent will use a service account for the secure communication with the AMPLIFY Platform.
* Generate a public/private key pair:
    ```
    openssl genpkey -algorithm RSA -out ./private_key.pem -pkeyopt rsa_keygen_bits:2048
    openssl rsa -pubout -in ./private_key.pem -out ./public_key.pem
    ```
* Create a new Service Account user in AMPLIFY Central using the `public_key.pem` from above. You may name this Service Account (for example, v7-Agent). For additional information, see [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).

### 2. Set up AMPLIFY Central environment

* Create an environment object in AMPLIFY Central that represents the effective AWS Gateway environment. Depending on your needs, you can create as many environments as required.
* Each discovered API or Traffic is associated to this environment and eases the filtering.
* Add your environment to AMPLIFY Central using either the [AMPLIFY Central CLI](/docs/central/cli_central/cli_environments/) or [the UI](/docs/central/connect-api-manager/prepare-amplify-central/#create-environment-using-the-ui).

### 3. Create the agent environment files

* Create a Discovery Agent environment file

    * Prepare an `da_env_vars.env` file with the following content:

    ```
    # AWS connectivity
    AWS_REGION=<AWS region where API are located>
    AWS_QUEUENAME=aws-apigw-discovery
    AWS_LOGGROUP=aws-apigw-traffic-logs
    AWS_FILTER=tag.publishToAmplify == true

    # AMPLIFY Central connectivity
    CENTRAL_ORGANIZATIONID=<YOUR ORGANIZATION ID>
    CENTRAL_ENVIRONMENT=<NAME OF THE CENTRAL TOPOLOGY ENVIRONMENT>
    CENTRAL_AUTH_CLIENTID=<SERVICE ACCOUNT NAME: DOSA_xxxxxxxxx>
    ```

    * Learn more about the [Discovery Agent](/docs/central/connect-aws-gateway/deploy-your-agents-1/#discovery-agent).

* Create a Traceability Agent environment file

    * Prepare an `ta_env_vars.env` file with the following content:

    ```
    # AWS connectivity
    AWS_REGION=<AWS region where API are located>
    AWS_QUEUENAME=aws-apigw-traceability

    # AMPLIFY Central connectivity
    CENTRAL_ORGANIZATIONID=<YOUR ORGANIZATION ID>
    CENTRAL_ENVIRONMENT=<NAME OF THE CENTRAL TOPOLOGY ENVIRONMENT>
    CENTRAL_AUTH_CLIENTID=<SERVICE ACCOUNT NAME: DOSA_xxxxxxxxx>
    ```

    * Learn more about the [Traceability Agent](/docs/central/connect-aws-gateway/deploy-your-agents-1/#traceability-agent).

### 4. Setup using AWS CloudFormation

* Get the required templates at [https://axway.jfrog.io/artifactory/ampc-public-generic-release/aws-agents/aws_apigw_agent_config/](<https://axway.jfrog.io/artifactory/ampc-public-generic-release/aws-agents/aws_apigw_agent_config/>).
* Setup and deploy the CloudFormation Stack [AWS CloudFormation](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/#set-up-the-cloudformation).
