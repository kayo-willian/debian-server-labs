# Lab 12 - Remoção do Zabbix e limpeza do servidor

[English](../../../../docs/labs/12-zabbix-removal-and-server-cleanup/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) · [Próximo laboratório](../13-dashboard-personalizado-grafana/README.md)

## Visão geral

Após migrar o ambiente de monitoramento Zabbix para uma máquina virtual Debian dedicada, o servidor Debian físico não precisava mais executar a pilha Zabbix Server.

O servidor físico foi, portanto, limpo, removendo a infraestrutura de monitoramento e preservando apenas o Zabbix Agent necessário para o monitoramento remoto pela nova VM.

O objetivo era transformar o servidor Debian físico em uma base limpa de infraestrutura para futuros serviços.

---

## Arquitetura anterior

Antes da limpeza, o servidor Debian físico executava a pilha completa de monitoramento:

```text
Physical Debian Server
192.168.1.117
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
├── Apache2
├── PHP
├── Grafana remnants
└── Zabbix Agent
```

Após a migração, essa arquitetura era redundante porque a pilha de monitoramento já havia sido transferida para a VM dedicada.

---

## Arquitetura pretendida

A arquitetura final separa o monitoramento dos serviços de infraestrutura:

```text
Debian Monitoring VM
192.168.1.219
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
└── Grafana
        │
        │ Zabbix Agent :10050
        ▼
Physical Debian Server
192.168.1.117
│
└── Zabbix Agent
```

O servidor físico passou a ser dedicado à hospedagem de futuros serviços de infraestrutura, enquanto a VM de monitoramento é responsável por monitorá-los.

---

## O que foi removido

Os seguintes componentes foram removidos do servidor Debian físico:

* Zabbix Server
* Zabbix Frontend
* Scripts SQL do Zabbix
* Pacote de repositório Zabbix
* MariaDB
* Apache2
* PHP
* Componentes e resquícios do Grafana

O seguinte componente foi intencionalmente preservado:

* Zabbix Agent

O agente é necessário porque a VM de monitoramento ainda precisa coletar métricas do servidor físico.

---

## Remoção da pilha Zabbix Server

O serviço Zabbix Server foi interrompido e os pacotes do lado servidor foram removidos:

```bash
sudo systemctl stop zabbix-server

sudo apt purge zabbix-server-mysql zabbix-frontend-php \
zabbix-apache-conf zabbix-sql-scripts zabbix-release -y
```

Os diretórios de configuração e logs do Zabbix Server foram então removidos:

```bash
sudo rm -rf /etc/zabbix/zabbix_server.conf
sudo rm -rf /var/log/zabbix
sudo rm -rf /var/lib/zabbix
```

O próprio diretório `/etc/zabbix` foi preservado porque ainda contém a configuração necessária ao Zabbix Agent.

---

## Remoção do banco de dados Zabbix

O antigo banco Zabbix já não era necessário no servidor físico.

O banco de dados e seu usuário MariaDB dedicado foram removidos:

```bash
sudo mariadb
```

```sql
DROP DATABASE zabbix;
DROP USER 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## Remoção do MariaDB

Após remover o banco Zabbix, o MariaDB já não era necessário no servidor físico:

```bash
sudo systemctl stop mariadb

sudo apt purge mariadb-server mariadb-client -y

sudo apt autoremove --purge -y
```

Os diretórios restantes do MariaDB foram removidos:

```bash
sudo rm -rf /var/lib/mysql
sudo rm -rf /etc/mysql
```

---

## Remoção do Apache2

Originalmente, o Apache2 era utilizado para servir o frontend Zabbix Web.

Como o frontend agora era executado na VM de monitoramento, o Apache2 foi removido:

```bash
sudo systemctl stop apache2

sudo apt purge apache2 apache2-bin apache2-data apache2-utils -y

