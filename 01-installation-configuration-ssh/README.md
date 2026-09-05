# Lab 01: Installation, Configuration and SSH

## Objective

The objective of this lab was to install Debian Server on a physical machine, configure it as a headless Linux server, establish network connectivity, create a dedicated laboratory subnet, and remotely administer the server through SSH.

This lab represents the initial foundation for the Debian Server Labs repository.

The goal was not only to make the server work, but to understand the reasoning behind each configuration, verify the results, identify problems, and document the troubleshooting process.

The main concepts covered were:

* Debian Server installation
* Headless server administration
* Network interface identification
* IPv4 addressing
* Subnets
* DHCP
* Default gateways
* Routing
* Network separation
* Network metrics
* SSH
* Remote administration
* Connectivity testing
* Basic troubleshooting

---

# 1. Lab Environment

## Physical Server

The server used in this lab is an old laptop repurposed as a physical Linux server.

| Component           | Specification                  |
| ------------------- | ------------------------------ |
| Device              | Positivo Motion                |
| CPU                 | Intel Atom x5-Z8350 @ 1.44 GHz |
| RAM                 | 2 GB DDR3                      |
| Storage             | Approximately 32 GB eMMC       |
| Network             | Wi-Fi                          |
| Operating System    | Debian Server                  |
| Desktop Environment | None                           |

The machine is being used as a headless server.

A headless server does not require a graphical desktop environment for normal administration. Instead, administration is performed remotely through SSH.

This is especially useful for this machine because of its limited hardware resources.

---

# 2. Why Debian Server?

Debian was chosen for this physical server because it is lightweight, stable, widely used, and provides a good environment for learning Linux server administration.

The installation was performed without a desktop environment.

This means the server boots into a command-line environment instead of loading a graphical interface.

The intention is to learn server administration through the terminal rather than relying on a graphical interface.

---

# 3. Installation

The Debian Server installation was performed directly on the physical machine.

During the installation, the system was configured as a minimal server environment.

A graphical desktop environment was not installed.

The installation focused on the components required for future server administration and networking labs.

## Installed Components

The relevant components selected during installation were:

* SSH Server
* Standard system utilities
* Debian base system

No desktop environment was installed.

No web server was installed at this stage.

The intention is to install additional services later as separate labs so that each service can be studied independently.

---

# 4. First Login

After installation, the server was accessed locally through the terminal.

The Debian user created for administration is:

```text
kayo
```

The system can be accessed through this user account.

To perform administrative tasks, the root account can be accessed with:

```bash
su -
```

The root shell provides administrative privileges for system-level configuration.

The normal user should be preferred for everyday administration, while root access should only be used when elevated privileges are actually required.

---

# 5. Initial Network Configuration

The Debian server connects to the network using Wi-Fi.

The server's wireless interface is:

```text
wlan0
```

The server initially received an IP address through DHCP.

The network configuration eventually used by the server was:

```text
IP Address: 192.168.2.161
Subnet Mask: 255.255.255.0
Network: 192.168.2.0/24
Gateway: 192.168.2.1
```

The `/24` prefix means that the subnet mask is:

```text
255.255.255.0
```

This provides:

```text
Network:    192.168.2.0
Usable IPs: 192.168.2.1 - 192.168.2.254
Broadcast:  192.168.2.255
```

The server therefore belongs to the `192.168.2.0/24` laboratory network.

---

# 6. The Original Network Problem

Before creating the dedicated laboratory subnet, the Ubuntu computer and the Debian server were both connected to the same network.

The relevant addresses were:

```text
Ubuntu Ethernet: 192.168.1.230
Ubuntu Wi-Fi:    192.168.1.222
Debian Server:   192.168.1.117
```

At first glance, this appeared to be convenient because all devices were inside:

```text
192.168.1.0/24
```

However, the Ubuntu computer had two network interfaces connected to the same subnet.

The interfaces were:

```text
enp2s0
wlx002e2d5b7b55
```

The routing table contained two possible paths to the same network.

The relevant routes were effectively:

```text
192.168.1.0/24 dev enp2s0
192.168.1.0/24 dev wlx002e2d5b7b55
```

Linux therefore had to decide which interface should be used when communicating with another `192.168.1.x` address.

The Ethernet interface had a better route metric.

For example:

```text
enp2s0      metric 100
Wi-Fi       metric 600
```

Lower metrics are preferred.

Therefore, when Ubuntu attempted to reach:

```text
192.168.1.117
```

the operating system preferred the Ethernet interface.

This could be verified with:

```bash
ip route get 192.168.1.117
```

The result showed that traffic was being directed through the Ethernet interface instead of the Wi-Fi interface.

This created an important networking problem.

