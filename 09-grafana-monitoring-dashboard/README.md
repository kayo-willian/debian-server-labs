# Lab 09 - Grafana Monitoring Dashboard

## Overview

This lab focused on creating a monitoring dashboard with Grafana using data collected by the existing Zabbix environment.

The objective was to build a centralized and visual representation of the Debian Server's monitoring data, combining system, network, web service and application monitoring in a single dashboard.

Grafana was integrated with Zabbix through the Zabbix API using API Token authentication.

---

## Environment

The monitoring environment used in this lab consisted of:

* Debian Server
* Zabbix Server
* Zabbix Agent
* Zabbix Web
* MariaDB
* Apache2
* Grafana
* Grafana Zabbix Plugin

Zabbix remained responsible for collecting and storing the monitoring data, while Grafana was used as the visualization layer.

The Zabbix API was successfully tested and returned version `7.4.15`.

---

## Dashboard

A dedicated dashboard was created to provide an overview of the Debian Server and its monitored services.

The dashboard was organized into several monitoring sections.

### Overview

The overview section provides a quick view of the server's current state, including:

* CPU utilization
* Memory utilization
* Disk space
* System uptime
* Zabbix Agent availability
* Load average

### CPU

CPU monitoring includes:

* CPU utilization
* CPU idle time
* Current CPU usage
* Load average
* 1-minute load average
* 5-minute load average
* 15-minute load average

### Memory

Memory monitoring includes:

* Available memory
* Total memory
* Memory utilization
* Swap information

### Disk

Disk monitoring includes:

* Free disk space
* Used disk space
* Disk utilization

### Network

Network monitoring provides visualization of inbound and outbound interface traffic collected by Zabbix.

The dashboard represents network interface traffic rather than assuming that the measured traffic is exclusively Internet traffic.

### Web and Apache

The dashboard also integrates the web and Apache monitoring already configured in Zabbix.

This section includes:

* Web monitoring
* URL monitoring
* HTTP response time
* Apache availability
* Apache response time

These panels provide a visual representation of the availability and response of the web services monitored by Zabbix.

### System

Additional system information includes:

* Number of processes
* Running processes
* Zabbix Agent version
* System information

### Zabbix Problems

A dedicated panel displays active Zabbix problems directly in the Grafana dashboard.

This provides a centralized view of current monitoring events alongside the system metrics.

---

## Web Monitoring

One of the main purposes of the dashboard was to visualize the web monitoring configuration already implemented in Zabbix.

The main elements highlighted in this lab are:

* **Monitoramento Web**
* **Apache Response**
* **URL**

These elements demonstrate how service-level monitoring configured in Zabbix can be represented through Grafana panels.

A Host Card was also included in the dashboard as a visual element, but it is not part of the technical monitoring scope of this laboratory.

---

## Grafana and Zabbix Integration

The integration was performed using the Grafana Zabbix plugin.

The Grafana data source communicates with the Zabbix API and retrieves monitoring information from the Zabbix environment.

The connection was configured using:

* Zabbix API endpoint
* API Token authentication
* Zabbix data source
* Grafana Zabbix plugin

The API connection was validated successfully before creating the dashboard.

---

## Dashboard Structure

The final dashboard was divided into the following sections:

```text
Grafana Monitoring Dashboard
│
├── Overview
├── CPU
├── Memory
├── Disk
├── Network
├── Internet / Connectivity
├── Apache / Web Server
├── System
└── Current Zabbix Problems
```

The dashboard uses a short refresh interval and a recent time range to provide an up-to-date view of the monitored server.

---

### Dashboard JSON

The following Grafana dashboard JSON was generated with assistance from **Claude Sonnet 5** and adapted for the context of this laboratory.

It is included below as part of the project's technical documentation and preserves the dashboard configuration used during the lab.

### Prompt

LAB 09 - DEBIAN SERVER MONITORING - GRAFANA DASHBOARD PROMPT

I want you to create a complete, importable Grafana dashboard JSON file, to be used in my Lab 09, Debian Server Monitoring.

