# Lab 01: Instalação, configuração e SSH

[English](../../../../docs/labs/01-installation-configuration-ssh/README.md) · [Índice dos laboratórios](../../README.md) · [Próximo laboratório](../02-servidor-de-monitoramento-zabbix/README.md)

> Nota de privacidade: identificadores pessoais de rede neste registro histórico foram mascarados; a topologia, os comandos e as observações registrados foram preservados.

## Objetivo

O objetivo deste laboratório foi instalar Debian Server em uma máquina física, configurá-la como servidor Linux headless, estabelecer conectividade de rede, criar uma sub-rede dedicada ao laboratório e administrar remotamente o servidor por SSH.

Este laboratório representa a base inicial do repositório Debian Server Labs.

O objetivo não era apenas fazer o servidor funcionar, mas compreender o raciocínio por trás de cada configuração, verificar os resultados, identificar problemas e documentar o processo de diagnóstico.

Os principais conceitos abordados foram:

* Instalação do Debian Server
* Administração de servidor headless
* Identificação de interfaces de rede
* Endereçamento IPv4
* Sub-redes
* DHCP
* Gateways padrão
* Roteamento
* Separação de redes
* Métricas de rede
* SSH
* Administração remota
* Testes de conectividade
* Diagnóstico básico

---

## 1. Ambiente do laboratório

### Servidor físico

O servidor utilizado neste laboratório é um notebook antigo reaproveitado como servidor Linux físico.

| Componente | Especificação |
| --- | --- |
| Dispositivo | Positivo Motion |
| CPU | Intel Atom x5-Z8350 @ 1.44 GHz |
| RAM | 2 GB DDR3 |
| Armazenamento | Aproximadamente 32 GB eMMC |
| Rede | Wi-Fi |
| Sistema operacional | Debian Server |
| Ambiente de desktop | Nenhum |

A máquina está sendo utilizada como servidor headless.

Um servidor headless não necessita de ambiente gráfico de desktop para a administração normal. Em vez disso, a administração é realizada remotamente por SSH.

Isso é especialmente útil nesta máquina devido aos recursos limitados de hardware.

---

## 2. Por que Debian Server?

O Debian foi escolhido para este servidor físico por ser leve, estável, amplamente utilizado e oferecer um bom ambiente para aprender administração de servidores Linux.

A instalação foi realizada sem ambiente de desktop.

Isso significa que o servidor inicia em um ambiente de linha de comando em vez de carregar uma interface gráfica.

A intenção é aprender administração de servidores pelo terminal, sem depender de uma interface gráfica.

---

## 3. Instalação

A instalação do Debian Server foi realizada diretamente na máquina física.

Durante a instalação, o sistema foi configurado como ambiente de servidor mínimo.

Nenhum ambiente gráfico de desktop foi instalado.

A instalação teve foco nos componentes necessários para futuros laboratórios de administração de servidores e redes.

### Componentes instalados

Os componentes relevantes selecionados durante a instalação foram:

* Servidor SSH
* Utilitários padrão do sistema
* Sistema base Debian

Nenhum ambiente de desktop foi instalado.

Nenhum servidor web foi instalado nesta etapa.

A intenção é instalar serviços adicionais posteriormente em laboratórios separados, para que cada serviço seja estudado de maneira independente.

---

## 4. Primeiro login

Após a instalação, o servidor foi acessado localmente pelo terminal.

O usuário Debian criado para administração é:

```text
kayo
```

O sistema pode ser acessado por essa conta de usuário.

Para realizar tarefas administrativas, a conta root pode ser acessada com:

```bash
su -
```

O shell root fornece privilégios administrativos para configuração em nível de sistema.

O usuário comum deve ser preferido para a administração cotidiana, utilizando acesso root somente quando privilégios elevados forem realmente necessários.

---

## 5. Configuração inicial de rede

O servidor Debian conecta-se à rede por Wi-Fi.

A interface sem fio do servidor é:

```text
wlan0
```

Inicialmente, o servidor recebeu um endereço IP por DHCP.

A configuração de rede que o servidor passou a utilizar foi:

```text
IP Address: 192.168.2.161
Subnet Mask: 255.255.255.0
Network: 192.168.2.0/24
Gateway: 192.168.2.1
```

O prefixo `/24` significa que a máscara de sub-rede é:

```text
255.255.255.0
```

Isso fornece:

