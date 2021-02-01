---
title: Deploy your agents with AMPLIFY CLI
linkTitle: Deploy your agents with AMPLIFY CLI
draft: false
weight: 20
description: Learn how to deploy your agents using AMPLIFY CLI so that you can
  manage your Axway API Gateway environment within AMPLIFY Central.
---
## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* You will need a basic knowledge of Axway API Management installation

    * Where the API Gateway is running (host / port / path to event logs)
    * Where the Admin node manager is running (host / port)
    * What users are available for the agent to use. It is recommended to have one API Manager user for Discovery Agent and one API Gateway operator user for Traceability Agent.

## Objectives

Learn how to quickly install and run your Discovery and Traceability agents with basic configuration using AMPLIFY Central CLI.

## AMPLIFY Central CLI pre-requisites

* Node.js 8 LTS or later
* Access to npm package (for installing AMPLIFY cli)
* Access to login.axway.com on port 443
* Minimum AMPLIFY Central CLI version: 0.1.15 (check version using `amplify central --version`)

More information is available at [Install AMPLIFY Central CLI](/docs/central/cli_central/cli_install/).

## Install the agents

### Step 1: Folder preparation

Create an empty directory where AMPLIFY CLI will generate files. Run all AMPLIFY Central CLI from this directory.

### Step 2: Identify yourself to AMPLIFY Platform with AMPLIFY CLI

To use Central CLI to log in with your AMPLIFY Platform credentials, run the following command:

```shell
amplify auth login
```

A browser will automatically open.
Enter your valid credentials (email address and password). Once the “Authorization Successful” message is displayed, go back to AMPLIFY CLI.

If you are a member of multiple AMPLIFY organizations, you may have to choose an organization.

{{< alert title="Note" color="primary" >}}If you do not have a graphical environment, forward the display to an X11 server (Xming or similar tools) using the `export DISPLAY=myLaptop:0.0` command .{{< /alert >}}

### Step 3: Run the agents' install procedure

Agents will be installed in the directory from where the CLI runs. You can install the agent from anywhere, but then you must transfer the agent and its configuration to the API Management system machine for the agent to operate correctly.

```shell
amplify central install agents
```

If your AMPLIFY subscription is hosted in the EU region, then the following installation command must be used to correctly configure the agents:

```shell
amplify central install agents --region=EU
```

The installation procedure will prompt for the following:

1. Select the type of gateway you want to connect to (V7 gateway in this scenario).
2. Platform connectivity:
   * **environment**: can be an existing environment or a new one that will be created by the installation procedure
   * **team**: can be an existing team or a new one that will be created by the installation procedure
   * **service account**: can be an existing service account or a new one that will be created by the installation procedure. If you choose an existing one, be sure you have the appropriate public and private keys, as they will be required for the agent to connect to the AMPLIFY Platform. If you choose to create a new one, the generated private and public keys will be provided.
3. Select the agents you want to install: Discovery / Traceability / all.
4. Select the agent deployment mode: binary / Docker image.
5. API Manager connectivity:
   * **hostname** of the API Manager (localhost by default)
   * **port** of the API Manager (8075 by default)
   * user/password
6. API Gateway connectivity:
   * **hostname** of the API Gateway (localhost by default)
   * **port** of the API Gateway (8090 by default)
   * user/password
   * event log path
7. Traceability module connectivity:
   * Traceability Agent protocol (**Lumberjack** (tcp) by default recommended for production environment or **HTTPs** recommended for testing purpose)

Once you have answered all questions, the agents are downloaded, the configuration files are updated, the Amplify Central resources are created and the key pair is generated (if you chose to create a new service account).

The current directory should contain the following files:

```shell
discovery_agent           *Binary deployment only
discovery_agent.yml       *Binary deployment only
traceability_agent        *Binary deployment only
traceability_agent.yml    *Binary deployment only
da_env_vars.env
ta_env_vars.env
private_key.pem           *Only present if a new service account is created
public_key.pem            *only present if a new service account is created
```

`da_env_vars.env` / `ta_env_vars.env` contains the specific configuration you entered during the installation procedure.

`discovery_agent.yml` and `traceability_agent.yml` contain the default minimum agent configuration.

`private_key.pem` and `public_key.pem` are the generated key pair the agent will use to securely talk with the AMPLIFY Platform (if you choose to let the installation generate them).

## Start the agents

### Binary mode

As mentioned in the installation procedure, agents can be started with the following commands:

Discovery Agent:

```shell
./discovery_agent --envFile ./da_env_vars.env
```

Traceability Agent:

```shell
./discovery_agent --envFile ./ta_env_vars.env
```

### Docker mode

