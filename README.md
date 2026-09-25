# Debian Server Labs

Hands-on Debian Server labs focused on Linux administration, networking, services, security, monitoring, and infrastructure.

This repository documents practical experiments and learning activities performed using a physical Debian Server.

The purpose of this repository is twofold: to demonstrate practical experience and to build a personal technical reference that can be consulted during future classes, laboratories, troubleshooting, and system administration tasks.

The focus is not simply on executing commands, but on understanding why a configuration is required, how it works, how to verify the result, and how to troubleshoot problems.

## Learning Approach

Each laboratory follows a practical learning cycle:

```text
Concept
   ↓
Why does it exist?
   ↓
Command / Configuration
   ↓
Observe the result
   ↓
Implementation
   ↓
Testing
   ↓
Troubleshooting
   ↓
Documentation
```

The documentation emphasizes understanding rather than memorization.

Commands are documented together with their purpose, expected behavior, observed results, and practical context.

## Labs

| Lab | Topic                                                                                                     | Description                                                                                                                                                                                                     |
| --- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 01  | [Installation, Configuration and SSH](./01-installation-configuration-ssh/README.md)                      | Installation and initial configuration of Debian Server, wireless networking, dedicated laboratory subnet, connectivity testing, and SSH administration                                                         |
| 02  | [Monitoring Server with Zabbix](./02-monitoring-server-with-zabbix/README.md)                             | Deployment of a complete Zabbix monitoring stack on resource constrained hardware: MariaDB preparation, schema import, web frontend setup, agent-based monitoring, data retention, and troubleshooting          |
| 03  | [Firewall and Access Control](./03-firewall-access-control/README.md)                                     | Securing the server with UFW: default deny policy, source address restrictions for SSH and the Zabbix frontend, agent traffic preservation, and multi client verification                                       |
| 04  | [Nginx Web Server](./04-nginx-web-server/README.md)                                                       | Installation and configuration of Nginx on Debian Server, custom web root and HTML page, non-default HTTP port, configuration validation, HTTP testing, logging, and Nginx Stub Status                          |
| 05  | [Zabbix Custom Dashboard](./05-zabbix-custom-dashboard/README.md)                                         | Creation and configuration of a custom Zabbix dashboard for the Debian Server, including CPU and memory gauges, uptime, load average, historical graphs, problem monitoring, web monitoring, and system metrics |
| 06  | [Apache Static Web Page](./06-apache-static-web-page/README.md)                                           | Installation and configuration of Apache2, creation of a custom static web page, HTTP validation, Zabbix Web Monitoring, response time monitoring, URL visualization, and dashboard integration                 |
| 07  | [DHCP Reservation and IP Address Persistence](./07-dhcp-reservation-and-ip-address-persistence/README.md) | Investigation of a changing DHCP address, identification of the DHCP client, network configuration troubleshooting, recovery of connectivity, and configuration of a router-side DHCP reservation               |

The first four laboratories were developed using the original isolated laboratory network. Starting with Lab 05, the project moved to the main local network to simplify communication between the server, administration workstation, and other devices used during the experiments.

This change also makes the current infrastructure easier to observe and manage because the server is now part of the same network used by the administration workstation and other laboratory equipment.

## Network Evolution

The network architecture changed during the development of the project.

### Labs 01-04

The first four laboratories used a dedicated wireless laboratory subnet:

```text
192.168.2.0/24
```

This provided network separation between the laboratory environment and the main home network.

The server and administration workstation communicated through the dedicated laboratory network during these initial experiments.

### Labs 05+

Starting with Lab 05, the server was moved to the main local network:

```text
192.168.1.0/24
```

The change was made to improve communication and visibility between the Debian Server, administration workstation, router, and other devices used in the laboratory.

The current environment therefore reflects a more practical infrastructure topology in which the server participates directly in the main LAN.

The change should be considered when reading the older laboratories. Their network addresses describe the environment that existed when those labs were performed and are not necessarily the current configuration.

## Environment

### Server Hardware

| Component           | Specification       |
| ------------------- | ------------------- |
| Device              | Positivo Motion     |
| CPU                 | Intel Atom x5-Z8350 |
| RAM                 | 2 GB DDR3           |
| Storage             | Approximately 32 GB |
| Network             | Wi-Fi               |
| Operating System    | Debian Server       |
| Desktop Environment | None                |

The server uses an older and resource constrained computer as a physical laboratory environment.

The limited hardware is intentional and provides a practical environment for learning how to operate services and systems with constrained resources.

## Current Network Environment

The current laboratory environment uses the main local network.

| Network                     | Subnet           | Purpose                                                       |
| --------------------------- | ---------------- | ------------------------------------------------------------- |
| Familia Silva               | `192.168.1.0/24` | Current main network and Debian Server laboratory environment |
| Previous Debian Lab Network | `192.168.2.0/24` | Isolated network used during Labs 01-04                       |

