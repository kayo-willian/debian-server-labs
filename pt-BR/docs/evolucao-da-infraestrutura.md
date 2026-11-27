# Evolução da infraestrutura

[Índice](README.md) · [English](../../docs/infrastructure-timeline.md)

Esta linha do tempo segue a ordem dos laboratórios e preserva as decisões de implantação, migração e remoção. Os endereços são históricos.

| Lab | Rede | Mudança documentada |
| --- | --- | --- |
| [01](laboratorios/01-instalacao-configuracao-ssh/README.md) | `192.168.2.0/24` | Debian instalado no hardware físico; SSH e sub-rede Wi-Fi dedicada |
| [02](laboratorios/02-servidor-de-monitoramento-zabbix/README.md) | `192.168.2.0/24` | Zabbix 7.4, MariaDB e Apache implantados; segundo host monitorado |
| [03](laboratorios/03-firewall-e-controle-de-acesso/README.md) | `192.168.2.0/24` | Política de entrada UFW com SSH/HTTP restritos e TCP 10051 aberto |
| [04](laboratorios/04-servidor-web-nginx/README.md) | `192.168.2.0/24` | Nginx TCP 8080 e Stub Status local |
| [05](laboratorios/05-dashboard-personalizado-zabbix/README.md) | `192.168.1.0/24` | LAN principal utilizada; dashboard personalizado Zabbix |
| [06](laboratorios/06-pagina-estatica-apache/README.md) | `192.168.1.0/24` | Página estática Apache e cenário web |
| [07](laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md) | `192.168.1.0/24` | DHCP alterou .117 para .116; reserva .117 configurada |
| [08](laboratorios/08-integracao-grafana-zabbix/README.md) | `192.168.1.0/24` | Grafana conectado à API Zabbix no servidor físico |
| [09](laboratorios/09-dashboard-de-monitoramento-grafana/README.md) | `192.168.1.0/24` | Dashboard Grafana e JSON registrado |
| [10](laboratorios/10-remocao-do-grafana-e-otimizacao-de-recursos/README.md) | `192.168.1.0/24` | Grafana removido do servidor físico para preservar recursos |
| [11](laboratorios/11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) | `192.168.1.0/24` | Zabbix migrado para VM .219; banco/configuração restaurados; consulta passiva ao agente .117 testada |
| [12](laboratorios/12-remocao-do-zabbix-e-limpeza-do-servidor/README.md) | `192.168.1.0/24` | Pilha Zabbix física, MariaDB, Apache e PHP removidos; agente preservado |
| [13](laboratorios/13-dashboard-personalizado-grafana/README.md) | `192.168.1.0/24` | Grafana na VM de monitoramento; dashboard personalizado em andamento, exportação ainda ausente |
| [14](laboratorios/14-servidor-dns-bind9/README.md) | `192.168.1.0/24` | BIND9 físico; consultas direta/reversa e verificação DNS Zabbix |

## Limites do registro

A tabela do Lab 12 já registra Grafana em execução na VM; o Lab 13 explica sua reinstalação e o desenvolvimento posterior do dashboard. Não há uma data exata de reinstalação registrada aqui. O Lab 13 continua aberto e o JSON do Lab 09 corresponde ao dashboard anterior, não ao novo.

Não se deve interpretar uma omissão de serviço em uma etapa posterior como prova de sua remoção. Por exemplo, o Lab 12 documenta a remoção do Apache, mas não uma remoção do Nginx.

[Arquitetura](arquitetura.md)