As mentioned in the installation procedure, agents can be started with the following commands:

Discovery Agent:

```shell
docker run -it --env-file $(pwd)/da_env_vars.env -v $(pwd):/keys docker run -it axway.jfrog.io/ampc-public-docker-release/agent/v7-discovery-agent:latest
```

Traceability Agent:

```shell
docker run -it --env-file $(pwd)/ta_env_vars.env -v $(pwd):/keys -v EVENT_LOG_PATH_ENTERED_DURING_INSTALLATION:/events docker run -it axway.jfrog.io/ampc-public-docker-release/agent/v7-traceability:latest
```

### Linux Service mode for binary agent

The agent can be installed as a Linux service with systemd. The following commands will help you utilize the service. These commands install the service abilities and must be run as a root user.

When running as a service, it is best to save your logging to a file rather than the console output. See [Customizing log section (log)](/docs/central/connect-api-manager/gateway-administration#customizing-log-section-log).

* Install the services and execute it as user axway and group axway:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service install -u axway -g axway --envFile /path/to/da_env_file.env
  sudo ./traceability_agent service install -u axway -g axway --envFile /path/to/ta_env_file.env
  ```

* Start the services:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service start
  sudo ./traceability_agent service start
  ```

* Read service logs, since the machine last booted:

  ```shell
  cd /home/APIC-agents
  ./discovery_agent service logs
  ./traceability_agent service logs
  ```

* Stop the services:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service stop
  sudo ./traceability_agent service stop
  ```

* Enable the services to start when the machine starts:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service enable
  sudo ./traceability_agent service enable
  ```

* Get service name:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service name
  sudo ./traceability_agent service name
  ```

* Uninstall the services from the machine:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service stop   # to ensure it is not running
  sudo ./discovery_agent service remove
  sudo ./traceability_agent service stop   # to ensure it is not running
  sudo ./traceability_agent service remove
  ```

## Check that agents are running

### Use the agent status command

Discovery agent:

  ```shell
  cd /home/APIC-agents
  ./discovery_agent --status
  ```

An empty result means the agent is not running; otherwise, you should receive this result:

```shell
/home/APIC-agents$ ./discovery_agent --status
{
  "name": "discovery_agent",
  "version": "0.0.19-658ebd93",
  "status": "OK",
  "statusChecks": {
    "apimanager": {
      "name": "API Manager",
      "endpoint": "apimanager",
      "status": {
        "result": "OK"
      }
    },
    "central": {
      "name": "AMPLIFY Central",
      "endpoint": "central",
      "status": {
        "result": "OK"
      }
    }
  }
}
```  

Traceability agent:

  ```shell
  cd /home/APIC-agents
  ./traceability_agent --status
  ```

An empty result means the agent is not running; otherwise, you should receive this result:

```shell
/home/APIC-agents$ ./traceability_agent status
{
  "name": "traceability_agent",
  "version": "0.0.19-658ebd93",
  "status": "OK",
  "statusChecks": {
    "apimanager": {
      "name": "API Manager",
      "endpoint": "apimanager",
      "status": {
        "result": "OK"
      }
    },
    "apigateway": {
      "name": "API Gateway",
      "endpoint": "apigateway",
      "status": {
        "result": "OK"
      }
    },
    "central": {
      "name": "AMPLIFY Central",
      "endpoint": "central",
      "status": {
        "result": "OK"
      }
    }
  }
}
```

### Use the agent service status command

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service status
  sudo ./traceability_agent service status
  ```

### Use AMPLIFY Central CLI

After being authenticated to the platform with `amplify auth login` command, run the following:

* `amplify central get edgeda` to get all discovery agent information.
* `amplify central get edgeta` to get all traceability agent information.

The STATUS column will help you identify which agent is running.

```shell
C:\Demos>amplify central get edgeda
√ Resource(s) successfully retrieved

NAME                                       STATUS   AGE             SCOPE KIND   SCOPE NAME
EdgeDiscoveryAgent/cli-1607014956109       started  26 minutes ago  Environment  pre-prod
EdgeDiscoveryAgent/lbean018-discovery      stopped  2 months ago    Environment  prod
```

```shell
C:\Demos>amplify central get edgeta
√ Resource(s) successfully retrieved

NAME                                             STATUS   AGE             SCOPE KIND   SCOPE NAME
EdgeTraceabilityAgent/cli-1607014956731          started  30 minutes ago  Environment  pre-prod
EdgeTraceabilityAgent/lbean018-traceability      stopped  2 months ago    Environment  prod
```

See [Administer API Gateway](/docs/central/connect-api-manager/gateway-administation/) for additional information about agent features.
