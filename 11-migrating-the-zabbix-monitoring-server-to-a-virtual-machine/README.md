# Lab 11 - Migrating the Zabbix Monitoring Server to a Virtual Machine

## Overview

This lab documents the migration of the monitoring environment from a physical Debian server to a dedicated Debian virtual machine.

The original physical server was responsible for both monitoring and infrastructure services. It was running Zabbix Server, Zabbix Agent, Zabbix Web, Apache, MariaDB and other components on low-resource hardware.

The objective of this lab was to separate these responsibilities:

```text
Ubuntu Host
│
├── Debian Physical Server
│   └── Infrastructure and network services
│
└── Debian-Monitoring-Server (VirtualBox VM)
    ├── Zabbix Server
    ├── Zabbix Agent
    ├── Zabbix Web
    ├── Apache
    ├── MariaDB
    └── Grafana
```

After the migration, the physical Debian server became a monitored host instead of being responsible for the Zabbix Server itself.

The final monitoring architecture is:

```text
Debian Physical Server
192.168.1.117
        │
        │ Zabbix Agent :10050
        ▼
Debian Monitoring Server
192.168.1.219
        │
        ├── Zabbix Server :10051
        ├── Zabbix Web :80
        ├── MariaDB :3306
        └── Grafana
```

---

# 1. Original Environment

Before the migration, the physical Debian server contained the monitoring stack.

The physical server used:

```text
IP address: 192.168.1.117
OS: Debian 13 (trixie)
Architecture: x86-64
```

Important services:

```text
Apache       → TCP 80
Zabbix Agent → TCP 10050
Zabbix Server → TCP 10051
MariaDB      → TCP 3306 locally
Zabbix Web   → /zabbix
```

The Zabbix API was available at:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

The existing Zabbix installation was version:

```text
Zabbix 7.4.15
```

The server already contained important monitoring data, including:

* Hosts
* Items
* Triggers
* History
* Templates
* Dashboards
* Web monitoring
* Zabbix configuration
* Database contents

Therefore, starting a completely new Zabbix installation and configuring everything manually would defeat the purpose of the migration.

---

# 2. Migration Alternatives

Several migration methods were considered.

## Alternative 1 - Bit-for-bit disk cloning

The first possibility was cloning the entire Debian installation.

The idea would be:

```text
Physical disk
      ↓
Complete disk image
      ↓
Virtual disk
      ↓
Virtual machine
```

Tools such as Clonezilla, SystemRescue or `dd` could theoretically be used.

The advantage would be preserving almost everything exactly as it existed.

However, this approach was not selected.

The physical server was using an eMMC device:

```text
/dev/mmcblk2
```

with partitions:

```text
mmcblk2
├── mmcblk2p1 → EFI
├── mmcblk2p2 → /
└── mmcblk2p3 → swap
```

The risk of accidentally modifying the original disk was unnecessary.

In addition, the physical installation and virtual machine would have different hardware characteristics.

---

# 3. Selected Migration Method

The chosen method was a **file-level migration combined with a database dump and restore**.

Instead of copying the entire operating system:

```text
Physical Debian
       ↓
backup configuration + database
       ↓
new Debian VM
       ↓
reinstall required packages
       ↓
restore database/configuration
```

This approach preserves the important application state while allowing the VM to have a clean Debian installation.

The original operating system itself is not cloned.

The Zabbix database is what carries most of the important monitoring state.

---

# 4. Backup Before Migration

The physical Debian server had approximately:

```text
/dev/mmcblk2p2
27G total
2.4G used
23G free
```

A separate 16 GB USB drive was used for the migration.

It was mounted as:

```text
/mnt/usb
```

The USB contained:

```text
/mnt/usb/
├── configs/
├── packages.txt
└── zabbix.sql.gz
```

---

# 5. Backing Up the Zabbix Database

The most important part of the migration was the Zabbix database.

A MariaDB dump was created:

```bash
sudo mariadb-dump --single-transaction --routines --triggers --events zabbix | gzip > /mnt/usb/zabbix.sql.gz
```

### What the command does

```text
mariadb-dump
```

