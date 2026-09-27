# Lab 13 - Grafana Custom Monitoring Dashboard

## Overview

This lab focuses on rebuilding and progressively developing a custom monitoring dashboard using Grafana and Zabbix.

The monitoring environment was previously migrated from the physical Debian Server to a dedicated Debian virtual machine. Grafana was subsequently reinstalled on the virtual machine to restore the monitoring environment.

The complete migration process is documented separately in **Lab 11 - Migrating the Zabbix Monitoring Server to a Virtual Machine**, so this lab only provides the necessary context before focusing on the Grafana dashboard itself.

This is an **ongoing lab**. The dashboard will be developed progressively as new monitoring requirements, visualizations, metrics, thresholds, and improvements are added.

## Environment

| Component            | Configuration               |
| -------------------- | --------------------------- |
| Grafana Server       | Debian Monitoring Server VM |
| Grafana              | 13.x                        |
| Zabbix               | 7.4.x                       |
| Datasource           | Zabbix                      |
| Monitored Host       | Debian Server               |
| Monitored Host IP    | 192.168.1.117               |
| Monitoring Server IP | 192.168.1.219               |
| Virtualization       | VirtualBox                  |

## Previous Monitoring Architecture

The original physical Debian Server previously hosted the complete monitoring stack, including Zabbix Server, Zabbix Web, MariaDB, Apache2, and Grafana.

As documented in Lab 11, the monitoring environment was migrated to a dedicated Debian virtual machine.

The resulting architecture is:

```text
Ubuntu Host
│
├── Debian Server
│   └── Zabbix Agent
│       └── 192.168.1.117
│
└── Debian Monitoring Server
    ├── Zabbix Server
    ├── Zabbix Web
    ├── MariaDB
    ├── Apache2
    └── Grafana
        └── 192.168.1.219
```

The physical Debian Server is monitored remotely by the Zabbix Server running on the virtual machine.

## Grafana Reinstallation

After the monitoring environment was moved to the virtual machine, Grafana was reinstalled on the Debian Monitoring Server.

The purpose of this step was to restore the visualization layer of the monitoring environment without repeating the complete migration process documented in Lab 11.

Grafana is accessed through:

```text
http://192.168.1.219:3000
```

The Zabbix datasource is used to retrieve monitoring data from the Zabbix Server.

## Custom Dashboard

The main objective of this lab is to create a personal Grafana dashboard from scratch.

Instead of relying entirely on pre-existing dashboards, the dashboard is being designed manually through the Grafana interface.

The dashboard will progressively include monitoring information such as:

* CPU utilization
* Load average
* Memory usage
* Available memory
* Disk usage
* Disk activity
* Network traffic
* Network errors
* System information
* Service availability
* Host availability
* Other relevant Linux monitoring metrics

The exact panels and organization may change as the dashboard evolves.

## Dashboard Development Approach

The dashboard is intentionally being developed incrementally.

The workflow is:

```text
Select metric
    ↓
Create panel
    ↓
Configure Zabbix query
    ↓
Configure visualization
    ↓
Configure units and thresholds
    ↓
Test the result
    ↓
Keep or improve the panel
```

This approach allows each visualization to be tested individually instead of attempting to build the entire dashboard at once.

## Dashboard Export

Once the dashboard reaches a stable state, it will be exported from Grafana as JSON.

The exported JSON will become part of the laboratory documentation and can be used to reproduce the dashboard in another Grafana environment.

The final project structure is expected to include the dashboard export, for example:

```text
13-grafana-custom-monitoring-dashboard/
├── README.md
└── dashboard.json
```

The JSON file will only be added when the dashboard reaches a suitable point for export.

## Current Status

This laboratory is intentionally **open and ongoing**.

The objective is not to complete the dashboard in a single session. New panels, metrics, thresholds, layouts, and improvements can be added progressively.

The lab will be considered complete only when the dashboard reaches the desired final state and its JSON export has been documented.

## Result

This laboratory will document the progressive development of a custom Grafana monitoring dashboard using Zabbix as the data source.

It also demonstrates the complete workflow from restoring the visualization environment to manually designing, testing, and eventually exporting a reusable Grafana dashboard.
