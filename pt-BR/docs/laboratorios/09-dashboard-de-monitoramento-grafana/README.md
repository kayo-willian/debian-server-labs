# Lab 09 - Dashboard de monitoramento Grafana

[English](../../../../docs/labs/09-grafana-monitoring-dashboard/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../08-integracao-grafana-zabbix/README.md) · [Próximo laboratório](../10-remocao-do-grafana-e-otimizacao-de-recursos/README.md)

## Visão geral

Este laboratório teve foco na criação de um dashboard de monitoramento com Grafana, utilizando dados coletados pelo ambiente Zabbix existente.

O objetivo foi construir uma representação visual e centralizada dos dados de monitoramento do servidor Debian, combinando monitoramento de sistema, rede, serviço web e aplicação em um único dashboard.

O Grafana foi integrado ao Zabbix pela API Zabbix utilizando autenticação por token de API.

---

## Ambiente

O ambiente de monitoramento utilizado neste laboratório consistia em:

* Debian Server
* Zabbix Server
* Zabbix Agent
* Zabbix Web
* MariaDB
* Apache2
* Grafana
* Plugin Zabbix para Grafana

O Zabbix permaneceu responsável por coletar e armazenar os dados de monitoramento, enquanto o Grafana foi utilizado como camada de visualização.

A API Zabbix foi testada com sucesso e retornou a versão `7.4.15`.

---

## Dashboard

Um dashboard dedicado foi criado para fornecer uma visão geral do servidor Debian e seus serviços monitorados.

O dashboard foi organizado em várias seções de monitoramento.

### Visão geral

A seção de visão geral fornece uma visão rápida do estado do servidor, incluindo:

* Utilização da CPU
* Utilização de memória
* Espaço em disco
* Tempo de atividade do sistema
* Disponibilidade do Zabbix Agent
* Load average

### CPU

O monitoramento da CPU inclui:

* Utilização da CPU
* Tempo ocioso da CPU
* Uso atual da CPU
* Load average
* Load average de 1 minuto
* Load average de 5 minutos
* Load average de 15 minutos

### Memória

O monitoramento de memória inclui:

* Memória disponível
* Memória total
* Utilização de memória
* Informações de swap

### Disco

O monitoramento de disco inclui:

* Espaço livre em disco
* Espaço utilizado em disco
* Utilização de disco

### Rede

O monitoramento de rede fornece a visualização do tráfego de entrada e saída das interfaces coletado pelo Zabbix.

O dashboard representa o tráfego das interfaces de rede, em vez de presumir que o tráfego medido seja exclusivamente tráfego de Internet.

### Web e Apache

O dashboard também integra o monitoramento web e Apache já configurado no Zabbix.

Esta seção inclui:

* Monitoramento web
* Monitoramento de URL
* Tempo de resposta HTTP
* Disponibilidade do Apache
* Tempo de resposta do Apache

Esses painéis fornecem uma representação visual da disponibilidade e resposta dos serviços web monitorados pelo Zabbix.

### Sistema

Informações adicionais do sistema incluem:

* Número de processos
* Processos em execução
* Versão do Zabbix Agent
* Informações do sistema

### Problemas do Zabbix

Um painel dedicado exibe problemas ativos do Zabbix diretamente no dashboard Grafana.

Isso fornece uma visão centralizada dos eventos de monitoramento atuais junto às métricas do sistema.

---

## Monitoramento web

Uma das principais finalidades do dashboard foi visualizar a configuração de monitoramento web já implementada no Zabbix.

Os principais elementos destacados neste laboratório são:

* **Monitoramento Web**
* **Apache Response**
* **URL**

Esses elementos demonstram como o monitoramento no nível de serviço configurado no Zabbix pode ser representado por painéis Grafana.

Um Host Card também foi incluído no dashboard como elemento visual, mas não faz parte do escopo técnico de monitoramento deste laboratório.

---

## Integração Grafana e Zabbix

A integração foi realizada utilizando o plugin Zabbix para Grafana.

A fonte de dados Grafana comunica-se com a API Zabbix e recupera informações de monitoramento do ambiente Zabbix.

A conexão foi configurada utilizando:

* Endpoint da API Zabbix
* Autenticação por token de API
* Fonte de dados Zabbix
* Plugin Zabbix para Grafana

A conexão à API foi validada com sucesso antes da criação do dashboard.