The server was connected through Wi-Fi, but Ubuntu had another interface on the same IP network through Ethernet.

The problem was not simply that the devices were disconnected.

The problem was that the host had two interfaces belonging to the same subnet, creating an ambiguous routing situation.

---

# 7. Why Create a Separate Laboratory Subnet?

Instead of trying to force Linux to choose the Wi-Fi interface manually, the network was redesigned.

The laboratory network was separated from the main home network.

The main network remained:

```text
192.168.1.0/24
```

The laboratory network became:

```text
192.168.2.0/24
```

This creates a clear separation:

```text
Main Network
192.168.1.0/24
        |
        |
     Ethernet
        |
      Ubuntu
        |
        |
       Wi-Fi
        |
        |
Lab Network
192.168.2.0/24
        |
        |
 Debian Server
```

This design also makes the laboratory environment easier to understand and manage.

Instead of having Ubuntu reach the Debian server through two interfaces belonging to the same subnet, each interface now belongs to a different IP network.

---

# 8. Creating the Laboratory Subnet

The network separation was created using the router's wireless configuration.

The router provides the main network through the primary SSID:

```text
Familia Silva
```

The dedicated laboratory SSID was:

```text
Debian
```

Initially, the `Debian` SSID was configured as an `External Guest` network.

This caused stronger client isolation and prevented the devices connected to the laboratory SSID from communicating with each other as required for the lab.

The SSID was therefore changed to:

```text
Home Guest
```

This allowed devices connected to the same laboratory network to communicate.

---

# 9. SSID Subnet Configuration

The router provides an option called:

```text
SSID Subnet
```

This was enabled for the `Debian` SSID.

The laboratory network was configured with the following parameters:

| Parameter      | Value          |
| -------------- | -------------- |
| SSID           | Debian         |
| Network        | 192.168.2.0/24 |
| LAN IP Address | 192.168.2.1    |
| Subnet Mask    | 255.255.255.0  |
| DHCP Start     | 192.168.2.100  |
| DHCP End       | 192.168.2.200  |

The router therefore acts as the gateway for the laboratory network:

```text
192.168.2.1
```

---

# 10. DHCP Configuration

DHCP was configured for the laboratory subnet.

The DHCP pool was defined as:

```text
192.168.2.100
-
192.168.2.200
```

This means that clients connecting to the `Debian` SSID can automatically receive addresses within this range.

For example:

```text
192.168.2.100
192.168.2.101
192.168.2.102
...
192.168.2.200
```

The router provides the network configuration to clients through DHCP, including information such as:

* IP address
* Subnet mask
* Default gateway
* Other network parameters

The server eventually received:

```text
192.168.2.161
```

which falls inside the configured DHCP range.

---

# 11. Final Network Architecture

After the network redesign, the environment became:

```text
                    Home Network
                  192.168.1.0/24
                         |
                         |
                     Ethernet
                         |
                         |
                  Ubuntu Computer
                         |
                         |
                        Wi-Fi
                         |
                         |
                    Lab Network
                  192.168.2.0/24
                         |
                         |
                  Debian Server
                   192.168.2.161
```

The Ubuntu computer therefore has two network interfaces connected to two different networks.

## Ubuntu

Ethernet:

```text
Interface: enp2s0
IP:        192.168.1.230/24
Network:   192.168.1.0/24
```

Wi-Fi:

```text
Interface: wlx002e2d5b7b55
IP:        192.168.2.143/24
Network:   192.168.2.0/24
```

## Debian Server

```text
Interface: wlan0
IP:        192.168.2.161/24
Network:   192.168.2.0/24
Gateway:   192.168.2.1
```

---

# 12. Verifying the Ubuntu Network Interfaces

The Ubuntu machine was used as the administration workstation.

The network interfaces were inspected with:

```bash
ip addr
```

The important interfaces were:

```text
enp2s0
wlx002e2d5b7b55
```

The resulting addressing was:

```text
enp2s0
192.168.1.230/24

wlx002e2d5b7b55
192.168.2.143/24
```

This confirmed that Ubuntu was simultaneously connected to both networks.

---

# 13. Verifying the Routing Table

The routing table was checked with:

```bash
ip route
```

The relevant routes were:

```text
default via 192.168.1.1 dev enp2s0 metric 100
default via 192.168.2.1 dev wlx002e2d5b7b55 metric 600

192.168.1.0/24 dev enp2s0 src 192.168.1.230 metric 100
192.168.2.0/24 dev wlx002e2d5b7b55 src 192.168.2.143 metric 600
```

The important difference compared with the original configuration is that the two interfaces now belong to different networks.

Ubuntu has:

```text
192.168.1.0/24
```

through Ethernet and:

