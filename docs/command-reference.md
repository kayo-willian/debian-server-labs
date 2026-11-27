# Command reference

[Index](README.md) · [Português (Brasil)](../pt-BR/docs/referencia-de-comandos.md)

Selective index of commands and tools covered in the records. Consult the corresponding lab for full syntax, parameters, results and the machine where each operation was performed. This is not an execution script.

| Lab | Commands and tools | Purpose |
| --- | --- | --- |
| [01](labs/01-installation-configuration-ssh/README.md) | `ip addr`, `ip route`, `ip route get`, `ping`, `ssh`, `passwd` | Interface, route, connectivity and remote access |
| [02](labs/02-monitoring-server-with-zabbix/README.md) | `free -h`, `apt update`, `apt policy`, `dpkg -L`, `systemctl`, `journalctl`, `zabbix_get` | Resource checks, package discovery, services and agent query |
| [03](labs/03-firewall-access-control/README.md) | `ufw enable`, `ufw status numbered` | Firewall activation and rule inspection |
| [04](labs/04-nginx-web-server/README.md) | `nginx -t`, `nginx -T`, `curl`, `ss` | Configuration validation and HTTP diagnostics |
| [06](labs/06-apache-static-web-page/README.md) | `curl`, `systemctl` | Apache HTTP availability and service status |
| [07](labs/07-dhcp-reservation-and-ip-address-persistence/README.md) | `ip addr`, `ip route`, `dhcpcd` | DHCP addressing and route inspection |
| [08](labs/08-grafana-zabbix-integration/README.md) | `curl`, `journalctl` | JSON-RPC testing and Grafana logs |
| [10](labs/10-grafana-removal-and-resource-optimization/README.md) | `systemctl`, `apt remove`, `apt purge`, `apt autoremove` | Service and package removal |
| [11](labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) | `mariadb-dump`, `gzip`, `mariadb`, `zabbix_get` | Database migration, archive check and agent query |
| [12](labs/12-zabbix-removal-and-server-cleanup/README.md) | `systemctl`, `ss`, `zabbix_get` | Post-cleanup verification and remote agent test |
| [14](labs/14-bind9-dns-server/README.md) | `named-checkconf`, `named-checkzone`, `dig`, `nc`, `zabbix_get` | DNS configuration, resolution and functional monitoring |

Labs 10–12 contain destructive package, file and database removal operations. Do not repeat them without checking the host, dependencies and backups. Historical examples with masked credentials require your own values; never publish passwords or tokens.

[Learning roadmap](learning-roadmap.md)
