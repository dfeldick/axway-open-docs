---
title: Deploy your agents - advanced functionality
linkTitle: Deploy your agents - advanced functionality
draft: true
weight: 30
description: Learn how to deploy your Discovery Agent and Traceability Agent so
  that you can manage your Axway API Gateway environment within AMPLIFY Central.
---
## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* Be sure you have [Prepared AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/)
* You will need a basic knowledge of Axway API Management solution

    * Where the solution is running (host / port / path to the logs / users)
    * How to create / publish an API
    * How to call an API
* For containerized agents, Docker must be installed and you will need a basic understanding of Docker commands

## Objectives

Learn how to install, customize and run your Discovery and Traceability agents as either a binary Linux executable or as a Docker container.

## Discovery Agent

The Discovery Agent is used to discover new published APIs or any updated APIs. Once they are discovered, the related APIs are published to AMPLIFY Central, in one of the following publication modes:

* **Environment / API Service publication** : Customers publish their APIs to the AMPLIFY platform.
* **Environment / API Service publication / Catalog item publication** (default mode): Same as previous plus automatically expose the APIs to the consumer via the AMPLIFY Catalog.

The Discovery Agent only discovers APIs that have the tag(s) defined in the agent configuration file. See [Discover APIs](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). By default, the filter is empty and thus the agent will discover all published APIs.

As soon as an API is published, the identifier of the asset in AMPLIFY Central is kept in a custom field at the API level in API Manager to help the agent remember what is already published.

The binary agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - discovery_agent.yaml:

    * **Default**: located in the same directory as the agent binary.
    * **Optional**: use a dedicated folder where the configuration file is located (use the –pathConfig flag in the agent command line to access the file path).
    * **Advanced configuration**: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Agent variables](/docs/central/connect-api-manager/agent-variables/).

