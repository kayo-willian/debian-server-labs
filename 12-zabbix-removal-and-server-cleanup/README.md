# Lab 12 - Zabbix Removal and Server Cleanup

## Overview

After migrating the Zabbix monitoring environment to a dedicated Debian virtual machine, the physical Debian Server no longer needed to run the Zabbix Server stack.

The physical server was therefore cleaned up, removing the monitoring infrastructure while preserving only the Zabbix Agent required for remote monitoring by the new monitoring VM.

The goal was to transform the physical Debian Server into a clean infrastructure server for future services.

---

## Previous Architecture

Before the cleanup, the physical Debian Server was running the complete monitoring stack:

```text
Physical Debian Server
192.168.1.117
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
├── Apache2
├── PHP
├── Grafana remnants
└── Zabbix Agent
```

After the migration, this architecture was redundant because the monitoring stack had already been moved to the dedicated VM.

---

## Target Architecture

The final architecture separates monitoring from infrastructure services:

```text
Debian Monitoring VM
192.168.1.219
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
└── Grafana
        │
        │ Zabbix Agent :10050
        ▼
Physical Debian Server
192.168.1.117
│
└── Zabbix Agent
```

The physical server is now dedicated to hosting future infrastructure services, while the monitoring VM is responsible for monitoring them.

---

## What Was Removed

The following components were removed from the physical Debian Server:

* Zabbix Server
* Zabbix Frontend
* Zabbix SQL scripts
* Zabbix repository package
* MariaDB
* Apache2
* PHP
* Grafana components and remnants

The following component was intentionally preserved:

* Zabbix Agent

The agent is required because the monitoring VM still needs to collect metrics from the physical server.

---

## Removing the Zabbix Server Stack

The Zabbix Server service was stopped and the server-side packages were removed:

```bash
sudo systemctl stop zabbix-server

sudo apt purge zabbix-server-mysql zabbix-frontend-php \
zabbix-apache-conf zabbix-sql-scripts zabbix-release -y
```

The Zabbix Server configuration and log directories were then removed:

```bash
sudo rm -rf /etc/zabbix/zabbix_server.conf
sudo rm -rf /var/log/zabbix
sudo rm -rf /var/lib/zabbix
```

The `/etc/zabbix` directory itself was preserved because it still contains the configuration required by the Zabbix Agent.

---

## Removing the Zabbix Database

The old Zabbix database was no longer required on the physical server.

The database and its dedicated MariaDB user were removed:

```bash
sudo mariadb
```

```sql
DROP DATABASE zabbix;
DROP USER 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## Removing MariaDB

After removing the Zabbix database, MariaDB was no longer required on the physical server:

```bash
sudo systemctl stop mariadb

sudo apt purge mariadb-server mariadb-client -y

sudo apt autoremove --purge -y
```

Remaining MariaDB directories were removed:

```bash
sudo rm -rf /var/lib/mysql
sudo rm -rf /etc/mysql
```

---

## Removing Apache2

Apache2 had originally been used to serve the Zabbix Web frontend.

Since the frontend was now running on the monitoring VM, Apache2 was removed:

```bash
sudo systemctl stop apache2

sudo apt purge apache2 apache2-bin apache2-data apache2-utils -y

sudo apt autoremove --purge -y
```

Remaining Apache configuration and web root directories were removed:

```bash
sudo rm -rf /etc/apache2
sudo rm -rf /var/www
```

---

## Removing PHP

PHP was previously required by the Zabbix Web frontend.

Since the frontend was removed from the physical server, PHP packages were also removed:

```bash
sudo apt purge 'php*' -y

sudo apt autoremove --purge -y
```

---

## Keeping the Zabbix Agent

The Zabbix Agent was intentionally kept installed and running.

This allows the monitoring VM to remotely collect metrics from the physical server.

The remaining Zabbix package was verified with:

```bash
dpkg -l | grep -E 'zabbix|apache2|php|mariadb|grafana'
```

The result showed only:

```text
zabbix-agent
```

---

## Service Verification

Running services were checked to confirm that no monitoring services other than the agent remained:

```bash
sudo systemctl --type=service --state=running | \
grep -E 'zabbix|apache2|php|mariadb|grafana'
```

The only result was:

```text
zabbix-agent.service loaded active running Zabbix Agent
```

This confirmed that the Zabbix Agent was the only remaining monitoring service.

---

## Port Verification

Listening ports were checked to confirm that the old monitoring services were no longer exposing their network ports:

```bash
sudo ss -tulpn | grep -E ':80 |:3306 |:10050 |:10051 |:3000 '
```

The only remaining relevant port was:

```text
0.0.0.0:10050
[::]:10050
```

Port `10050` belongs to the Zabbix Agent.

The following ports were no longer listening:

```text
80      Apache2
3306    MariaDB
10051   Zabbix Server
3000    Grafana
```

---

## Package Cleanup

The system was checked for unused dependencies:

```bash
sudo apt autoremove --purge -y
```

The system reported:

```text
Atualizando: 0, Instalando: 0, Removendo: 0, Não atualizando: 0
```

This confirmed that no additional packages were identified for automatic removal.

---

## Final Monitoring Test

The final test was performed from the dedicated monitoring VM.

From the Debian Monitoring Server:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

The result was:

```text
1
```

This confirmed that the monitoring VM could still communicate with the Zabbix Agent running on the physical Debian Server.

---

## Final State

The physical Debian Server was successfully cleaned of the Zabbix monitoring stack.

```text
Physical Debian Server
192.168.1.117
│
└── Zabbix Agent :10050
        ▲
        │
        │ Remote monitoring
        │
        ▼
Debian Monitoring Server
192.168.1.219
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
└── Grafana
```

The physical server is now free to host future infrastructure services without running its own monitoring stack.

New services installed on the physical server can be monitored centrally by the dedicated monitoring VM.

---

## Verification Summary

| Component                 | Physical Debian Server | Monitoring VM |
| ------------------------- | ---------------------- | ------------- |
| Zabbix Server             | Removed                | Running       |
| Zabbix Web                | Removed                | Running       |
| Zabbix Agent              | Running                | Running       |
| MariaDB                   | Removed                | Running       |
| Apache2                   | Removed                | Running       |
| PHP                       | Removed                | Running       |
| Grafana                   | Removed                | Running       |
| Monitoring responsibility | None                   | Centralized   |

---

## Result

The monitoring architecture was successfully separated from the physical infrastructure server.

The physical Debian Server now provides a clean base for future infrastructure services, while the dedicated Debian Monitoring Server centralizes Zabbix and Grafana.

This separation reduces unnecessary resource consumption on the physical server and establishes a clearer architecture for future labs.