creates a logical backup of the database.

```text
--single-transaction
```

allows the dump to be created consistently without locking the entire database in the same way as a traditional locking dump.

```text
--routines
```

includes stored procedures and functions.

```text
--triggers
```

includes database triggers.

```text
--events
```

includes MariaDB scheduled events.

```text
zabbix
```

is the database being backed up.

```text
| gzip
```

compresses the output.

```text
> /mnt/usb/zabbix.sql.gz
```

writes the compressed backup to the USB drive.

The backup was then verified:

```bash
gzip -t /mnt/usb/zabbix.sql.gz && echo "Backup OK"
```

The result was:

```text
Backup OK
```

This confirmed that the compressed database dump was not corrupted.

---

# 6. Backing Up Configuration Files

The main configuration directories were also copied.

```bash
sudo cp -a /etc/zabbix /mnt/usb/configs/
sudo cp -a /etc/apache2 /mnt/usb/configs/
sudo cp -a /etc/mysql /mnt/usb/configs/
sudo cp -a /etc/php /mnt/usb/configs/
sudo cp -a /etc/ssh /mnt/usb/configs/
```

Additional system configuration files were preserved:

```bash
sudo cp -a /etc/fstab /mnt/usb/configs/
sudo cp -a /etc/hostname /mnt/usb/configs/
sudo cp -a /etc/hosts /mnt/usb/configs/
```

The purpose was not to blindly copy every configuration into the VM.

Instead, these files were kept as references and used selectively when rebuilding the environment.

---

# 7. Recording Installed Packages

The physical server's installed packages were recorded:

```bash
dpkg-query -W | sort > /mnt/usb/packages.txt
```

This generated a list containing approximately 515 packages.

This was useful for identifying the original environment and determining which packages were necessary on the new VM.

The important monitoring packages included:

```text
zabbix-agent
zabbix-apache-conf
zabbix-frontend-php
zabbix-server-mysql
zabbix-sql-scripts
zabbix-release
```

The original versions included:

```text
Apache 2.4.68
PHP 8.4.24
MariaDB 11.8.6
Zabbix 7.4.15
```

---

# 8. Creating the Virtual Machine

A new VirtualBox VM was created.

```text
Name: Debian-Monitoring-Server
RAM: 4 GB
CPU: 2 vCPUs
Disk: 32 GB dynamically allocated VDI
Network: Bridged Adapter
OS: Debian 13.7.0 amd64 netinst
```

The VM was intentionally dedicated to monitoring.

The physical Debian server would later be available for infrastructure services such as:

```text
DNS
DHCP
Samba
NFS
SSH
VPN
Docker
```

This creates a cleaner separation of responsibilities.

---

# 9. Installing the New Debian System

Debian 13 was installed normally in VirtualBox.

The VM initially received its IP address through DHCP:

```text
192.168.1.219
```

The user `kayo` was added to sudo.

SSH was installed so the VM could be administered remotely.

The hostname was configured as:

```text
debian-monitoring-server
```

The system was updated:

```bash
sudo apt update
sudo apt full-upgrade -y
```

---

# 10. Attaching the Migration USB to the VM

The USB was originally available to the physical server.

VirtualBox was configured to pass the USB device to the new VM.

Inside the VM:

```bash
lsblk -f
```

showed:

```text
/dev/sdb1
```

as the migration USB.

It was mounted:

```bash
sudo mkdir -p /mnt/usb
sudo mount /dev/sdb1 /mnt/usb
```

The contents were confirmed:

```bash
ls -lah /mnt/usb
```

Result:

```text
configs/
lost+found/
packages.txt
zabbix.sql.gz
```

The database backup was tested again:

```bash
gzip -t /mnt/usb/zabbix.sql.gz
```

No error was returned.

---

# 11. Installing MariaDB

MariaDB was installed in the VM:

```bash
sudo apt install mariadb-server mariadb-client -y
```

MariaDB was started automatically and verified.

Initially, the database server contained only its default databases.

A new Zabbix database was created:

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

A database user was created:

```sql
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixTemp2026!';
```

Privileges were granted:

```sql
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
```

