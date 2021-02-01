---
title: Administer API Manager Gateway
linkTitle: Administer API Manager Gateway
draft: false
weight: 25
description: Learn how to deploy your Discovery Agent and Traceability Agent so
  that you can manage your Axway API Gateway environment within AMPLIFY Central.
---
## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* Be sure you have [Prepared AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/)
* You will need a basic knowledge of Axway API Management solution:

    * Where the solution is running (host / port / path to the logs / users)
    * How to create / publish an API
    * How to call an API
* For containerized agents, Docker must be installed and you will need a basic understanding of Docker commands

## Objectives

Learn how to install, customize and run your Discovery and Traceability agents.

## Discovery Agent

The Discovery Agent is used to discover new published APIs or any updated APIs. Once they are discovered, the related APIs are published to AMPLIFY Central, in one of the following publication modes:

* **Environment / API Service publication**: Customers publish their APIs to the AMPLIFY platform.
* **Environment / API Service publication / Catalog item publication** (default mode): Same as previous plus automatically expose the APIS to the consumer via the AMPLIFY Catalog.

The Discovery Agent only discovers APIs that have the tag(s) defined in the agent configuration file. See [Discover APIs](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). By default, the filter is empty and thus the agent will discover all published API.

The binary agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - `discovery_agent.yml`:

    * Default: located in the same directory as the agent binary.
    * Optional: use a dedicated folder where the configuration file is located (use the --pathConfig flag in the agent command line to access the file path).
    * Advanced configuration: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Discovery Agent variables](/docs/central/connect-api-manager/agent-variables/).
