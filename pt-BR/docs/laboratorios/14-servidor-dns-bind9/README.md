# Lab 14 - Servidor DNS BIND9

[English](../../../../docs/labs/14-bind9-dns-server/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../13-dashboard-personalizado-grafana/README.md)

## Objetivo

Configurar um servidor DNS utilizando BIND9 em um servidor Debian físico, configurar zonas DNS direta e reversa, testar a resolução DNS a partir de outra máquina da rede e integrar o monitoramento DNS ao Zabbix.

## Ambiente

| Dispositivo | Função | IP |
| --- | --- | --- |
| Debian Server | Servidor DNS BIND9 | `192.168.1.117` |
| Debian Monitoring Server | Cliente DNS / Zabbix Server | `192.168.1.219` |
| Ubuntu PC | Cliente DNS opcional | `192.168.1.230` |

### Serviços

| Serviço | Porta |
| --- | ---: |
| DNS | 53/UDP |
| DNS | 53/TCP |
| Zabbix Agent | 10050/TCP |

## 1. Instalação do BIND9

Instale o BIND9 e os utilitários DNS necessários:

```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils
```

Verifique o serviço:

```bash
sudo systemctl status bind9
```

Nesta instalação Debian, o serviço systemd efetivo é `named.service`, com `bind9.service` funcionando como uma unidade vinculada.

## 2. Configuração das opções do BIND9

Edite:

```bash
sudo nano /etc/bind/named.conf.options
```

Configuração:

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

A rede `192.168.1.0/24` permite consultas DNS a partir da rede local.

Valide a configuração:

```bash
sudo named-checkconf
```

A ausência de saída, com código de saída bem-sucedido, indica que a configuração passou na verificação de sintaxe.

## 3. Criação das zonas DNS

Crie o diretório de zonas:

```bash
sudo mkdir -p /etc/bind/zones
```

Edite:

```bash
sudo nano /etc/bind/named.conf.local
```

Adicione a zona direta:

```conf
zone "example.lan" IN {
    type master;
    file "/etc/bind/zones/db.example.lan";
    allow-update { none; };
};
```

Adicione a zona reversa:

```conf
zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/bind/zones/db.192.168.1";
    allow-update { none; };
};
```

Nenhum servidor DNS secundário foi configurado neste laboratório.

## 4. Configuração da zona direta

Crie:

```bash
sudo nano /etc/bind/zones/db.example.lan
```

Configuração:

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

Valide a zona:

```bash
sudo named-checkzone example.lan /etc/bind/zones/db.example.lan
```

Resultado esperado:

```text
zone example.lan/IN: loaded serial 2026092801
OK
```

## 5. Configuração da zona reversa

Crie:

```bash
sudo nano /etc/bind/zones/db.192.168.1
```

Configuração:

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

Valide a zona:

```bash
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1
```

Resultado esperado:

```text
zone 1.168.192.in-addr.arpa/IN: loaded serial 2026092801
OK
```

## 6. Inicialização do BIND9

Reinicie o serviço:

```bash
sudo systemctl restart bind9
```

Verifique seu estado:

```bash
sudo systemctl status bind9
```

O serviço deve mostrar:

```text
Active: active (running)
```

As zonas configuradas também devem aparecer como carregadas no log do serviço.

## 7. Teste de resolução DNS direta

A partir do servidor DNS Debian:

```bash
dig @127.0.0.1 ns1.example.lan
```

Uma resposta autoritativa bem-sucedida retornou:

```text
ns1.example.lan.    604800    IN    A    192.168.1.117
```

Uma versão mais curta pode ser utilizada:

```bash
dig @127.0.0.1 ns1.example.lan +short
```

Resultado:

```text
192.168.1.117
```

A flag `aa` na resposta completa indica que a resposta é autoritativa para a zona configurada.

## 8. Teste de resolução DNS reversa

Execute:

```bash
dig @127.0.0.1 -x 192.168.1.117
```

Resultado:

```text
117.1.168.192.in-addr.arpa.    604800    IN    PTR    ns1.example.lan.
```

