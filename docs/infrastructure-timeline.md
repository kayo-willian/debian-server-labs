# Infrastructure timeline

[Index](README.md) · [Português (Brasil)](../pt-BR/docs/evolucao-da-infraestrutura.md)

This timeline follows lab order and preserves deployment, migration and removal decisions. Addresses are historical.

| Lab | Network | Documented change |
| --- | --- | --- |
| [01](labs/01-installation-configuration-ssh/README.md) | `192.168.2.0/24` | Debian installed on physical hardware; SSH and dedicated Wi-Fi subnet |
| [02](labs/02-monitoring-server-with-zabbix/README.md) | `192.168.2.0/24` | Zabbix 7.4, MariaDB and Apache deployed; second host monitored |
| [03](labs/03-firewall-access-control/README.md) | `192.168.2.0/24` | UFW inbound policy with scoped SSH/HTTP and open TCP 10051 |
| [04](labs/04-nginx-web-server/README.md) | `192.168.2.0/24` | Nginx TCP 8080 and local Stub Status |
| [05](labs/05-zabbix-custom-dashboard/README.md) | `192.168.1.0/24` | Main LAN used; custom Zabbix dashboard |
| [06](labs/06-apache-static-web-page/README.md) | `192.168.1.0/24` | Apache static page and web scenario |
| [07](labs/07-dhcp-reservation-and-ip-address-persistence/README.md) | `192.168.1.0/24` | DHCP changed .117 to .116; reservation .117 configured |
| [08](labs/08-grafana-zabbix-integration/README.md) | `192.168.1.0/24` | Grafana connected to the Zabbix API on the physical server |
| [09](labs/09-grafana-monitoring-dashboard/README.md) | `192.168.1.0/24` | Grafana dashboard and recorded JSON |
| [10](labs/10-grafana-removal-and-resource-optimization/README.md) | `192.168.1.0/24` | Grafana removed from physical server to conserve resources |
| [11](labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) | `192.168.1.0/24` | Zabbix migrated to VM .219; database/configuration restored; passive agent query .117 tested |
| [12](labs/12-zabbix-removal-and-server-cleanup/README.md) | `192.168.1.0/24` | Physical Zabbix stack, MariaDB, Apache and PHP removed; agent retained |
| [13](labs/13-grafana-custom-monitoring-dashboard/README.md) | `192.168.1.0/24` | Grafana on monitoring VM; custom dashboard ongoing, export not yet present |
| [14](labs/14-bind9-dns-server/README.md) | `192.168.1.0/24` | Physical BIND9; forward/reverse queries and Zabbix DNS check |

## Record limits

Lab 12's table already records Grafana running on the VM; Lab 13 explains its reinstallation and subsequent dashboard development. No exact reinstallation date is recorded here. Lab 13 remains open and Lab 09's JSON belongs to the earlier dashboard, not the new one.

A service's omission in a later stage must not be interpreted as proof of removal. For example, Lab 12 documents Apache removal, but not Nginx removal.

[Architecture](architecture.md)
