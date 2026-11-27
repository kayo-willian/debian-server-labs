# Laboratórios de Debian Server

<div align="center">

[![Read in English](https://img.shields.io/badge/READ%20IN%20ENGLISH-0B3D91?style=for-the-badge)](../README.md)
[![Documentação técnica](https://img.shields.io/badge/DOCUMENTA%C3%87%C3%83O%20T%C3%89CNICA-1E293B?style=for-the-badge)](docs/README.md)

![Debian](https://img.shields.io/badge/Debian-A81D33?style=flat-square) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square) ![SSH](https://img.shields.io/badge/SSH-475569?style=flat-square) ![Zabbix](https://img.shields.io/badge/Zabbix-D40000?style=flat-square) ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square) ![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=flat-square) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square) ![Apache](https://img.shields.io/badge/Apache-D22128?style=flat-square) ![UFW](https://img.shields.io/badge/UFW-64748B?style=flat-square) ![BIND9](https://img.shields.io/badge/BIND9-2563EB?style=flat-square)
[![MIT](https://img.shields.io/badge/Licen%C3%A7a-MIT-2563EB?style=flat-square)](../LICENSE)

</div>

Quatorze registros de laboratórios acompanham um Positivo Motion com Intel Atom x5-Z8350 e 2 GB de RAM, da instalação inicial do Debian até serviços DNS e monitoramento remoto. Os registros preservam os endereços históricos, comandos e observações de cada etapa, com identificadores pessoais mascarados.

## Laboratórios

| Lab | Tema | Escopo | EN |
| --- | --- | --- | --- |
| 01 | [Instalação, configuração e SSH](docs/laboratorios/01-instalacao-configuracao-ssh/README.md) | Debian físico, Wi-Fi, sub-rede dedicada e administração remota | [EN](../docs/labs/01-installation-configuration-ssh/README.md) |
| 02 | [Servidor de monitoramento Zabbix](docs/laboratorios/02-servidor-de-monitoramento-zabbix/README.md) | MariaDB, Zabbix, agentes, configuração web, retenção e diagnóstico | [EN](../docs/labs/02-monitoring-server-with-zabbix/README.md) |
| 03 | [Firewall e controle de acesso](docs/laboratorios/03-firewall-e-controle-de-acesso/README.md) | UFW, regras de entrada, SSH, HTTP e comunicação de agentes | [EN](../docs/labs/03-firewall-access-control/README.md) |
| 04 | [Servidor web Nginx](docs/laboratorios/04-servidor-web-nginx/README.md) | Diretório web personalizado, HTTP 8080, logs e Stub Status | [EN](../docs/labs/04-nginx-web-server/README.md) |
| 05 | [Dashboard personalizado Zabbix](docs/laboratorios/05-dashboard-personalizado-zabbix/README.md) | CPU, memória, uptime, carga, gráficos históricos e problemas | [EN](../docs/labs/05-zabbix-custom-dashboard/README.md) |
| 06 | [Página estática Apache](docs/laboratorios/06-pagina-estatica-apache/README.md) | HTML/CSS, verificação HTTP, monitoramento web Zabbix e widget URL | [EN](../docs/labs/06-apache-static-web-page/README.md) |
| 07 | [Reserva DHCP e persistência do IP](docs/laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md) | Cliente DHCP, conflito de endereçamento, recuperação e reserva no roteador | [EN](../docs/labs/07-dhcp-reservation-and-ip-address-persistence/README.md) |
| 08 | [Integração Grafana e Zabbix](docs/laboratorios/08-integracao-grafana-zabbix/README.md) | Plugin, token de API, JSON-RPC, logs e validação da fonte de dados | [EN](../docs/labs/08-grafana-zabbix-integration/README.md) |
| 09 | [Dashboard de monitoramento Grafana](docs/laboratorios/09-dashboard-de-monitoramento-grafana/README.md) | Métricas de sistema, rede e web; JSON do dashboard | [EN](../docs/labs/09-grafana-monitoring-dashboard/README.md) |
| 10 | [Remoção do Grafana e otimização de recursos](docs/laboratorios/10-remocao-do-grafana-e-otimizacao-de-recursos/README.md) | Remoção de serviço, limpeza de pacotes e gestão de recursos | [EN](../docs/labs/10-grafana-removal-and-resource-optimization/README.md) |
| 11 | [Migração do Zabbix para uma máquina virtual](docs/laboratorios/11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) | Backup/restauração do banco, recuperação do frontend, agente e IP estático | [EN](../docs/labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) |
| 12 | [Remoção do Zabbix e limpeza do servidor](docs/laboratorios/12-remocao-do-zabbix-e-limpeza-do-servidor/README.md) | Remoção da pilha do servidor físico; agente preservado e testado | [EN](../docs/labs/12-zabbix-removal-and-server-cleanup/README.md) |
| 13 | [Dashboard personalizado Grafana](docs/laboratorios/13-dashboard-personalizado-grafana/README.md) | Grafana na VM de monitoramento; dashboard incremental (em andamento) | [EN](../docs/labs/13-grafana-custom-monitoring-dashboard/README.md) |
| 14 | [Servidor DNS BIND9](docs/laboratorios/14-servidor-dns-bind9/README.md) | Zonas direta/reversa, registros DNS, consultas remotas e verificação Zabbix | [EN](../docs/labs/14-bind9-dns-server/README.md) |

## Documentação e evidências

[Ambiente e metodologia](docs/README.md) · [Arquitetura](docs/arquitetura.md) · [Evolução da infraestrutura](docs/evolucao-da-infraestrutura.md) · [Trilha de aprendizado](docs/trilha-de-aprendizado.md) · [Referência de comandos](docs/referencia-de-comandos.md)

[Galeria Zabbix](docs/laboratorios/05-dashboard-personalizado-zabbix/galeria.md) · [Galeria Apache](docs/laboratorios/06-pagina-estatica-apache/galeria.md)

Registros históricos de diferentes etapas, não um inventário atual. O Lab 13 permanece em andamento. [MIT](../LICENSE).