---

## Estrutura do dashboard

O dashboard final foi dividido nas seguintes seções:

```text
Grafana Monitoring Dashboard
│
├── Overview
├── CPU
├── Memory
├── Disk
├── Network
├── Internet / Connectivity
├── Apache / Web Server
├── System
└── Current Zabbix Problems
```

O dashboard utiliza um intervalo curto de atualização e um período de tempo recente para fornecer uma visão atualizada do servidor monitorado.

---

### JSON do dashboard

O JSON do dashboard Grafana a seguir foi gerado com auxílio do **Claude Sonnet 5** e adaptado ao contexto deste laboratório.

Ele está incluído abaixo como parte da documentação técnica do projeto e preserva a configuração do dashboard utilizada durante o laboratório.

### JSON completo do dashboard

```text
{
  "id": null,
  "uid": "lab09-debian-server-monitoring",
  "title": "Lab 09 - Debian Server Monitoring",
  "description": "Dashboard de monitoramento do Debian Server (Lab 09) via Zabbix 7.4, usando o plugin Alexander Zobnin Zabbix datasource. CPU, memoria, disco, rede, conectividade web, Apache e problemas ativos.",
  "tags": [
    "zabbix",
    "grafana",
    "debian",
    "linux",
    "lab-09"
  ],
  "timezone": "browser",
  "editable": true,
  "graphTooltip": 1,
  "schemaVersion": 39,
  "version": 1,
  "refresh": "1m",
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "templating": {
    "list": [
      {
        "name": "hostgroup",
        "type": "query",
        "label": "Host Group",
        "datasource": {
          "type": "alexanderzobnin-zabbix-datasource",
          "uid": "alexanderzobnin-zabbix-datasource-1"
        },
        "query": "groups()",
        "current": {
          "selected": true,
          "text": "Linux Servers",
          "value": "Linux Servers"
        },
        "refresh": 1,
        "regex": "",
        "sort": 1,
        "multi": false,
        "includeAll": false,
        "hide": 0
      },
      {
        "name": "host",
        "type": "query",
        "label": "Host",
        "datasource": {
          "type": "alexanderzobnin-zabbix-datasource",
          "uid": "alexanderzobnin-zabbix-datasource-1"
        },
        "query": "hosts($hostgroup)",
        "current": {
          "selected": true,
          "text": "Debian Servers",
          "value": "Debian Servers"
        },
        "refresh": 1,
        "regex": "",
        "sort": 1,
        "multi": false,
        "includeAll": false,
        "hide": 0
      }
    ]
  },
  "annotations": {
    "list": [
      {
        "name": "Annotations & Alerts",
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "type": "dashboard",
        "builtIn": 1
      }
    ]
  },
  "panels": [
    {
      "id": 1,
      "type": "row",
      "title": "Overview",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "panels": []
    },
    {
      "id": 2,
      "type": "gauge",
      "title": "CPU Usage",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 0,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 3,
      "type": "gauge",
      "title": "Memory Usage",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 4,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Memory utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 4,
      "type": "gauge",
      "title": "Disk Free (/)",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 8,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on / (percentage)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 10
              },
              {
                "color": "yellow",
                "value": 25
              },
              {
                "color": "green",
                "value": 40
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 5,
      "type": "stat",
      "title": "Uptime",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 12,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "System uptime"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 6,
      "type": "stat",
      "title": "Zabbix Agent",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 16,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Agent ping"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "background",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          },
          "mappings": [
            {
              "type": "value",
              "options": {
                "0": {
                  "text": "DOWN",
                  "color": "red",
                  "index": 1
                },
                "1": {
                  "text": "UP",
                  "color": "green",
                  "index": 0
                }
              }
            }
          ]
        },
        "overrides": []
      }
    },
    {
      "id": 7,
      "type": "stat",
      "title": "Load Average (1m)",
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 20,
        "y": 1
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (1m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 2,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 1.5
              },
              {
                "color": "red",
                "value": 3
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 8,
      "type": "row",
      "title": "CPU",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 7
      },
      "panels": []
    },
    {
      "id": 9,
      "type": "timeseries",
      "title": "CPU Utilization (%)",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 25,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 10,
      "type": "stat",
      "title": "CPU - Current",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 16,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 11,
      "type": "stat",
      "title": "CPU - Peak (6h)",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 20,
        "y": 8
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          },
          "custom": {}
        },
        "overrides": []
      }
    },
    {
      "id": 12,
      "type": "stat",
      "title": "CPU - Average (6h)",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 16,
        "y": 12
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 13,
      "type": "stat",
      "title": "CPU - Idle",
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 20,
        "y": 12
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "CPU idle time"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 14,
      "type": "timeseries",
      "title": "Load Average (1m / 5m / 15m)",
      "gridPos": {
        "h": 7,
        "w": 24,
        "x": 0,
        "y": 16
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (1m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (5m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "C",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Load average (15m avg)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 15,
      "type": "row",
      "title": "Memory",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 23
      },
      "panels": []
    },
    {
      "id": 16,
      "type": "timeseries",
      "title": "Memory Usage History",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Available memory"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Total memory"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 20,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 17,
      "type": "gauge",
      "title": "Memory Utilization %",
      "gridPos": {
        "h": 8,
        "w": 4,
        "x": 16,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Memory utilization"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 60
              },
              {
                "color": "orange",
                "value": 75
              },
              {
                "color": "red",
                "value": 90
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 18,
      "type": "gauge",
      "title": "Swap Usage %",
      "gridPos": {
        "h": 8,
        "w": 4,
        "x": 20,
        "y": 24
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free swap space in %"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 20
              },
              {
                "color": "yellow",
                "value": 50
              },
              {
                "color": "green",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 19,
      "type": "row",
      "title": "Disk",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 32
      },
      "panels": []
    },
    {
      "id": 20,
      "type": "timeseries",
      "title": "Disk Space Usage (/)",
      "gridPos": {
        "h": 8,
        "w": 16,
        "x": 0,
        "y": 33
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on /"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Used disk space on /"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 25,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 21,
      "type": "gauge",
      "title": "Free Disk Space (/) %",
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 16,
        "y": 33
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Free disk space on / (percentage)"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "fieldConfig": {
        "defaults": {
          "unit": "percent",
          "min": 0,
          "max": 100,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "orange",
                "value": 10
              },
              {
                "color": "yellow",
                "value": 25
              },
              {
                "color": "green",
                "value": 40
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 22,
      "type": "row",
      "title": "Network",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 41
      },
      "panels": []
    },
    {
      "id": 23,
      "type": "timeseries",
      "title": "Network Traffic (In / Out - eth0)",
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 42
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits received"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits sent"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 20,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 24,
      "type": "stat",
      "title": "Current Inbound Traffic",
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 0,
        "y": 50
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits received"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 25,
      "type": "stat",
      "title": "Current Outbound Traffic",
      "gridPos": {
        "h": 4,
        "w": 12,
        "x": 12,
        "y": 50
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Interface eth0: Bits sent"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "bps",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "purple",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 26,
      "type": "row",
      "title": "Internet / Connectivity",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 54
      },
      "panels": []
    },
    {
      "id": 27,
      "type": "stat",
      "title": "Web Scenario Availability",
      "gridPos": {
        "h": 6,
        "w": 6,
        "x": 0,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Web monitoring"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Download speed for scenario"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 1,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 28,
      "type": "timeseries",
      "title": "Ping / Latency",
      "gridPos": {
        "h": 6,
        "w": 9,
        "x": 6,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "ICMP ping"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "ms",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 29,
      "type": "timeseries",
      "title": "HTTP Response Time",
      "gridPos": {
        "h": 6,
        "w": 9,
        "x": 15,
        "y": 55
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Web monitoring"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Response time for step"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 30,
      "type": "row",
      "title": "Apache / Web Server",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 61
      },
      "panels": []
    },
    {
      "id": 31,
      "type": "stat",
      "title": "Apache Availability",
      "gridPos": {
        "h": 6,
        "w": 6,
        "x": 0,
        "y": 62
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Apache"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Apache: Service status"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "background",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "green",
                "value": 1
              }
            ]
          },
          "mappings": [
            {
              "type": "value",
              "options": {
                "0": {
                  "text": "DOWN",
                  "color": "red",
                  "index": 1
                },
                "1": {
                  "text": "UP",
                  "color": "green",
                  "index": 0
                }
              }
            }
          ]
        },
        "overrides": []
      }
    },
    {
      "id": 32,
      "type": "timeseries",
      "title": "Apache Response Time",
      "gridPos": {
        "h": 6,
        "w": 18,
        "x": 6,
        "y": 62
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": "Apache"
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Apache: Response time"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "s",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 33,
      "type": "row",
      "title": "System",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 68
      },
      "panels": []
    },
    {
      "id": 34,
      "type": "timeseries",
      "title": "Number of Processes",
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        },
        {
          "refId": "B",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes running"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "custom": {
            "drawStyle": "line",
            "lineWidth": 1,
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "spanNulls": true,
            "pointSize": 3,
            "stacking": {
              "mode": "none",
              "group": "A"
            },
            "showPoints": "never"
          },
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "options": {
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        },
        "legend": {
          "displayMode": "table",
          "placement": "bottom",
          "calcs": [
            "mean",
            "max",
            "last"
          ]
        }
      }
    },
    {
      "id": 35,
      "type": "stat",
      "title": "Processes - Current",
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 12,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Number of processes"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "auto",
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "short",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 250
              },
              {
                "color": "red",
                "value": 400
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 36,
      "type": "stat",
      "title": "Zabbix Agent Version",
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 18,
        "y": 69
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 0,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "itemTag": {
            "filter": ""
          },
          "item": {
            "filter": "Version of zabbix_agent(d) running"
          },
          "functions": [],
          "options": {
            "showDisabledItems": false,
            "skipEmptyValues": false,
            "disableDataAlignment": false
          },
          "resultFormat": "time_series"
        }
      ],
      "options": {
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "orientation": "auto",
        "textMode": "value",
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto"
      },
      "fieldConfig": {
        "defaults": {
          "unit": "none",
          "decimals": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      }
    },
    {
      "id": 37,
      "type": "row",
      "title": "Current Zabbix Problems",
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 76
      },
      "panels": []
    },
    {
      "id": 38,
      "type": "table",
      "title": "Current Problems",
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 77
      },
      "datasource": {
        "type": "alexanderzobnin-zabbix-datasource",
        "uid": "alexanderzobnin-zabbix-datasource-1"
      },
      "targets": [
        {
          "refId": "A",
          "datasource": {
            "type": "alexanderzobnin-zabbix-datasource",
            "uid": "alexanderzobnin-zabbix-datasource-1"
          },
          "queryType": 4,
          "group": {
            "filter": "$hostgroup"
          },
          "host": {
            "filter": "$host"
          },
          "application": {
            "filter": ""
          },
          "proxy": {
            "filter": ""
          },
          "tags": {
            "filter": ""
          },
          "options": {
            "minSeverity": 2,
            "sortProblems": "default",
            "showAckButton": true,
            "limit": 100
          },
          "resultFormat": "table"
        }
      ],
      "options": {
        "showHeader": true,
        "cellHeight": "sm",
        "footer": {
          "show": false
        }
      },
      "fieldConfig": {
        "defaults": {
          "custom": {
            "align": "auto",
            "cellOptions": {
              "type": "auto"
            }
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Severity"
            },
            "properties": [
              {
                "id": "custom.cellOptions",
                "value": {
                  "type": "color-background"
                }
              }
            ]
          }
        ]
      }
    }
  ],
  "style": "dark",
  "fiscalYearStartMonth": 0,
  "liveNow": false,
  "weekStart": ""
}
```

---

## Resultado

O laboratório resultou em um dashboard Grafana centralizado conectado ao Zabbix.

O dashboard reúne dados de monitoramento de infraestrutura e serviços em uma única interface, permitindo que informações coletadas pelo Zabbix sejam visualizadas por diferentes painéis Grafana.

O projeto proporcionou experiência prática com:

* Grafana
* Integração Zabbix
* API Zabbix
* Autenticação por token de API
* Fontes de dados
* Criação de dashboards
* Painéis de monitoramento
* Métricas de sistema
* Métricas de rede
* Monitoramento web
* Monitoramento Apache
* Problemas do Zabbix

---

## Habilidades demonstradas

* Monitoramento de servidores Linux
* Zabbix
* Grafana
* API Zabbix
* Autenticação por token de API
* Dashboards de monitoramento
* Monitoramento de CPU
* Monitoramento de memória
* Monitoramento de disco
* Monitoramento de rede
* Monitoramento Apache
* Monitoramento web
* Visualização de infraestrutura
* Documentação técnica

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
