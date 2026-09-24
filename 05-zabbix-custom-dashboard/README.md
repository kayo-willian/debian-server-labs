# Lab 05 - Zabbix Custom Dashboard

## Description

This lab documents the creation and configuration of a custom Zabbix dashboard for the physical Debian Server.

Unlike the previous labs, which used the old network subnet, this laboratory uses the main local network:

```text
192.168.1.0/24
```

The monitored Debian server is currently using:

```text
192.168.1.117
```

The Zabbix installation and server configuration were documented in previous laboratories. This lab focuses specifically on **dashboard design, visualization, monitored metrics, widgets, and the interpretation of the information displayed by Zabbix**.

The objective is to create a centralized dashboard that provides a quick overview of the server's current condition and historical behavior.

---

## 1. Dashboard Overview

The custom dashboard was designed specifically for the `Debian Server` host.

The dashboard combines different Zabbix widgets so that the most relevant information can be observed without navigating through individual monitoring pages.

The final dashboard contains:

* CPU utilization
* Memory utilization
* System uptime
* Load average
* CPU historical graph
* Memory historical graph
* Active problems

![Dashboard Overview](screenshots/01-dashboard-overview.png)

The dashboard is organized to provide both **current-state information** and **historical information**.

Gauge widgets provide an immediate visual representation of resource utilization, while the historical graphs make it possible to observe changes over time. The Problems widget provides visibility into events detected by Zabbix.

---

## 2. Dashboard Layout

The dashboard was organized into seven main widgets:

| Widget             | Purpose                                       |
| ------------------ | --------------------------------------------- |
| CPU Utilization    | Displays current CPU usage                    |
| Memory Utilization | Displays current memory usage                 |
| System Uptime      | Displays how long the server has been running |
| Load Average       | Displays the current Linux load average       |
| CPU Graph          | Displays CPU activity over time               |
| Memory Graph       | Displays memory usage over time               |
| Problems           | Displays detected Zabbix problems             |

The intention was to keep the dashboard simple enough to be read quickly while still providing enough information to investigate the server's current state.

---

## 3. CPU Utilization

The CPU utilization widget displays the current percentage of CPU resources being used by the Debian server.

The server has:

```text
4 CPU cores
```

At the time the dashboard data was collected, the server reported:

```text
CPU utilization: 6.6633%
CPU idle: 93.3367%
```

Other CPU metrics available through Zabbix included:

```text
CPU user:    4.6056%
CPU system:  1.6895%
CPU iowait:  0.2059%
CPU softirq: 0.05043%
```

The gauge was configured with thresholds at:

```text
70%
90%
```

These thresholds provide visual reference points for increasing CPU utilization.

A CPU stress test was also performed during the laboratory using:

```bash
stress --cpu 4 --timeout 60
```

The purpose was to generate CPU activity and observe how the dashboard responds to an increase in workload.

---

## 4. Memory Utilization

The memory utilization widget displays the percentage of RAM currently being used by the server.

The monitored server has:

```text
Total memory: 1.79 GB
```

At the time of the captured monitoring data:

```text
Memory utilization: 33.7464%
Available memory: 1.19 GB
Available memory: 66.2536%
```

The server also had:

```text
Total swap: 1.51 GB
Free swap: 1.51 GB
Free swap: 100%
```

The memory gauge was configured to provide a quick visual indication of memory consumption.

---

## 5. System Uptime

The dashboard includes an Item value widget dedicated to system uptime.

The server boot time recorded by Zabbix was:

```text
24-09-2026 10:42:09
```

The uptime at the time of the captured data was approximately:

```text
01:41:05
```

Displaying uptime directly on the dashboard makes it possible to quickly determine whether the server has recently restarted.

---

## 6. Load Average

The Load Average widget displays the Linux load average for three different periods.

The captured values were:

| Period     | Load Average |
| ---------- | -----------: |
| 1 minute   |       0.1797 |
| 5 minutes  |       0.2549 |
| 15 minutes |       0.3608 |