```text
Network:    192.168.2.0
Usable IPs: 192.168.2.1 - 192.168.2.254
Broadcast:  192.168.2.255
```

Portanto, o servidor pertence à rede de laboratório `192.168.2.0/24`.

---

## 6. O problema de rede original

Antes de criar a sub-rede dedicada ao laboratório, o computador Ubuntu e o servidor Debian estavam conectados à mesma rede.

Os endereços relevantes eram:

```text
Ubuntu Ethernet: 192.168.1.230
Ubuntu Wi-Fi:    192.168.1.222
Debian Server:   192.168.1.117
```

À primeira vista, isso parecia conveniente porque todos os dispositivos estavam dentro de:

```text
192.168.1.0/24
```

Entretanto, o computador Ubuntu possuía duas interfaces de rede conectadas à mesma sub-rede.

As interfaces eram:

```text
enp2s0
wlxXXXXXXXXXXXX
```

A tabela de roteamento continha dois caminhos possíveis para a mesma rede.

As rotas relevantes eram, na prática:

```text
192.168.1.0/24 dev enp2s0
192.168.1.0/24 dev wlxXXXXXXXXXXXX
```

O Linux precisava, portanto, decidir qual interface utilizar ao se comunicar com outro endereço `192.168.1.x`.

A interface Ethernet tinha uma métrica de rota melhor.

Por exemplo:

```text
enp2s0      metric 100
Wi-Fi       metric 600
```

Métricas menores são preferidas entre rotas comparáveis.

Assim, quando o Ubuntu tentava alcançar:

```text
192.168.1.117
```

o sistema operacional preferia a interface Ethernet.

Isso podia ser verificado com:

```bash
ip route get 192.168.1.117
```

O resultado mostrou que o tráfego estava sendo direcionado pela interface Ethernet em vez da interface Wi-Fi.

Isso criou um problema importante de rede.

O servidor estava conectado por Wi-Fi, mas o Ubuntu tinha outra interface na mesma rede IP por Ethernet.

O problema não era simplesmente que os dispositivos estavam desconectados.

O problema era que o host possuía duas interfaces pertencentes à mesma sub-rede, criando uma situação ambígua de roteamento.

---

## 7. Por que criar uma sub-rede separada para o laboratório?

Em vez de tentar forçar manualmente o Linux a escolher a interface Wi-Fi, a rede foi redesenhada.

A rede de laboratório foi separada da rede doméstica principal.

A rede principal permaneceu:

```text
192.168.1.0/24
```

A rede de laboratório tornou-se:

```text
192.168.2.0/24
```

Isso cria uma separação clara:

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

Esse projeto também torna o ambiente de laboratório mais fácil de compreender e gerenciar.

Em vez de o Ubuntu alcançar o servidor Debian por duas interfaces pertencentes à mesma sub-rede, cada interface agora pertence a uma rede IP diferente.

---

## 8. Criação da sub-rede de laboratório

A separação de rede foi criada utilizando a configuração sem fio do roteador.

O roteador fornece a rede principal pelo SSID primário:

```text
[redacted SSID]
```

O SSID dedicado ao laboratório era:

```text
Debian
```

Inicialmente, o SSID `Debian` foi configurado como rede `External Guest`.

Isso causou um isolamento mais forte entre clientes e impediu os dispositivos conectados ao SSID de laboratório de se comunicarem entre si conforme necessário.

O SSID foi então alterado para:

```text
Home Guest
```

Isso permitiu que dispositivos conectados à mesma rede de laboratório se comunicassem.

---

## 9. Configuração de sub-rede do SSID

O roteador fornece uma opção chamada:

```text
SSID Subnet
```

Ela foi habilitada para o SSID `Debian`.

A rede de laboratório foi configurada com os seguintes parâmetros:

| Parâmetro | Valor |
| --- | --- |
| SSID | Debian |
| Rede | 192.168.2.0/24 |
| Endereço IP LAN | 192.168.2.1 |
| Máscara de sub-rede | 255.255.255.0 |
| Início do DHCP | 192.168.2.100 |
| Fim do DHCP | 192.168.2.200 |

O roteador, portanto, atua como gateway da rede de laboratório:

```text
192.168.2.1
```

---

## 10. Configuração DHCP

O DHCP foi configurado para a sub-rede de laboratório.

O pool DHCP foi definido como:

```text
192.168.2.100
-
192.168.2.200
```

Isso significa que os clientes que se conectam ao SSID `Debian` podem receber automaticamente endereços dentro desse intervalo.

Por exemplo:

```text
192.168.2.100
192.168.2.101
192.168.2.102
...
192.168.2.200
```

O roteador fornece a configuração de rede aos clientes por DHCP, incluindo informações como:

* Endereço IP
* Máscara de sub-rede
* Gateway padrão
* Outros parâmetros de rede

O servidor acabou recebendo:

```text
192.168.2.161
```

que está dentro do intervalo DHCP configurado.

---

## 11. Arquitetura final de rede

Após o redesenho da rede, o ambiente ficou assim:

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

O computador Ubuntu, portanto, possui duas interfaces de rede conectadas a duas redes diferentes.

### Ubuntu

Ethernet:

```text
Interface: enp2s0
IP:        192.168.1.230/24
Network:   192.168.1.0/24
```

Wi-Fi:

```text
Interface: wlxXXXXXXXXXXXX
IP:        192.168.2.143/24
Network:   192.168.2.0/24
```

### Servidor Debian

```text
Interface: wlan0
IP:        192.168.2.161/24
Network:   192.168.2.0/24
Gateway:   192.168.2.1
```

---

## 12. Verificação das interfaces de rede do Ubuntu

A máquina Ubuntu foi utilizada como estação de administração.

As interfaces de rede foram inspecionadas com:

```bash
ip addr
```

As interfaces importantes eram:

```text
enp2s0
wlxXXXXXXXXXXXX
```

O endereçamento resultante era:

```text
enp2s0
192.168.1.230/24

wlxXXXXXXXXXXXX
192.168.2.143/24
```

Isso confirmou que o Ubuntu estava conectado simultaneamente às duas redes.

---

## 13. Verificação da tabela de roteamento

A tabela de roteamento foi verificada com:

```bash
ip route
```

As rotas relevantes eram:

```text
default via 192.168.1.1 dev enp2s0 metric 100
default via 192.168.2.1 dev wlxXXXXXXXXXXXX metric 600

192.168.1.0/24 dev enp2s0 src 192.168.1.230 metric 100
192.168.2.0/24 dev wlxXXXXXXXXXXXX src 192.168.2.143 metric 600
```

A diferença importante em relação à configuração original é que as duas interfaces agora pertencem a redes diferentes.

O Ubuntu possui:

```text
192.168.1.0/24
```

por Ethernet e:

```text
192.168.2.0/24
```

por Wi-Fi.

Portanto, não há ambiguidade sobre qual interface deve ser utilizada para alcançar o servidor Debian.

---

## 14. Verificação da rota até o servidor

A rota exata até o servidor Debian foi verificada utilizando:

```bash
ip route get 192.168.2.161
```

O resultado foi:

```text
192.168.2.161 dev wlxXXXXXXXXXXXX src 192.168.2.143
```

Esta é uma verificação importante.

Ela demonstra que o Ubuntu alcança:

```text
192.168.2.161
```

por meio de:

```text
wlxXXXXXXXXXXXX
```

utilizando:

```text
192.168.2.143
```

como seu endereço de origem.

Em outras palavras:

```text
Ubuntu Wi-Fi
192.168.2.143
      |
      |
      v
Debian Server
192.168.2.161
```

O tráfego não é mais enviado incorretamente pela interface Ethernet.

---

## 15. Teste de conectividade

A conectividade entre Ubuntu e Debian foi testada com:

```bash
ping -c 4 192.168.2.161
```

O teste retornou:

```text
4 packets transmitted
4 packets received
0% packet loss
```

Isso confirmou a conectividade IP básica entre as duas máquinas.

O ping bem-sucedido demonstrou que:

* O servidor Debian tinha conectividade de rede.
* O Ubuntu conseguia alcançar a sub-rede de laboratório.
* Os dois dispositivos conseguiam se comunicar por Wi-Fi.
* A infraestrutura de rede permitia o tráfego necessário.
* A configuração da sub-rede estava funcionando.
* A decisão de roteamento no Ubuntu estava correta.

---

## 16. Servidor SSH

O SSH foi instalado durante a instalação do Debian.

O SSH fornece administração remota segura de sistemas Linux.

Em vez de conectar monitor e teclado à máquina Debian, o computador Ubuntu pode abrir uma sessão de shell remota.