The Debian Server currently operates on the `192.168.1.0/24` network.

The administration workstation can communicate with the server directly through the current LAN.

The current Debian Server configuration includes:

| Interface | Network          | Address         |
| --------- | ---------------- | --------------- |
| `wlan0`   | `192.168.1.0/24` | `192.168.1.117` |

The router acts as the network gateway and DHCP server.

The server uses DHCP, with a DHCP reservation configured on the router to maintain the expected address `192.168.1.117`.

This approach allows the Debian Server to continue using DHCP while maintaining a predictable IP address for services such as SSH, Apache, and Zabbix.

## DHCP Reservation

The current server address is maintained through a DHCP reservation on the network router.

The reservation associates the server's network interface with its assigned address:

| Parameter          | Value               |
| ------------------ | ------------------- |
| Interface          | `wlan0`             |
| MAC Address        | `dc:35:f1:b3:9a:1a` |
| Reserved Address   | `192.168.1.117`     |
| Gateway            | `192.168.1.1`       |
| Address Assignment | DHCP                |

This configuration was documented in [Lab 07](./07-dhcp-reservation-and-ip-address-persistence/README.md).

The reservation was introduced after the server received `192.168.1.116` instead of its previously used address `192.168.1.117`.

The investigation demonstrated the difference between a DHCP address, a manually configured static address, and a DHCP reservation.

## Documentation Philosophy

Each laboratory should document the relevant parts of the learning process, including:

* Objective
* Scenario or problem
* Environment
* Concepts learned
* Implementation
* Important commands
* Expected results
* Actual results
* Testing and verification
* Troubleshooting
* Lessons learned
* Key Takeaways

The objective is not to create a transcript of every command executed.

Instead, the documentation should explain what was done, why it was done, what happened, and what was learned from the process.

## Linux and Networking

An important part of this project is connecting Linux administration with networking concepts.

The same fundamental networking concepts studied with Cisco IOS can also be observed and managed in Linux, although the commands and implementation methods are different.

For example:

| Concept                 | Cisco IOS                            | Linux       |
| ----------------------- | ------------------------------------ | ----------- |
| View interfaces         | `show ip interface brief`            | `ip addr`   |
| View routing table      | `show ip route`                      | `ip route`  |
| Test connectivity       | `ping`                               | `ping`      |
| View listening services | `show control-plane host open-ports` | `ss -tulpn` |

These comparisons are intended to reinforce concepts across different platforms rather than imply that the commands are equivalent.

## Debian Server Documentation

The primary technical reference for this project is the official Debian documentation:

https://www.debian.org/doc/

The documentation provides technical information about Debian systems, installation, administration, configuration, and available software.

Learning how to locate and use official documentation is itself part of the administration skill being developed through this project.

## Physical Server

Unlike the Ubuntu Server Labs repository, which uses a virtual machine, this project uses a physical computer as the server.

This provides an opportunity to work with aspects of real hardware and physical server administration, including:

* Wireless network connectivity
* Limited hardware resources
* Physical storage
* System boot and installation
* Headless administration
* Remote administration through SSH
* Service management on a physical system
* Network troubleshooting

The server is intended to operate without requiring a dedicated monitor, keyboard, or graphical interface during normal administration.

## SSH Administration

SSH is the primary method used to administer the Debian Server remotely.

The current administration model is:

```text
Ubuntu Workstation
       |
       | SSH
       |
       v
Debian Server
192.168.1.117
```

The current server and administration workstation communicate through the main local network.

Older laboratories may show the previous `192.168.2.0/24` laboratory network. Those addresses belong to the earlier network architecture and are retained in the historical documentation for accuracy.

Commands executed after establishing the SSH session operate on the Debian Server rather than on the Ubuntu workstation.

This allows the physical server to operate independently while being managed remotely.

## Services and Infrastructure

As the laboratories progress, the server environment is being expanded beyond basic operating system administration.

The current project includes practical work with:

* SSH
* UFW
* Zabbix
* Zabbix Agent
* Apache2
* Web monitoring
* DHCP
* Network troubleshooting
* Static web content
* System monitoring
* Resource monitoring
* Linux networking

The laboratory environment is intentionally developed incrementally. Each new service provides an opportunity to connect Linux administration with networking, monitoring, security, and infrastructure concepts.

## Purpose

This repository is intended to become a practical Debian Server administration knowledge base built through real configuration, experimentation, verification, and troubleshooting.

The goal is to have documentation that remains useful after completing the individual laboratories and can be consulted when similar concepts appear in future studies or real administration scenarios.

The project will progressively expand from initial server configuration into Linux administration, networking, services, monitoring, traffic analysis, hardening, and security experiments.

The network architecture may also evolve as new laboratories introduce additional services and infrastructure requirements. Historical configurations are preserved in the corresponding laboratory documentation, while this README describes the current environment whenever possible.