```text
192.168.2.0/24
```

through Wi-Fi.

There is therefore no ambiguity about which interface should be used to reach the Debian server.

---

# 14. Verifying the Route to the Server

The exact route to the Debian server was checked using:

```bash
ip route get 192.168.2.161
```

The result was:

```text
192.168.2.161 dev wlx002e2d5b7b55 src 192.168.2.143
```

This is an important verification.

It proves that Ubuntu reaches:

```text
192.168.2.161
```

through:

```text
wlx002e2d5b7b55
```

using:

```text
192.168.2.143
```

as its source address.

In other words:

```text
Ubuntu Wi-Fi
192.168.2.143
      |
      |
      v
Debian Server
192.168.2.161
```

The traffic is no longer incorrectly sent through the Ethernet interface.

---

# 15. Testing Connectivity

Connectivity between Ubuntu and Debian was tested with:

```bash
ping -c 4 192.168.2.161
```

The test returned:

```text
4 packets transmitted
4 packets received
0% packet loss
```

This confirmed basic IP connectivity between the two machines.

The successful ping demonstrated that:

* The Debian server had network connectivity.
* Ubuntu could reach the laboratory subnet.
* The two devices could communicate through Wi-Fi.
* The router was forwarding the required traffic.
* The subnet configuration was working.
* The routing decision on Ubuntu was correct.

---

# 16. SSH Server

SSH was installed during the Debian installation.

SSH provides secure remote administration of Linux systems.

Instead of connecting a monitor and keyboard to the Debian machine, the Ubuntu computer can open a remote shell session.

This is particularly useful for the physical server because the machine is being used as a headless server.

The server can therefore be administered from the main computer without requiring a graphical environment.

---

# 17. SSH User

The Debian user created for remote administration is:

```text
kayo
```

The server can be accessed using the following command from Ubuntu:

```bash
ssh kayo@192.168.2.161
```

The command follows this structure:

```text
ssh USER@SERVER_IP
```

Therefore:

```text
ssh
```

starts the SSH client.

```text
kayo
```

is the user account on the Debian server.

```text
192.168.2.161
```

is the destination IP address of the server.

---

# 18. First Remote Access

After executing:

```bash
ssh kayo@192.168.2.161
```

the SSH client establishes a connection to the Debian server.

After authentication, the shell changes to the remote Debian environment.

The prompt becomes similar to:

```text
kayo@debian:~$
```

At this point, commands typed into the terminal are executed on the Debian server, not on Ubuntu.

This distinction is important when administering a remote machine.

For example:

```bash
hostname
```

executed inside the SSH session checks the hostname of the Debian server.

Commands executed before establishing the SSH connection continue to operate on the Ubuntu machine.

---

# 19. Password Recovery

During the initial SSH configuration, the password for the `kayo` account was not available.

Because local administrative access was still available, the password was reset using the root account.

First, root access was obtained:

```bash
su -
```

Then the password for the user was changed:

```bash
passwd kayo
```

The system requested a new password and confirmation.

After the password was changed, SSH authentication was attempted again:

```bash
ssh kayo@192.168.2.161
```

The connection succeeded.

This demonstrated an important administration concept:

A user's password can be changed by an administrator with sufficient privileges without requiring the administrator to know the user's previous password.

---

# 20. What Happens When SSH Is Closed?

SSH provides a remote session to the server.

Closing the SSH connection does not shut down the Debian machine.

For example, the following command:

```bash
exit
```

ends the current SSH session.

The Debian server continues running normally.

The relationship can be understood as:

```text
Ubuntu
   |
   | SSH connection
   v
Debian Server
   |
   | Server continues running
   v
Services remain active
```

SSH is therefore a method of remotely accessing the server, not the mechanism that keeps the server powered on.

---

# 21. Troubleshooting Process

One of the most important parts of this lab was not the final configuration, but understanding why the original network design caused problems.

The troubleshooting process followed a basic networking methodology.

## Step 1: Identify the interfaces

```bash
ip addr
```

This showed that Ubuntu had both Ethernet and Wi-Fi interfaces.

## Step 2: Check the IP addresses

The two interfaces were initially inside the same network:

```text
192.168.1.0/24
```

## Step 3: Check the routing table

```bash
ip route
```

This showed that Ethernet had the preferred route because of its lower metric.

## Step 4: Ask Linux how it would reach the server

```bash
ip route get 192.168.1.117
```

This demonstrated that the traffic was being directed through Ethernet.

## Step 5: Redesign the network

Instead of manually manipulating route metrics, a dedicated laboratory subnet was created.

```text
Main network:
192.168.1.0/24

Lab network:
192.168.2.0/24
```

## Step 6: Verify the new route