IMPORTANT: I do not want just a JSON example. I want you to generate the complete dashboard JSON, ready for me to save as a .json file and import into Grafana.

LAB CONTEXT

I am setting up an infrastructure lab on a physical Debian Server.

The server has the following stack:

Operating system: Debian Server
IP: 192.168.1.117
Zabbix Server: 7.4.15
Grafana: 13.2.2
Zabbix plugin for Grafana: Alexander Zobnin 6.8.0
Grafana datasource: alexanderzobnin-zabbix-datasource-1
Apache2: port 80
Zabbix Server: port 10051
Zabbix Agent: port 10050
Grafana: port 3000
MariaDB: port 3306

Zabbix was already working before Grafana was installed.

The Grafana to Zabbix integration has already been completed successfully.

Grafana was configured to access: http://192.168.1.117/zabbix/api_jsonrpc.php

The authentication method used is Zabbix API Token.

The datasource test in Grafana returned: Zabbix API version 7.4.15

So the communication between Grafana and Zabbix is working.

HOST AND GROUP

In Zabbix, the server host group is: Linux Servers

The host is: Debian Servers

I want the dashboard to be configured initially for these values.

Datasource: alexanderzobnin-zabbix-datasource-1

DASHBOARD GOAL

I want a beautiful, professional and visually complete dashboard, similar to professional Linux server monitoring dashboards, but without overdoing the number of elements.

This dashboard will be used to showcase my lab on GitHub.

The goal is to show as much useful information as possible about how my Debian Server is running, using the data already collected by Zabbix.

I want it to look like a real infrastructure monitoring dashboard, not just a few basic charts.

IMPORTANT ABOUT MY SERVER

The server is a machine with limited resources.

So the dashboard should be visually complete, but the queries should be relatively simple.

I do not want dozens of unnecessary queries.

Grafana is consuming a lot of memory on my server and I plan to remove Grafana after testing and documenting the lab.

So the goal is:

1. Create the dashboard.
2. Import it into Grafana.
3. Test it.
4. Take screenshots.
5. Document it on GitHub.
6. Remove Grafana from the server.

WHAT I WANT IN THE DASHBOARD

Create a professional visual structure.

You can organize it into sections.

1. HEADER / SUMMARY

At the top I want quick information about the server:

Host
Host Group
Zabbix Agent status
Uptime
Current CPU
Current memory
Current disk
Network traffic

You can use Stat, Gauge or another appropriate panel type.

I want the top area to make it possible to quickly understand the server state.

2. CPU

I want to show:

CPU usage in percent
CPU usage history
Average
Peak
Current value

If the available data allows it, also show:

User
System
I/O wait
Idle
Load average

I want at least one very visual CPU chart.

Use a 0 to 100 percent scale where it makes sense.

3. RAM MEMORY

I want to show:

Used memory
Available memory
Total memory, if possible
Usage percentage
Usage history
Swap, if it exists in Zabbix

I want a memory chart and a visual indicator of the current percentage.

4. DISK

I want to show:

Disk usage in percent
Used space
Free space
Filesystem, if possible
Usage history

If data is available for it, also show disk I/O.

I want it to be possible to quickly identify if storage is running low.

5. NETWORK

This part is very important.

I want to show the Debian Server network traffic.

Ideally:

Download / received traffic
Upload / sent traffic
Bytes received
Bytes sent
Throughput
Traffic history

If there is more than one interface, allow selecting or identifying the interface.

I want the dashboard to clearly show incoming and outgoing traffic.

6. INTERNET / CONNECTIVITY

In previous labs I already worked with web monitoring.

So I want to take advantage of that.

If the existing data in Zabbix allows it, include:

Availability
Ping
Latency
HTTP response time
Web monitoring
URL
Apache response

I already have in Zabbix monitoring related to:

Web monitoring
Apache Response
URL

These elements are important because they are part of my previous labs.

I want the Grafana dashboard to be able to display this data when it is available.

7. APACHE / WEB SERVER

