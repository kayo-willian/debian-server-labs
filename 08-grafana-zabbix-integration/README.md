# Lab 08 - Grafana and Zabbix Integration

## Overview

This lab documents the installation and integration of Grafana with the existing Zabbix monitoring environment running on the Debian Server.

The objective was to extend the existing monitoring infrastructure by connecting Grafana to the Zabbix API, allowing Zabbix monitoring data to be queried and visualized through Grafana.

The integration was performed using the official Grafana package repository, the Alexander Zobnin Zabbix plugin, and a Zabbix API Token.

The laboratory also included API testing and troubleshooting of the communication between Grafana, Apache, and the Zabbix API.

---

## Objectives

* Install Grafana on the Debian Server.
* Understand the relationship between Grafana and Zabbix.
* Install the Zabbix plugin for Grafana.
* Understand the Zabbix JSON-RPC API endpoint.
* Create a Zabbix API Token.
* Configure a Grafana Zabbix data source.
* Connect Grafana to the existing Zabbix API.
* Test the API independently using `curl`.
* Troubleshoot failed Grafana connection attempts.
* Validate the final integration.
* Prepare the environment for Grafana dashboards and data visualization.

---

## Environment

| Item                  | Information                                   |
| --------------------- | --------------------------------------------- |
| Operating System      | Debian Server                                 |
| Server IP             | `192.168.1.117`                               |
| Zabbix Version        | `7.4.15`                                      |
| Grafana Version       | `13.2.2`                                      |
| Zabbix Grafana Plugin | Alexander Zobnin `6.8.0`                      |
| Web Server            | Apache2                                       |
| Zabbix Web            | `http://192.168.1.117/zabbix`                 |
| Grafana Web           | `http://192.168.1.117:3000`                   |
| Zabbix API            | `http://192.168.1.117/zabbix/api_jsonrpc.php` |
| Zabbix Server Port    | `10051/TCP`                                   |
| Zabbix Agent Port     | `10050/TCP`                                   |
| Apache Port           | `80/TCP`                                      |
| Grafana Port          | `3000/TCP`                                    |
| Database              | MariaDB                                       |

---

## Existing Monitoring Architecture

Before this laboratory, the Debian Server already had a functional Zabbix monitoring environment.

The existing architecture was:

```text
                    Debian Server
                         |
        +----------------+----------------+
        |                |                |
     Apache2          Zabbix           MariaDB
      :80             :10051             :3306
        |                |
        |                |
   Zabbix Web       Zabbix Server
        |                |
        +------- Zabbix Data -------+
```

Grafana was added as a visualization layer.

The final architecture became:

```text
                     User Browser
                          |
                          |
                    Grafana :3000
                          |
                          |
             Zabbix Grafana Plugin
                          |
                          | HTTP / JSON-RPC
                          |
                    Apache2 :80
                          |
                          |
             /zabbix/api_jsonrpc.php
                          |
                          |
                    Zabbix API
                          |
                          |
                    Zabbix Data
```

Grafana does not replace Zabbix.

Zabbix remains responsible for monitoring, collecting and storing monitoring information, while Grafana provides an additional interface for querying and visualizing that information.

---

## Installing Grafana

Grafana was installed on the Debian Server using the official Grafana APT repository.

After installation, the service was started and enabled:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

The service was then restarted during the configuration process:

```bash
sudo systemctl restart grafana-server
```

The service status was verified with:

```bash
sudo systemctl status grafana-server --no-pager
```

The Grafana service was confirmed as:

```text
Active: active (running)
```

Grafana was configured to listen on:

```text
TCP 3000
```

The local web interface was also tested:

```bash
curl http://localhost:3000/
```

The response redirected to:

```text
/login
```

This confirmed that the Grafana web service was responding.

---

## Grafana Port

Grafana uses TCP port `3000` by default.

The listening services were inspected with:

```bash
ss -tulpn
```

The result showed Grafana listening on port `3000` and Apache listening on port `80`.

The distinction between the two services is important:

```text
Apache
192.168.1.117:80
        |
        +-- Zabbix Web
        +-- Zabbix API

Grafana
192.168.1.117:3000
        |
        +-- Grafana Web Interface
```

---

## Installing the Zabbix Plugin

Grafana requires a plugin to communicate with Zabbix.

The Alexander Zobnin Zabbix plugin was installed using:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
```

The installed plugins were verified with:

```bash
sudo grafana cli plugins ls
```

The Zabbix plugin was identified as:

```text
alexanderzobnin-zabbix-app @ 6.8.0
```

The Grafana service was then restarted:

```bash
sudo systemctl restart grafana-server
```

---

## Understanding the Zabbix API

The Zabbix web interface and Zabbix API are different endpoints.

The normal Zabbix web interface is:

```text
http://192.168.1.117/zabbix
```

The JSON-RPC API endpoint is:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

The API endpoint is used by applications such as Grafana to communicate programmatically with Zabbix.

The API uses JSON-RPC requests.

It expects HTTP `POST` requests containing JSON data.

---

## Testing the API

A first test was performed using a normal HTTP request:

```bash
curl -i http://localhost/zabbix/api_jsonrpc.php
```

The server returned:

```text
HTTP/1.0 412 Precondition Failed
```

This was not evidence that the API was broken.

The API expects a JSON-RPC `POST` request rather than a normal browser-style `GET` request.

A valid API request was then performed:

```bash
curl -i -X POST http://localhost/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

The API returned:

```json
{
  "jsonrpc": "2.0",
  "result": "7.4.15",
  "id": 1
}
```

This confirmed that the Zabbix API was operational.

The same test was performed using the server's network address:

```bash
curl -i -X POST http://192.168.1.117/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

The result again returned:

```text
7.4.15
```

This confirmed that the API was also reachable through the server's network interface.

---

## Zabbix API Token

Grafana was configured to authenticate against Zabbix using an API Token.

The token was created in Zabbix and used as the authentication credential for the Grafana data source.

The token itself is intentionally not documented in this repository.

Sensitive credentials should never be committed to Git.

---

## Grafana Data Source

A new Zabbix data source was created in Grafana.

The API URL configured in Grafana was:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

The general data source authentication method was configured as:

```text
No Authentication
```

The Zabbix connection authentication method was configured as:

```text
API Token
```

The previously generated Zabbix API Token was then entered into the plugin configuration.

---

## Troubleshooting

The first Grafana connection attempt failed.

Grafana reported:

```text
Could not connect to given url
```

The Grafana service logs were inspected with:

```bash
sudo journalctl -u grafana-server -n 50 --no-pager
```

The logs contained errors such as:

```text
Error querying Zabbix version
invalid character '<' looking for beginning of value
```

and:

```text
Error connecting Zabbix server
invalid character '<' looking for beginning of value
```

The error indicated that the plugin was receiving a response that was not valid JSON.

Another error appeared during the investigation:

```text
basic auth is not supported for Zabbix v7.2 and later
```

This helped identify that the authentication configuration needed to be corrected for the current Zabbix API version.

The Zabbix plugin was subsequently removed and reinstalled to restart the configuration from a clean state.

The plugin was removed with:

```bash
sudo grafana-cli plugins remove alexanderzobnin-zabbix-app
```

It was then installed again:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
```

The plugin installation was verified:

```bash
sudo grafana cli plugins ls
```

After restarting Grafana, the data source was configured again using the correct API endpoint and API Token authentication.

---

## Successful Connection

After the data source was reconfigured, Grafana successfully connected to Zabbix.

The Grafana data source test returned:

```text
Zabbix API version 7.4.15
```

This was the final confirmation that the integration was working.

The communication path was therefore successfully established:

```text
Grafana
   |
   v
Zabbix Plugin
   |
   v
Zabbix API
   |
   v
Zabbix 7.4.15
```

---

## Grafana and Zabbix Roles

The laboratory demonstrated that Grafana and Zabbix have different responsibilities.

### Zabbix

Zabbix remains responsible for:

* Monitoring hosts
* Collecting metrics
* Processing monitoring data
* Triggering problems
* Storing monitoring information
* Providing the API

### Grafana

Grafana provides:

* Data visualization
* Dashboards
* Graphs
* Panels
* Queries
* Additional visualization options

The Grafana Zabbix plugin acts as the connection between the two systems.

---

## Verification

The final environment was verified through multiple tests.

### Grafana service

```bash
sudo systemctl status grafana-server --no-pager
```

Expected result:

```text
Active: active (running)
```

### Grafana HTTP service

```bash
curl http://localhost:3000/
```

Expected behavior:

```text
/login
```

### Zabbix API

```bash
curl -i -X POST http://192.168.1.117/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

Expected result:

```text
"result":"7.4.15"
```

### Grafana Data Source

The Grafana data source test successfully returned:

```text
Zabbix API version 7.4.15
```

---

## Final Architecture

```text
                         Administration PC
                                |
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
              Zabbix Web                 Grafana
              TCP 80                   TCP 3000
                    |                       |
                    |                       |
                    v                       v
              Zabbix API             Zabbix Plugin
                    |                       |
                    +-----------+-----------+
                                |
                                v
                         Zabbix Data
                                |
                                v
                           Zabbix Server
                           TCP 10051
                                |
                                v
                         Zabbix Monitored
                              Host
```

The important point is that Grafana accesses Zabbix through its web API. The Zabbix Server process and the Zabbix API are separate components with different roles.

---

## What I Learned

* Grafana can be integrated with an existing Zabbix installation.
* Grafana does not replace Zabbix.
* The Zabbix Grafana plugin provides the integration layer.
* The Zabbix API is exposed through the Zabbix web frontend.
* The JSON-RPC API expects `POST` requests.
* A normal `GET` request to the API endpoint does not constitute a valid API query.
* `curl` can be used to independently verify whether the Zabbix API is working.
* The Zabbix API returned version `7.4.15`.
* Grafana uses TCP port `3000` by default.
* Apache continues to serve the Zabbix frontend and API on TCP port `80`.
* Zabbix Server continues operating independently on TCP port `10051`.
* API Tokens can be used instead of username/password authentication.
* Grafana logs are useful for identifying datasource and plugin communication problems.
* An HTML response received where JSON was expected can indicate an incorrect endpoint or configuration.
* Testing each component independently makes troubleshooting easier.

---

## Result

The Grafana and Zabbix integration was successfully completed.

Grafana `13.2.2` is running on the Debian Server and communicates with Zabbix `7.4.15` through the Zabbix JSON-RPC API using the Alexander Zobnin Zabbix plugin `6.8.0` and an API Token.

The environment is now prepared for the next stage: querying Zabbix data in Grafana and creating custom visualization panels and dashboards.

---

## Project Status

Completed

The Grafana data source successfully connected to the Zabbix API and returned the expected Zabbix API version.