* With command line arguments. See [Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

The containerized agent can run in the following mode:

* With an environment variables configuration file, `env_vars`, supplied as a command line argument when running the Docker image.

### Installing the Discovery Agent

#### To install the Binary Discovery Agent

1. Download the latest version of the zip file from the Axway public repository using the following command:

   ```shell
   curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_discovery_agent/latest/discovery_agent-latest.zip" -o discovery_agent-latest.zip
   ```

2. Unzip the file `discovery_agent-latest.zip` to get the agent binary (`discovery_agent`) and the template configuration (`discovery_agent.yml`).
3. Copy both files (`discovery_agent`, `discovery_agent.yml`) into a folder (i.e., `/home/APIC-agents`) on the machine where the API Manager environment is located.
4. Customize apimanager section by setting configuration values to point to the API Gateway, API Manager and AMPLIFY Central.  There are two options to set values:

   * `env_vars` file
   * `discovery_agent.yml`:

   ```shell
   apimanager:
    host: localhost
    port: 8075
    discoveryIgnoreTags: tag1, tag2
    filter:
    subscriptionApplicationField: subscriptions
    pollInterval: 30s
    ssl:
     insecureSkipVerify: true
    auth:
     username: apiManagerUser
     password: apiManagerUserPassword

   central:
    url: https://apicentral.axway.com
    platformURL: https://platform.axway.com
    team: Dev
    organizationID: 655431797898152
    environment: my-v7-env
    apiServerVersion: v1alpha1
    mode: publishToEnvironmentAndCatalog
    pollInterval: 10s
    auth:
     url: https://login-preprod.axway.com/auth
     realm: Broker
     clientId: DOSA_66743...
     privateKey: /home/APIC-agents/private_key.pem
     publicKey: /home/APIC-agents/public_key.pem
     keyPassword:
     timeout: 10s

   log:
    level: debug
    format: json
    output: stdout
    maskedValues: keyword1, keyword2, keyword3

   status:
    healthCheckInterval: 30s
   ```

   * The value for *team* can be found in [AMPLIFY Central > Access > Teams](https://apicentral.axway.com/access/teams/).
   * The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
   * The value for *clientId* can be found in Service Account. See [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).
   * The value for *healthCheckInterval* can be between 30 seconds and 5 minutes. To specify the value in seconds use an 's' character (e.g. 30s). To specify the value in minutes, use a an 'm' character (e.g. 5m).
5. Run the binary Discovery Agent:

   * Open a shell and run the following commands to start up your agent.  Add necessary [Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

     ```shell
     cd /home/APIC-agents
     ./discovery_agent
     ```
   * To use env_vars file for overriding configuration, create env_vars file with definition of environment variables and run the following command. See [Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable descriptions.

     ```shell
     cd /home/APIC-agents
     ./discovery_agent --envFile ./env_vars
     ```
   * To verify that the agent is up and running, open another shell command and run:

     ```shell
     cd /home/APIC-agents
     ./discovery_agent --status
     ```

##### To install the Dockerized Discovery Agent

Create your Discovery Agent environment file, `env_vars`. See [Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable descriptions.
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
CENTRAL_TEAM=<AMPLIFYCentralTeamName>
CENTRAL_AUTH_CLIENTID=<CLIENTID, ie. DOSA_12345...>
```

* The value for *team* can be found in [AMPLIFY Central > Access > Team Assets](https://apicentral.axway.com/access/teams/).
* The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
* The value for *clientId* can be found in Service Account. See [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).

##### Run the Dockerized Discovery Agent

1. Copy the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment.

2. Pull the latest image of the Discovery Agent:

   ```shell
   docker pull axway.jfrog.io/ampc-public-docker-release/agent/v7-discovery-agent:latest
   ```

3. Start the Docker Discovery Agent pointing to the `env_vars` file and the keys directory. `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.

   ```shell
   docker run --env-file ./env_vars -v <pwd>/keys:/keys -it axway.jfrog.io/ampc-public-docker-release/agent/v7-discovery-agent:latest
   ```

4. Run the following health check command to ensure the agent is up and running:

   ```shell
   docker inspect --format='{{json .State.Health}}' <container>
   ```

## Traceability Agent

The traceability agent is used to filter the Axway API Gateway logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform. Each time an already discovered API is called by a consumer, an event (summary + detail) is sent to AMPLIFY Central and is visible in API Observer.

The agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - `traceability_agent.yaml`:

    * Default: located in the same directory as the agent binary.
    * Optional: use a dedicated folder where the configuration file is located (use the --path.config flag in the agent command line to access the file path).
    * Advanced configuration: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Agent variables](/docs/central/connect-api-manager/agent-variables/).
* With command line argument. See [Traceability Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

### Installing the Traceability Agent

To install the binary Traceability Agent:

1. Download the Traceability Agent using the following command:

   ```shell
   curl -L "https://axway.jfrog.io/artifactory/ampc-public-generic-release/v7-agents/v7_traceability_agent/latest/traceability_agent-latest.zip" -o traceability_agent-latest.zip
   ```

2. Unzip the file `traceability_agent-latest.zip` to get the agent binary (`traceability_agent`) and the template configuration (`traceability_agent.yml`).
3. Copy both files (`traceability_agent`, `traceability_agent.yml`) into a folder (i.e., `/home/APIC-agents`) on the machine where the API Manager environment is located.
4. Customize traceability_agent section by setting configuration values to point to the event logs path,  API Gateway, API Manager, and AMPLIFY Central.  There are 2 options to set values:

   * `env_vars` file
   * `traceability_agent.yml`:

   ```yaml
   ################### Beat Configuration #########################
   traceability_agent:
     inputs:
       - type: log
        paths:
         - <PATH_TO>/group-X_instance-Y.log
        include_lines: ['.*"type":"transaction".*"type":"http".*']

     central:
        url: https://apicentral.axway.com
        organizationID: 68794y2
        team: Dev
        deployment: prod
        environment: my-v7-env
        auth:
          url: https://login.axway.com/auth
          realm: Broker
          clientId: "DOSA_68732642t64545..."
          privateKey: /home/APIC-agents/private_key.pem}
          publicKey: /home/APIC-agents/public_key.pem}
          keyPassword: ""
        timeout: 10s
     apigateway:
       getHeaders: true
       host: localhost
       port: 8090
       pollInterval: 1m
       auth:
         username: myApiGatewayOperatorUser
         password: myApiGatewayOperatorUserPassword
     apimanager:
       host: localhost
       port: 8075
       pollInterval: 1m
       apiVersion: 1.3
       auth:
         username: myApiManagerUserName
         password: myApiManagerUserPassword

   # Send output to Central Database
   output.traceability:
     enabled: true
     hosts: ${TRACEABILITY_HOST:ingestion-lumberjack.datasearch.axway.com:453}
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
     level: ${LOG_LEVEL:info}
   ```

   * The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
   * The value for *clientId* can be found in Service Account. See [Create a Service in AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/).
   * Traceability Agent variables can be found at [Agent variables](/docs/central/connect-api-manager/agent-variables/).
5. Run the binary Traceability Agent:

   * Once the YAML file is updated, start the Traceability Agent. If the YAML file is in the same folder, run the following command. Otherwise, pass the command-line flags that are documented at [Traceability Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

     ```shell
     cd /home/APIC-agents
     ./traceability_agent
     ```
   * To use env_vars file for overriding configuration, create env_vars file with definition of environment variables and run the following command. See [Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable description.

     ```shell
     cd /home/APIC-agents
     ./traceability_agent --envFile ./env_vars
     ```
6. The Traceability Agent parses through the files based on the `event-file` path and pattern provided. Depending on the data found, the agent pushes it to AMPLIFY Central.
7. Go to AMPLIFY Central and open the API Observer tab to verify that the agent is working. You should see the monitoring data for the APIs discovered earlier. If you don’t see any data, then invoke a few different API methods in the exposed API.
8. Click on any of the transactions to see the details. You will see the lifecycle of an API call, such as time taken / request and response headers / etc.

#### To install the Docker Traceability Agent

Create your Discovery Agent environment file, `env_vars`. See [Agent variables](/docs/central/connect-api-manager/agent-variables/) for a reference to variable descriptions.
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
CENTRAL_TEAM=<THE TEAM NAME>
CENTRAL_AUTH_CLIENTID=<CLIENTID, ie. DOSA_12345...>
CENTRAL_ENVIRONMENT=<Environment>
```

* The value for *team* can be found in [AMPLIFY Central > Access > Team Assets](https://apicentral.axway.com/access/teams/).
* The value for *organizationID* can be found in AMPLIFY Central Platform > Organization.
* The value for *clientId* can be found in Service Account. See [Create a service account](/docs/central/cli_central/cli_install/#create-a-service-account).

##### Install and run Dockerized Traceability Agent

1. Copy the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment.
2. Pull the latest Docker image of the Traceability Agent:

   ```shell
   docker pull axway.jfrog.io/ampc-public-docker-release/agent/v7-traceability-agent:latest
   ```

3. Start the Traceability Agent pointing to the `env_vars` file, `keys`, and the logging `events` directory. `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.

   ```shell
   docker run --env-file ./env_vars -v <pwd>/keys:/keys -v <pwd>/events:/events axway.jfrog.io/ampc-public-docker-release/agent/v7-traceability-agent:latest
   ```

   * See [Create and start API Gateway Docker container](/docs/apim_installation/apigw_containers/docker_script_gwimage/#mount-volumes-to-persist-logs-outside-the-api-gateway-container/) for more  information regarding the persistent API Gateway trace and event logs to a directory on your host machine.
   * Run the following health check command to ensure the agent is up and running:

   ```shell
   docker inspect --format='{{json .State.Health}}' <container>
   ```

## Configuring API Manager

### Tags

1. Configure filters in yaml file to determine which [APIs will be discovered](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/) and published to AMPLIFY Central.
2. Discovery API tags.  Set filtering based on tag name, tag value, partial value and MatchRegEx to discover APIs that can be published to AMPLIFY Central:

   * In API Manager, select front-end API to edit.
   * The tags should match the values in your apimanager configuration file.
   * Save the API and publish.  Once published, the Discovery Agent attempts to match the polling criteria. If it matches, it publishes it to the AMPLIFY Central Catalog and Environments.
   * Once the API is published to the catalog, a reference value is generated by AMPLIFY Central and sent back to the API Manager (`APIC_ID`).  If you want to see that field or customize it, refer to **Add a custom property to APIs** in [Customize API Manager](/docs/apim_administration/apimgr_admin/api_mgmt_custom/#customize-api-manager-data) documentation.

### Custom field for subscription identifier

The Discovery Agent uses a custom field in the application's resource on API Manager to store identifiers for Central subscriptions. The default value for the custom field name is `subscriptions` and is not visible in the API Manager UI, as it is a specific configuration. This field helps the Discovery Agent to know if AMPLIFY Central subscriptions are used for particular API/application combinations. Each time an AMPLIFY Central subscription succeeds, the subscription identifier is stored in the field value of the corresponding API Manager application used to initiate the subscription. When the subscription is removed from AMPLIFY Central, the corresponding subscription identifier is removed in the corresponding API Manager application.

In order to visualize those fields in API Manager UI, update the file `app.config` located in `<INSTALL-DIR>/apigateway/webapps/apiportal/vordel/apiportal/app` with the following:

```json
    customPropertiesConfig: {
        user: {
            // custom properties...
        },
        organization: {
            // custom properties...
        },
        application: {
            // custom properties...
             subscriptions: {
                     label: 'Amplify Central subscription ID',
                     disabled:true
             }
        }
    },
```

You can also change the property value (`subscriptions`) to suit your custom fields pattern. If you do so, be sure you update the agent configuration files accordingly (`subscriptionApplicationField`).

### Subscriptions - Manage client applications on APIManager for subscription process

You can use the **Clients** tab to manage client applications (for example, create, update, or remove client applications that invoke specific APIs). When an application is created, API administrators can also set authentication, quota, and sharing settings on the appropriate tab.

The API administrator must first specify the APIs that an organization is allowed to access before any of its client applications can have access to them. In API Manager, you can only add APIs to an application when they have been added to the organization. For more details, see [Manage access to APIs](/docs/apim_administration/apimgr_admin/api_mgmt_admin/).

#### Create application

To create an application:

1. Click **New application** in the toolbar and configure the following general fields:

   * **Image**: Click to add a graphical image for the application (for example, .png, .gif, or .jpeg file).
   * **Application name**: Enter the name of the application. This field is required.
   * **Organization**: Enter the name of the organization that the application belongs to. This field is required. The choice of organization determines which APIs are available to the application.
   * **Enabled**: Select whether the application is enabled. Applications are enabled by default.
2. Configure the following additional attributes:

   * **Email**: Enter an email address for the application.
   * **Phone**: Enter a phone number for the application.
   * **Description**: Enter a short description of the application.
3. Click **Add API** to select the APIs and methods used by the application. You can add multiple APIs for an application.
4. Click **Create** in the toolbar.

#### Edit application

Once applications have been created, you can click an application name in the *Managing applications* screen to edit its existing settings on the **Application** tab.

1. Add API to be recognized as an application to be used for subscription.
2. Make sure **Enabled** is active.

### Manage subscription in AMPLIFY Central

After configuring the application in API Manager, set up catalog subscriptions in AMPLIFY Central.

A consumer initiates the subscription in AMPLIFY Central:

1. Open an AMPLIFY Catalog Item.
2. Click **Subscribe**.
3. Enter the Team
4. Select an API Manager Application name (The displayed list should be the on set up on API Manager associated to the current API the subscriber is subscribing to).
5. Click **Subscribe**.

The Discovery Agent listens to the subscription event.

### Subscribing workflow

When a consumer wants to consume an API in AMPLIFY Central, he needs to subscribe to the API. For now, only an auto-approved workflow is available:

* The discovery agent listens to the subscription event.
* Check that the selected API and application are existing/published on API Manager.
* Search for the corresponding API credentials (API key or OAuth token) and send the credentials to the subscriber email.
* Send back to AMPLIFY Central the subscription status:

    * If active, the subscription ID is automatically added to the custom field of the application (`subscriptions`).
    * If failure, the subscription status **Subscription failed** will appear. You can delete the subscription and start again from the **Save the API and publish** step.
* In case of success, the subscriber consumes the API otherwise API cannot be consumed.

In case the application is not available because it is not in the same organization as the API, you can grant specific access to the API:

1. In the UI, select the API.
2. Expand **Manage selected**.
3. Select **Grant access**.

### Unsubscription workflow

In API Manager, assume there is a FrontEnd API that is published, has been discovered by the Discovery Agent, and has an active subscription to it in AMPLIFY Central. To initiate an unsubscribe to AMPLIFY Central for that Catalog item:

* Unpublish that API in API Manager.
* The Discovery Agent discovers the change.
* The subscription status is set to **Unsubscribed**.
* The subscription ID is removed from the application’s **Custom** field.
* The subscription status is set to **Unsubscribed**.

### Managing Subscription notifications

```shell
subscriptions:
  notifications:
    smtp:
      host: mail.axway.int
      port: 25
      fromAddress: fromaddress@axway.com
      username: fromaddress@axway.com
      password:
      subscribe:
        subject: Subscription Notification
        body: |
          Subscription created for Catalog Item:  <a href= ${catalogItemUrl}> ${catalogItemName} </a> <br/>
          Subscription key: <b>${key}</b>
      unsubscribe:
        subject: Subscription Removal Notification
        body: |
          Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a> has been unsubscribed
      subscribeFailed:
        subject: Subscription Failed Notification
        body: |
          Could not subscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>
      unsubscribeFailed:
        subject: Subscription Removal Failed Notification
        body: |
          Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>
```
