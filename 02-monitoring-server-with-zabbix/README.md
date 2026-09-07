# Lab 02: Monitoring Server with Zabbix

Hands-on laboratory focused on deploying a complete monitoring solution on a resource constrained physical Debian Server, covering database preparation, repository configuration, service installation, web interface setup, distributed agent monitoring, and data retention management.

This lab documents the complete path from an empty Debian Server to a functional monitoring system observing two hosts over the network, including every error encountered and resolved during the process.

The goal is the same as in previous laboratories: not only to make the system work, but to understand why each component exists, how the pieces interact, how to verify each stage, and how to recover from failure.

## Objective

The objectives of this lab were:

* Understand what a monitoring system is and why it is a core tool in network and server administration
* Install and configure a three tier web application: database, application server, and web frontend
* Import an initial database schema and understand what it contains
* Tune a service to operate within limited hardware resources
* Monitor a second host across the network using the Zabbix agent
* Verify communication between server and agent end to end
* Configure data retention to protect limited disk space
* Practice reading service logs as the primary troubleshooting tool

---

## 1. What Is a Monitoring System

A monitoring system continuously collects measurements from computers and network equipment, stores them, evaluates rules against them, and notifies administrators when something breaks or is about to break.

Without monitoring, an administrator only learns about a failure when a user complains. With monitoring, the system detects the failure itself, records when it started, and can notify responsible people automatically.

Zabbix is an enterprise grade open source monitoring solution. It was chosen for this lab because it is widely used in the job market, runs acceptably on modest hardware when correctly tuned, and exposes every fundamental monitoring concept: hosts, items, triggers, events, alerts, dashboards, and templates.

### Why This Matters for Network Administration

Monitoring is a daily activity in network operations. Almost every infrastructure job expects familiarity with at least one monitoring platform. The concepts learned here transfer directly to any other tool because the concepts, not the product, are the actual skill.

---

## 2. Lab Environment

### Server Hardware

| Component        | Specification       |
| ---------------- | ------------------- |
| Device           | Positivo Motion     |
| CPU              | Intel Atom x5-Z8350 |
| RAM              | 2 GB DDR3           |
| Storage          | Approximately 32 GB |
| Network          | Wi-Fi               |
| Operating System | Debian 13 (trixie)  |
| Desktop          | None                |

The same physical server from Lab 01 was reused. The limited hardware is intentional and makes resource tuning a mandatory part of the exercise instead of an optional refinement.

### Software Versions

| Component   | Version                     |
| ----------- | --------------------------- |
| Debian      | 13 (trixie)                 |
| Kernel      | Linux 6.12 (amd64)          |
| MariaDB     | 11.8                        |
| Zabbix      | 7.4 (official repository)   |
| Web Server  | Apache 2.4                  |
| PHP         | 8.4                         |

The operating system version was confirmed before choosing the repository package with:

```bash
cat /etc/os-release
```

This step matters because repository package names include the target distribution version. Installing a package built for a different Debian release causes dependency errors.

### Network

The laboratory network architecture from Lab 01 was kept unchanged.

```text
Administration Workstation
            |
            | SSH and HTTP
            |
            v
Debian Server (Zabbix Server, Database, Frontend)
            |
            | SNMP capable network (planned)
            v
Network Devices
```

The Zabbix server, the database, and the web frontend all run on the Debian server. The administration workstation runs a Zabbix agent and is monitored by the server as a regular managed host.

Note: in the commands below, placeholder values are used in place of real addresses and credentials. Replace them during installation and never publish real values in a public repository.

| Placeholder             | Meaning                              |
| ----------------------- | ------------------------------------ |
| `<SERVER_IP>`           | IP address of the Debian server      |
| `<WORKSTATION_IP>`      | IP address of the monitored Ubuntu workstation |
| `<STRONG_DB_PASSWORD>`  | Password of the Zabbix database user |
| `<STRONG_UI_PASSWORD>`  | Password of the Zabbix web Admin user |

---

## 3. Zabbix Architecture

Zabbix is composed of several independent components that communicate with each other. Understanding this architecture explains almost every configuration file used in this lab.