```bash
ip route get 192.168.2.161
```

The result showed:

```text
dev wlx002e2d5b7b55
```

## Step 7: Test connectivity

```bash
ping -c 4 192.168.2.161
```

The test succeeded with:

```text
0% packet loss
```

## Step 8: Test remote administration

```bash
ssh kayo@192.168.2.161
```

SSH access succeeded.

---

# 22. Important Networking Concepts Learned

## Subnet

A subnet is a logical division of an IP network.

In this lab:

```text
192.168.1.0/24
```

and:

```text
192.168.2.0/24
```

are two different IPv4 networks.

---

## Default Gateway

The default gateway is the device used to reach networks outside the local subnet.

For the Debian laboratory network:

```text
Gateway:
192.168.2.1
```

This address belongs to the router.

---

## DHCP

DHCP automatically provides network configuration to clients.

The laboratory DHCP pool was:

```text
192.168.2.100 - 192.168.2.200
```

The Debian server received:

```text
192.168.2.161
```

---

## Routing

Routing determines where network traffic should be sent.

The Ubuntu computer had two networks:

```text
192.168.1.0/24 -> Ethernet
192.168.2.0/24 -> Wi-Fi
```

The destination network determines which interface is appropriate.

---

## Route Metric

A route metric is used to determine preference when multiple routes are available.

Lower values are preferred.

The original configuration contained:

```text
Ethernet metric: 100
Wi-Fi metric:    600
```

Therefore, Ethernet was preferred when both interfaces could potentially reach the same network.

The final design avoided this ambiguity by placing the interfaces in different subnets.

---

## SSH

SSH provides encrypted remote access to a system.

In this lab:

```text
Ubuntu
192.168.2.143
      |
      | SSH
      |
Debian
192.168.2.161
```

The Debian machine can therefore be administered remotely.

---

# 23. Final Configuration

## Debian Server

```text
Hostname: Debian
User: kayo

Interface:
wlan0

IP Address:
192.168.2.161/24

Gateway:
192.168.2.1

Network:
192.168.2.0/24
```

## Ubuntu Administration Machine

### Ethernet

```text
Interface:
enp2s0

IP:
192.168.1.230/24

Network:
192.168.1.0/24
```

### Wi-Fi

```text
Interface:
wlx002e2d5b7b55

IP:
192.168.2.143/24

Network:
192.168.2.0/24
```

## Router Laboratory Network

```text
SSID:
Debian

Network:
192.168.2.0/24

Router:
192.168.2.1

Subnet Mask:
255.255.255.0

DHCP Range:
192.168.2.100 - 192.168.2.200
```

---

# 24. Verification Checklist

The following tests were completed successfully:

* [x] Debian installed on physical hardware
* [x] Desktop environment not installed
* [x] SSH server installed
* [x] Administrative user created
* [x] User password configured
* [x] Wi-Fi interface identified
* [x] Dedicated laboratory SSID created
* [x] SSID configured as Home Guest
* [x] SSID Subnet enabled
* [x] Laboratory subnet configured
* [x] DHCP range configured
* [x] Debian received an address from the laboratory network
* [x] Ubuntu connected to both networks
* [x] Routing table verified
* [x] Route to Debian verified with `ip route get`
* [x] ICMP connectivity tested with `ping`
* [x] SSH connectivity tested
* [x] Remote shell access confirmed

---

# 25. Key Takeaways

This lab established the basic architecture that will be used by the remaining Debian Server Labs.

The most important lesson was that network problems are not always caused by a disconnected device or an incorrect IP address.

A machine can have valid IP addresses on all interfaces and still experience connectivity problems because of routing decisions.

The original configuration demonstrated the problem of having multiple interfaces connected to the same subnet.

The final configuration solved the problem by separating the networks:

```text
192.168.1.0/24
        |
     Ethernet
        |
      Ubuntu
        |
       Wi-Fi
        |
192.168.2.0/24
        |
   Debian Server
```

This provided a cleaner laboratory architecture and made the routing behavior predictable.

The lab also established SSH as the primary method of administering the Debian server.

The server can now operate without a graphical environment or constant physical access.

---

# 26. Next Stage

With the Debian server installed, reachable, and accessible through SSH, the next labs can focus on Linux system administration and services.

Possible topics include:

```text
Linux Administration
        ↓
Users and Groups
        ↓
Permissions
        ↓
Processes and Services
        ↓
Logs and Troubleshooting
        ↓
Firewall
        ↓
Web Server
        ↓
DNS
        ↓
Traffic Analysis
        ↓
Server Hardening
```

Each topic will be implemented as a separate practical lab.

The intention is to maintain the same learning cycle throughout the repository:

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