sudo apt autoremove --purge -y
```

Os diretórios restantes de configuração e raiz web do Apache foram removidos:

```bash
sudo rm -rf /etc/apache2
sudo rm -rf /var/www
```

---

## Remoção do PHP

O PHP era anteriormente necessário para o frontend Zabbix Web.

Como o frontend foi removido do servidor físico, os pacotes PHP também foram removidos:

```bash
sudo apt purge 'php*' -y

sudo apt autoremove --purge -y
```

---

## Preservação do Zabbix Agent

O Zabbix Agent foi intencionalmente mantido instalado e em execução.

Isso permite à VM de monitoramento coletar remotamente métricas do servidor físico.

O pacote Zabbix restante foi verificado com:

```bash
dpkg -l | grep -E 'zabbix|apache2|php|mariadb|grafana'
```

O resultado mostrou apenas:

```text
zabbix-agent
```

---

## Verificação de serviços

Os serviços em execução foram verificados para confirmar que não restavam serviços de monitoramento além do agente:

```bash
sudo systemctl --type=service --state=running | \
grep -E 'zabbix|apache2|php|mariadb|grafana'
```

O único resultado foi:

```text
zabbix-agent.service loaded active running Zabbix Agent
```

Isso confirmou que, entre os serviços pesquisados, o Zabbix Agent era o único serviço de monitoramento restante.

---

## Verificação de portas

As portas em escuta foram verificadas para confirmar que os antigos serviços de monitoramento já não disponibilizavam suas portas de rede:

```bash
sudo ss -tulpn | grep -E ':80 |:3306 |:10050 |:10051 |:3000 '
```

A única porta relevante restante era:

```text
0.0.0.0:10050
[::]:10050
```

A porta `10050` pertence ao Zabbix Agent.

As seguintes portas já não estavam em escuta:

```text
80      Apache2
3306    MariaDB
10051   Zabbix Server
3000    Grafana
```

---

## Limpeza de pacotes

O sistema foi verificado quanto a dependências não utilizadas:

```bash
sudo apt autoremove --purge -y
```

O sistema informou:

```text
Atualizando: 0, Instalando: 0, Removendo: 0, Não atualizando: 0
```

Isso confirmou que nenhum pacote adicional foi identificado para remoção automática.

---

## Teste final de monitoramento

O teste final foi realizado a partir da VM dedicada de monitoramento.

A partir do Debian Monitoring Server:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

O resultado foi:

```text
1
```

Isso confirmou que a VM de monitoramento ainda conseguia comunicar-se com o Zabbix Agent executado no servidor Debian físico.

---

## Estado final

A pilha de monitoramento Zabbix foi removida com sucesso do servidor Debian físico.

```text
Physical Debian Server
192.168.1.117
│
└── Zabbix Agent :10050
        ▲
        │
        │ Remote monitoring
        │
        ▼
Debian Monitoring Server
192.168.1.219
│
├── Zabbix Server
├── Zabbix Web
├── MariaDB
└── Grafana
```

O servidor físico ficou disponível para hospedar futuros serviços de infraestrutura sem executar sua própria pilha de monitoramento.

Novos serviços instalados no servidor físico podem ser monitorados centralmente pela VM dedicada.

---

## Resumo da verificação

| Componente | Servidor Debian físico | VM de monitoramento |
| --- | --- | --- |
| Zabbix Server | Removido | Em execução |
| Zabbix Web | Removido | Em execução |
| Zabbix Agent | Em execução | Em execução |
| MariaDB | Removido | Em execução |
| Apache2 | Removido | Em execução |
| PHP | Removido | Em execução |
| Grafana | Removido | Em execução |
| Responsabilidade pelo monitoramento | Nenhuma | Centralizada |

---

## Resultado

A arquitetura de monitoramento foi separada com sucesso do servidor físico de infraestrutura.

Nesta etapa, o servidor Debian físico fornecia uma base limpa para futuros serviços de infraestrutura, enquanto o Debian Monitoring Server dedicado centralizava Zabbix e Grafana.

Essa separação reduz o consumo desnecessário de recursos no servidor físico e estabelece uma arquitetura mais clara para futuros laboratórios.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