```text
                 Web Browser
                      |
                      | HTTP
                      v
  +--------------------------------------+
  |            Debian Server             |
  |                                      |
  |  Apache + PHP  (Frontend)            |
  |         |                            |
  |         | SQL                        |
  |         v                            |
  |  MariaDB  (Historical data)          |
  |         ^                            |
  |         |                            |
  |  Zabbix Server  (Collector/Worker)   |
  |         ^                            |
  |         | TCP 10051                  |
  +---------|----------------------------+
            |
            | TCP 10050
            v
     Zabbix Agent on remote hosts
```

| Component       | Role                                                              | Listens on          |
| --------------- | ----------------------------------------------------------------- | ------------------- |
| Zabbix Server   | Collects data, evaluates triggers, writes to the database          | TCP 10051           |
| Zabbix Agent    | Small service installed on each monitored host, answers metric queries | TCP 10050  |
| Database        | Stores configuration, collected values, and events                | Local socket / 3306 |
| Frontend        | PHP application served by Apache, the web interface              | HTTP 80             |

The frontend never collects anything itself. It reads from the database. The server collects and writes. The agent answers questions about the machine where it runs. When something stops working, identifying which of these four components broke is the first diagnostic step.

---

## 4. Important Concept: Database User Versus Application User

This lab produced two completely different credential pairs, and confusing them caused a real login failure that is documented in the troubleshooting section.

| Credential pair | Where it lives            | Used by                          | Purpose                        |
| --------------- | ------------------------- | -------------------------------- | ------------------------------- |
| Database user   | Inside MariaDB            | Zabbix Server and Frontend       | Internal application to database communication |
| Web user        | Inside the Zabbix database | The human opening the browser   | Login to the administrative interface |

The database user is created by the administrator with SQL statements. The web user is created automatically by the schema import with a well known default password, and must be changed immediately after the first login. This separation between service accounts and human accounts exists in practically every web application.

---

# 5. Installation Procedure

## Step 1: Confirm the Distribution Version

The repository package depends on the exact Debian release.

```bash
cat /etc/os-release
```

Expected result: `VERSION_ID="13"` and codename `trixie`.

The version determines which Zabbix repository package must be downloaded in a later step.

## Step 2: Create a Swap File

The server has 2 GB of RAM and will host a database, a PHP application, and a monitoring server simultaneously. A swap file acts as an emergency overflow area and prevents the out of memory killer from terminating services.

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

Verification: `free -h` must list the swap total as approximately 1 GiB, and the `/etc/fstab` entry guarantees it survives a reboot.

Why swap exists: when physical RAM is exhausted the system starts moving inactive memory pages to disk. Without swap, memory exhaustion terminates processes abruptly. With swap, the system degrades gracefully instead.

## Step 3: Install and Verify the Database Server

```bash
sudo apt update
sudo apt install mariadb-server -y
systemctl status mariadb
ss -tulpn | grep 3306
```

Expected result: the service shows `active (running)` and the socket listens on `127.0.0.1:3306`.

The listen address deserves attention: MariaDB binds to `127.0.0.1` by default, which means it accepts connections only from the local machine. This is correct for this lab because the Zabbix server and frontend run on the same host. A database exposed to the network without need is a security defect.

## Step 4: Create the Database and the Application User

```bash
sudo mysql
```

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '<STRONG_DB_PASSWORD>';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

Explanation of each statement:

| Statement | Why it exists |
| --------- | ------------- |
| `CREATE DATABASE ... utf8mb4 ... utf8mb4_bin` | Zabbix requires this exact character set and binary collation for correct storage and comparisons |
| `CREATE USER ... @'localhost'` | Creates a dedicated account restricted to local connections, following the principle of least privilege |
| `GRANT ALL PRIVILEGES ON zabbix.*` | Grants rights only over the Zabbix database, not over the entire database server |
| `SET GLOBAL log_bin_trust_function_creators = 1` | Temporarily permits the schema import to create stored functions |

Important detail: `SET GLOBAL` changes a runtime variable that does not survive a database restart. It must be set again before an import if the database was restarted, and should be set back to `0` after the import completes.

## Step 5: Add the Official Zabbix Repository

Distribution repositories may lag behind or lack some components. Zabbix publishes its own repository with up to date packages.

