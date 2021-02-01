---
title: Deploy your agents - quickstart
linkTitle: Deploy your agents - quickstart
draft: true
weight: 25
description: Learn how to quickly deploy your Discovery Agent and Traceability
  Agent so that you can manage your Axway API Gateway environment within AMPLIFY
  Central.
---
## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* Be sure you have [Prepared AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/)
* You will need a basic knowledge of Axway API Management solution

    * Where the solution is running (host / port / path to the logs / users)
    * How to create / publish an API
    * How to call an API

## Objectives

Learn how to quickly install and run your Discovery and Traceability agents with basic configuration.

## 1. Create environment variable file

Prepare an `env_vars_agents` file with the following content:

```
#
#API Manager connectivity
#
APIMANAGER_HOST=localhost
APIMANAGER_PORT=8075
APIMANAGER_AUTH_USERNAME=<apiManagerUser>
APIMANAGER_AUTH_PASSWORD=<apiManagerUserPassword>
#
#API Gateway connectivity
#
APIGATEWAY_HOST=localhost
APIGATEWAY_PORT=8090
APIGATEWAY_AUTH_USERNAME=<apiGatewayOperatorUser>
APIGATEWAY_AUTH_PASSWORD=<apiGatewayOperatorUserPassword>
#
# API GATEWAY EVENT LOG PATH
#
# When multiple log paths are defined, either separate each path with a comma or use the wildcard to regroup them all in a single value.
EVENT_LOG_PATHS=<apiGatewayInstallDir>/events/group-*_instance-*.log
#
#AMPLIFY Central connectivity
#
CENTRAL_ORGANIZATIONID=<YOUR ORGANIZATION ID>
CENTRAL_ENVIRONMENT=<NAME OF THE CENTRAL TOPOLOGY ENVIRONMENT>
CENTRAL_AUTH_CLIENTID=<SERVICE ACCOUNT NAME: DOSA_xxxxxxxxx>
```

The required values represented in `<>` are either coming from the API Management system installation or from AMPLIFY platform configuration.
An explanation for each variable can be found in the [Agent variables section](/docs/central/connect-api-manager/agent-variables/).

## 2. Download Discovery Agent

```shell
curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_discovery_agent/latest/discovery_agent-latest.zip" -o discovery_agent-latest.zip
```

Unzip the file `discovery_agent-latest.zip` to get the agent binary (`discovery_agent`) and the template configuration (`discovery_agent.yml`).

## 3. Start the Discovery Agent

Before starting the agent, you need to copy the private/public key into the same folder as the agent. You also need to copy the `env_vars_agents` file into the agent directory. Then execute the following command:

```shell
./discovery_agent --envFile ./env_vars_agents
```

If all parameters are corrects, the following message should be displayed:

```shell
{"level":"info","msg":"Starting Discovery agent for V7 APIGateway (0.0.10-5ee9dca6)","time":"2020-08-01T01:58:30+02:00"}
{"level":"info","msg":"Services are Ready","time":"2020-08-01T01:58:32+02:00"}
```

And the agent should start discovering already published APIs.

## 4. Download Traceability Agent

```shell
curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_traceability_agent/latest/traceability_agent-latest.zip" -o traceability_agent-latest.zip
```

Unzip the file `traceability_agent-latest.zip` to get the agent binary (`traceability_agent`) and the template configuration (`traceability_agent.yml`) into the same folder as the discovery agent.

## 5. Start the Traceability Agent

```
./traceability_agent --envFile ./env_vars_agents
```

If all parameter are correct, the following message should be displayed:

```shell
2020-08-01T02:14:10.696+0200    INFO    instance/beat.go:297    Setup Beat: traceability_agent; Version: 0.0.10-48172a35
2020-08-01T02:14:54.894+0200    INFO    instance/beat.go:438    traceability_agent start running.
```

And the traceability agent should start publishing the logs in AMPLIFY platform.
