# Lab 14 - BIND9 DNS Server

## Objective

Set up a DNS server using BIND9 on a physical Debian server, configure forward and reverse DNS zones, test DNS resolution from another machine on the network, and integrate DNS monitoring with Zabbix.

## Environment

| Device                   | Role                       | IP              |
| ------------------------ | -------------------------- | --------------- |
| Debian Server            | BIND9 DNS Server           | `192.168.1.117` |
| Debian Monitoring Server | DNS Client / Zabbix Server | `192.168.1.219` |
| Ubuntu PC                | Optional DNS Client        | `192.168.1.230` |

### Services

| Service      |      Port |
| ------------ | --------: |
| DNS          |    53/UDP |
| DNS          |    53/TCP |
| Zabbix Agent | 10050/TCP |

## 1. Install BIND9

Install BIND9 and the required DNS utilities:

```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils
```

Check the service:

```bash
sudo systemctl status bind9
```

On this Debian installation, the actual systemd service is `named.service`, with `bind9.service` acting as a linked unit.

## 2. Configure BIND9 Options

Edit:

```bash
sudo nano /etc/bind/named.conf.options
```

Configuration:

```conf
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    listen-on { any; };
    listen-on-v6 { any; };

    allow-query { localhost; 192.168.1.0/24; };

    recursion yes;
    allow-recursion { localhost; 192.168.1.0/24; };

    allow-transfer { none; };

    dnssec-validation auto;
};
```

The `192.168.1.0/24` network allows DNS queries from the local network.

Validate the configuration:

```bash
sudo named-checkconf
```

No output indicates that the configuration syntax is valid.

## 3. Create DNS Zones

Create the zones directory:

```bash
sudo mkdir -p /etc/bind/zones
```

Edit:

```bash
sudo nano /etc/bind/named.conf.local
```

Add the forward zone:

```conf
zone "example.lan" IN {
    type master;
    file "/etc/bind/zones/db.example.lan";
    allow-update { none; };
};
```

Add the reverse zone:

```conf
zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/bind/zones/db.192.168.1";
    allow-update { none; };
};
```

No secondary DNS server was configured in this lab.

## 4. Configure the Forward Zone

Create:

```bash
sudo nano /etc/bind/zones/db.example.lan
```

Configuration:

```dns
$TTL    604800
@       IN      SOA     ns1.example.lan. admin.example.lan. (
                         2026092801
                         604800
                         86400
                         2419200
                         604800
)

; Name server
@       IN      NS      ns1.example.lan.

; Name server address
ns1     IN      A       192.168.1.117

; Mail exchanger
@       IN      MX 10   mail.example.lan.

; A records for hosts
www     IN      A       192.168.1.117
mail    IN      A       192.168.1.117

; CNAME records
ftp     IN      CNAME   www.example.lan.
```

Validate the zone:

```bash
sudo named-checkzone example.lan /etc/bind/zones/db.example.lan
```

Expected result:

```text
zone example.lan/IN: loaded serial 2026092801
OK
```

## 5. Configure the Reverse Zone

Create:

```bash
sudo nano /etc/bind/zones/db.192.168.1
```

Configuration:

```dns
$TTL    604800
@       IN      SOA     ns1.example.lan. admin.example.lan. (
                         2026092801
                         604800
                         86400
                         2419200
                         604800
)

; Name server
@       IN      NS      ns1.example.lan.

; PTR records
117     IN      PTR     ns1.example.lan.
```

Validate the zone:

```bash
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1
```

Expected result:

```text
zone 1.168.192.in-addr.arpa/IN: loaded serial 2026092801
OK
```

## 6. Start BIND9

Restart the service:

```bash
sudo systemctl restart bind9
```

Check its status:

```bash
sudo systemctl status bind9
```

The service should show:

```text
Active: active (running)
```

The configured zones should also appear as loaded in the service log.

## 7. Test Forward DNS Resolution

From the Debian DNS server:

```bash
dig @127.0.0.1 ns1.example.lan
```