The temporary password was later replaced with the original Zabbix database password.

---

# 12. Restoring the Zabbix Database

The original Zabbix database was restored into the newly created database:

```bash
sudo mariadb zabbix < <(gzip -dc /mnt/usb/zabbix.sql.gz)
```

The command works in two stages:

```text
gzip -dc
     ↓
decompress zabbix.sql.gz
     ↓
mariadb zabbix
     ↓
restore SQL into the zabbix database
```

The database was then inspected:

```sql
SHOW TABLES;
```

Many existing Zabbix tables appeared, including tables such as:

```text
acknowledges
actions
alerts
auditlog
autoreg_host
changelog
conditions
config_autoreg_tls
connector
```

This confirmed that the original Zabbix database had been successfully restored.

---

# 13. Installing the Zabbix Repository

The VM needed the same Zabbix version used by the original server.

The Zabbix repository for Debian 13 was installed using:

```bash
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
```

Then:

```bash
sudo apt update
```

The available version was checked:

```bash
apt-cache policy zabbix-server-mysql
```

The candidate version was:

```text
7.4.15
```

This matched the original physical server.

Using the same major/minor version reduced compatibility risks during migration.

---

# 14. Installing the Zabbix Components

The required packages were installed:

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

This installed:

```text
Zabbix Server
Zabbix Web
Zabbix Apache configuration
Zabbix SQL scripts
Zabbix Agent
```

The VM therefore had the software required to use the restored database.

---

# 15. Restoring the Zabbix Server Configuration

The newly installed configuration was compared with the original configuration.

The important original settings were:

```text
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_server.pid
DBName=zabbix
DBUser=zabbix
Timeout=4
```

The original configuration also contained the Zabbix database password.

Before replacing the configuration, the new file was backed up:

```bash
sudo cp /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.new
```

The original configuration was then restored:

```bash
sudo cp /mnt/usb/configs/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf
```

---

# 16. Restoring the Database Password

The Zabbix database user was configured with the original password:

```sql
ALTER USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixLab2026!';
FLUSH PRIVILEGES;
```

Database access was tested:

```bash
mariadb -u zabbix -p zabbix -e "SELECT COUNT(*) FROM users;"
```

The query returned:

```text
COUNT(*)
3
```

This confirmed that the Zabbix user could access the restored database.

---

# 17. Starting Zabbix Server

The Zabbix server was started:

```bash
sudo systemctl start zabbix-server
```

Its status was checked:

```bash
sudo systemctl status zabbix-server
```

The service was running and spawned its normal Zabbix processes, including configuration syncers, history syncers, HTTP pollers and browser pollers.

This was an important confirmation that:

```text
Zabbix Server
      ↓
MariaDB
      ↓
restored Zabbix database
```

was working.

---

# 18. Configuring the Zabbix Web Interface

Apache was already installed and running.

Its status was checked:

```bash
sudo systemctl status apache2 --no-pager
```

PHP was verified:

```bash
sudo apache2ctl -M | grep -E 'php|rewrite'
```

The PHP module was loaded.

The Zabbix Apache configuration existed:

```text
/etc/apache2/conf-enabled/zabbix.conf
```

It provided the:

```text
/zabbix
```

URL.

Initially, accessing:

```text
http://192.168.1.219/zabbix
```

returned an Apache 404.

However, testing locally showed:

```bash
curl -I http://127.0.0.1/zabbix/
```

returned:

```text
HTTP/1.1 302 Found
Location: setup.php
```

This proved that Apache and the Zabbix frontend were actually working.

---

# 19. Fixing the Zabbix Web Database Configuration

The Zabbix frontend configuration was a symbolic link:

```text
/usr/share/zabbix/ui/conf/zabbix.conf.php
        ↓
/etc/zabbix/web/zabbix.conf.php
```

The target did not exist.

Therefore, the configuration file was created at:

```text
/etc/zabbix/web/zabbix.conf.php
```

with the database connection information:

