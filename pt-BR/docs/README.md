# Documentação dos laboratórios Debian Server

> Nota de privacidade: identificadores pessoais de rede neste registro histórico foram mascarados; a topologia, os comandos e as observações registrados foram preservados.

[Visão geral do repositório](../README.md) · [English](../../docs/README.md)

A numeração dos laboratórios segue a ordem dos experimentos originais.

[Arquitetura](arquitetura.md) · [Evolução da infraestrutura](evolucao-da-infraestrutura.md) · [Trilha de aprendizado](trilha-de-aprendizado.md) · [Referência de comandos](referencia-de-comandos.md)

- [Lab 01 — Instalação, configuração e SSH](laboratorios/01-instalacao-configuracao-ssh/README.md)
- [Lab 02 — Servidor de monitoramento Zabbix](laboratorios/02-servidor-de-monitoramento-zabbix/README.md)
- [Lab 03 — Firewall e controle de acesso](laboratorios/03-firewall-e-controle-de-acesso/README.md)
- [Lab 04 — Servidor web Nginx](laboratorios/04-servidor-web-nginx/README.md)
- [Lab 05 — Dashboard personalizado Zabbix](laboratorios/05-dashboard-personalizado-zabbix/README.md)
- [Lab 06 — Página estática Apache](laboratorios/06-pagina-estatica-apache/README.md)
- [Lab 07 — Reserva DHCP e persistência do IP](laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md)
- [Lab 08 — Integração Grafana e Zabbix](laboratorios/08-integracao-grafana-zabbix/README.md)
- [Lab 09 — Dashboard de monitoramento Grafana](laboratorios/09-dashboard-de-monitoramento-grafana/README.md)
- [Lab 10 — Remoção do Grafana e otimização de recursos](laboratorios/10-remocao-do-grafana-e-otimizacao-de-recursos/README.md)
- [Lab 11 — Migração do Zabbix para uma máquina virtual](laboratorios/11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md)
- [Lab 12 — Remoção do Zabbix e limpeza do servidor](laboratorios/12-remocao-do-zabbix-e-limpeza-do-servidor/README.md)
- [Lab 13 — Dashboard personalizado Grafana](laboratorios/13-dashboard-personalizado-grafana/README.md)
- [Lab 14 — Servidor DNS BIND9](laboratorios/14-servidor-dns-bind9/README.md)

## Sobre estes registros

Laboratórios práticos de Debian Server com foco em administração Linux, redes, serviços, segurança, monitoramento e infraestrutura.

Este repositório documenta experimentos práticos e atividades de aprendizado realizados utilizando um servidor Debian físico e, nas etapas posteriores de monitoramento, uma VM Debian dedicada.

O propósito deste repositório é duplo: demonstrar experiência prática e construir uma referência técnica pessoal que possa ser consultada em futuras aulas, laboratórios, diagnósticos e tarefas de administração de sistemas.

O foco não é simplesmente executar comandos, mas compreender por que uma configuração é necessária, como funciona, como verificar o resultado e como diagnosticar problemas.

## Abordagem de aprendizado

Cada laboratório segue um ciclo de aprendizado prático:

```text
Conceito
   ↓
Por que existe?
   ↓
Comando / Configuração
   ↓
Observar o resultado
   ↓
Implementação
   ↓
Testes
   ↓
Diagnóstico
   ↓
Documentação
```

A documentação enfatiza a compreensão em vez da memorização.

Os comandos são documentados junto com sua finalidade, comportamento esperado, resultados observados e contexto prático.

## Evolução da rede

A arquitetura de rede mudou durante o desenvolvimento do projeto.

### Labs 01–04

Os quatro primeiros laboratórios utilizaram uma sub-rede sem fio dedicada ao laboratório:

```text
192.168.2.0/24
```

Isso forneceu separação de rede entre o ambiente de laboratório e a rede doméstica principal.

O servidor e a estação de administração comunicavam-se pela rede dedicada durante esses experimentos iniciais.