The correct package name includes the distribution version. At the time of this lab, Debian 13 support existed starting with Zabbix 7.4.

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest+debian13_all.deb
sudo dpkg -i zabbix-release_latest+debian13_all.deb
sudo apt update
```

The `apt update` after installing the repository package is mandatory. Skipping it causes the next installation step to fail with package not found errors, even though the package names are correct. This happened in practice and is documented in the troubleshooting section.

Verification that the repository is active:

```bash
apt-cache policy zabbix-server-mysql | head -5
```

The candidate version must reference `repo.zabbix.com`.

## Step 6: Install the Zabbix Components

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

What each package provides:

| Package                | Function                                             |
| ---------------------- | ---------------------------------------------------- |
| `zabbix-server-mysql`  | The monitoring server compiled for MySQL/MariaDB      |
| `zabbix-frontend-php`  | The PHP web interface                                |
| `zabbix-apache-conf`   | Apache configuration that publishes the frontend     |
| `zabbix-sql-scripts`   | SQL schema and data used to initialize the database  |
| `zabbix-agent`         | Monitoring agent, used here for local self monitoring |

During this installation the package manager also installed and enabled Apache and PHP automatically, switched Apache to the prefork multiprocessing module required by mod_php, and enabled the Zabbix Apache configuration.

Verification:

```bash
dpkg -l | grep zabbix
```

All five Zabbix packages plus `zabbix-release` must appear in the list with status `ii`.

## Step 7: Import the Initial Schema

The schema archive contains the complete database structure: more than 150 tables covering hosts, items, history, triggers, users, and events.

Locate the actual path of the schema file inside the installed package:

```bash
dpkg -L zabbix-sql-scripts | grep server.sql.gz
```

In current Zabbix versions the file is located at:

```text
/usr/share/zabbix/sql-scripts/mysql/server.sql.gz
```

Older documentation frequently cites `/usr/share/zabbix-sql-scripts/mysql/server.sql.gz`, which no longer exists. When a tutorial references a path that does not exist, listing the files of the owning package with `dpkg -L` reveals the correct location. This is a generalizable troubleshooting technique.

Run the import with the path returned by the command above:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

The command takes several minutes on this hardware and appears frozen near the end. This is normal.

Verify the import:

```bash
sudo mysql -e "USE zabbix; SHOW TABLES;" | wc -l
```

A count above 150 confirms the schema was fully imported.

After the import, restore the runtime flag:

```bash
sudo mysql -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

## Step 8: Configure the Zabbix Server

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Ensure the following parameters are set:

```text
DBName=zabbix
DBUser=zabbix
DBPassword=<STRONG_DB_PASSWORD>
CacheSize=64M
HistoryCacheSize=16M
```

`DBName`, `DBUser`, and `DBPassword` connect the server to the database created earlier. The two cache directives reduce the amount of RAM the server reserves for caching, which is essential on a 2 GB machine. On default settings the server allocates significantly more memory and competes with the database and the web server.

## Step 9: Start and Enable the Services

```bash
sudo systemctl restart zabbix-server apache2
sudo systemctl enable zabbix-server apache2
```

Verify:

```bash
systemctl status zabbix-server --no-pager
systemctl status apache2 --no-pager
```

Both must show `active (running)`.

If the Zabbix server fails to start, the log files contain the reason in almost every case:

```bash
journalctl -u zabbix-server -n 30 --no-pager
sudo tail -30 /var/log/zabbix/zabbix_server.log
```

## Step 10: Complete the Web Wizard

Open the frontend from the administration workstation:

```text
http://<SERVER_IP>/zabbix/
```

The installation wizard requests the database connection parameters:

| Field                | Value                  |
| -------------------- | ---------------------- |
| Database type        | MySQL                  |
| Database host        | localhost              |
| Database port        | 0 (default)            |
| Database name        | zabbix                 |
| Store credentials in | Plain text             |
| User                 | zabbix                 |
| Password             | `<STRONG_DB_PASSWORD>` |
| Database TLS         | Disabled               |

TLS is unnecessary here because the connection never leaves the server: both processes run on the same machine.

The next screen defines the instance name and default timezone. Set the timezone to the real local zone so event timestamps and graphs match reality.

On completion the wizard writes the connection settings to:

```text
/etc/zabbix/web/zabbix.conf.php
```

This file is how the frontend remembers the database parameters. Reading this file after installation is a quick way to recall which credentials the frontend uses.

## Step 11: First Login and Immediate Hardening

Default credentials of a freshly imported Zabbix instance:

```text
Username: Admin
Password: zabbix
```

The username has a capital A. The lowercase form does not authenticate.

This password is public knowledge because it is identical in every new Zabbix installation. Changing it is not optional. Navigate to Users, Users, select Admin, and change the password to a value stored in a password manager.