```php
$DB['TYPE'] = 'MYSQL';
$DB['SERVER'] = 'localhost';
$DB['PORT'] = '0';
$DB['DATABASE'] = 'zabbix';
$DB['USER'] = 'zabbix';
$DB['PASSWORD'] = 'ZabbixLab2026!';
```

The Zabbix server name was configured as:

```php
$ZBX_SERVER_NAME = 'Debian Monitoring Server';
```

Apache was reloaded.

The Zabbix Web interface then became accessible.

---

# 20. Verifying the Migrated Zabbix Environment

The most important test was not simply whether the login page appeared.

The existing Zabbix data was checked.

The migrated environment contained the original:

```text
Hosts
Items
Triggers
History
Templates
Dashboards
Web monitoring
```

The existing Zabbix configuration appeared in the VM.

This confirmed that the migration was successful at the application/database level.

The migration therefore did not require rebuilding the monitoring environment manually.

---

# 21. Installing Zabbix Agent and Testing the Physical Server

The new VM also had Zabbix Agent installed.

The VM's own agent was configured for the local Zabbix server.

The physical Debian server remained at:

```text
192.168.1.117
```

The VM needed to communicate with its Zabbix Agent on:

```text
192.168.1.117:10050
```

Initially, the test was:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

The response was:

```text
ZBX_NOTSUPPORTED: Received empty response from Zabbix Agent at [192.168.1.117].
Assuming that agent dropped connection because of access permissions.
```

This indicated that the network connection existed but the physical server's Zabbix Agent did not allow the VM's IP.

---

# 22. Allowing the New Zabbix Server

On the physical Debian server, the Zabbix Agent configuration was checked:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Originally:

```text
Server=127.0.0.1,192.168.1.117
ServerActive=127.0.0.1
Hostname=Debian Server
```

The VM address was added:

```text
Server=127.0.0.1,192.168.1.117,192.168.1.219
```

The agent was restarted:

```bash
sudo systemctl restart zabbix-agent
```

The service was verified as running.

---

# 23. Testing Zabbix Agent Communication

The test was repeated from the VM:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

The result was:

```text
1
```

This is the expected successful response from a Zabbix Agent.

The physical server therefore became accessible to the new Zabbix Server.

The Zabbix Web interface subsequently showed the physical Debian host as green.

The interface continued to display:

```text
192.168.1.117:10050
```

This is correct.

The address belongs to the **physical server's Agent**, not to the new Zabbix Server.

---

# 24. Enabling Services at Boot

The monitoring services were configured to start automatically:

```bash
sudo systemctl enable zabbix-server zabbix-agent apache2 mariadb
```

The architecture was now persistent across VM reboots:

```text
MariaDB
   ↓
Zabbix Server
   ↓
Zabbix Web
   ↓
Zabbix Agent
```

---

# 25. Why a Static IP Was Necessary

At this point the VM was working with:

```text
192.168.1.219
```

However, this address had initially been assigned through DHCP.

That creates a problem for a monitoring server.

For example:

```text
Today:
VM → 192.168.1.219

After DHCP lease changes:
VM → 192.168.1.150
```

The Zabbix environment and other configurations could then reference the wrong address.

For a server whose job is to monitor other systems, a stable address is important.

Therefore, the VM was configured with a static IPv4 address.

---

# 26. First Static IP Configuration Attempt

The first approach was to configure the address in:

```text
/etc/network/interfaces
```

The configuration became:

```text
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.219
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.1
```

IPv6 remained:

```text
iface enp0s3 inet6 auto
```

However, the system was also running `dhcpcd`.

This created two possible network configuration mechanisms:

```text
ifupdown
     +
dhcpcd
```

The running `dhcpcd` process was confirmed with:

```bash
ps aux | grep '[d]hcpcd'
```

The process was running as PID 661 with parent PID 1.

There was no visible:

```text
dhcpcd.service
```

and no obvious `dhcpcd` hook inside:

```text
/etc/network/if-*.d/
```

Therefore, using both configuration mechanisms simultaneously was undesirable.

---

# 27. Final Static IP Configuration

The final approach was to allow `dhcpcd` to manage the interface while explicitly assigning a static address.

The following was added to:

```text
/etc/dhcpcd.conf
```