A successful authoritative response returned:

```text
ns1.example.lan.    604800    IN    A    192.168.1.117
```

A shorter version can be used:

```bash
dig @127.0.0.1 ns1.example.lan +short
```

Result:

```text
192.168.1.117
```

The `aa` flag in the full response indicates that the answer is authoritative for the configured zone.

## 8. Test Reverse DNS Resolution

Run:

```bash
dig @127.0.0.1 -x 192.168.1.117
```

Result:

```text
117.1.168.192.in-addr.arpa.    604800    IN    PTR    ns1.example.lan.
```

This confirms that the reverse zone is working.

## 9. Test DNS from Another Machine

The Debian Monitoring Server at `192.168.1.219` was used as the DNS client.

Forward lookup:

```bash
dig @192.168.1.117 ns1.example.lan
```

Result:

```text
ns1.example.lan.    604800    IN    A    192.168.1.117
```

Reverse lookup:

```bash
dig @192.168.1.117 -x 192.168.1.117
```

Result:

```text
117.1.168.192.in-addr.arpa.    604800    IN    PTR    ns1.example.lan.
```

These tests confirm that the DNS server can be accessed from another machine on the network.

## 10. Verify DNS Ports

From the Monitoring Server:

### UDP

```bash
nc -zuv 192.168.1.117 53
```

Result:

```text
53 (domain) open
```

### TCP

```bash
nc -zv 192.168.1.117 53
```

Result:

```text
53 (domain) open
```

BIND9 listens on both UDP/53 and TCP/53.

Check the listeners directly on the DNS server:

```bash
sudo ss -lntup | grep ':53'
```

The output confirms that `named` is listening on port 53.

## 11. Zabbix DNS Monitoring

The Debian Server already runs the Zabbix Agent.

A custom `UserParameter` was added to test an actual DNS resolution instead of only checking whether port 53 is open.

Edit:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Add:

```ini
UserParameter=dns.example.lan,dig @127.0.0.1 ns1.example.lan +short
```

Restart the agent:

```bash
sudo systemctl restart zabbix-agent
```

Verify:

```bash
sudo systemctl status zabbix-agent
```

From the Zabbix Monitoring Server, test the custom key:

```bash
zabbix_get -s 192.168.1.117 -k dns.example.lan
```

Result:

```text
192.168.1.117
```

This confirms that Zabbix can remotely execute the DNS health check through the Zabbix Agent.

## 12. Zabbix Item

A custom item was created on the `Debian Server` host:

```text
Name: DNS resolution - ns1.example.lan
Type: Zabbix agent
Key: dns.example.lan
Type of information: Text
Update interval: 1m
```

The item successfully returned:

```text
192.168.1.117
```

## 13. Zabbix Trigger

A trigger was created to detect an unexpected DNS response:

```text
Name: DNS resolution failed - ns1.example.lan
Severity: High
```

Expression:

```text
last(/Debian Server/dns.example.lan)<>"192.168.1.117"
```

The trigger allows Zabbix to identify when the expected DNS resolution is no longer being returned.

## Result

The lab successfully implemented a functional BIND9 DNS server on the physical Debian machine.

The environment now includes:

* Forward DNS zone
* Reverse DNS zone
* Authoritative DNS resolution
* `A`, `PTR`, `NS`, `MX`, and `CNAME` records
* DNS recursion and forwarders
* UDP/53 and TCP/53
* Remote DNS queries from another machine
* Zabbix monitoring of an actual DNS resolution
* Zabbix trigger for DNS resolution failure

## What I Practiced

* BIND9 installation and configuration
* DNS forward and reverse zones
* DNS record types
* Authoritative DNS queries
* Recursive DNS and forwarders
* DNS port 53 over UDP and TCP
* `dig` and `nc` for network testing
* BIND9 zone validation with `named-checkzone`
* BIND9 configuration validation with `named-checkconf`
* Zabbix `UserParameter`
* Custom Zabbix items and triggers
* Monitoring a network service based on its actual function rather than only checking its port