If repeated failed logins occur, Zabbix temporarily locks the account. Waiting briefly before retrying resolves the lockout.

---

# 6. Adding the Second Monitored Host

At this point the server monitors itself only. A monitoring system observing a single machine teaches little about distributed operation. The second host is the administration workstation.

## Step 12: Install and Configure the Agent on the Workstation

```bash
sudo apt install zabbix-agent -y
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Set the following parameters:

```text
Server=<SERVER_IP>
ServerActive=<SERVER_IP>
Hostname=<HOST_NAME>
ListenIP=0.0.0.0
```

Field meanings:

| Parameter     | Meaning                                                                 |
| ------------- | ----------------------------------------------------------------------- |
| `Server`      | Addresses allowed to query this agent (passive checks)                  |
| `ServerActive`| Address the agent pushes data to (active checks)                        |
| `Hostname`    | Name the agent identifies itself with. Must match the frontend exactly  |
| `ListenIP`    | Network address the agent binds to. Default is loopback only           |

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
systemctl status zabbix-agent --no-pager
```

Verification that the agent is listening on all interfaces:

```bash
ss -tulpn | grep 10050
```

The output must show `0.0.0.0:10050`. If it shows `127.0.0.1:10050`, the ListenIP directive was not applied and the server cannot reach the agent over the network.

## Step 13: Register the Host in the Frontend

Navigate to Data collection, then Hosts, then Create host.

| Field             | Value                          |
| ----------------- | ------------------------------ |
| Host name         | `<HOSTNAME>` (must equal the agent `Hostname`) |
| Templates         | Linux by Zabbix agent          |
| Host groups       | Linux servers                  |
| Interface         | Agent, `<WORKSTATION_IP>`, port 10050 |
| Description       | Free text identifying machine role and network |

The template field is the powerful part. A template is a predefined bundle of items, graphs, and triggers. Attaching `Linux by Zabbix agent` instantly provides dozens of CPU, memory, disk, and network metrics without manual definition. In production, templates are how consistent monitoring scales across hundreds of hosts.

The `Hostname` value in the agent configuration and the host name in the frontend are matched literally. A single character of difference makes the server reject the agent data.

## Step 14: Verify Server to Agent Communication

Install the testing utility on the server:

```bash
sudo apt install zabbix-get -y
zabbix_get -s <WORKSTATION_IP> -k system.hostname
```

A returned hostname proves the full path works: server process, network, firewall, agent process, agent configuration. This separates network problems from Zabbix configuration problems.

In the frontend, the Availability column for the new host changes the ZBX indicator to green within one or two minutes.

Collected values are visible under Monitoring, Hosts, then Latest data. Ready made graphs are under Monitoring, Hosts, then Graphs.

---

# 7. Data Retention

By default Zabbix stores collected history for a long period. On a 32 GB disk this eventually becomes a storage incident. Retention is configured under Administration, Housekeeping.

| Setting | Recommended value | Reason |
| ------- | ----------------- | ------ |
| History retention override | Enabled, 7d | Raw collected values occupy the most space |
| Trend retention override | Enabled, 30d | Hourly aggregates are compact and still power long term graphs |

History and trends are different data. History is every individual measurement. Trends are hourly averages, minimums, and maximums computed by Zabbix. Reducing history does not damage long term graphs because trends preserve the aggregate view.

---

# 8. Observing Real Alerts

Immediately after installation a warning level problem appeared on the Zabbix server host:

```text
Linux: Number of installed packages has been changed
```

This trigger compares the number of installed packages between evaluations. It exists to detect unauthorized software changes on monitored systems. In this lab it fired because dozens of packages were installed minutes earlier, which is expected and harmless.

This event demonstrates several concepts:

| Concept | Observation |
| ------- | ----------- |
| Severity levels | The warning color reflects a Notice severity, informational rather than critical |
| Trigger purpose | Triggers encode operational expectations, such as software remaining unchanged |
| Acknowledgment | Closing an event while writing the reason documents why it occurred, which is standard operations practice |

In production environments these low severity notices accumulate noise. Administrators tune or disable irrelevant triggers so that meaningful alerts remain visible. Alert fatigue is a real operational problem.

---

# 9. Troubleshooting Summary

Every failure encountered during this lab, its cause, and its resolution.

