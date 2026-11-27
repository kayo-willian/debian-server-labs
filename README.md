# Debian Server Labs

<div align="center">

[![Leia em português](https://img.shields.io/badge/LEIA%20EM%20PORTUGU%C3%8AS-009C3B?style=for-the-badge)](pt-BR/README.md)
[![Technical documentation](https://img.shields.io/badge/TECHNICAL%20DOCS-1E293B?style=for-the-badge)](docs/README.md)

![Debian](https://img.shields.io/badge/Debian-A81D33?style=flat-square) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square) ![SSH](https://img.shields.io/badge/SSH-475569?style=flat-square) ![Zabbix](https://img.shields.io/badge/Zabbix-D40000?style=flat-square) ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square) ![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=flat-square) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square) ![Apache](https://img.shields.io/badge/Apache-D22128?style=flat-square) ![UFW](https://img.shields.io/badge/UFW-64748B?style=flat-square) ![BIND9](https://img.shields.io/badge/BIND9-2563EB?style=flat-square)
[![MIT](https://img.shields.io/badge/License-MIT-2563EB?style=flat-square)](LICENSE)

</div>

Fourteen lab records follow a Positivo Motion with an Intel Atom x5-Z8350 and 2 GB of RAM from its first Debian installation through DNS service and remote monitoring. The records retain historical addresses, commands and observations from each stage, with personal identifiers masked.

## Labs

| Lab | Topic | Scope | PT-BR |
| --- | --- | --- | --- |
| 01 | [Installation, Configuration and SSH](docs/labs/01-installation-configuration-ssh/README.md) | Physical Debian, Wi-Fi, dedicated subnet and remote administration | [PT-BR](pt-BR/docs/laboratorios/01-instalacao-configuracao-ssh/README.md) |
| 02 | [Monitoring Server with Zabbix](docs/labs/02-monitoring-server-with-zabbix/README.md) | MariaDB, Zabbix, agents, web setup, retention and diagnostics | [PT-BR](pt-BR/docs/laboratorios/02-servidor-de-monitoramento-zabbix/README.md) |
| 03 | [Firewall and Access Control](docs/labs/03-firewall-access-control/README.md) | UFW, inbound rules, SSH, HTTP and agent communication | [PT-BR](pt-BR/docs/laboratorios/03-firewall-e-controle-de-acesso/README.md) |
| 04 | [Nginx Web Server](docs/labs/04-nginx-web-server/README.md) | Custom web root, HTTP 8080, logs and Stub Status | [PT-BR](pt-BR/docs/laboratorios/04-servidor-web-nginx/README.md) |
| 05 | [Zabbix Custom Dashboard](docs/labs/05-zabbix-custom-dashboard/README.md) | CPU, memory, uptime, load, historical graphs and problems | [PT-BR](pt-BR/docs/laboratorios/05-dashboard-personalizado-zabbix/README.md) |
| 06 | [Apache Static Web Page](docs/labs/06-apache-static-web-page/README.md) | HTML/CSS, HTTP verification, Zabbix web monitoring and URL widget | [PT-BR](pt-BR/docs/laboratorios/06-pagina-estatica-apache/README.md) |
| 07 | [DHCP Reservation and IP Address Persistence](docs/labs/07-dhcp-reservation-and-ip-address-persistence/README.md) | DHCP client, addressing conflict, recovery and router reservation | [PT-BR](pt-BR/docs/laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md) |
| 08 | [Grafana and Zabbix Integration](docs/labs/08-grafana-zabbix-integration/README.md) | Plugin, API token, JSON-RPC, logs and data source validation | [PT-BR](pt-BR/docs/laboratorios/08-integracao-grafana-zabbix/README.md) |
| 09 | [Grafana Monitoring Dashboard](docs/labs/09-grafana-monitoring-dashboard/README.md) | System, network and web metrics; dashboard JSON | [PT-BR](pt-BR/docs/laboratorios/09-dashboard-de-monitoramento-grafana/README.md) |
| 10 | [Grafana Removal and Resource Optimization](docs/labs/10-grafana-removal-and-resource-optimization/README.md) | Service removal, package cleanup and resource management | [PT-BR](pt-BR/docs/laboratorios/10-remocao-do-grafana-e-otimizacao-de-recursos/README.md) |
| 11 | [Migrating Zabbix to a Virtual Machine](docs/labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) | Database backup/restore, frontend recovery, agent and static IP | [PT-BR](pt-BR/docs/laboratorios/11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) |
| 12 | [Zabbix Removal and Server Cleanup](docs/labs/12-zabbix-removal-and-server-cleanup/README.md) | Removal of the physical server stack; agent retained and tested | [PT-BR](pt-BR/docs/laboratorios/12-remocao-do-zabbix-e-limpeza-do-servidor/README.md) |
| 13 | [Grafana Custom Monitoring Dashboard](docs/labs/13-grafana-custom-monitoring-dashboard/README.md) | Grafana on the monitoring VM; incremental dashboard (ongoing) | [PT-BR](pt-BR/docs/laboratorios/13-dashboard-personalizado-grafana/README.md) |
| 14 | [BIND9 DNS Server](docs/labs/14-bind9-dns-server/README.md) | Forward/reverse zones, DNS records, remote queries and Zabbix check | [PT-BR](pt-BR/docs/laboratorios/14-servidor-dns-bind9/README.md) |

## Documentation and evidence

[Environment and methodology](docs/README.md) · [Architecture](docs/architecture.md) · [Infrastructure timeline](docs/infrastructure-timeline.md) · [Learning roadmap](docs/learning-roadmap.md) · [Command reference](docs/command-reference.md)

[Zabbix screenshot gallery](docs/labs/05-zabbix-custom-dashboard/gallery.md) · [Apache screenshot gallery](docs/labs/06-apache-static-web-page/gallery.md)

Historical records of different stages, not a current inventory. Lab 13 remains ongoing. [MIT](LICENSE).
