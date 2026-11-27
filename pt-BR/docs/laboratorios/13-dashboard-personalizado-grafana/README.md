# Lab 13 - Dashboard personalizado de monitoramento Grafana

[English](../../../../docs/labs/13-grafana-custom-monitoring-dashboard/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../12-remocao-do-zabbix-e-limpeza-do-servidor/README.md) · [Próximo laboratório](../14-servidor-dns-bind9/README.md)

## Visão geral

Este laboratório tem foco na reconstrução e no desenvolvimento progressivo de um dashboard personalizado de monitoramento utilizando Grafana e Zabbix.

O ambiente de monitoramento havia sido migrado do servidor Debian físico para uma máquina virtual Debian dedicada. Posteriormente, o Grafana foi reinstalado na máquina virtual para restaurar o ambiente de monitoramento.

O processo completo de migração está documentado separadamente no **[Lab 11](../11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md) - Migração do servidor de monitoramento Zabbix para uma máquina virtual**, portanto este laboratório fornece apenas o contexto necessário antes de concentrar-se no próprio dashboard Grafana.

Este é um **laboratório em andamento**. O dashboard será desenvolvido progressivamente conforme novos requisitos de monitoramento, visualizações, métricas, limites e melhorias forem adicionados.

## Ambiente

| Componente | Configuração |
| --- | --- |
| Servidor Grafana | VM Debian Monitoring Server |
| Grafana | 13.x |
| Zabbix | 7.4.x |
| Fonte de dados | Zabbix |
| Host monitorado | Debian Server |
| IP do host monitorado | 192.168.1.117 |
| IP do servidor de monitoramento | 192.168.1.219 |
| Virtualização | VirtualBox |

## Arquitetura de monitoramento anterior

O servidor Debian físico original hospedava anteriormente a pilha completa de monitoramento, incluindo Zabbix Server, Zabbix Web, MariaDB, Apache2 e Grafana.

Conforme documentado no [Lab 11](../11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md), o ambiente de monitoramento foi migrado para uma máquina virtual Debian dedicada.

A arquitetura resultante é:

```text
Ubuntu Host
│
├── Debian Server
│   └── Zabbix Agent
│       └── 192.168.1.117
│
└── Debian Monitoring Server
    ├── Zabbix Server
    ├── Zabbix Web
    ├── MariaDB
    ├── Apache2
    └── Grafana
        └── 192.168.1.219
```

O servidor Debian físico é monitorado remotamente pelo Zabbix Server executado na máquina virtual.

## Reinstalação do Grafana

Após a transferência do ambiente de monitoramento para a máquina virtual, o Grafana foi reinstalado no Debian Monitoring Server.

A finalidade dessa etapa foi restaurar a camada de visualização do ambiente de monitoramento sem repetir o processo completo de migração documentado no [Lab 11](../11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md).

O Grafana é acessado por:

```text
http://192.168.1.219:3000
```

A fonte de dados Zabbix é utilizada para recuperar dados de monitoramento do Zabbix Server.

## Dashboard personalizado

O objetivo principal deste laboratório é criar um dashboard pessoal no Grafana desde o início.

Em vez de depender inteiramente de dashboards preexistentes, o dashboard está sendo projetado manualmente pela interface Grafana.

O dashboard incluirá progressivamente informações de monitoramento como:

* Utilização da CPU
* Load average
* Uso de memória
* Memória disponível
* Uso de disco
* Atividade de disco
* Tráfego de rede
* Erros de rede
* Informações do sistema
* Disponibilidade de serviços
* Disponibilidade do host
* Outras métricas relevantes de monitoramento Linux

Os painéis exatos e a organização podem mudar conforme o dashboard evolui.

## Abordagem de desenvolvimento do dashboard

O dashboard está sendo intencionalmente desenvolvido de maneira incremental.

O fluxo de trabalho é:

```text
Select metric
    ↓
Create panel
    ↓
Configure Zabbix query
    ↓
Configure visualization
    ↓
Configure units and thresholds
    ↓
Test the result
    ↓
Keep or improve the panel
```

Essa abordagem permite testar cada visualização individualmente, em vez de tentar construir todo o dashboard de uma só vez.

## Exportação do dashboard

Quando o dashboard atingir um estado estável, será exportado do Grafana como JSON.

O JSON exportado passará a fazer parte da documentação do laboratório e poderá ser utilizado para reproduzir o dashboard em outro ambiente Grafana.

Espera-se que a estrutura final do projeto inclua a exportação do dashboard, por exemplo:

```text
13-grafana-custom-monitoring-dashboard/
├── README.md
└── dashboard.json
```

O arquivo JSON somente será adicionado quando o dashboard atingir um ponto apropriado para exportação.

## Estado atual

Este laboratório está intencionalmente **aberto e em andamento**.

O objetivo não é concluir o dashboard em uma única sessão. Novos painéis, métricas, limites, disposições e melhorias podem ser adicionados progressivamente.

O laboratório somente será considerado concluído quando o dashboard atingir o estado final desejado e sua exportação JSON estiver documentada.

## Resultado

Este laboratório documentará o desenvolvimento progressivo de um dashboard personalizado de monitoramento Grafana utilizando Zabbix como fonte de dados.

Também demonstra o fluxo completo desde a restauração do ambiente de visualização até o projeto manual, os testes e a eventual exportação de um dashboard Grafana reutilizável.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
