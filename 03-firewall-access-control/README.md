# Lab 03: Firewall and Access Control with UFW

Hands-on laboratory focused on securing a physical Debian Server with a host based firewall, building an explicit access control policy, and verifying that protection works from multiple network clients.

This lab builds directly on Lab 02. The Zabbix monitoring stack deployed previously now serves as the test subject: the firewall is configured to protect its web interface while preserving every legitimate communication path of the monitoring system.

The goal is to move from "the service works" to "the service works and only authorized parties can reach it".

## Objective

The objectives of this lab were:

* Understand what a host based firewall is and how it differs from network perimeter security
* Learn that a firewall operates on a default deny model and how an allowlist is built
* Install and configure UFW on Debian Server
* Restrict SSH and HTTP access to specific trusted source addresses
* Preserve required service communication while blocking everything else
* Validate the policy from both allowed and blocked devices
* Understand the relationship between DHCP addressing and IP based firewall rules

---

## 1. Why a Host Based Firewall

The server previously accepted connections from any device on the laboratory subnet. Any device connected to the laboratory wireless network could open the Zabbix web interface simply by entering the server address in a browser. The web login page of an administrative interface should not be reachable by every device on the network.

There are two complementary levels of protection:

| Level | Mechanism | Scope |
| ----- | --------- | ----- |
| Perimeter filtering | Router based rules separating subnets | Controls what crosses between networks |
| Host firewall | Filters traffic on the server itself | Controls every connection that reaches the machine |

The network design from Lab 01 already separates the laboratory subnet from the home network. However, devices connected to the laboratory wireless network could still reach every service on the server. The host firewall closes this gap by enforcing the policy directly on the machine.

UFW, the Uncomplicated Firewall, is a frontend for the kernel packet filtering subsystem. It was chosen because it expresses packet filtering rules in readable commands while managing the underlying complexity automatically.

---

## 2. The Access Control Model

UFW operates on the principle of default deny: every connection that does not match an allow rule is dropped. There is no need for explicit block rules. Blocking is the default state, and the administrator builds an explicit list of permitted communications.

This model is called an allowlist and mirrors how access control is designed in production environments: state exactly who may connect, for which purpose, and deny everything else implicitly.

The policy designed for this server:

| Source | Destination port | Protocol | Purpose |
| ------ | ---------------- | -------- | ------- |
| Administration workstation | 22 | TCP | SSH administration |
| Administration workstation | 80 | TCP | Zabbix web frontend |
| Secondary admin device | 22 | TCP | SSH administration from mobile terminal |
| Secondary admin device | 80 | TCP | Zabbix frontend viewing |
| Any monitored host | 10051 | TCP | Agents pushing collected data |
| Everyone else | Nothing | | Implicitly denied |

Two components required exceptions and two components deliberately received none:

* SSH must remain reachable because it is the only administrative channel to a headless server
* TCP 10051 must accept connections from agents, since agents push data to the server
* The database requires no rule because MariaDB listens only on the loopback interface and never leaves the machine
* The frontend was restricted by source address instead of opened to the entire subnet

---

## 3. Installation

```bash
sudo apt install ufw -y
```

Rules are created while the firewall is still inactive. This ordering is critical and is explained in the troubleshooting section: activating a default deny firewall before adding an SSH rule locks the administrator out of the machine.

---

## 4. Building the Rule Set

Each rule grants a specific source address a specific destination port.

### Allow SSH only from the administration workstation

```bash
sudo ufw allow from <WORKSTATION_IP> to any port 22 proto tcp comment 'SSH from administration workstation'
```

Without this rule, activating the firewall would terminate the active session and make the server unreachable over the network.

### Restrict the Web Frontend

The Zabbix frontend listens on port 80. Opening it only to the administration workstation and the secondary admin device:

```bash
sudo ufw allow from <WORKSTATION_IP> to any port 80 proto tcp comment 'Zabbix frontend from workstation'
sudo ufw allow from <SECONDARY_DEVICE_IP> to any port 80 proto tcp comment 'Zabbix frontend from secondary admin device'
```

Every other address on the network can no longer reach the web interface.

### Permit Agent Traffic

Agents send collected data to the server on TCP 10051:

```bash
sudo ufw allow 10051/tcp comment 'Zabbix agent data'
```

This rule has no source restriction because agents may run on multiple hosts.

### Activate the Firewall

```bash
sudo ufw enable
sudo ufw status numbered
```

`status numbered` displays the rule set with indexes, which are used to reference rules when deleting or inspecting them.

---

## 5. Rules by Scope

UFW rules can grant access at different granularities:

```bash
# Everything from one address
sudo ufw allow from <IP>

# One port from one address
sudo ufw allow from <IP> to any port <PORT> proto tcp

# One port from an entire subnet
sudo ufw allow from 192.168.2.0/24 to any port <PORT> proto tcp
```

