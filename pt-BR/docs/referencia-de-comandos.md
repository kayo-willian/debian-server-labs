# Referência de comandos

[Índice](README.md) · [English](../../docs/command-reference.md)

Índice seletivo de comandos e ferramentas abordados nos registros. Consulte o laboratório correspondente para a sintaxe completa, os parâmetros, os resultados e a máquina em que cada operação foi realizada. Não é um script para execução.

| Lab | Comandos e ferramentas | Finalidade |
| --- | --- | --- |
| [01](laboratorios/01-instalacao-configuracao-ssh/README.md) | `ip addr`, `ip route`, `ip route get`, `ping`, `ssh`, `passwd` | Interface, rota, conectividade e acesso remoto |
| [02](laboratorios/02-servidor-de-monitoramento-zabbix/README.md) | `free -h`, `apt update`, `apt policy`, `dpkg -L`, `systemctl`, `journalctl`, `zabbix_get` | Recursos, localização de pacotes, serviços e consulta ao agente |
| [03](laboratorios/03-firewall-e-controle-de-acesso/README.md) | `ufw enable`, `ufw status numbered` | Ativação do firewall e inspeção de regras |
| [04](laboratorios/04-servidor-web-nginx/README.md) | `nginx -t`, `nginx -T`, `curl`, `ss` | Validação da configuração e diagnóstico HTTP |
| [06](laboratorios/06-pagina-estatica-apache/README.md) | `curl`, `systemctl` | Disponibilidade HTTP Apache e estado do serviço |
| [07](laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md) | `ip addr`, `ip route`, `dhcpcd` | Endereçamento DHCP e inspeção de rotas |
| [08](laboratorios/08-integracao-grafana-zabbix/README.md) | `curl`, `journalctl` | Teste JSON-RPC e logs do Grafana |
| [10](laboratorios/10-remocao-do-grafana-e-otimizacao-de-recursos/README.md) | `systemctl`, `apt remove`, `apt purge`, `apt autoremove` | Remoção de serviços e pacotes |
| [11](laboratorios/11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) | `mariadb-dump`, `gzip`, `mariadb`, `zabbix_get` | Migração de banco, verificação de arquivo compactado e consulta ao agente |
| [12](laboratorios/12-remocao-do-zabbix-e-limpeza-do-servidor/README.md) | `systemctl`, `ss`, `zabbix_get` | Verificação após limpeza e teste remoto do agente |
| [14](laboratorios/14-servidor-dns-bind9/README.md) | `named-checkconf`, `named-checkzone`, `dig`, `nc`, `zabbix_get` | Configuração DNS, resolução e monitoramento funcional |

Os Labs 10–12 contêm operações destrutivas de remoção de pacotes, arquivos e bancos de dados. Não as repita sem verificar o host, as dependências e os backups. Os exemplos históricos com credenciais mascaradas exigem valores próprios; nunca publique senhas ou tokens.

[Trilha de aprendizado](trilha-de-aprendizado.md)