### Labs 05 em diante

A partir do Lab 05, o servidor foi transferido para a rede local principal:

```text
192.168.1.0/24
```

A mudança foi realizada para melhorar a comunicação e a visibilidade entre o servidor Debian, a estação de administração, o roteador e outros dispositivos utilizados no laboratório.

O ambiente nessa etapa refletia, portanto, uma topologia de infraestrutura mais prática na qual o servidor participa diretamente da LAN principal.

A mudança deve ser considerada ao ler os laboratórios antigos. Seus endereços de rede descrevem o ambiente existente quando foram realizados e não necessariamente a configuração atual.

## Ambiente

### Hardware do servidor

| Componente | Especificação |
| --- | --- |
| Dispositivo | Positivo Motion |
| CPU | Intel Atom x5-Z8350 |
| RAM | 2 GB DDR3 |
| Armazenamento | Aproximadamente 32 GB |
| Rede | Wi-Fi |
| Sistema operacional | Debian Server |
| Ambiente de desktop | Nenhum |

O servidor utiliza um computador antigo com recursos limitados como ambiente físico de laboratório.

O hardware limitado é intencional e fornece um ambiente prático para aprender a operar serviços e sistemas com recursos restritos.

## LAN principal após o Lab 07

A tabela descreve a rede local principal após a reserva DHCP registrada no Lab 07.

| Rede | Sub-rede | Finalidade |
| --- | --- | --- |
| [redacted SSID] | `192.168.1.0/24` | Rede principal e ambiente de laboratório Debian Server |
| Rede Debian de laboratório anterior | `192.168.2.0/24` | Rede isolada utilizada nos Labs 01–04 |

O servidor Debian operava na rede `192.168.1.0/24`.

Naquela etapa, a estação de administração podia comunicar-se com o servidor diretamente pela LAN principal.

A configuração registrada do servidor Debian incluía:

| Interface | Rede | Endereço |
| --- | --- | --- |
| `wlan0` | `192.168.1.0/24` | `192.168.1.117` |

O roteador atuava como gateway de rede e servidor DHCP.

O servidor utilizava DHCP, com uma reserva configurada no roteador para manter o endereço esperado `192.168.1.117`.

Essa abordagem permitiu que o servidor Debian continuasse utilizando DHCP enquanto mantinha um endereço previsível para serviços como SSH, Apache e Zabbix.

## Reserva DHCP

No Lab 07, o endereço do servidor foi mantido por meio de uma reserva DHCP no roteador.

A reserva associa a interface de rede do servidor ao endereço atribuído:

| Parâmetro | Valor |
| --- | --- |
| Interface | `wlan0` |
| Endereço MAC | `xx:xx:xx:xx:xx:xx` |
| Endereço reservado | `192.168.1.117` |
| Gateway | `192.168.1.1` |
| Atribuição de endereço | DHCP |

Essa configuração foi documentada no [Lab 07](laboratorios/07-reserva-dhcp-e-persistencia-de-ip/README.md).

A reserva foi introduzida após o servidor receber `192.168.1.116` em vez do endereço anteriormente utilizado `192.168.1.117`.

A investigação demonstrou a diferença entre um endereço DHCP, um endereço estático configurado manualmente e uma reserva DHCP.

## Filosofia da documentação

Cada laboratório deve documentar as partes relevantes do processo de aprendizado, incluindo:

* Objetivo
* Cenário ou problema
* Ambiente
* Conceitos aprendidos
* Implementação
* Comandos importantes
* Resultados esperados
* Resultados reais
* Testes e verificação
* Diagnóstico
* Lições aprendidas
* Principais aprendizados

O objetivo não é criar uma transcrição de cada comando executado.

Em vez disso, a documentação deve explicar o que foi feito, por que foi feito, o que aconteceu e o que foi aprendido durante o processo.

## Linux e redes

Uma parte importante deste projeto é conectar a administração Linux aos conceitos de redes.