Isso é especialmente útil para o servidor físico porque a máquina está sendo utilizada como servidor headless.

O servidor pode, portanto, ser administrado a partir do computador principal sem exigir um ambiente gráfico.

---

## 17. Usuário SSH

O usuário Debian criado para administração remota é:

```text
kayo
```

O servidor pode ser acessado utilizando o seguinte comando a partir do Ubuntu:

```bash
ssh kayo@192.168.2.161
```

O comando segue esta estrutura:

```text
ssh USER@SERVER_IP
```

Portanto:

```text
ssh
```

inicia o cliente SSH.

```text
kayo
```

é a conta de usuário no servidor Debian.

```text
192.168.2.161
```

é o endereço IP de destino do servidor.

---

## 18. Primeiro acesso remoto

Após executar:

```bash
ssh kayo@192.168.2.161
```

o cliente SSH estabelece uma conexão com o servidor Debian.

Após a autenticação, o shell muda para o ambiente Debian remoto.

O prompt torna-se semelhante a:

```text
kayo@debian:~$
```

Nesse momento, os comandos digitados no terminal são executados no servidor Debian, não no Ubuntu.

Essa distinção é importante ao administrar uma máquina remota.

Por exemplo:

```bash
hostname
```

executado dentro da sessão SSH verifica o hostname do servidor Debian.

Comandos executados antes de estabelecer a conexão SSH continuam operando na máquina Ubuntu.

---

## 19. Recuperação de senha

Durante a configuração inicial do SSH, a senha da conta `kayo` não estava disponível.

Como o acesso administrativo local ainda estava disponível, a senha foi redefinida utilizando a conta root.

Primeiro, foi obtido acesso root:

```bash
su -
```

Em seguida, a senha do usuário foi alterada:

```bash
passwd kayo
```

O sistema solicitou uma nova senha e sua confirmação.

Após alterar a senha, a autenticação SSH foi tentada novamente:

```bash
ssh kayo@192.168.2.161
```

A conexão foi bem-sucedida.

Isso demonstrou um conceito importante de administração:

A senha de um usuário pode ser alterada por um administrador com privilégios suficientes sem que ele precise conhecer a senha anterior do usuário.

---

## 20. O que acontece quando o SSH é encerrado?

O SSH fornece uma sessão remota ao servidor.

Encerrar a conexão SSH não desliga a máquina Debian.

Por exemplo, o seguinte comando:

```bash
exit
```

encerra a sessão SSH atual.

O servidor Debian continua em execução normalmente.

A relação pode ser compreendida como:

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

O SSH é, portanto, um método de acesso remoto ao servidor, e não o mecanismo que mantém o servidor ligado.

---

## 21. Processo de diagnóstico

Uma das partes mais importantes deste laboratório não foi a configuração final, mas compreender por que o projeto de rede original causava problemas.

O processo de diagnóstico seguiu uma metodologia básica de redes.

### Etapa 1: Identificar as interfaces

```bash
ip addr
```

Isso mostrou que o Ubuntu tinha interfaces Ethernet e Wi-Fi.

### Etapa 2: Verificar os endereços IP

Inicialmente, as duas interfaces estavam dentro da mesma rede:

```text
192.168.1.0/24
```

### Etapa 3: Verificar a tabela de roteamento

```bash
ip route
```

Isso mostrou que a Ethernet tinha a rota preferida devido à métrica menor.

### Etapa 4: Consultar o Linux sobre como alcançaria o servidor

```bash
ip route get 192.168.1.117
```

Isso demonstrou que o tráfego estava sendo direcionado pela Ethernet.

### Etapa 5: Redesenhar a rede

Em vez de manipular manualmente as métricas das rotas, foi criada uma sub-rede dedicada ao laboratório.

```text
Main network:
192.168.1.0/24

Lab network:
192.168.2.0/24
```

### Etapa 6: Verificar a nova rota

```bash
ip route get 192.168.2.161
```

O resultado mostrou:

```text
dev wlxXXXXXXXXXXXX
```

### Etapa 7: Testar a conectividade

```bash
ping -c 4 192.168.2.161
```

O teste foi bem-sucedido com:

```text
0% packet loss
```

### Etapa 8: Testar a administração remota

```bash
ssh kayo@192.168.2.161
```

O acesso SSH foi bem-sucedido.

---