```text
interface enp0s3
static ip_address=192.168.1.219/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

The IPv4 configuration in:

```text
/etc/network/interfaces
```

was changed to:

```text
auto enp0s3
iface enp0s3 inet manual

iface enp0s3 inet6 auto
```

This prevents `ifupdown` from attempting to assign another IPv4 configuration.

---

# 28. Applying the Network Configuration

The interface was restarted:

```bash
sudo ifdown enp0s3 && sudo ifup enp0s3
```

Because the operation was being performed through SSH, there was a possibility that the connection would temporarily disconnect.

The VM was then accessed again through:

```bash
ssh kayo@192.168.1.219
```

The connection succeeded.

---

# 29. Verifying the Static Address

The address was checked:

```bash
ip -4 addr show enp0s3
```

The result included:

```text
inet 192.168.1.219/24
```

and, importantly:

```text
valid_lft forever
preferred_lft forever
```

This confirms that the address is configured as a permanent/static address rather than a temporary DHCP lease.

The final `dhcpcd` configuration was also checked:

```bash
grep -E '^(interface enp0s3|static ip_address|static routers|static domain_name_servers)' /etc/dhcpcd.conf
```

Result:

```text
interface enp0s3
static ip_address=192.168.1.219/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

The final routing table contained:

```text
default via 192.168.1.1 dev enp0s3 src 192.168.1.219
192.168.1.0/24 dev enp0s3 proto dhcp scope link src 192.168.1.219
```

Although the local route still displayed `proto dhcp`, the actual address was verified as:

```text
valid_lft forever
preferred_lft forever
```

and the explicit static configuration existed in `dhcpcd.conf`.

Therefore, the VM was no longer dependent on receiving a different IPv4 address from the router.

---

# 30. Final Architecture

The migration changed the architecture from:

```text
Before

Physical Debian Server
192.168.1.117
│
├── Zabbix Server
├── Zabbix Agent
├── Zabbix Web
├── Apache
├── MariaDB
└── Monitoring data
```

to:

```text
After

Ubuntu Host
│
├── Debian Physical Server
│   192.168.1.117
│   │
│   └── Zabbix Agent :10050
│
└── Debian Monitoring Server VM
    192.168.1.219
    │
    ├── Zabbix Server :10051
    ├── Zabbix Agent :10050
    ├── Zabbix Web :80
    ├── Apache
    └── MariaDB
```

The communication path is now:

```text
Debian Physical Server
        │
        │ Zabbix Agent
        │ 192.168.1.117:10050
        ▼
Debian Monitoring Server
        │
        ├── Zabbix Server
        ├── MariaDB
        └── Zabbix Web
```

---

# 31. Result

The monitoring environment was successfully migrated from the physical Debian server to the dedicated Debian virtual machine.

The migration preserved the existing Zabbix application state through a database backup and restore rather than rebuilding the environment manually.

The physical Debian server remained intact and became a monitored host.

The new monitoring VM now has a fixed address:

```text
192.168.1.219
```

The physical server remains at:

```text
192.168.1.117
```

Communication was verified with:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

returning:

```text
1
```

The Zabbix interface also showed the physical Debian server as healthy.

At this point, the architecture can be considered:

```text
              ┌──────────────────────────────┐
              │ Debian Monitoring Server VM  │
              │ 192.168.1.219                │
              │                              │
              │ Zabbix Server                │
              │ Zabbix Web                   │
              │ MariaDB                      │
              │ Apache                       │
              └──────────────┬───────────────┘
                             │
                         Zabbix Agent
                         TCP 10050
                             │
                             ▼
              ┌──────────────────────────────┐
              │ Debian Physical Server      │
              │ 192.168.1.117               │
              │                              │
              │ Infrastructure services     │
              │ + Zabbix Agent              │
              └──────────────────────────────┘
```

The key architectural change is that **the physical Debian server is no longer responsible for running the monitoring platform. The virtual machine is now the dedicated Zabbix monitoring server, while the physical Debian machine is one of the systems being monitored.**

This separation also leaves the physical server available for future infrastructure projects without mixing those services with the monitoring platform.