Os mesmos conceitos fundamentais de redes estudados com Cisco IOS também podem ser observados e gerenciados no Linux, embora os comandos e métodos de implementação sejam diferentes.

Por exemplo:

| Conceito | Cisco IOS | Linux |
| --- | --- | --- |
| Visualizar interfaces | `show ip interface brief` | `ip addr` |
| Visualizar tabela de roteamento | `show ip route` | `ip route` |
| Testar conectividade | `ping` | `ping` |
| Visualizar serviços em escuta | `show control-plane host open-ports` | `ss -tulpn` |

Essas comparações destinam-se a reforçar conceitos em diferentes plataformas, e não a afirmar que os comandos são equivalentes.

## Documentação do Debian Server

A principal referência técnica deste projeto é a documentação oficial do Debian:

https://www.debian.org/doc/

A documentação fornece informações técnicas sobre sistemas Debian, instalação, administração, configuração e software disponível.

Aprender a localizar e utilizar a documentação oficial é, por si só, parte da habilidade de administração desenvolvida neste projeto.

## Servidor físico

Diferentemente do repositório Ubuntu Server Labs, que utiliza uma máquina virtual, este projeto começou com um computador físico como servidor e posteriormente transferiu o monitoramento para uma VM dedicada.

Isso oferece uma oportunidade de trabalhar com aspectos do hardware real e da administração de servidores físicos, incluindo:

* Conectividade de rede sem fio
* Recursos limitados de hardware
* Armazenamento físico
* Inicialização e instalação do sistema
* Administração headless
* Administração remota por SSH
* Gerenciamento de serviços em um sistema físico
* Diagnóstico de rede

O servidor foi concebido para operar sem exigir monitor, teclado ou interface gráfica dedicados durante a administração normal.

## Administração por SSH

O SSH é o principal método utilizado para administrar remotamente o servidor Debian.

O modelo de administração registrado na LAN principal era:

```text
Estação Ubuntu
       |
       | SSH
       |
       v
Servidor Debian
192.168.1.117
```

O servidor e a estação de administração comunicavam-se pela rede local principal.

Laboratórios antigos podem mostrar a rede anterior de laboratório `192.168.2.0/24`. Esses endereços pertencem à arquitetura de rede anterior e são mantidos na documentação histórica para preservar a precisão.

Comandos executados após estabelecer a sessão SSH operam no servidor Debian, e não na estação Ubuntu.

Isso permite que o servidor físico opere independentemente enquanto é gerenciado remotamente.

## Serviços e infraestrutura

À medida que os laboratórios progridem, o ambiente de servidor é expandido para além da administração básica do sistema operacional.

Nas etapas registradas, o projeto inclui trabalho prático com:

* SSH
* UFW
* Zabbix
* Zabbix Agent
* Apache2
* Monitoramento web
* DHCP
* Diagnóstico de rede
* Conteúdo web estático
* Monitoramento de sistemas
* Monitoramento de recursos
* Redes Linux

O ambiente de laboratório é intencionalmente desenvolvido de maneira incremental. Cada novo serviço oferece uma oportunidade de conectar administração Linux com conceitos de redes, monitoramento, segurança e infraestrutura.

## Propósito

Este repositório pretende tornar-se uma base prática de conhecimento em administração Debian Server, construída por meio de configuração real, experimentação, verificação e diagnóstico.

O objetivo é ter documentação que continue útil após concluir os laboratórios individuais e que possa ser consultada quando conceitos semelhantes aparecerem em estudos futuros ou em cenários reais de administração.

O projeto será expandido progressivamente da configuração inicial de servidor para administração Linux, redes, serviços, monitoramento, análise de tráfego, hardening e experimentos de segurança.

A arquitetura de rede também pode evoluir conforme novos laboratórios introduzam serviços adicionais e requisitos de infraestrutura. As configurações históricas são preservadas na documentação dos laboratórios correspondentes.
