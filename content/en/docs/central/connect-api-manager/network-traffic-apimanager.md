---
title: Administer API Manager network traffic
linkTitle: Administer API Manager network traffic
draft: false
weight: 30
description: Traffic is always initiated by the Agent to API Manager, API
  Gateway, and Amplify Central. No sessions are ever initiated back to the
  Agent.
---

## Overview

![Agent networking](/Images/central/connect-api-manager/agents-proxy2.png "Agent networking")

All communications are initiated by the agents so there are no requirements to change the inbound rules on your network infrastructure.

## Data destinations

The destination for:

* Agent Authentication data is `login.axway.com`
* API definition (Swagger or WSDL) and API documentation data is `apicentral.axway.com`
* API Event data, the transaction summary and headers, is `ingestion-lumberjack.datasearch.axway.com`
* Subscription notification for getting platform user information is `platform.axway.com`
* API usage statistics, the number of calls placed to, is `https://lighthouse.admin.axway.com`

## Data exchanged

### Discovery Agent

Use variable `apimanager.filter` to select which API should be sent to Axway Amplify platform. Only the matching APIs are transferred to Axway Amplify platform. See [Discover APIs](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). The Discovery Agent sends the following information to the Axway Amplify platform:

* API definition using Swagger or WSDL depending on the API type (REST vs SOAP)
* API documentation

### Traceability Agent

The Traceability Agent reports two sets of data to Amplify platform:

* Usage data
* Transactions data

#### Usage data