A subnet wide rule grants access to every device in that range and defeats the purpose of per host restriction. The value of this design lies in granting access narrowly: per address, per port, per purpose.

---

## 6. Verification

Validation was performed from three different origins:

| Test | Origin | Expectation | Result |
| ---- | ------ | ----------- | ------ |
| Frontend from workstation | `http://<SERVER_IP>/zabbix/` | Loads normally | Confirmed |
| Frontend from secondary device | Same address | Loads normally | Confirmed |
| Frontend from unlisted device | Same address | Connection times out, page never loads | Confirmed |
| SSH from allowed sources | Port 22 | Session establishes | Confirmed |
| SSH from unlisted device | Port 22 | Connection blocked | Confirmed |
| Agent data flow | ZBX indicator | Remains green | Confirmed |

The mobile device test is the clearest demonstration: before the rule, the page never finishes loading because the firewall silently drops the packets; after adding the rule, the dashboard loads normally.

## Checking the Active Policy

```bash
sudo ufw status numbered
```

The output lists every rule with its position number. Rules can be removed by number with:

```bash
sudo ufw delete <NUMBER>
```

Rules can also be removed by restating them with `delete`, and the comment field keeps each rule self documenting in listings.

---

## 7. Defense in Depth Beyond the Firewall

The firewall controls which addresses may open a connection. Two additional layers complete the protection model.

### Authentication Quality

Firewall rules grant network reachability, not authorization. Whoever passes the firewall still needs valid credentials. Strong passwords stored in a password manager, and preferably key based authentication instead of passwords, are what actually protect an open SSH port.

### Brute Force Protection

A firewall controls who may connect. A companion service called fail2ban monitors authentication logs and temporarily blocks sources that repeatedly fail to authenticate. It adds a dynamic layer on top of static firewall rules and is planned as a separate laboratory.

### Naming a Fundamental Principle

Defense in depth: no single control is trusted alone. Firewall restricts reachability, strong authentication restricts access, and log monitoring detects abuse. Each layer covers a failure of the layer before it.

---

## 8. Limitations Encountered

Two constraints shaped this configuration and are worth documenting.

### Router Does Not Support DHCP Reservations on the Laboratory Subnet

The laboratory subnet is provided by an SSID subnet feature whose firmware exposes static DHCP bindings only for the primary LAN. Assigning a reservation to the server address was therefore rejected by the router interface with a subnet validation error.

Consequences:

* Server and workstation addresses remain dynamically assigned
* Firewall rules reference specific source addresses, so if DHCP reassigns a workstation address, its access stops until the rule is updated
* The probability of change is low but not zero, and this trade off is documented consciously

An alternative addressing scheme was considered and intentionally deferred: configuring a static address on the server outside the DHCP pool. The procedure was analyzed, including the safe migration sequence of adding a secondary address before removing DHCP, but was postponed to keep the current environment stable.

### IP Based Rules Depend on Stable Addressing

The entire policy rests on addresses staying predictable. This lab documented that limitation explicitly rather than hiding it. In larger environments the standard solutions are DHCP reservations, statically addressed infrastructure, or both.

---

## 9. Troubleshooting Summary

| Situation | Detection | Resolution |
| --------- | --------- | ---------- |
| Lockout risk before activation | First firewall activation with no SSH rule | Always create the SSH allowance before `ufw enable` |
| Legitimate device blocked | Page does not load from trusted phone | Add specific allow rule for its address |
| Rule not taking effect | `ufw status numbered` review | Verify rule order and source address correctness |

The design lesson: change firewall configuration in a safe order. Permissive rules first, activation last, verification immediately after.

---

## 10. Verification Checklist

* [x] UFW installed on the Debian server
* [x] SSH rule scoped to the administration workstation address
* [x] SSH rule scoped to the secondary administrative device
* [x] HTTP rule scoped to the administration workstation address
* [x] HTTP rule scoped to the secondary administrative device
* [x] Agent data port open for incoming connections
* [x] Firewall enabled with default deny inbound policy
* [x] Frontend loads from both authorized devices
* [x] Frontend unreachable from unauthorized devices
* [x] SSH sessions unaffected for authorized devices
* [x] Agent data delivery verified through continued green availability indicators

---

## 11. Key Takeaways

A host firewall converts an open server into a server with an explicit access policy. Before the firewall, every device on the laboratory network could reach every listening service. After the firewall, reachability became a deliberate decision recorded as a rule.

Default deny is the property that makes this work. Without it, each new service added to the server would automatically become reachable by everyone. With it, exposing anything new requires an explicit, reviewed decision.

Firewall and authentication solve different problems. The firewall decides whether a packet may arrive at the service; authentication decides whether the requester may use it. Layers are additive, not redundant.

Network reachability rules are only as stable as the addressing they reference. Dynamic addresses create a standing dependency between DHCP behavior and firewall correctness, which is a real operational trade off worth understanding rather than ignoring.

---