Load average complements CPU utilization by providing another view of system workload.

The three values allow the current workload to be compared with the recent workload over longer intervals.

---

## 7. CPU Historical Graph

A historical CPU graph was added to the dashboard to complement the instantaneous CPU gauge.

While the gauge shows the current state, the graph makes it possible to observe how CPU activity changes over time.

The graph can be used to identify:

* Temporary CPU spikes
* Sustained CPU usage
* Periods of low activity
* Effects of workloads and tests
* Changes in CPU behavior over time

This is particularly useful when investigating an issue that is no longer occurring at the moment of analysis.

---

## 8. Memory Historical Graph

The memory graph provides historical information about RAM utilization.

The graph complements the memory gauge because the gauge only represents the current state, while the graph can reveal changes over time.

It can be used to identify:

* Increasing memory consumption
* Sudden changes in memory usage
* Periods of high utilization
* Long-term usage patterns

For a server with only 1.79 GB of RAM, observing memory behavior over time is particularly useful.

---

## 9. Problems Widget

The Problems widget was configured to display problems associated with the:

```text
Debian Server
```

During the laboratory, Zabbix detected a real problem related to a change in the number of installed packages:

```text
Linux: Number of installed packages has been changed
```

The captured event had:

```text
Event ID: 88
Trigger ID: 32564
Duration at capture: 9m 35s
```

The server currently reported:

```text
Installed packages: 513
```

This provided a practical demonstration of how a change detected by an item can result in a Zabbix trigger and subsequently appear as a problem on the dashboard.

The monitoring flow can be represented as:

```text
Monitored Item
      ↓
Trigger
      ↓
Event
      ↓
Problem
      ↓
Dashboard
```

This is one of the most important functions of the Problems widget because it allows operational events to be seen alongside the server's performance metrics.

---

## 10. Network Information

Although the dashboard focuses primarily on system resources, the monitored host also provides network information through Zabbix.

The Debian server uses:

```text
Interface: wlan0
IP: 192.168.1.117
```

The captured network metrics included:

| Metric                     |      Value |
| -------------------------- | ---------: |
| Incoming traffic           | 14.52 Kbps |
| Outgoing traffic           |  22.5 Kbps |
| Incoming errors            |          0 |
| Outgoing errors            |          0 |
| Incoming discarded packets |          5 |
| Outgoing discarded packets |          0 |
| Operational status         |         Up |

The network information provides additional context when analyzing the server's behavior.

---

## 11. Storage Information

The dashboard's monitored host also provides storage metrics.

The main storage device is:

```text
mmcblk2
```

At the time of the monitoring snapshot:

```text
Disk utilization: 4.9531%
Write rate: 5.4998 writes/s
```

The root filesystem is:

```text
/
```

with:

```text
Filesystem: ext4
Total: 26.07 GB
Used: 1.93 GB
Available: 22.79 GB
Usage: 7.7952%
Free inodes: 97.43%
```

These metrics are available through the host's monitoring data even though storage is not one of the primary widgets in this dashboard.

---

## 12. Server Monitoring Snapshot

The following values represent the server at the time the dashboard was documented:

| Metric                |     Value |
| --------------------- | --------: |
| CPU utilization       |   6.6633% |
| CPU idle              |  93.3367% |
| Memory utilization    |  33.7464% |
| Available memory      |   1.19 GB |
| Total memory          |   1.79 GB |
| Total swap            |   1.51 GB |
| Root filesystem usage |   7.7952% |
| Root filesystem total |  26.07 GB |
| Load average 1m       |    0.1797 |
| Load average 5m       |    0.2549 |
| Load average 15m      |    0.3608 |
| CPU cores             |         4 |
| Installed packages    |       513 |
| Processes             |       227 |
| Logged-in users       |         3 |
| Zabbix Agent          |    7.4.15 |
| Agent availability    | Available |

These values represent a snapshot of the monitored system and are expected to change as the server operates.