The usage data represent the total number of APIs called during a certain period of time. This usage is automatically reported every 15 minutes by default to Amplify platform and cannot be inactivated. See [Reporting Gateway usage event](/docs/central/connected_agent_common_reference/traceability_usage/#reporting-gateway-usage-event).

It contains the following information:

Structural Fields:

* Unique ID
* Timestamp
* Event Identifier
* Environment Identifier
* Event version

Data Fields:

* Count
* Observation Window (start / end)

#### Transactions data

The transaction data represent the number and details of transactions processed by the Gateway during a period of time. You can limit the number of transactions sent to the platform, or completely turn this feature off, by applying a sampling configuration to the Traceability Agent. See [Sampling](/docs/central/connected_agent_common_reference/trace_sampling/#sampling).

The transactions can be redacted (by default) and/or sanitized, according to your need, before sending the information to Amplify platform. See [Trace redaction](/docs/central/connected_agent_common_reference/trace_redaction/).

It contains the following information:

Structural Fields:

* Unique ID
* Timestamp
* Event Identifier
* App Identifier (v7 Gateway application or AWS Usage plan or Azure subscription)
* Amplify Central environment Identifier
* Event version

Data Fields:

* API Identifier
* Transaction status (i.e., HTTP status of the API)
* URLs (frontend / backend API)
* Duration and timestamp
* Service called: method (POST / GET…) + URI path
* Request/response headers from each API call (this can be turned off using the `APIGATEWAY_GETHEADERS=false` property)

{{< alert title="Note" color="primary" >}}You can disable sending the headers by using the following property:  `APIGATEWAY_GETHEADERS=false`. By default, the property is set to true. If collecting the headers is disabled, they will not be visible in Axway Amplify platform Observability module, as the Traceability Agent will send only the transaction summary data (status / url / duration / timestamp / transaction service called) to the platform.{{< /alert >}}

## Communication ports

All outbound traffic is sent over SSL via TCP / UDP.

Open the following ports so that agents can communicate to the Amplify platform:

**Outbound**:

| Region | Host                                                                                      | IP             | port       | Protocol     | data                               |
|--------|-------------------------------------------------------------------------------------------|----------------|------------|--------------|------------------------------------|
| EU/US  | platform.axway.com                                                                        | 34.211.114.227 | 443        | HTTPS        |                                    |
|        |                                                                                           | 54.201.86.113  |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| EU/US  | lighthouse.admin.axway.com                                                                |                | 443        | HTTPS        | API usage statics                  |
|        |                                                                                           |                |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| EU/US  | login.axway.com                                                                           | 52.58.132.2    | 443        | HTTPS        |                                    |
|        |                                                                                           | 52.29.4.35     |            |              |                                    |
|        |                                                                                           | 54.93.140.145  |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| US     | apicentral.axway.com                                                                      | 3.94.245.118   | 443        | HTTPS        | API definitions, Subscription info |
|        |                                                                                           | 54.208.199.251 |            |              |                                    |
|        |                                                                                           | 3.212.78.217   |            |              |                                    |
|        |                                                                                           | 52.202.95.208  |            |              |                                    |
|        |                                                                                           | 107.23.176.64  |            |              |                                    |
|        |                                                                                           | 3.225.16.120   |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| EU     | central.eu-fr.axway.com                                                                   | 52.47.84.198   | 443        | HTTPS        | API definitions, Subscription info |
|        |                                                                                           | 13.36.25.69    |            |              |                                    |
|        |                                                                                           | 35.181.21.87   |            |              |                                    |
|        |                                                                                           | 13.36.2.143    |            |              |                                    |
|        |                                                                                           | 13.36.52.216   |            |              |                                    |
|        |                                                                                           | 15.236.7.112   |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| US     | ingestion.datasearch.axway.com                                                            | 54.225.171.111 | 5044 or 443 | TCP or HTTPS | API event data                     |
|        |                                                                                           | 54.225.2.221   |            |              |                                    |
|        |                                                                                           | 54.146.97.250  |            |              |                                    |
|        |                                                                                           | 54.147.98.128  |            |              |                                    |
|        |                                                                                           | 52.206.193.184 |            |              |                                    |
|        |                                                                                           | 54.225.92.97   |            |              |                                    |
|        |                                                                                           |                |            |              |                                    |
| EU     | ingestion.visibility.eu-fr.axway.com                                                      | 15.236.125.123 | 5044 or 443 | TCP or HTTPS | API event data                     |
|        |                                                                                           | 35.180.77.202  |            |              |                                    |
|        |                                                                                           | 13.36.27.97    |            |              |                                    |
|        |                                                                                           | 13.36.33.229   |            |              |                                    |

Note: _Region_ column is representing the region where your Amplify organization is deployed. EU means deployed in European data center and US meaning deployed in US data center. Be sure to use the corresponding _Host_/_Port_ for your agents to operate correctly.

Other ports which may need to be opened so that the Agent may monitor API Gateway / Manager are:

**Internal**:

| Host             | Port           | Protocol | Data                                                                                                                                                    |
|------------------|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| API Manager Host | 8075 (default) | HTTPS    | API Discovery                                                                                                                                           |
| API Gateway Host | 8090 (default) | HTTPS    | API Transaction Header data (see [APIGATEWAY GETHEADERS](/docs/central/connect-api-manager/agent-variables/#specific-variables-for-traceability-agent)) |

**Inbound (used for the agent status server)**:

| Host          | Port           | Protocol | Data                                                               |
|---------------|----------------|----------|--------------------------------------------------------------------|
| Agent Host(s) | 8989 (default) | HTTPS    | Serves the status of the agent and its dependencies for monitoring |

## Subscription notifications

SMTP and/or a webhook URL can be set up to send subscription notifications on both subscribe and unsubscribe actions.  You can find the configuration to set up the SMTP or webhook URL at [Deploy your agents](/docs/central/connect-api-manager/deploy-your-agents-with-amplify-cli/).

## Using proxies

If your network policy restricts outbound traffic such that outbound traffic must pass through a proxy. A proxy can be configured in the configuration file of the agents.

### HTTP/HTTPS Proxy

Use a HTTP/HTTPS Proxy for communication to the Amplify Platform.  This configuration is set for both the [Discovery](/docs/central/connect-api-manager/agent-variables/) and [Traceability Agents](/docs/central/connect-api-manager/agent-variables/).

### SOCKS5 Proxy

Use a SOCKS5 Proxy for communication to the Amplify Platform when sending API Traffic Events.  This configuration is set only for [Traceability Agents](/docs/central/connect-api-manager/agent-variables/#specific-variables-for-traceability-agent).

### Proxy authentication

Both proxy types will use one of two authentication mechanisms, none or username/password authentication. The username authentication is specified within the URL `http://{user}:{pass}@{proxy}:{port}`.

## Validation

### Direct Connection

**Connecting to Amplify Central and Login hosts:**

```shell
curl -s -o /dev/null -w "%{http_code}"  https://apicentral.axway.com
```

```shell
curl -s -o /dev/null -w "%{http_code}"  https://login.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to Amplify Central Event Traffic host, HTTPS:**

```shell
curl -s -o /dev/null -w "%{http_code}" https://ingestion.datasearch.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to Amplify Central Event Traffic host, Lumberjack:**

```shell
curl ingestion-lumberjack.datasearch.axway.com:453
```

A return of **"curl: (52) Empty reply from server"** validates the connection was established.

### Connection via Proxy

**Connecting to Amplify Central and Login hosts:**

```shell
curl -x {{proxy_host}}:{{proxy_port}} -s -o /dev/null -w "%{http_code}"  https://apicentral.axway.com
```

```shell
curl -x {{proxy_host}}:{{proxy_port}} -s -o /dev/null -w "%{http_code}"  https://login.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to Amplify Central Event Traffic host, HTTPS:**

```shell
curl -x {{proxy_host}}:{{proxy_port}} -s -o /dev/null -w "%{http_code}" https://ingestion.datasearch.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to Amplify Central Event Traffic host, Lumberjack:**

```shell
curl -x socks5://{{proxy_host}}:{{proxy_port}} ingestion-lumberjack.datasearch.axway.com:453
```

A return of **"curl: (52) Empty reply from server"** validates the connection was established.

## Troubleshooting

### Curl connection to ingestion-lumberjack.datasearch.axway.com

* **Error:**

  ```shell
  curl: (6) Could not resolve host: ingestion-lumberjack.datasearch.axway.com
  ```

    * **Cause:** The host making the call can’t resolve the ingestion-lumberjack DNS name.

    * **Possible Resolution:** Tell curl to resolve the hostname on the proxy:

    ```shell
    curl -x socks5h://{{proxy_host}}:{{proxy_port}} ingestion-lumberjack.datasearch.axway.com
    ```
* **Error:**

  ```shell
  curl: (7) No authentication method was acceptable.
  ```

    * **Cause:** The SOCKS proxy server expected an authentication type other than what was specified.

    * **Possible Resolution:** Provide authentication to the proxy:

    ```shell
    socks5://{{username}}:{{password}}@{{proxy_host}}:{{proxy_port}}
    ```

    The Agents only support the use of username/password authentication method for SOCKS connections.