| Problem | Symptom | Cause | Resolution |
| ------- | ------- | ----- | ---------- |
| Zabbix packages not found | `apt install` fails for `zabbix-apache-conf` and `zabbix-sql-scripts` | `apt update` was not run after installing the repository package | Run `sudo apt update`, then repeat the installation |
| Schema import fails | `server.sql.gz: No such file or directory` | Schema path changed in newer Zabbix versions | Locate the file with `dpkg -L zabbix-sql-scripts | grep server.sql.gz` |
| Frontend login rejected | Incorrect username or password message | Confusion between the database password and the initial web password | The initial web login is Admin with password zabbix, independent of database credentials |
| Host URL confusion | Difficulty locating the host creation screen in the translated interface | Menu labels vary between translations | Reached via Data collection, then Hosts, then Create host |
| Testing tool missing | `zabbix_get: command not found` | Utility ships in a separate package | `sudo apt install zabbix-get` |
| Database connection test impossible | `curl: command not found` | Minimal install lacks curl | `sudo apt install curl` |
| zabbix_server does not start | Service inactive | Most commonly wrong database credentials in the server configuration | Check `journalctl -u zabbix-server` and the server log, verify DBPassword |

The recurring lesson: when a package or file is not where documentation says it should be, ask the package manager where its files actually are. Package documentation ages, package managers do not lie.

---

# 10. Verification Checklist

* [x] MariaDB installed and running
* [x] Zabbix database created with utf8mb4 and binary collation
* [x] Dedicated database user with privileges limited to the Zabbix database
* [x] Official Zabbix repository installed for Debian 13
* [x] Server, frontend, agent, and SQL scripts installed
* [x] Initial schema imported, table count verified
* [x] Runtime function creation flag reset after import
* [x] Server configuration edited with database credentials and reduced caches
* [x] Zabbix server service enabled and running
* [x] Apache serving the frontend under /zabbix
* [x] Web wizard completed with database parameters, name, and timezone
* [x] Default Admin password changed and stored in a password manager
* [x] Local host collecting data with ZBX green
* [x] Agent installed and configured on the administration workstation
* [x] Server to agent communication proven with zabbix_get
* [x] Second host registered with template and green ZBX
* [x] Housekeeping configured for 7 day history and 30 day trends
* [x] Warning level trigger observed and interpreted

---

# 11. Resource Notes

The server runs the complete stack within 2 GB of RAM by applying deliberate reductions:

| Measure | Effect |
| ------- | ------ |
| No desktop environment | Saves hundreds of megabytes of RAM |
| `CacheSize=64M` | Limits the Zabbix configuration cache |
| `HistoryCacheSize=16M` | Limits the in memory history buffer |
| 1 GB swap file | Safety margin when all tiers peak simultaneously |
| Short retention periods | Keeps the database small on a 32 GB disk |

Monitoring resource consumption of the monitoring server itself, using its own data, closes the loop: the system watches the machine that runs it.

Physical care matters as much as configuration. This machine operates 24 hours a day plugged into wall power with an aged battery. Periodic physical inspection and awareness of battery swelling risk are part of running hardware as a server.

---

# 12. Key Takeaways

A monitoring stack is a three tier web application, and installing one is a complete exercise in service administration: database, application server, and interface must each be configured and verified independently before the whole functions.

Two credential domains exist in every such system. The database user and the application user are unrelated, and treating them as the same thing produces avoidable confusion during setup.

The apt package index is refreshed per repository. Adding a repository without updating the index produces package not found errors that look like missing packages but are only stale metadata.

Documentation that ships with software decays. The file locations in this lab differed from widely copied older guides. Locating real paths with package tools is more reliable than trusting tutorials.

A monitoring system is not finished when its interface opens. It is finished when it proves it can detect failure. The agent stop and start test, the availability indicators, and the alert observation are what convert an installed program into an operational monitoring capability.

---

# 13. Next Stages

Planned continuations of this laboratory:

```text
Dashboard construction for presentation
        |
Notification channels (email on trigger)
        |
Custom trigger authoring
        |
SNMP based monitoring of network devices
        |
Database backup with mysqldump and cron
        |
Host inventory and automation
```

SNMP deserves emphasis. Network equipment such as switches, routers, access points, and printers generally cannot run a Zabbix agent. SNMP is the protocol family for monitoring them, which makes it essential for network administration work. A future lab will enable the SNMP daemon on this server and consume it from Zabbix through the Linux by SNMP template, exercising community configuration, port 161, and OID based data collection.