## 22. Conceitos importantes de redes aprendidos

### Sub-rede

Uma sub-rede é uma divisão lógica de uma rede IP.

Neste laboratório:

```text
192.168.1.0/24
```

e:

```text
192.168.2.0/24
```

são duas redes IPv4 diferentes.

---

### Gateway padrão

O gateway padrão é o dispositivo utilizado para alcançar redes fora da sub-rede local quando não há uma rota mais específica.

Para a rede de laboratório Debian:

```text
Gateway:
192.168.2.1
```

Esse endereço pertence ao roteador.

---

### DHCP

O DHCP fornece automaticamente configurações de rede aos clientes.

O pool DHCP do laboratório era:

```text
192.168.2.100 - 192.168.2.200
```

O servidor Debian recebeu:

```text
192.168.2.161
```

---

### Roteamento

O roteamento determina para onde o tráfego de rede deve ser enviado.

O computador Ubuntu possuía duas redes:

```text
192.168.1.0/24 -> Ethernet
192.168.2.0/24 -> Wi-Fi
```

A rede de destino determina qual interface é apropriada.

---

### Métrica de rota

Uma métrica de rota é utilizada para determinar preferência quando múltiplas rotas comparáveis estão disponíveis.

Valores menores são preferidos.

A configuração original continha:

```text
Ethernet metric: 100
Wi-Fi metric:    600
```

Portanto, a Ethernet era preferida quando ambas as interfaces potencialmente alcançavam a mesma rede.

O projeto final evitou essa ambiguidade colocando as interfaces em sub-redes diferentes.

---

### SSH

O SSH fornece acesso remoto criptografado a um sistema.

Neste laboratório:

```text
Ubuntu
192.168.2.143
      |
      | SSH
      |
Debian
192.168.2.161
```

A máquina Debian pode, portanto, ser administrada remotamente.

---

## 23. Configuração final

### Servidor Debian

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

### Máquina Ubuntu de administração

#### Ethernet

```text
Interface:
enp2s0

IP:
192.168.1.230/24

Network:
192.168.1.0/24
```

#### Wi-Fi

```text
Interface:
wlxXXXXXXXXXXXX

IP:
192.168.2.143/24

Network:
192.168.2.0/24
```

### Rede de laboratório do roteador

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

## 24. Lista de verificação

Os seguintes testes foram concluídos com sucesso:

* [x] Debian instalado em hardware físico
* [x] Ambiente de desktop não instalado
* [x] Servidor SSH instalado
* [x] Usuário administrativo criado
* [x] Senha do usuário configurada
* [x] Interface Wi-Fi identificada
* [x] SSID dedicado ao laboratório criado
* [x] SSID configurado como Home Guest
* [x] SSID Subnet habilitado
* [x] Sub-rede de laboratório configurada
* [x] Intervalo DHCP configurado
* [x] Debian recebeu um endereço da rede de laboratório
* [x] Ubuntu conectado às duas redes
* [x] Tabela de roteamento verificada
* [x] Rota até o Debian verificada com `ip route get`
* [x] Conectividade ICMP testada com `ping`
* [x] Conectividade SSH testada
* [x] Acesso ao shell remoto confirmado

---

## 25. Principais aprendizados

Este laboratório estabeleceu a arquitetura básica que será utilizada pelos demais Debian Server Labs.

A lição mais importante foi que problemas de rede nem sempre são causados por um dispositivo desconectado ou um endereço IP incorreto.

Uma máquina pode ter endereços IP válidos em todas as interfaces e ainda apresentar problemas de conectividade devido às decisões de roteamento.

A configuração original demonstrou o problema de ter múltiplas interfaces conectadas à mesma sub-rede.

A configuração final resolveu o problema separando as redes:

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

Isso forneceu uma arquitetura de laboratório mais clara e tornou o comportamento de roteamento previsível.

O laboratório também estabeleceu o SSH como método principal de administração do servidor Debian.

O servidor agora pode operar sem um ambiente gráfico ou acesso físico constante.

---

## 26. Próxima etapa

Com o servidor Debian instalado, alcançável e acessível por SSH, os próximos laboratórios podem se concentrar na administração de sistemas Linux e em serviços.

Possíveis temas incluem:

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

Cada tema será implementado como laboratório prático separado.

A intenção é manter o mesmo ciclo de aprendizado em todo o repositório:

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

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
