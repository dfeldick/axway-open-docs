---
title: Prepare AWS API Gateway
linkTitle: Prepare AWS API Gateway
draft: true
weight: null
description: Learn how to use the provided CoudFormation templates to initialize
  AWS Services, which enables the agents to collect API / traffic data.
---
## Before you start

* Read [AMPLIFY Central AWS API Gateway connected overview](/docs/central/connect-aws-gateway/)
* You will need a basic knowledge of Amazon Web Services (AWS) and associated tools

## Objectives

Learn how to configure CloudFormation to initialize the required AWS Services for operating in the continuous discovery mode.

## Download the CloudFormation templates

```
curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/aws-agents/aws_apigw_agent_config/latest/aws_apigw_agent_config-latest.zip" -o aws_apigw_agent_config-latest.zip
```

This zip contains three files:

* `cloudformation-continuous-discovery.zip`: CloudFormation templates for operating in a continuous discovery mode
* `cloudformation-synchronous-discovery.zip`: CloudFormation templates for operating in a synchronous discovery mode
* `traceability_lambda.zip`: a lambda used to parse the log

Extract the files from the `cloudformation-continuous-discovery.zip` archive

## Upload all files to an S3 bucket

Create a new bucket on S3 and upload the following (note the bucket name and URL to the `amplify-agents-deploy-all.yaml` file).

* Upload the yaml files from the `cloudformation-continuous-discovery.zip` archive and the `traceability_lambda.zip` archive to an AWS S3 bucket located on the same region the Cloud Formation template will be deployed.

* For deployment within an EC2 instance, setup the following:

    * Within this bucket, create a folder named **resources** and add the following:

        * **da_env_vars.env** which has all environment variable overrides to run the Discovery Agent
        * **ta_env_vars.env** which has all environment variable overrides to run the Traceability Agent

The bucket file structure should look like the following:

```
[my-bucket-name]
    amplify-agents-deploy-all.yaml
    amplify-agents-ec2.yaml
    amplify-agents-ecs-fargate.yaml
    amplify-agents-resources.yaml
    resources                          **EC2 only**
    |    da_env_vars.env               **EC2 only**
    |    ta_env_vars.env               **EC2 only**
```

## Create an EC2 SSH key (EC2 Only)

An SSH key is required to use the EC2 instance. If you don't have one for the region where the stack will be deployed, you must create one. Once created, use the name of this key as the EC2StackKeyname below. The Cloudformation will assign it to the EC2 instance for being able to connect to it. See [Amazon EC2 key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).

```bash
aws ec2 create-key-pair \
    --key-name MyKeyPair \
    --query 'KeyMaterial' \
    --output text > MyKeyPair.pem
```

## Save AMPLIFY Central keys within AWS Systems Manager Parameter Store (EC2 and ECS)

* Create a secure string parameter for both the private_key.pem and public_key.pem files.

    * [Within the AWS Console](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-create-console.html)
    * [Using the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html)

    ```sh
    aws ssm put-parameter \
        --type "SecureString" \
        --name "AmplifyPrivateKey" \
        --value "$(cat private_key.pem)"

    aws ssm put-parameter \
        --type "SecureString" \
        --name "AmplifyPublicKey" \
        --value "$(cat public_key.pem)"
    ```

* The CloudFormation expects the parameter names to be **AmplifyPrivateKey** and **AmplifyPublicKey** by default.

## Configure all roles and services via CloudFormation

For details on all CloudFormation options, See [Administer AWS Gateway cloud](/docs/central/connect-aws-gateway/cloud-administration-operation).

Create a new stack with the S3 URL of the `amplify-agents-deploy-all.yaml` template. Use the "**With new resources (standard)**" option and configure the parameters:

* Amplify Agents Stack Options

    * Set the **AgentResourcesBucket** value to the S3 Bucket where you saved all of the files.
    * Enable **APIGWCWRoleSetup** to create an IAM role for writing API Gateway logs in CloudWatch.
    * Accept the **APIGWTrafficLogGroupName** default value of `aws-apigw-traffic-logs` to store AWS API Gateway call logs in CloudWatch.
    * Enable **ConfigServiceSetup** to create an IAM role for using Config Service.
    * Set the **ConfigBucketExists** value to `true` to use an existing S3 bucket to store the AWS logs. Be sure the S3 Bucket is accessible and is located in the same region where you run the Cloud Formation template.
    * Set the ConfigBucketName value to the S3 Bucket where you saved all of the files, to store AWS Config logs. When ConfigBucketExists is false the CloudFormation will attempt to create a new bucket using the this name value appending the account id and region to it.
    * Accept the **DiscoveryQueueName** default value of `aws-apigw-discovery` to hold changes made to the API Gateway resources.
    * Accept the **TraceabilityQueueName** default value of `aws-apigw-traceability` to hold the API call so that the Traceability Agent can push them into the AMPLIFY platform.

* Deployment Type

    * Set **DeploymentType** to match how the agents will be deployed for your installation. Quickstart guide is `EC2` which is also the default.