Apache is installed on the Debian Server and uses TCP port 80.

I want a section to show information related to the web service.

If there are corresponding Items in Zabbix, show:

Response time
HTTP status
Availability
Monitored URL
Apache response

Do not invent metrics.

If a metric cannot be safely queried, leave the panel ready for me to adjust manually.

8. ZABBIX AGENT

I want an indicator showing whether the Zabbix Agent is available.

It can include:

Agent availability
Agent ping
Status
Last communication

9. LOAD AVERAGE

If the corresponding Item exists:

Load 1m
Load 5m
Load 15m

It can be a chart or three indicators.

10. PROCESSES

If the Items exist:

Number of processes
Active processes
Running processes

No need to invent Items.

11. UPTIME

I want to clearly show the server uptime.

It can be:

Stat
Gauge
Formatted text

Example: 12 days 04:32:10

If the unit provided by Zabbix is different, configure the appropriate transformation/unit in Grafana.

12. PROBLEMS / ALERTS

If the datasource/plugin allows it in a compatible way, I want a section showing current Zabbix problems.

Something like:

Problems
Severity
Status
Host
Timestamp

I want it to be visually easy to identify problems.

13. SYSTEM

If data is available:

Uptime
Load
Processes
CPU
Memory
Swap
Disk
Network

I want an overall view of the Linux server.

VARIABLES

I want the dashboard to have variables at the top.

Create at least:

Host Group
Host

Initial values:

Host Group: Linux Servers
Host: Debian Servers

The goal is to be able to select a different group/host in the future without having to edit every panel manually.

IMPORTANT: the variables need to be compatible with the plugin alexanderzobnin-zabbix-datasource and with the datasource alexanderzobnin-zabbix-datasource-1.

DATASOURCE

All panels must use: alexanderzobnin-zabbix-datasource-1

Do not create a fictitious datasource.

Do not use Prometheus.

Do not use InfluxDB.

Do not use Loki.

Do not use a generic datasource.

The only datasource should be Zabbix.

COMPATIBILITY

The JSON must be compatible with Grafana 13.2.2 and with the Alexander Zobnin Zabbix plugin 6.8.0.

Zabbix is 7.4.15.

Use the current Grafana dashboard JSON model.

ABOUT THE ITEMS

This point is VERY important.

I do not want you to simply make up Item names.

Since you cannot directly query my Zabbix, make the dashboard as robust as possible.

When you need to reference Items, use queries compatible with the Alexander Zobnin plugin.

If filters/regex for Items are needed, use broad filters that are clearly editable.

For example, if you need something related to CPU, use a structure that I can later adjust to the real Item.

I am willing to manually change the Item names in Grafana.

But I want the JSON to already come with the full visual structure ready.

VISUAL STYLE

I want a beautiful dashboard.

Style:

Professional infrastructure monitoring
Linux server monitoring
Dark theme
Visually organized
No excess of colors
Good hierarchy
Well distributed panels
Clear titles
Correct units
Line charts for history
Stat/Gauge for current values
Thresholds where it makes sense

I do not want something childish or full of effects.

I want something that looks good in a screenshot to put on GitHub.

THRESHOLDS

Use reasonable thresholds for metrics such as:

CPU:
Green: 0 to 60
Yellow: 60 to 75
Orange: 75 to 90
Red: 90 to 100

Memory:
Green: 0 to 60
Yellow: 60 to 75
Orange: 75 to 90
Red: 90 to 100

Disk:
Green: 0 to 60
Yellow: 60 to 75
Orange: 75 to 90
Red: 90 to 100

But do not apply absurd thresholds to metrics where percentage does not make sense.

TIME RANGE

Set initially: Last 6 hours

And refresh: 30s

If you think 1m is more appropriate to reduce load on the server, you can use 1m.

TIMEZONE

Very important: the Debian is correctly configured as America/Belem.

The server shows Local time: -03.

Grafana should use the browser timezone or America/Belem, avoiding showing times in UTC.

Do not change the Debian timezone.

TITLE

