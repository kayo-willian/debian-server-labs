# Lab 10 - Grafana Removal and Resource Optimization

## Overview

This lab documents the removal of Grafana from the Debian Server laboratory environment after completing the Grafana and Zabbix integration documented in Lab 09.

The Grafana dashboard was successfully created and used to visualize monitoring data collected by Zabbix.

After completing the practical objectives of the Grafana laboratory, the decision was made to remove Grafana from the physical Debian Server.

The decision was primarily based on the limited hardware resources available in the laboratory environment and the need to preserve those resources for future projects.

---

## Why Grafana Was Removed

The Debian Server used in this laboratory runs on limited hardware.

The available resources include restricted:

* RAM
* CPU processing capacity
* Disk storage

The server is being used as a practical laboratory environment where several services and future projects will be tested.

Keeping every service permanently installed would gradually consume storage and system resources, even when those services were no longer actively being used.

Grafana was therefore removed after its learning objectives had been completed.

This was not a result of a problem with Grafana or with the Zabbix integration.

The integration worked successfully and provided the expected monitoring dashboard.

The removal was a resource-management decision.

---

## Hardware Limitations

Running multiple infrastructure services on a low-resource machine requires careful management of available resources.

The laboratory server already contains services required for the monitoring environment, including:

* Zabbix Server
* Zabbix Agent
* MariaDB
* Apache2
* Zabbix Web

Adding Grafana introduced another application and its associated dependencies.

Although Grafana was useful during the dashboard laboratory, permanently maintaining it on the same machine was not necessary after the practical objectives had been completed.

Removing it reduces the amount of software permanently installed on the server and preserves resources for other experiments.

---

## Prioritizing Future Laboratories

The main reason for removing Grafana is to prepare the laboratory environment for the projects that will follow.

This server is not intended to be a production monitoring platform. It is a learning environment that will continuously change as new technologies and services are tested.

Storage and system resources therefore need to be treated as limited laboratory resources.

Instead of keeping software installed simply because it was previously used, completed experiments can be cleaned up when they are no longer required.

This makes room for:

* New services
* New Linux experiments
* New networking projects
* New server configurations
* Future monitoring experiments
* Additional infrastructure laboratories

The goal is to keep the environment flexible enough to support the next stages of the laboratory.

---

## Grafana Remains Part of the Learning Process

Removing Grafana from the physical server does not mean that the Grafana laboratory is lost.

The previous laboratory contains the dashboard documentation and exported configuration.

The dashboard configuration can also be reused in another environment if Grafana is installed again in the future.

A separate virtual machine can also be used for future Grafana experiments, allowing additional RAM and disk resources to be assigned without affecting the main Debian Server.

This provides a more suitable environment for continuing Grafana studies while keeping the physical server focused on its primary laboratory services.

---

## Removal Process

The removal was performed through the Debian package management system.

The Grafana service was first stopped and disabled.

```bash
sudo systemctl stop grafana-server
sudo systemctl disable grafana-server
```

The installed Grafana package was then removed:

```bash
sudo apt remove grafana
```

To remove configuration files associated with the package:

```bash
sudo apt purge grafana
```

Unused dependencies were then cleaned:

```bash
sudo apt autoremove
```

The package cache could also be cleaned to recover additional disk space:

```bash
sudo apt clean
```

---

## Verification

After the removal, the system was checked to confirm that Grafana was no longer running.

The service status can be checked with:

```bash
systemctl status grafana-server
```

The Grafana port can also be checked:

```bash
ss -tulpn | grep :3000
```

If Grafana has been successfully removed and no other application is using the port, there should be no Grafana service listening on TCP port `3000`.

Installed packages can also be searched with:

```bash
dpkg -l | grep grafana
```

These checks help confirm that the service and package were removed from the environment.

---

## Resource Management

This laboratory demonstrates that system administration is not only about installing and configuring software.

It also involves deciding which services should remain installed and which can be removed when they are no longer necessary.

In a resource-constrained laboratory, disk space, memory and processing capacity are valuable resources.

Removing completed laboratory components helps maintain a cleaner environment and makes it possible to continue building new projects without unnecessarily accumulating software and dependencies.

---

## Result

Grafana was successfully removed from the Debian Server environment after the completion of the Grafana dashboard laboratory.

The Zabbix monitoring environment remains available for the server's monitoring requirements.

The resources previously allocated to Grafana can now be used by future laboratories and experiments.

The Grafana configuration and documentation from Lab 09 remain available as a record of the completed integration.

---

## What This Lab Demonstrated

* Linux service management
* `systemctl`
* Debian package management
* Package removal
* Package purging
* Dependency cleanup
* Disk space management
* Resource management
* Infrastructure maintenance
* Laboratory environment management
* Planning for future projects

---

## Conclusion

Grafana fulfilled its purpose in the laboratory by providing practical experience with Zabbix integration and monitoring dashboards.

After completing that stage, maintaining Grafana permanently on the physical server was no longer necessary.

Because the laboratory hardware has limited RAM, processing capacity and disk storage, the available resources are better reserved for future experiments.

The removal of Grafana therefore represents the transition from one completed laboratory stage to the next.

The laboratory environment remains focused, lightweight and ready for the new projects that will follow.
