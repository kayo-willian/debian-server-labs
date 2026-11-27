# Debian Server Labs

Fourteen lab records follow a Positivo Motion with an Intel Atom x5-Z8350 and 2 GB of RAM from its first Debian installation through DNS service and remote monitoring. The records retain the addresses, commands and observations from each stage.

[Documentation and environment](docs/README.md) · [Zabbix screenshots](assets/images/05-zabbix-custom-dashboard/README.md) · [Apache screenshots](assets/images/06-apache-static-web-page/README.md)

## Labs

| Lab | Topic | What the lab covers |
| --- | --- | --- |
| 01 | [Installation, Configuration and SSH](docs/labs/01-installation-configuration-ssh/README.md) | Installation and initial configuration of Debian Server, wireless networking, dedicated laboratory subnet, connectivity testing, and SSH administration |
| 02 | [Monitoring Server with Zabbix](docs/labs/02-monitoring-server-with-zabbix/README.md) | Deployment of a complete Zabbix monitoring stack on resource constrained hardware: MariaDB preparation, schema import, web frontend setup, agent-based monitoring, data retention, and troubleshooting |
| 03 | [Firewall and Access Control](docs/labs/03-firewall-access-control/README.md) | Securing the server with UFW: default deny policy, source address restrictions for SSH and the Zabbix frontend, agent traffic preservation, and multi client verification |
| 04 | [Nginx Web Server](docs/labs/04-nginx-web-server/README.md) | Installation and configuration of Nginx on Debian Server, custom web root and HTML page, non-default HTTP port, configuration validation, HTTP testing, logging, and Nginx Stub Status |
| 05 | [Zabbix Custom Dashboard](docs/labs/05-zabbix-custom-dashboard/README.md) | Creation and configuration of a custom Zabbix dashboard for the Debian Server, including CPU and memory gauges, uptime, load average, historical graphs, problem monitoring, web monitoring, and system metrics |
| 06 | [Apache Static Web Page](docs/labs/06-apache-static-web-page/README.md) | Installation and configuration of Apache2, creation of a custom static web page, HTTP validation, Zabbix Web Monitoring, response time monitoring, URL visualization, and dashboard integration |
| 07 | [DHCP Reservation and IP Address Persistence](docs/labs/07-dhcp-reservation-and-ip-address-persistence/README.md) | Investigation of a changing DHCP address, identification of the DHCP client, network configuration troubleshooting, recovery of connectivity, and configuration of a router-side DHCP reservation |
| 08 | [Grafana and Zabbix Integration](docs/labs/08-grafana-zabbix-integration/README.md) | Installation and configuration of Grafana, Zabbix plugin integration, API Token authentication, JSON-RPC API testing, troubleshooting, and datasource validation |
| 09 | [Grafana Monitoring Dashboard](docs/labs/09-grafana-monitoring-dashboard/README.md) | Creation and configuration of a monitoring dashboard in Grafana using Zabbix as the data source, including system metrics, web monitoring, Apache response monitoring, URL monitoring, and dashboard visualization |
| 10 | [Grafana Removal and Resource Optimization](docs/labs/10-grafana-removal-and-resource-optimization/README.md) | Removal of Grafana from the physical Debian Server to reduce resource consumption, analysis of memory usage, service cleanup, and preparation of the server for future infrastructure services |
| 11 | [Migrating the Zabbix Monitoring Server to a Virtual Machine](docs/labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) | Migration of the Zabbix monitoring environment from the physical Debian Server to a dedicated Debian virtual machine, including database backup and restoration, configuration migration, Zabbix Web recovery, agent reconfiguration, service validation, and static IP configuration |
| 12 | [Zabbix Removal and Server Cleanup](docs/labs/12-zabbix-removal-and-server-cleanup/README.md) | Removal of the Zabbix Server stack from the physical Debian Server while preserving the Zabbix Agent for remote monitoring, including removal of the Zabbix frontend, MariaDB, Apache2, PHP, Grafana remnants, service cleanup, port verification, and final validation of the dedicated monitoring architecture |
| 13 | [Grafana Custom Monitoring Dashboard](docs/labs/13-grafana-custom-monitoring-dashboard/README.md) | Reinstallation of Grafana on the dedicated Debian monitoring virtual machine and progressive development of a custom monitoring dashboard using Zabbix as the data source, including manual panel creation, metric visualization, thresholds, dashboard organization, and eventual JSON export |
| 14 | [BIND9 DNS Server](docs/labs/14-bind9-dns-server/README.md) | Configuration of a BIND9 DNS server on a physical Debian machine, including forward and reverse zones, DNS records, authoritative and remote resolution tests, DNS port validation, and custom Zabbix monitoring of DNS resolution |

## Reading the timeline

Labs 01–04 used an isolated wireless subnet. In Lab 05 the server joined the main LAN. Lab 07 records why a router-side DHCP reservation was introduced.

Labs 08–09 ran Grafana on the physical server. Lab 10 removed it to reclaim resources. Labs 11–12 moved Zabbix to a dedicated VM and removed the server stack from the physical host, retaining its agent. Lab 13 records Grafana on the VM and leaves the custom dashboard open. Lab 14 tests BIND9 on the physical server.

The addresses and service lists refer to the stage in which they were recorded.
