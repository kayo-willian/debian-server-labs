# Debian Server Labs

Hands-on Debian Server labs focused on Linux administration, networking, services, security, and infrastructure.

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

| Lab | Topic                                                                       | Description                                                                                                                                             |
| --- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 01  | [Installation, Configuration and SSH](./01-installation-configuration-ssh/) | Installation and initial configuration of Debian Server, wireless networking, dedicated laboratory subnet, connectivity testing, and SSH administration |

This table will grow as new laboratories are created.

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

### Network Environment

The laboratory uses a dedicated wireless network separated from the main home network.

| Network       | Subnet           | Purpose            |
| ------------- | ---------------- | ------------------ |
| Familia Silva | `192.168.1.0/24` | Main network       |
| Debian        | `192.168.2.0/24` | Laboratory network |

The Debian wireless network uses a dedicated SSID subnet configured on the network gateway.

The Ubuntu workstation used for administration connects to both networks:

| Interface         | Network       | IP Address      |
| ----------------- | ------------- | --------------- |
| `enp2s0`          | Familia Silva | `192.168.1.230` |
| `wlx002e2d5b7b55` | Debian        | `192.168.2.143` |

The Debian Server currently uses:

| Interface | Network | IP Address      |
| --------- | ------- | --------------- |
| `wlan0`   | Debian  | `192.168.2.161` |

The separation between the two IPv4 subnets allows the administration workstation to maintain connectivity with both the main network and the isolated laboratory environment.

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

The server is intended to operate without requiring a dedicated monitor, keyboard, or graphical interface during normal administration.

## SSH Administration

SSH is the primary method used to administer the Debian Server remotely.

The current administration model is:

```text
Ubuntu Workstation
192.168.2.143
       |
       | SSH
       |
       v
Debian Server
192.168.2.161
```

Commands executed after establishing the SSH session operate on the Debian Server rather than on the Ubuntu workstation.

This allows the physical server to operate independently while being managed remotely.

## Purpose

This repository is intended to become a practical Debian Server administration knowledge base built through real configuration, experimentation, verification, and troubleshooting.

The goal is to have documentation that remains useful after completing the individual laboratories and can be consulted when similar concepts appear in future studies or real administration scenarios.

The project will progressively expand from initial server configuration into Linux administration, networking, services, monitoring, traffic analysis, hardening, and security experiments.