The dashboard title should be: Lab 09 - Debian Server Monitoring

Tags: zabbix, grafana, debian, linux, lab-09

VISUAL ORGANIZATION

I want approximately this organization:

Header: LAB 09 - DEBIAN SERVER MONITORING

Host Group: Linux Servers
Host: Debian Servers

Row of four: CPU, MEMORY, DISK, UPTIME

Row of two: CPU USAGE graph, MEMORY USAGE graph

Row of two: DISK USAGE graph, NETWORK TRAFFIC graph

Full width: NETWORK / INTERNET TRAFFIC

Row of two: SYSTEM LOAD, PROCESSES

Row of two: APACHE / WEB RESPONSE, ZABBIX AGENT

Full width: CURRENT ZABBIX PROBLEMS

You can improve this organization if you have a better idea.

DO NOT DO

Do not do:

Prometheus
Node Exporter
InfluxDB
Telegraf
Loki
Docker
Kubernetes
fictitious data
made up metrics
fictitious datasource
API Token inside the JSON
password inside the JSON

The API Token must not appear in the JSON.

The datasource already exists in Grafana.

IMPORTANT ABOUT THE JSON

I want you to deliver:

1. The complete JSON.
2. Valid.
3. No comments inside the JSON.
4. No ellipsis placeholders.
5. No fake fields that would prevent import.
6. No Markdown inside the JSON file.
7. Compatible with Grafana 13.2.2.
8. Datasource configured for alexanderzobnin-zabbix-datasource-1.
9. Initial Host Group Linux Servers.
10. Initial Host Debian Servers.

Before delivering it, do a logical validation of the JSON:

Valid JSON?
Unique panel IDs?
Valid variables?
Correct datasource?
No token/password included?
Structure compatible with Grafana 13?
No trailing commas?
No invalid comments?
Importable dashboard?

WHAT I WANT TO RECEIVE

At the end, deliver the complete JSON inside a code block.

Do not summarize.

Do not give me only a partial structure.

I want the complete dashboard to save as: lab-09-grafana-debian-server.json

I will then import it into Grafana and test it.

If any panel shows No data, I will manually adjust the Items/regex afterward.

The most important thing is that the dashboard is beautiful, organized, complete and actually importable, using my existing Zabbix datasource.

### Complete Dashboard JSON