* With command line arguments. See [Discovery Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

The containerized agent can run in the following mode:

* With an environment variables configuration file, `env_vars`, supplied as a command line argument when running the Docker image.

### Installing the Discovery Agent

#### To install the Binary Discovery Agent

**Step 1**: Download the latest version of the zip file from the Axway public repository using the following command:

```shell
curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_discovery_agent/latest/discovery_agent-latest.zip" -o discovery_agent-latest.zip
```

**Step 2**: Unzip the file discovery_agent-latest.zip to get the agent binary (discovery_agent) and a template configuration file (discovery_agent.yml).

```shell
unzip discovery_agent-latest.zip
```

**Step 3**: Copy those 2 files into a folder (/home/APIC-agents for instance) on the machine where the API Manager environment is located.

**Step 4**: Move the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to the agent directory (APIC-agents). Note that the `public_key.pem` comes from Steps 3 or 4 of [Create a Service Account](/docs/central/connect-api-manager/prepare-amplify-central/#create-a-service-account) depending if you choose to use the `der` format or not.

#### To install the Dockerized Discovery Agent

Create your Discovery Agent environment file, `env_vars`. See [Discovery Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable descriptions.
After customizing all the sections, your `env_vars` file should look like this example file:

```shell
# API MANAGER connectivity
APIMANAGER_HOST=<HOST>
APIMANAGER_AUTH_USERNAME=<USER>
APIMANAGER_AUTH_PASSWORD=<PASSWORD>

# API GATEWAY connectivity
APIGATEWAY_HOST=<HOST>
APIGATEWAY_AUTH_USERNAME=<USER>
APIGATEWAY_AUTH_PASSWORD=<PASSWORD>

# AMPLIFY connectivity
CENTRAL_ORGANIZATIONID=<ORGANIZATIONID>
CENTRAL_TEAM=<TEAM>
CENTRAL_AUTH_CLIENTID=<CLIENTID, ie. DOSA_12345...>
```

* The value for *team* can be found in [AMPLIFY Central > Access > Team Assets](https://apicentral.axway.com/access/teams/).
* The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
* The value for *clientId* can be found in Service Account. See [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).

Pull the latest image of the Discovery Agent:

```shell
docker pull axway.jfrog.io/ampc-public-docker-release/agent/v7-discovery-agent:latest
```

### Customizing the Discovery Agent configuration file

The `discovery_agent.yml` configuration file contain 3 sections to personalize: apimanager, central and log.

#### Customizing apimanager section

This section connects the agent to API Manager and determines which APIs should be discovered and published to AMPLIFY Central.

`host`: Machine name where API Manager is running. Use a hostname according to the certificate returned by the API-Manager.

`port`: API Manager port number (8075 by default).

`discoveryIgnoreTags` (optional): Comma-separated blacklist of tags. If an API has one or several of these blacklist tags, the agent ignores this API and will not publish it to AMPLIFY Central. This property takes precedence over the filter property below. The default value is empty, which means no API is ignored.

`filter` (optional): Expression to filter the API you want the agent to discover. See [Discover APIs](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). Leaving this field empty tells the agent to discover all published APIs (REST / SOAP).

`subscriptionApplicationField` (optional): The field name used to store AMPLIFY Central subscription identifier inside the API Manager application securing the front end proxy. Default value is **subscriptions**. If you do not intend to change it, comment this property. Be aware that the field will not be visible in the API Manager application, as it is a specific configuration. If you want to see that field or customize it, refer to Add a custom property to applications in [Customize API Manager](/docs/apim_administration/apimgr_admin/api_mgmt_custom/#customize-api-manager-data/) documentation.

`pollInterval`: The frequency in which API Manager is polled for new endpoints. Default value is 30s.

`allowApplicationAutoCreation` (optional): When creating a subscription on AMPLIFY Central, setting this value to true will enable a selection in the App name dropdown for 'Create an application.' This allows the user to either select from an existing API Manager application, or to create a new application in API Manager. The new application in API Manager will be given the name of the subscription ID from AMPLIFY Central. A value of false will cause 'Create an application' to not be shown in the dropdown. Default value is **false**.

`auth.username`: An API Manager user the agent will use to connect to the API Manager. This user must have either the “API Manager Administrator” or “Organization administrator” role. Based on the role of this user, the agent is able to:

* discover any API from any organization (“API Manager Administrator”)  
* discovery any API from a specific organization (“Organization administrator”)

`auth.password`: The password of the API Manager user in clear text.

Once all data is gathered, this section should looks like:

```yaml
apimanager:
  host: localhost
  port: 8075
  discoveryIgnoreTags: tag1, tag2
  filter:
#  subscriptionApplicationField: subscriptions
  pollInterval: 30s
  allowApplicationAutoCreation: true
  auth:
    username: apiManagerUser
    password: apiManagerUserPassword
```

#### Customizing central section

This section connects the agent to AMPLIFY Central and determines how to published the discovered APIs.

`url`: The AMPLIFY Central url. Default value is **<https://apicentral.axway.com>**.

`platformURL:` The AMPLIFY platform url. Needed for finding the user email during the subscription process with email notification. Default value is **<https://platform.axway.com>**.

`team`: The Team name in AMPLIFY Central that all APIs will be linked to. Locate this at AMPLIFY Central > Access > Team Assets.).

`organizationID`: The Organization ID from AMPLIFY Central. Locate this at Platform > User > Organization > Org ID field.

`environment`: The environment name you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/).

`apiServerVersion`: The version of AMPLIFY Central API the agent is using. Default value is **v1alpha1**.

`mode`: The method to send endpoints back to Central. (publishToEnvironment = API Service, publishToEnvironmentAndCatalog = API Service and Catalog asset).  

`pollInterval`: The frequency the agent is polling AMPLIFY Central to get some event. Default value is **30s**.

`auth.url`: The AMPLIFY login URL. Default value is **<https://login.axway.com/auth>**.

`auth.realm`: The Realm used to authenticate for AMPLIFY Central. Default value is **Broker**.

`auth.clientId`: The Client ID of the Service Account (DOSA_....) you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Locate this at AMPLIFY Central > Access > Service Accounts.

`auth.privateKey`: The location of the private key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.

`auth.publicKey`:  The location of the public key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.  

`auth.keyPassword`: The key password to open the key. None set up by default.

`auth.timeout`: Timeout for the authentication. Default value is **10s**.

Once all data is gathered, this section should look like:

```yaml
central:
  url: https://apicentral.axway.com
  platformURL: https://platform.axway.com
  team: Dev
  organizationID: 68794y2
  environment: my-v7-env
  apiServerVersion: v1alpha1
  mode: publishToEnvironmmentAndCatalog
  pollInterval:
  auth:
    url: https://login.axway.com/auth
    realm: Broker
    clientId: DOSA_66743...
    privateKey: /home/APIC-agents/private_key.pem
    publicKey: /home/APIC-agents/public_key.pem
    keyPassword:
    timeout: 10s
```

#### Customizing SMTP Notification (subscription)

The SMTP Notification section defines how the agent manages email settings for subscriptions.

`host`: SMTP server where the email notifications will originate from.

`port`: Port of the SMTP server.

`fromAddress`: Email address which will represent the sender.

`username`: Login user for the SMTP server.

`password`: Login password for the SMTP server.

`authtype`: The authentication type based on the email server.  You may have to refer to the email server properties and specifications. This value defaults to NONE.

`subscribe.subject`: Subject of the email notification for action subscribe. Default is `Subscription Notification`.

`subscribe.body`: Body of the email notification for action subscribe. Default is `Subscription created for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a><br/>${authtemplate}<br/>`.

`subscribe.oauth:` Body of the email notification for action subscribe on OAuth authorization is `Your API is secured using OAuth token. You can obtain your token using grant_type=client_credentials with the following client_id=<b>${clientID}</b> and client_secret=<b>${clientSecret}</b>`.

`subscribe.apikeys`: Body of the email notification for action subscribe on APIKey authorization is `Your API is secured using an APIKey credential: header: <b>${keyHeaderName}</b> / value: <b>${key}</b>`.

`unsubscribe.subject`: Subject of the email notification for action unsubscribe. Default is `Subscription Removal Notification`.

`unsubscribe.body`: Body of the email notification for action unsubscribe. Default is `Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a> has been unsubscribed`.

`subscribeFailed.subject`: Subject of the email notification for action subscribe failed. Default is `Subscription Failed Notification`.

`subscribeFailed.body`: Body of the email notification for action subscribe failed. Default is `Could not subscribe to CatalogItem: <a href= ${catalogItemUrl}> ${catalogItemName}</a>`.

`unsubscribeFailed.subject`: Subject of the email notification for action unsubscribe failed. Default is `Subscription Removal Failed Notification`.

`unsubscribeFailed.body` : Body of the email notification for action unsubscribe failed. Default is `Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a>`.

#### Customizing email servers

The `host`, which represents the email server, can be configured with minimal setup.  This section represents the email servers that have been currently tested. Please note, that all testing has been set up on port 587 signifying TLS support.  

```
# Google/Gmail server
host: smtp.gmail.com
port: 587
username: your GMAIL account
password: application generated GMAIL password (see note below)
authtype: PLAIN

# Microsoft office server
host: smtp.office365.com
port: 587
username: your Office Mail account
password: your Office Mail password
authtype: LOGIN

# Microsoft outlook server
host: smtp-mail.outlook.com
port: 587
username: your Outlook Mail account
password: your Outlook Mail password
authType: PLAIN

# Yahoo email server
host: smtp.mail.yahoo.com
port: 587
username: your Yahoo Mail account
password: application generated Yahoo password (see note below)
authtype: PLAIN

# Amazon Simple Email Service
host: email-smtp.<region>.amazonaws.com
port: 587
username: user access key (see note below)
password: user secret key (see note below)
authtype: PLAIN
```

**Note**: You will be required to use an application generated password instead of the actual user email password for the following email servers. Follow the links for application generated passwords.

* Gmail - [Application generated gmail password](https://support.google.com/accounts/answer/185833?hl=en). Use this password in place of your actual password in the agent configuration `password:` field.
* Yahoo - [Application generated yahoo password](https://help.yahoo.com/kb/generate-third-party-passwords-sln15241.html). Use this password in place of your actual password in the agent configuration `password:` field.
* [AWS  Simple email service](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-smtp.html). Create your [SMTP credentials](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/smtp-credentials.html) and use them in the username (ACCESS KEY) and password (SECRET KEY) of the agent configuration.

### Customizing Webhook Notification (subscription)

The webhook Notification section defines how the agent manages to send notifications to a webhook URL.

`url`: url where the webhook server is defined.

`headers`: information used to verify the webhook. Provided by the customer, and may include such information as contentType and Authorization.

Both webhook and smtp  sections can be configured at the same time.  The agent will attempt the subscription notifications that are set in the agent config.

Once all data is gathered, this section should look like this for subscription Notification:

```yaml
subscriptions:
  notifications:
    webhook:
      url:
      headers:
    smtp:
      host: mail.outlook.com
      port: 25
      fromAddress: fromAddress@outlook.com
      username: outlookuser
      password:
```

If you want to customize your SMTP email notifications to be something different than the defaults, you can add to the configuration file as follows:

```yaml
subscriptions:
  notifications:
    smtp:
      subscribe:
        subject: Subscription Notification
        body: |
          Subscription created for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a><br/>${authtemplate}<br/>
          ${authtemplate}<br/>
        oauth: Your API is secured using OAuth token. You can obtain your token using grant_type=client_credentials with the following client_id=<b>${clientID}</b> and client_secret=<b>${clientSecret}</b>
        apikeys: Your API is secured using an APIKey credential: header: <b>${keyHeaderName}</b> / value: <b>${key}</b>
      unsubscribe:
        subject: Subscription Removal Notification
        body: |
          Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a> has been unsubscribed
      subscribeFailed:
        subject: Custom Subscription Failed Notification
        body: |
          Could not subscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a> Error: ${message}
      unsubscribeFailed:
        subject: Custom Subscription Removal Failed Notification
        body: |
          Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a>
```

#### Customizing log section (log)

The log section defines how the agent is managing its logs.

`level`: The log level for output messages (debug, info, warn, error). Default value is **info**.

`format`: The format to print log messages (json, line, package). Default value is **json**.

`output`: The output for the log lines (stdout, file, both). Default value is **stdout**.

`maskedValues` : Comma-separated list of keywords to identify within the agent config, which is used to mask its corresponding sensitive data. Keywords are matched by whole words and are case-sensitive. Displaying of agent config to the console requires that the log.level be at debug (level: debug).

Once all data is gathered, this section should look like:

```yaml
log:
  level: info
  format: json
  output: stdout
  maskedValues: keyword1, keyword2, keyword3
```

See [Set up agent configuration](/docs/central/connect-api-manager/agent-variables/) for more options.

#### Validating your custom Discovery Agent configuration file

After customizing all the sections, your discovery_agent.yml file should look like:

```yaml
apimanager:
  host: localhost
  port: 8075
  discoveryIgnoreTags: tag1, tag2
  filter:
#  subscriptionApplicationField: subscriptions
  pollInterval: 30s
  allowApplicationAutoCreation: true
  auth:
    username: apiManagerUser
    password: apiManagerUserPassword

central:
  url: https://apicentral.axway.com
  platformURL: https://platform.axway.com
  team: Dev
  organizationID: 68794y2
  environment: my-v7-env
  apiServerVersion: v1alpha1
  mode: publishToEnvironmmentAndCatalog
  pollInterval:
  auth:
    url: https://login.axway.com/auth
    realm: Broker
    clientId: DOSA_66743...
    privateKey: /home/APIC-agents/private_key.pem
    publicKey: /home/APIC-agents/public_key.pem
    keyPassword:
    timeout: 10s

subscriptions:
  notifications:
    webhook:
      url:
      headers:
    smtp:
      host: mail.outlook.com
      port: 25
      fromAddress: fromAddress@outlook.com
      username: outlookuser
      password:
      subscribe:
        subject: Subscription Notification
        body: |
          Subscription created for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a><br/>${authtemplate}<br/>
          ${authtemplate}<br/>
        oauth: Your API is secured using OAuth token. You can obtain your token using grant_type=client_credentials with the following client_id=<b>${clientID}</b> and client_secret=<b>${clientSecret}</b>
        apikeys: Your API is secured using an APIKey credential: header: <b>${keyHeaderName}</b> / value: <b>${key}</b>
      unsubscribe:
        subject: Subscription Removal Notification
        body: |
          Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a> has been unsubscribed
      subscribeFailed:
        subject: Custom Subscription Failed Notification
        body: |
          Could not subscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a> Error: ${message}
      unsubscribeFailed:
        subject: Custom Subscription Removal Failed Notification
        body: |
          Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName}</a>

log:
  level: info
  format: json
  output: stdout
  path: logs
  maskedValues: keyword1, keyword2, keyword3
```

### Running the Discovery Agent

#### Execute binary Discovery Agent in Foreground

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./discovery_agent
{"level":"info","msg":"Starting Discovery agent for V7 APIGateway (-)","time":"2020-07-06T02:56:20-07:00"}
{"level":"info","msg":"Services are Ready","time":"2020-07-06T02:56:22-07:00"}
{"level":"info","msg":"Found new frontend proxy: EMR-System-Surgery","time":"2020-07-06T02:56:22-07:00"}
{"level":"info","msg":"Found new frontend proxy: Security-HIPAA-Control","time":"2020-07-06T02:56:22-07:00"}
...
```

To stop your binary agent, press Ctrl+C within the shell.

#### Execute binary Discovery Agent in Background

When executing in the background, it is best to save your logging to a file rather than the console output. See [Customizing log section (log)](#customizing-log-section-log) above.

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./discovery_agent &
[1] 13186
```

Notice that the line after the execution returns the PID (Process Identifier).

Run the following commands to kill the PID and stop your agent:

```shell
# to find the PID, if you do not know it
ps -ef | grep discovery_agent
ubuntu     13186    4615 16 13:37 pts/1    00:00:02 ./bin/discovery_agent

# to stop the PID
kill 13186
```

#### Execute Discovery Agent as a Service

The agent can be installed as a Linux service with systemd. The following commands will help you utilize the service. These commands install the service abilities and must be run as a root user.

When running as a service, it is best to save your logging to a file rather than the console output. See [Customizing log section (log)](#customizing-log-section-log) above.

* Install

  To install the service and have it execute as user axway and group axway:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service install -u axway -g axway
  ```

  Optionally, provide an environment file with all the configuration overwrites for the agent:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service install -u axway -g axway --envFile /path/to/da_env_file.env
  ```

* Start

  To start the service:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service start
  ```

* Logs

  To get all logs for the service, since the machine last booted:

  ```shell
  cd /home/APIC-agents
  ./discovery_agent service logs
  ```

* Stop

  To stop the service:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service stop
  ```

* Enable

  To enable the service to start when the machine starts:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service enable
  ```

* Name

  To get the name of the service:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service name
  ```

* Remove

  To uninstall the service from the machine:

  ```shell
  cd /home/APIC-agents
  sudo ./discovery_agent service stop   # to ensure it is not running
  sudo ./discovery_agent service remove
  ```

#### Verify Discovery Agent is Running

To verify if the agent is up and running, open a shell and run:

```shell
cd /home/APIC-agents
./discovery_agent --status

{
  "name": "discovery_agent",
  "version": "-",
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

#### Run the Dockerized Discovery Agent

1. Copy the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment.
2. Start the Docker Discovery Agent pointing to the `env_vars` file and the keys directory. `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.

   ```shell
   docker run --env-file ./env_vars -v <pwd>/keys:/keys axway.jfrog.io/ampc-public-docker-release/agent/v7-discovery-agent:latest
   ```
3. Run the following health check command to ensure the agent is up and running:

   ```shell
   docker inspect --format='{{json .State.Health}}' <container>
   ```

## Traceability Agent

The traceability agent is used to filter the Axway API Gateway logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform. Each time an already discovered API is called by a consumer, an event (summary + detail) is sent to AMPLIFY Central and is visible in API Observer.

The agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - `traceability_agent.yaml`:

    * Default: located in the same directory as the agent binary.
    * Optional: use a dedicated folder where the configuration file is located (use the --path.config flag in the agent command line to access the file path).
    * Advanced configuration: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Discovery Agent variables](/docs/central/connect-api-manager/agent-variables/).
* With command line argument. See [Traceability Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

### Installing the Traceability Agent

#### To install the binary Traceability Agent

**Step 1**: Download the latest version of the zip file from the Axway public repository using the following command:

```shell
curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_traceability_agent/latest/traceability_agent-latest.zip" -o traceability_agent-latest.zip
```

**Step 2**: Unzip the file traceability_agent-latest.zip to get the agent binary (traceability_agent) and a template configuration file (traceability_agent.yml).

```shell
unzip traceability_agent-latest.zip
```

**Step 3**: Copy those 2 files into a folder (/home/APIC-agents for instance) on the machine where the API Manager environment is located.

**Step 4**: If not done yet, move the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to the agent directory (APIC-agents). Note that the `public_key` comes from Steps 3 or 4 of [Create a Service Account](/docs/central/connect-api-manager/prepare-amplify-central/#create-a-service-account) depending if you choose to use the `der` format or not.

#### To install the Dockerized Traceability Agent

Create your Discovery Agent environment file, `env_vars`. See [Traceability Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable descriptions.
After customizing all the sections, your `env_vars` file should look like this example file:

```shell
# API MANAGER connectivity
APIMANAGER_HOST=<HOST>
APIMANAGER_AUTH_USERNAME=<USER>
APIMANAGER_AUTH_PASSWORD=<PASSWORD>

# API GATEWAY connectivity
APIGATEWAY_HOST=<HOST>
APIGATEWAY_AUTH_USERNAME=<USER>
APIGATEWAY_AUTH_PASSWORD=<PASSWORD>

# AMPLIFY connectivity
CENTRAL_ORGANIZATIONID=<ORGANIZATIONID>
CENTRAL_TEAM=<TEAM>
CENTRAL_AUTH_CLIENTID=<CLIENTID, ie. DOSA_12345...>
CENTRAL_ENVIRONMENT=<Environment>
```

* The value for *team* can be found in [AMPLIFY Central > Access > Team Assets](https://apicentral.axway.com/access/teams/).
* The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
* The value for *clientId* can be found in Service Account. See [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).

Pull the latest Docker image of the Traceability Agent:

```shell
docker pull axway.jfrog.io/ampc-public-docker-release/agent/v7-traceability-agent:latest
```

### Customizing the Traceability Agent configuration file

The `traceability_agent.yml` configuration file contain six sections to customize: the beat input, central, apigateway, apimanager, output ingestion service and log.

By default the `traceability_agent.yml` file contains references to environment variables: ${VARIABLE_NAME: value}. You can remove the ${VARIABLE_NAME} and keep only the value for simplifying the configuration.

#### Customizing input section (traceability_agent.input)

This section describes where the API Gateway logs are located on the machine so the beat can read them.

`paths`: List all API Gateway event log files (absolute path) the beat will listen to. It could be a single file if there is only one gateway installed on the machine, or multiple files if several gateways are installed on the same machine.

Single Gateway - explicit file name

```yaml
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-1.log
```

Multiple Gateways on the same machine - explicit file names

```yaml
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-1.log
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-3.log
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-7.log
```

Multiple Gateways on the same machine - file path with wildcard

```yaml
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-?.log
```

#### Customizing central section (traceability_agent.central)

This section connects the agent to AMPLIFY Central and determine how to published the discovered APIs.

`url`: The amplify central url. Default value is **<https://apicentral.axway.com>**.

`organizationID`: The Organization ID from AMPLIFY Central. Locate this at Platform > User > Organization > ORrg ID field.

`deployment`: The APIC deployment environment. Default value is **prod**.

`environment`: The environment name you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/).

`auth.url`: The AMPLIFY login URL. Default value is **<https://login.axway.com/auth>**.

`auth.realm`: The Realm used to authenticate for AMPLIFY Central. Default value is **Broker**.

`auth.clientId`: The name of the Service Account you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Locate this at AMPLIFY Central > Access > Service Accounts.

`auth.privateKey`: The location of the private key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.

`auth.publicKey`:  The location of the public key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.  

`auth.keyPassword`: The key password to open the key. None set up by default.

`auth.timeout`: Timeout for the authentication. Default value is **10s**.

`proxy_url`: The URL for the proxy for Amplify Central `<http://username:password@hostname:port>`. If empty, no proxy is defined.

`ssl` settings: By default, for connecting to AMPLIFY Central, agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [Administer API Manager agent security](/docs/central/connect-api-manager/agent-security-api-manager/) section for changing this behavior.

Once all data is gathered, this section should look like:

```yaml
traceability_agent:
  ...
  central:
    url: https://apicentral.axway.com
    organizationID: 68794y2
    deployment: prod
    environment: my-v7-env
    auth:
      url: https://login.axway.com/auth
       realm: Broker
      clientId: "DOSA_68732642t64545..."
      privateKey: /home/APIC-agents/private_key.pem
      publicKey: /home/APIC-agents/public_key.pem
      keyPassword: ""
      timeout: 10s
    ssl:
#       minVersion: {CENTRAL_SSL_MINVERSION:""}
#       maxVersion: ${CENTRAL_SSL_MAXVERSION:""}
#       nextProtos: ${CENTRAL_SSL_NEXTPROTOS:[]}
#       cipherSuites: ${CENTRAL_SSL_CIPHERSUITES:[]}
#       insecureSkipVerify: ${CENTRAL_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: "http://username:password@hostname:port"
```

#### Customizing apigateway section (traceability_agent.apigateway)

This section helps the agent to collect the header from request/response from the API Gateway system.

`getHeaders`: Tells the agent to  call the API Gateway API to get additional transaction details (headers). Default value is **true**. If false, API Gateway config does not need to be set and no headers will be send to AMPLIFY Central.

`host`: The host that Axway API Gateway is running on. Default value is **localhost**.

`port`: The port that Axway API Gateway is listening on. Default value is **8090**.

`pollInterval`: The frequency in which the agent polls the logs in us, ms, s, m, h. Default value is **1m**.

`auth.username`: An Axway API Gateway username with the "API Gateway operator" role.

`auth.password`: The Axway API Gateway username password in clear text.

`ssl` settings: By default, for connecting to API Gateway, the agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [Administer API Manager agent security](/docs/central/connect-api-manager/agent-security-api-manager/) section for changing this behavior.

Once all data is gathered, this section should look like:

```yaml
traceability_agent:
  ...
  apigateway:
    getHeaders: true
    host: localhost
    port: 8090
    pollInterval: 1m
    auth:
      username: myApiGatewayOperatorUser
      password: myApiGatewayOperatorUserPassword
    ssl:
#      minVersison: ${APIGATEWAY_SSL_MINVERSION:""}
#      maxVersion: ${APIGATEWAY_SSL_MAXVERSION:""}
#      nextProtos: ${APIGATEWAY_SSL_NEXTPROTOS:[]}
#      cipherSuites: ${APIGATEWAY_SSL_CIPHERSUITES:[]}
#      insecureSkipVerify: ${APIGATEWAY_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: ${APIGATEWAY_PROXYURL:""}
```

#### Customizing apimanager section (traceability_agent.apimanager)

This section tells the agent which API needs to be monitored: one that has been discovered by the Discovery Agent.

`host`: The Machine name where API Manager is running. localhost value can be used, as the agent is installed on the same machine as the API Manager.

`port`: The API Manager port number (**8075** by default).

`pollInterval`: The frequency in which API Manager is polled for new endpoints. Default value is 30s.

`apiVersion`: The API Manager API version to use. Default value is **1.3**.

`auth.username`: An API Manager user the agent will use to connect to the API Manager. This user must have either an “API Manager Administrator” or “Organization administrator” role. Based on the role of this user, the agent is able to:

* discover any API from any organization (“API Manager Administrator”)  
* discovery any API from a specific organization (“Organization administrator”)

For the Traceability Agent to report correctly the discovered API traffic, it is recommended to use the same user as the one used for discovering APIs.

`auth.password`: The password of the API Manager user in clear text.

`ssl` settings: By default, for connecting to API Manager, the agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [Administer API Manager agent security](/docs/central/connect-api-manager/agent-security-api-manager/) section for changing this behavior.

Once all data is gathered, this section should look like:

```yaml
traceability_agent:
  ...
  apimanager:
    host: localhost
    port: 8075
    pollInterval: 1m
    apiVersion: 1.3
    auth:
      username: myAPIManagerUserName
      password: myAPIManagerUserPassword
    ssl:
#      minVersion: ${APIMANAGER_SSL_MINVERSION:""}
#      maxVersion: ${APIMANAGER_SSL_MAXVERSION:""}
#      nextProtos: ${APIMANAGER_SSL_NEXTPROTOS:[]}
#      cipherSuites: ${APIMANAGER_SSL_CIPHERSUITES:[]}
#      insecureSkipVerify: ${APIMANAGER_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: ${APIMANAGER_PROXYURL:""}
```

#### Customizing output ingestion service section (output.traceability)

This section describes where the logs should be sent on AMPLIFY Central.

`hosts`: The host and port of the ingestion service to forward the transaction log entries. Default value is **ingestion-lumberjack.datasearch.axway.com:453**.

`protocol`: The protocol (https or tcp) to be used for communicating with ingestion service. Default value is **tcp**.

`compression_level`: The gzip compression level for the output event. Default value is **3**.

`ssl.cipher_suites`: List the cipher suites for the TLS connectivity. See the [Administer API Manager agent security](/docs/central/connect-api-manager/agent-security-api-manager/) topic for more information.

`proxy_url`: The socks5 or http URL of the proxy server for ingestion service (**socks5://username:password@hostname:port**) to use when the API Management eco-system is not allowed to access the internet world where AMPLIFY Central is installed. **username** and **password** are optional and can be omitted if not required by the proxy configuration. Leaving this value empty means that no proxy will be used to connect to AMPLIFY Central ingestion service.

Once all data is gathered, the section should look like this if you do not use a proxy:

```yaml
# Send output to Central Database
output.traceability:
  enabled: true
  hosts: ingestion-lumberjack.datasearch.axway.com:453
  protocol: tcp
  compression_level: 3
  ssl:
    enabled: true
    verification_mode: none
    cipher_suites:
      - "ECDHE-ECDSA-AES-128-GCM-SHA256"
      - "ECDHE-ECDSA-AES-256-GCM-SHA384"
      - "ECDHE-ECDSA-AES-128-CBC-SHA256"
      - "ECDHE-ECDSA-CHACHA20-POLY1305"
      - "ECDHE-RSA-AES-128-CBC-SHA256"
      - "ECDHE-RSA-AES-128-GCM-SHA256"
      - "ECDHE-RSA-AES-256-GCM-SHA384"
   pipelining: 0
#   proxy_url: socks5://username:password@hostname:port
```

#### Customizing beat queuing section (queue)

The queue section defines the internal Filebeat queue to store events before publishing them. The queue is responsible for buffering and combining events into batches that can be consumed by the outputs. The outputs use bulk operations to send a batch of events in one transaction.

`QUEUE_MEM_EVENTS`: Number of events the queue can store (2048 by default).

`QUEUE_MEM_FLUSH_MINEVENTS`: Minimum number of events required for publishing. If this value is set to 0, the output starts publishing events without additional waiting times. Otherwise, the output must wait for more events to become available (100 by default).

`QUEUE_MEM_FLUSH_TIMEOUT`: Maximum wait time for `QUEUE_MEM_FLUSH_MINEVENTS` to be fulfilled (1 second by default). If set to 0s, events are immediately available for consumption.

Once all data is gathered, this section should look like:

```yaml
queue:
  mem:
    events: ${QUEUE_MEM_EVENTS:2048}
    flush:
      min_events: ${QUEUE_MEM_FLUSH_MINEVENTS:100}
      timeout: ${QUEUE_MEM_FLUSH_TIMEOUT:1s}
```

#### Customizing log section (logging)

The log section defines how the agent manages its logs.

`to_stderr`: (default configuration) The output is logged onto the screen.

`to_file`:  (alternate configuration) The output is logged into a file. Requires more configuration (refer to <https://www.elastic.co/guide/en/beats/filebeat/current/configuration-logging.html>).

`level`: The log level for output messages (debug, info, warn, error). Default value is **info**.

Once all data is gathered, this section should look like this for standard output logging:

```yaml
logging:
  metrics:
    enabled: false
  # Send all logging output to stderr
  to_stderr: true
  # Set log level
  level: info
  # Send all logging output to file - change value to_files: true and to_stderr: false
  to_files: false
  files:
    path: ./logs
    name: traceability_agent.log
    keepfiles: 7
    permissions: 0644
```

#### Validating your custom Traceability Agent configuration file

After customizing all the sections, your traceability_agent.yaml file should look like:

```yaml
################### Beat Configuration #########################
traceability_agent:
  inputs:
    - type: log
      paths:
        - <PATH TO>/group-X_instance-Y.log
      include_lines: ['.*"type":"transaction".*"type":"http".*']
  central:
    url: https://apicentral.axway.com
    organizationID: 68794y2
    deployment: prod
    environment: my-v7-env
    auth:
      url: https://login.axway.com/auth
      realm: Broker
      clientId: "DOSA_68732642t64545..."
      privateKey: /home/APIC-agents/private_key.pem
      publicKey: /home/APIC-agents/public_key.pem
      keyPassword: ""
      timeout: 10s
    ssl:
#      minVersion: {CENTRAL_SSL_MINVERSION:""}
#      maxVersion: ${CENTRAL_SSL_MAXVERSION:""}
#      nextProtos: ${CENTRAL_SSL_NEXTPROTOS:[]}
#      cipherSuites: ${CENTRAL_SSL_CIPHERSUITES:[]}
#      insecureSkipVerify: ${CENTRAL_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: "http://username:password@hostname:port"
  apigateway:
    getHeaders: true
    host: localhost
    port: 8090
    pollInterval: 1m
    auth:
      username: myApiGatewayOperatorUser
      password: myApiGatewayOperatorUserPassword
    ssl:
#      minVersison: ${APIGATEWAY_SSL_MINVERSION:""}
#      maxVersion: ${APIGATEWAY_SSL_MAXVERSION:""}
#      nextProtos: ${APIGATEWAY_SSL_NEXTPROTOS:[]}
#      cipherSuites: ${APIGATEWAY_SSL_CIPHERSUITES:[]}
#      insecureSkipVerify: ${APIGATEWAY_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: ${APIGATEWAY_PROXYURL:""}
  apimanager:
    host: localhost
    port: 8075
    pollInterval: 1m
    apiVersion: 1.3
    auth:
      username: myAPIManagerUserName
      password: myAPIManagerUserPassword
    ssl:
#      minVersion: ${APIMANAGER_SSL_MINVERSION:""}
#      maxVersion: ${APIMANAGER_SSL_MAXVERSION:""}
#      nextProtos: ${APIMANAGER_SSL_NEXTPROTOS:[]}
#      cipherSuites: ${APIMANAGER_SSL_CIPHERSUITES:[]}
#      insecureSkipVerify: ${APIMANAGER_SSL_INSECURESKIPVERIFY:false}
#    proxyUrl: ${APIMANAGER_PROXYURL:""}

# Send output to Central Database
output.traceability:
  enabled: true
  hosts: ${TRACEABILITY_URL:ingestion-lumberjack.datasearch.axway.com:453}
  protocol: ${TRACEABILITY_PROTOCOL:"tcp"}
  compression_level: ${TRACEABILITY_COMPRESSIONLEVEL:3}
  bulk_max_size: ${TRACEABILITY_BULKMAXSIZE:100}
  timeout: ${TRACEABILITY_TIMEOUT:300s}
  pipelining: 0
  ssl:
    enabled: true
    verification_mode: none
    cipher_suites:
      - "ECDHE-ECDSA-AES-128-GCM-SHA256"
      - "ECDHE-ECDSA-AES-256-GCM-SHA384"
      - "ECDHE-ECDSA-AES-128-CBC-SHA256"
      - "ECDHE-ECDSA-CHACHA20-POLY1305"
      - "ECDHE-RSA-AES-128-CBC-SHA256"
      - "ECDHE-RSA-AES-128-GCM-SHA256"
      - "ECDHE-RSA-AES-256-GCM-SHA384"
  proxy_url: ${TRACEABILITY_PROXYURL:""}

queue:
  mem:
    events: ${QUEUE_MEM_EVENTS:2048}
    flush:
      min_events: ${QUEUE_MEM_FLUSH_MINEVENTS:100}
      timeout: ${QUEUE_MEM_FLUSH_TIMEOUT:1s}

logging:
  metrics:
    enabled: false
  # Send all logging output to stderr
  to_stderr: true
  # Set log level
  level: info
  # Send all logging output to file - change value to_files: true and to_stderr: false
  to_files: false
  files:
    path: ./logs
    name: traceability_agent.log
    keepfiles: 7
    permissions: 0644
```

### Running the binary Traceability Agent

Open a shell and run the following command to start up your agent:

#### Execute binary Traceability Agent in Foreground

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./traceability_agent
...
```

To stop your agent, press Ctrl+C within the shell.

#### Execute binary Traceability Agent in Background

When executing in the background, it is best to save your logging to a file rather than the console output. See [Customizing log section (logging)](#customizing-log-section-logging) above.

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./traceability_agent &
[1] 13186
```

Notice that the line after the execution returns the PID (Process Identifier).

Run the following commands to kill the PID and stop your agent:

```shell
# to find the PID, if you do not know it
ps -ef | grep traceability_agent
ubuntu     13186    4615 16 13:37 pts/1    00:00:02 ./bin/traceability_agent

# to stop the PID
kill 13186
```

#### Execute binary Traceability Agent as a Service

The agent can be installed as a Linux service, with systemd. The following commands will help you utilize the service. These commands install the service abilities and must be run as a root user.

When running as a service, it is best to save your logging to a file rather than the console output. See [Customizing log section (logging)](#customizing-log-section-logging) above.

* Install

  To install the service and have it execute as user axway and group axway:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service install -u axway -g axway
  ```

  Optionally, provide an environment file with all the configuration overwrites for the agent:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service install -u axway -g axway --envFile /path/to/ta_env_file.env
  ```

* Start

  To start the service:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service start
  ```

* Logs

  To get all logs for the service, since the machine last booted:

  ```shell
  cd /home/APIC-agents
  ./traceability_agent service logs
  ```

* Stop

  To stop the service:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service stop
  ```

* Enable

  To enable the service to start when the machine starts:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service enable
  ```

* Name

  To get the name of the service:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service name
  ```

* Remove

  To uninstall the service from the machine:

  ```shell
  cd /home/APIC-agents
  sudo ./traceability_agent service stop   # to ensure it is not running
  sudo ./traceability_agent service remove
  ```

#### Verify binary Traceability Agent is Running

To verify if the agent is up and running, open a shell command and run:

```shell
cd /home/APIC-agents
./traceability_agent status
```

##### Install and run Dockerized Traceability Agent

* See "To install the Dockerized Discovery Agent" section above for the `env_vars` configuration.

1. Copy the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment.
2. Start the Traceability Agent pointing to the `env_vars` file, `keys`, and the logging `events` directory. `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.

   ```shell
   docker run --env-file ./env_vars -v <pwd>/keys:/keys -v <pwd>/events:/events axway.jfrog.io/ampc-public-docker-release/agent/v7-traceability:latest
   ```

   * See [Create and start API Gateway Docker container](/docs/apim_installation/apigw_containers/docker_script_gwimage/#mount-volumes-to-persist-logs-outside-the-api-gateway-container/) for more  information regarding the persistent API Gateway trace and event logs to a directory on your host machine.
   * Run the following health check command to ensure the agent is up and running:

   ```shell
   docker inspect --format='{{json .State.Health}}' <container>
   ```