Isso confirma o funcionamento da zona reversa para a consulta testada.

## 9. Teste DNS a partir de outra máquina

O Debian Monitoring Server em `192.168.1.219` foi utilizado como cliente DNS.

Consulta direta:

```bash
dig @192.168.1.117 ns1.example.lan
```

Resultado:

```text
ns1.example.lan.    604800    IN    A    192.168.1.117
```

Consulta reversa:

```bash
dig @192.168.1.117 -x 192.168.1.117
```

Resultado:

```text
117.1.168.192.in-addr.arpa.    604800    IN    PTR    ns1.example.lan.
```

Esses testes confirmam que o servidor DNS pode ser acessado a partir de outra máquina da rede.

## 10. Verificação das portas DNS

A partir do Monitoring Server:

### UDP

```bash
nc -zuv 192.168.1.117 53
```

Resultado:

```text
53 (domain) open
```

### TCP

```bash
nc -zv 192.168.1.117 53
```

Resultado:

```text
53 (domain) open
```

O BIND9 escuta tanto em UDP/53 quanto em TCP/53 nesta configuração. Um teste UDP com nc, isoladamente, não comprova uma resposta DNS válida.

Verifique as portas de escuta diretamente no servidor DNS:

```bash
sudo ss -lntup | grep ':53'
```

A saída confirma que `named` está escutando na porta 53.

## 11. Monitoramento DNS no Zabbix

O servidor Debian já executa o Zabbix Agent.

Um `UserParameter` personalizado foi adicionado para testar uma resolução DNS efetiva, em vez de apenas verificar se a porta 53 está aberta.

Edite:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Adicione:

```ini
UserParameter=dns.example.lan,dig @127.0.0.1 ns1.example.lan +short
```

Reinicie o agente:

```bash
sudo systemctl restart zabbix-agent
```

Verifique:

```bash
sudo systemctl status zabbix-agent
```

A partir do servidor de monitoramento Zabbix, teste a chave personalizada:

```bash
zabbix_get -s 192.168.1.117 -k dns.example.lan
```

Resultado:

```text
192.168.1.117
```

Isso confirma que o Zabbix consegue solicitar remotamente a verificação de saúde DNS pelo Zabbix Agent.

## 12. Item do Zabbix

Um item personalizado foi criado no host `Debian Server`:

```text
Name: DNS resolution - ns1.example.lan
Type: Zabbix agent
Key: dns.example.lan
Type of information: Text
Update interval: 1m
```

O item retornou com sucesso:

```text
192.168.1.117
```

## 13. Trigger do Zabbix

Uma trigger foi criada para detectar uma resposta DNS inesperada:

```text
Name: DNS resolution failed - ns1.example.lan
Severity: High
```

Expressão:

```text
last(/Debian Server/dns.example.lan)<>"192.168.1.117"
```

A trigger permite ao Zabbix identificar quando a resposta DNS recebida difere do valor esperado.

## Resultado

O laboratório implementou com sucesso um servidor DNS BIND9 funcional na máquina Debian física.

O ambiente documentado nesta etapa inclui:

* Zona DNS direta
* Zona DNS reversa
* Resolução DNS autoritativa
* Registros `A`, `PTR`, `NS`, `MX` e `CNAME`
* Recursão DNS e encaminhadores
* UDP/53 e TCP/53
* Consultas DNS remotas a partir de outra máquina
* Monitoramento Zabbix de uma resolução DNS efetiva
* Trigger Zabbix para falha de resolução DNS

## O que pratiquei

* Instalação e configuração do BIND9
* Zonas DNS direta e reversa
* Tipos de registros DNS
* Consultas DNS autoritativas
* DNS recursivo e encaminhadores
* Porta DNS 53 por UDP e TCP
* `dig` e `nc` para testes de rede
* Validação de zonas BIND9 com `named-checkzone`
* Validação da configuração BIND9 com `named-checkconf`
* `UserParameter` do Zabbix
* Itens e triggers personalizados do Zabbix
* Monitoramento de um serviço de rede com base em sua função efetiva em vez de verificar apenas sua porta

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