* EC2 Deployment Settings (ignore if **DeploymentType** is **NOT** `EC2`). Suggested values for quickstart.

    * Accept the **EC2InstanceType** default value of `t3.micro` for the type of EC2 instance the agents will run in.
    * Set the **EC2KeyName** to the KeyName you want to use for connecting to the EC2 instance.
 `MyKeyName` is the value from the command above.
    * Accept the **EC2VPCID** default value of blank to create all EC2 resources.
    * Enable **EC2PublicIPAddress** value to assign a public IP to your EC2 instance so it can connect to the Internet and the AMPIFY Platform.
    * Accept the **EC2SSHLocation** default value of `0.0.0.0/0` to allow all SSH connections with a valid key access to the instance.

* ECS Deployment Settings (ignore if **DeploymentType** is **NOT** `ECS Fargate`).

    * Accept the **ECSClusterName** default value of blank, for quickstart.
    * Accept the **ECSCentralOrganizationID** default value of blank, for quickstart.
    * Accept the **ECSCentralEnvironmentName** default value of blank, for quickstart.
    * Accept the **ECSCentralDiscoveryAgentName** default value of blank, for quickstart.
    * Accept the **ECSCentralTraceabilityAgentName** default value of blank, for quickstart.
    * Accept the **ECSCentralURL** default value of blank, for quickstart.
    * Accept the **ECSCentralDeployment** default value of blank, for quickstart.
    * Accept the **ECSCentralTraceabilityHost** default value of blank, for quickstart.
    * Accept the **ECSCentralClientID** default value of blank, for quickstart.

* EC2/ECS Common Deployment Settings (ignore if **DeploymentType** is `Other`). Suggested values for quickstart.

    * Accept the **DiscoveryAgentLogGroupName** default value of `amplify-discovery-agent-logs` to create a log group for the Discovery agent logs.
    * Accept the **TraceabilityAgentLogGroupName** default value of `amplify-traceability-agent-logs` to create a log group for the Traceability agent logs.
    * Set the **SSMPrivateKeyParameter** to the name of the Parameter holding the private_key.pem contents. `AmplifyPrivateKey` by default.
    * Set the **SSMPublicKeyParameter** to the name of the Parameter holding the public_key.pem contents. `AmplifyPublicKey` by default.
    * Accept the **SecurityGroup** default value of blank, for quickstart.
    * Accept the **Subnet** default value of blank, for quickstart.

Execute the stack creation. The following is an example of running it with the AWS CLI.

Replace values in **[[BRACKETS]]**

```bash
aws cloudformation create-stack \
    --stack-name amplify-agents \
    --template-url [[S3 URL to amplify-agents-deploy-all.yaml]] \
    --capabilities CAPABILITY_IAM  CAPABILITY_AUTO_EXPAND \
    --parameters '[
                    {"ParameterKey": "AgentResourcesBucket",            "ParameterValue": "[[MY-BUCKET-NAME]]"},
                    {"ParameterKey": "APIGWCWRoleSetup",                "ParameterValue": "true"},
                    {"ParameterKey": "APIGWTrafficLogGroupName",        "ParameterValue": "aws-apigw-traffic-logs"},
                    {"ParameterKey": "ConfigServiceSetup",              "ParameterValue": "true"},
                    {"ParameterKey": "ConfigBucketName",                "ParameterValue": "[[MY-BUCKET-NAME]]"},
                    {"ParameterKey": "ConfigBucketExists",              "ParameterValue": "true"},
                    {"ParameterKey": "DiscoveryQueueName",              "ParameterValue": "aws-apigw-discovery"},,
                    {"ParameterKey": "TraceabilityQueueName",           "ParameterValue": "aws-apigw-traceability"},
                    {"ParameterKey": "DeploymentType",                  "ParameterValue": "EC2"},
                    {"ParameterKey": "EC2InstanceType",                 "ParameterValue": "t3.micro"},
                    {"ParameterKey": "EC2KeyName",                      "ParameterValue": "[[MY EC2 SSH KEY NAME]]"},
                    {"ParameterKey": "EC2VPCID",                        "ParameterValue": ""},
                    {"ParameterKey": "EC2PublicIPAddress",              "ParameterValue": "true"},
                    {"ParameterKey": "EC2SSHLocation",                  "ParameterValue": "0.0.0.0/0"},
                    {"ParameterKey": "ECSClusterName",                  "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralOrganizationID",        "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralEnvironmentName",       "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralDiscoveryAgentName",    "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralTraceabilityAgentName", "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralURL",                   "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralDeployment",            "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralTraceabilityHost",      "ParameterValue": ""},
                    {"ParameterKey": "ECSCentralClientID",              "ParameterValue": ""},
                    {"ParameterKey": "DiscoveryAgentLogGroupName",      "ParameterValue": "amplify-discovery-agent-logs"},
                    {"ParameterKey": "TraceabilityAgentLogGroupName",   "ParameterValue": "amplify-traceability-agent-logs"},
                    {"ParameterKey": "SSMPrivateKeyParameter",          "ParameterValue": "AmplifyPrivateKey"},
                    {"ParameterKey": "SSMPublicKeyParameter",           "ParameterValue": "AmplifyPublicKey"},
                    {"ParameterKey": "SecurityGroup",                   "ParameterValue": ""},
                    {"ParameterKey": "Subnet",                          "ParameterValue": ""}
                  ]'
```

With EC2 or ECS deployments, your agents will have been deployed and started, respectively. Go to AWS CloudWatch to see the logs.

With Other deployments, you are ready to [deploy agents](/docs/central/connect-aws-gateway/deploy-your-agents-1).