---

## 13. Dashboard Color Convention

The dashboard uses colors to make different types of information easier to distinguish.

The visual convention used during the lab was:

| Information      | Color  |
| ---------------- | ------ |
| Download         | Blue   |
| Upload           | Green  |
| Available memory | Green  |
| CPU utilization  | Orange |
| Problems         | Red    |

The colors are used as visual indicators. The actual metric values remain the primary source of information.

---

## 14. Dashboard Design Goals

The dashboard was designed around three main goals:

### Visibility

The most important server metrics should be visible immediately after opening the dashboard.

### Simplicity

The dashboard should avoid unnecessary information and focus on metrics useful for quickly understanding the server's condition.

### Troubleshooting

The dashboard should provide enough information to identify abnormal behavior and determine when further investigation is necessary.

The combination of gauges, Item values, graphs, and the Problems widget provides different levels of information without requiring multiple monitoring pages.

---

## 15. Real-Time vs Historical Monitoring

One of the main concepts demonstrated by this dashboard is the difference between current and historical monitoring.

### Current values

The gauges and Item value widgets provide an immediate view of the server.

Examples:

```text
CPU utilization
Memory utilization
Load average
Uptime
```

### Historical values

The CPU and memory graphs provide information about what happened over time.

This distinction is important when monitoring servers because a problem may have already disappeared by the time an administrator investigates it.

For example, a CPU spike may no longer be visible on the CPU gauge, but it can still be observed in the historical graph.

---

## 16. Zabbix Items, Triggers and Problems

The dashboard also helped demonstrate the relationship between the main Zabbix monitoring components.

### Items

Items collect individual pieces of information from the monitored host.

Examples include:

```text
CPU utilization
Memory utilization
Network traffic
Filesystem usage
Load average
Number of installed packages
```

### Triggers

Triggers evaluate collected values and conditions.

A trigger can determine when a monitored condition should generate an event.

### Problems

When a trigger condition is met, Zabbix can generate a problem that becomes visible through the Problems widget.

This makes the dashboard more than a collection of graphs. It also becomes a place where detected conditions can be investigated.

---

## 17. Dashboard Result

The final dashboard provides a centralized view of the Debian server's monitoring data.

It combines:

```text
                    Debian Server
                         │
          ┌──────────────┼──────────────┐
          │              │              │
        CPU           Memory          System
          │              │              │
       Gauge          Gauge       Uptime / Load
          │              │              │
          └──────────────┼──────────────┘
                         │
                   Historical Data
                         │
                  CPU / Memory Graphs
                         │
                         ▼
                    Problems
                         │
                         ▼
                 Custom Dashboard
```

The result is a dashboard that can be used as a central monitoring interface for the physical Debian server.

---

## 18. What I Learned

This laboratory focused on the practical use of Zabbix dashboards rather than the installation of the monitoring stack.

The main concepts practiced were:

* Creating a custom Zabbix dashboard
* Selecting appropriate dashboard widgets
* Configuring Item value widgets
* Configuring gauge widgets
* Configuring historical graphs
* Filtering the Problems widget by host
* Understanding the difference between current and historical metrics
* Interpreting CPU utilization
* Interpreting memory utilization
* Interpreting Linux load average
* Observing server uptime
* Understanding Zabbix Items
* Understanding Zabbix Triggers
* Understanding Zabbix Problems
* Using dashboard visual conventions
* Generating CPU activity and observing the monitoring response
* Using a dashboard as a centralized monitoring interface

---

## 19. Project Status

**Completed**

The custom Zabbix dashboard was successfully created for the `Debian Server` host.

The dashboard provides a centralized view of CPU utilization, memory utilization, uptime, load average, historical CPU and memory data, and detected problems.

The monitored server is using the main local network:

```text
192.168.1.0/24
```

with the Debian server accessible at:

```text
192.168.1.117
```

This laboratory therefore represents the current version of the monitoring environment, replacing the old subnet configuration used in the previous labs.
