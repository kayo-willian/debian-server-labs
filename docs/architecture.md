# Historical architecture

[Index](README.md) · [Português (Brasil)](../pt-BR/docs/arquitetura.md)

These diagrams summarize the configurations documented at different stages, not the equipment's present state. Query arrows indicate who initiates communication; they do not represent every traffic flow.

## Stage 1 — dedicated network (01–04)

```mermaid
flowchart LR
    H["Ubuntu workstation"]
    E["Main LAN<br/>192.168.1.0/24"]
    W["Lab network<br/>192.168.2.0/24"]
    D["Physical Debian<br/>192.168.2.161"]
    H ---|"Ethernet"| E
    H ---|"Wi-Fi"| W
    W --- D
    H -. "SSH TCP 22" .-> D
```

The workstation had interfaces on separate subnets. The server received its address through DHCP. Zabbix, MariaDB and the Apache frontend were added in Lab 02; UFW in Lab 03; Nginx on TCP 8080 in Lab 04, with Stub Status restricted to loopback.

[Installation and SSH](labs/01-installation-configuration-ssh/README.md) · [Zabbix server](labs/02-monitoring-server-with-zabbix/README.md) · [UFW](labs/03-firewall-access-control/README.md) · [Nginx](labs/04-nginx-web-server/README.md)

## Stage 2 — main LAN and local monitoring (05–10)

```mermaid
flowchart LR
    H["Ubuntu workstation"]
    R["DHCP router<br/>192.168.1.1"]
    subgraph D["Physical Debian<br/>192.168.1.117"]
      A["Zabbix Agent"]
      Z["Zabbix Server"]
      M["MariaDB"]
      W["Apache / Zabbix Web"]
      G["Grafana (08–09)"]
      Z --> M
      Z --> A
      G -->|"JSON-RPC"| W
      W --> M
    end
    R --- H
    R --- D
    H -->|"SSH"| D
```

Lab 07 documents the DHCP reservation after the temporary change to 192.168.1.116. Grafana was installed in Labs 08–09 and removed in Lab 10 for resource management. The diagram does not imply that every service was verified simultaneously in each lab.

[DHCP reservation](labs/07-dhcp-reservation-and-ip-address-persistence/README.md) · [Grafana integration](labs/08-grafana-zabbix-integration/README.md) · [Grafana removal](labs/10-grafana-removal-and-resource-optimization/README.md)

## Stage 3 — separate monitoring and infrastructure (11–14)

```mermaid
flowchart LR
    subgraph V["Debian monitoring VM<br/>192.168.1.219"]
      Z["Zabbix Server"]
      M["MariaDB"]
      W["Apache / Zabbix Web"]
      G["Grafana"]
      Z --> M
      W --> M
      G -->|"JSON-RPC"| W
    end
    subgraph D["Physical Debian server<br/>192.168.1.117"]
      A["Zabbix Agent"]
      B["BIND9 (14)"]
      A -->|"dig @127.0.0.1"| B
    end
    Z -->|"Passive query TCP 10050"| A
    V -->|"DNS 53 UDP/TCP"| B
```

The VirtualBox VM uses a bridged adapter, 2 vCPUs, 4 GB of RAM and a 32 GB VDI disk. Its static IP was configured with dhcpcd. Lab 12 records removal of the physical server's Zabbix stack while retaining its agent; Lab 13 records Grafana on the VM and an ongoing dashboard; Lab 14 adds physical BIND9 and a functional DNS resolution check.

[Zabbix migration](labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) · [Physical server cleanup](labs/12-zabbix-removal-and-server-cleanup/README.md) · [Grafana on VM](labs/13-grafana-custom-monitoring-dashboard/README.md) · [BIND9](labs/14-bind9-dns-server/README.md)

For the sequence of changes, see the [infrastructure timeline](infrastructure-timeline.md).