```text
{
  "id": null,
  "uid": "lab09-debian-server-monitoring",
  "title": "Lab 09 - Debian Server Monitoring",
  "description": "Dashboard de monitoramento do Debian Server (Lab 09) via Zabbix 7.4, usando o plugin Alexander Zobnin Zabbix datasource. CPU, memoria, disco, rede, conectividade web, Apache e problemas ativos.",
  "tags": [
    "zabbix",
    "grafana",
    "debian",
    "linux",
    "lab-09"
  ],
  "timezone": "browser",
  "editable": true,
  "graphTooltip": 1,
  "schemaVersion": 39,
  "version": 1,
  "refresh": "1m",
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "templating": {
    "list": [
      {
        "name": "hostgroup",
        "type": "query",
        "label": "Host Group",
        "datasource": {
          "type": "alexanderzobnin-zabbix-datasource",
          "uid": "alexanderzobnin-zabbix-datasource-1"
        },
        "query": "groups()",
        "current": {
          "selected": true,
          "text": "Linux Servers",
          "value": "Linux Servers"
        },
        "refresh": 1,
        "regex": "",
        "sort": 1,
        "multi": false,
        "includeAll": false,
        "hide": 0
      },
      {
        "name": "host",
        "type": "query",
        "label": "Host",
        "datasource": {
          "type": "alexanderzobnin-zabbix-datasource",
          "uid": "alexanderzobnin-zabbix-datasource-1"
        },
        "query": "hosts($hostgroup)",
        "current": {
          "selected": true,
          "text": "Debian Servers",
          "value": "Debian Servers"
        },
        "refresh": 1,
        "regex": "",
        "sort": 1,
        "multi": false,
        "includeAll": false,
        "hide": 0
      }
    ]
  },
  "annotations": {
    "list": [
      {
        "name": "Annotations & Alerts",
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "type": "dashboard",
        "builtIn": 1
      }
    ]
  },
  "panels": [
    {
      "id": 1,
      "type": "row",
      "title": "Overview",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "panels": []
    },
    {
      "id": 2,
      "type": "gauge",
      "title": "CPU Usage",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 0,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 3,
      "type": "gauge",
      "title": "Memory Usage",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 4,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Memory utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 4,
      "type": "gauge",
      "title": "Disk Free (/)",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 8,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on / (percentage)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 10
              },
              {
                "color": "yellow",
                "value": 25
              },
              {
                "color": "green",
                "value": 40
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 5,
      "type": "stat",
      "title": "Uptime",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 12,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "System uptime"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 6,
      "type": "stat",
      "title": "Zabbix Agent",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 16,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Agent ping"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "background",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          },
          "mappings": [
            {
              "type": "value",
              "options": {
                "0": {
                  "text": "DOWN",
                  "color": "red",
                  "index": 1
                },
                "1": {
                  "text": "UP",
                  "color": "green",
                  "index": 0
                }
              }
            }
          ]
        },
        "overrides": []
      }
    },
    {
      "id": 7,
      "type": "stat",
      "title": "Load Average (1m)",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 20,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (1m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 2,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 1.5
              },
              {
                "color": "red",
                "value": 3
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 8,
      "type": "row",
      "title": "CPU",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 7
      },
      "panels": []
    },
    {
      "id": 9,
      "type": "timeseries",
      "title": "CPU Utilization (%)",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 25,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 10,
      "type": "stat",
      "title": "CPU - Current",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 16,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 11,
      "type": "stat",
      "title": "CPU - Peak (6h)",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 20,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          },
          "custom": {}
        },
        "overrides": []
      }
    },
    {
      "id": 12,
      "type": "stat",
      "title": "CPU - Average (6h)",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 16,
        "y": 12
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 13,
      "type": "stat",
      "title": "CPU - Idle",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 20,
        "y": 12
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU idle time"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 14,
      "type": "timeseries",
      "title": "Load Average (1m / 5m / 15m)",
      "gridPos": {
        "h": 7,
        "w": 24,
        "x": 0,
        "y": 16
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (1m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (5m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "C",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (15m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 15,
      "type": "row",
      "title": "Memory",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 23
      },
      "panels": []
    },
    {
      "id": 16,
      "type": "timeseries",
      "title": "Memory Usage History",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Available memory"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Total memory"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 20,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 17,
      "type": "gauge",
      "title": "Memory Utilization %",
      "gridPos": {
        "h": 8,
        "w": 4,
        "x": 16,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Memory utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 18,
      "type": "gauge",
      "title": "Swap Usage %",
      "gridPos": {
        "h": 8,
        "w": 4,
        "x": 20,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free swap space in %"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 20
              },
              {
                "color": "yellow",
                "value": 50
              },
              {
                "color": "green",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 19,
      "type": "row",
      "title": "Disk",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 32
      },
      "panels": []
    },
    {
      "id": 20,
      "type": "timeseries",
      "title": "Disk Space Usage (/)",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 33
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on /"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Used disk space on /"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 25,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 21,
      "type": "gauge",
      "title": "Free Disk Space (/) %",
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 16,
        "y": 33
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on / (percentage)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 10
              },
              {
                "color": "yellow",
                "value": 25
              },
              {
                "color": "green",
                "value": 40
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 22,
      "type": "row",
      "title": "Network",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 41
      },
      "panels": []
    },
    {
      "id": 23,
      "type": "timeseries",
      "title": "Network Traffic (In / Out - eth0)",
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 42
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits received"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits sent"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 20,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 24,
      "type": "stat",
      "title": "Current Inbound Traffic",
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 0,
        "y": 50
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits received"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 25,
      "type": "stat",
      "title": "Current Outbound Traffic",
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 12,
        "y": 50
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits sent"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "purple",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 26,
      "type": "row",
      "title": "Internet / Connectivity",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 54
      },
      "panels": []
    },
    {
      "id": 27,
      "type": "stat",
      "title": "Web Scenario Availability",
      "gridPos": {
        "h": 6,
        "w": 6,
        "x": 0,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Web monitoring"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Download speed for scenario"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 28,
      "type": "timeseries",
      "title": "Ping / Latency",
      "gridPos": {
        "h": 6,
        "w": 9,
        "x": 6,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "ICMP ping"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "ms",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 29,
      "type": "timeseries",
      "title": "HTTP Response Time",
      "gridPos": {
        "h": 6,
        "w": 9,
        "x": 15,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Web monitoring"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Response time for step"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 30,
      "type": "row",
      "title": "Apache / Web Server",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 61
      },
      "panels": []
    },
    {
      "id": 31,
      "type": "stat",
      "title": "Apache Availability",
      "gridPos": {
        "h": 6,
        "w": 6,
        "x": 0,
        "y": 62
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Apache"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Apache: Service status"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "background",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          },
          "mappings": [
            {
              "type": "value",
              "options": {
                "0": {
                  "text": "DOWN",
                  "color": "red",
                  "index": 1
                },
                "1": {
                  "text": "UP",
                  "color": "green",
                  "index": 0
                }
              }
            }
          ]
        },
        "overrides": []
      }
    },
    {
      "id": 32,
      "type": "timeseries",
      "title": "Apache Response Time",
      "gridPos": {
        "h": 6,
        "w": 18,
        "x": 6,
        "y": 62
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Apache"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Apache: Response time"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 33,
      "type": "row",
      "title": "System",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 68
      },
      "panels": []
    },
    {
      "id": 34,
      "type": "timeseries",
      "title": "Number of Processes",
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes running"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 35,
      "type": "stat",
      "title": "Processes - Current",
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 12,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 250
              },
              {
                "color": "red",
                "value": 400
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 36,
      "type": "stat",
      "title": "Zabbix Agent Version",
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 18,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Version of zabbix_agent(d) running"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "value",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 37,
      "type": "row",
      "title": "Current Zabbix Problems",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 76
      },
      "panels": []
    },
    {
      "id": 38,
      "type": "table",
      "title": "Current Problems",
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 77
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 4,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "proxy": {
            "filter": ""
          },
          "tags": {
            "filter": ""
          },
          "options": {
            "minSeverity": 2,
            "sortProblems": "default",
            "showAckButton": true,
            "limit": 100
          },
          "resultFormat": "table"
        }
      ],
      "options": {
        "showHeader": true,
        "cellHeight": "sm",
        "footer": {
          "show": false
        }
      },
      "fieldConfig": {
        "defaults": {
          "custom": {
            "align": "auto",
            "cellOptions": {
              "type": "auto"
            }
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Severity"
            },
            "properties": [
              {
                "id": "custom.cellOptions",
                "value": {
                  "type": "color-background"
                }
              }
            ]
          }
        ]
      }
    }
  ],
  "style": "dark",
  "fiscalYearStartMonth": 0,
  "liveNow": false,
  "weekStart": ""
}
```

---

## Result

The laboratory resulted in a centralized Grafana dashboard connected to Zabbix.

The dashboard brings together infrastructure and service monitoring data in a single interface, allowing information collected by Zabbix to be visualized through different Grafana panels.

The project provided practical experience with:

* Grafana
* Zabbix integration
* Zabbix API
* API Token authentication
* Data sources
* Dashboard creation
* Monitoring panels
* System metrics
* Network metrics
* Web monitoring
* Apache monitoring
* Zabbix problems

---

## Skills Demonstrated

* Linux server monitoring
* Zabbix
* Grafana
* Zabbix API
* API Token authentication
* Monitoring dashboards
* CPU monitoring
* Memory monitoring
* Disk monitoring
* Network monitoring
* Apache monitoring
* Web monitoring
* Infrastructure visualization
* Technical documentation
