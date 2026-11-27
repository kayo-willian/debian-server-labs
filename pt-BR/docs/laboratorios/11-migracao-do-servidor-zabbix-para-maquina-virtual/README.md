# Lab 11 - Migração do servidor de monitoramento Zabbix para uma máquina virtual

[English](../../../../docs/labs/11-migrating-the-zabbix-monitoring-server-to-a-virtual-machine/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../10-remocao-do-grafana-e-otimizacao-de-recursos/README.md) · [Próximo laboratório](../12-remocao-do-zabbix-e-limpeza-do-servidor/README.md)

As senhas históricas do banco de dados estão mascaradas nos comandos e configurações abaixo.

## Visão geral

Este laboratório documenta a migração do ambiente de monitoramento de um servidor Debian físico para uma máquina virtual Debian dedicada.

O servidor físico original era responsável tanto pelo monitoramento quanto pelos serviços de infraestrutura. Executava Zabbix Server, Zabbix Agent, Zabbix Web, Apache, MariaDB e outros componentes em hardware com poucos recursos.

O objetivo deste laboratório foi separar essas responsabilidades:

```text
Ubuntu Host
│
├── Debian Physical Server
│   └── Infrastructure and network services
│
└── Debian-Monitoring-Server (VirtualBox VM)
    ├── Zabbix Server
    ├── Zabbix Agent
    ├── Zabbix Web
    ├── Apache
    ├── MariaDB
    └── Grafana
```

Após a migração, o servidor Debian físico tornou-se um host monitorado, em vez de ser responsável pelo próprio Zabbix Server.

A arquitetura final de monitoramento é:

```text
Debian Physical Server
192.168.1.117
        │
        │ Zabbix Agent :10050
        ▼
Debian Monitoring Server
192.168.1.219
        │
        ├── Zabbix Server :10051
        ├── Zabbix Web :80
        ├── MariaDB :3306
        └── Grafana
```

---

## 1. Ambiente original

Antes da migração, o servidor Debian físico continha a pilha de monitoramento.

O servidor físico utilizava:

```text
IP address: 192.168.1.117
OS: Debian 13 (trixie)
Architecture: x86-64
```

Serviços importantes:

```text
Apache       → TCP 80
Zabbix Agent → TCP 10050
Zabbix Server → TCP 10051
MariaDB      → TCP 3306 locally
Zabbix Web   → /zabbix
```

A API Zabbix estava disponível em:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

A instalação Zabbix existente era da versão:

```text
Zabbix 7.4.15
```

O servidor já continha dados importantes de monitoramento, incluindo:

* Hosts
* Itens
* Triggers
* Histórico
* Templates
* Dashboards
* Monitoramento web
* Configuração Zabbix
* Conteúdo do banco de dados

Portanto, iniciar uma instalação Zabbix completamente nova e configurar tudo manualmente contrariaria a finalidade da migração.

---

## 2. Alternativas de migração

Vários métodos de migração foram considerados.

### Alternativa 1 - Clonagem bit a bit do disco

A primeira possibilidade era clonar toda a instalação Debian.

A ideia seria:

```text
Physical disk
      ↓
Complete disk image
      ↓
Virtual disk
      ↓
Virtual machine
```

Ferramentas como Clonezilla, SystemRescue ou `dd` poderiam teoricamente ser utilizadas.

A vantagem seria preservar quase tudo exatamente como existia.

Entretanto, essa abordagem não foi selecionada.

O servidor físico utilizava um dispositivo eMMC:

```text
/dev/mmcblk2
```

com as partições:

```text
mmcblk2
├── mmcblk2p1 → EFI
├── mmcblk2p2 → /
└── mmcblk2p3 → swap
```

O risco de modificar acidentalmente o disco original era desnecessário.

Além disso, a instalação física e a máquina virtual teriam características de hardware diferentes.

---

## 3. Método de migração selecionado

O método escolhido foi uma **migração em nível de arquivos combinada com dump e restauração do banco de dados**.

Em vez de copiar todo o sistema operacional:

```text
Physical Debian
       ↓
backup configuration + database
       ↓
new Debian VM
       ↓
reinstall required packages
       ↓
restore database/configuration
```

Essa abordagem preserva o estado importante da aplicação, permitindo que a VM tenha uma instalação Debian limpa.

O próprio sistema operacional original não é clonado.

O banco de dados Zabbix carrega a maior parte do estado importante de monitoramento.

---

## 4. Backup antes da migração

O servidor Debian físico tinha aproximadamente:

```text
/dev/mmcblk2p2
27G total
2.4G used
23G free
```

Uma unidade USB separada de 16 GB foi utilizada para a migração.

Ela foi montada como:

```text
/mnt/usb
```

A unidade USB continha:

```text
/mnt/usb/
├── configs/
├── packages.txt
└── zabbix.sql.gz
```

---

## 5. Backup do banco de dados Zabbix

A parte mais importante da migração foi o banco de dados Zabbix.

Um dump do MariaDB foi criado:

```bash
sudo mariadb-dump --single-transaction --routines --triggers --events zabbix | gzip > /mnt/usb/zabbix.sql.gz
```

### O que o comando faz

```text
mariadb-dump
```

cria um backup lógico do banco de dados.

```text
--single-transaction
```

permite criar um dump consistente de tabelas transacionais sem bloquear todo o banco da mesma forma que um dump tradicional com bloqueios.

```text
--routines
```

inclui procedimentos armazenados e funções.

```text
--triggers
```

inclui triggers do banco de dados.

```text
--events
```

inclui eventos agendados do MariaDB.

```text
zabbix
```

é o banco de dados do qual o backup está sendo realizado.

```text
| gzip
```

compacta a saída.

```text
> /mnt/usb/zabbix.sql.gz
```

grava o backup compactado na unidade USB.

O backup foi então verificado:

```bash
gzip -t /mnt/usb/zabbix.sql.gz && echo "Backup OK"
```

O resultado foi:

```text
Backup OK
```

Isso confirmou a integridade do arquivo compactado no teste realizado, não a completude lógica do dump.

---

## 6. Backup dos arquivos de configuração

Os principais diretórios de configuração também foram copiados.

```bash
sudo cp -a /etc/zabbix /mnt/usb/configs/
sudo cp -a /etc/apache2 /mnt/usb/configs/
sudo cp -a /etc/mysql /mnt/usb/configs/
sudo cp -a /etc/php /mnt/usb/configs/
sudo cp -a /etc/ssh /mnt/usb/configs/
```

Arquivos adicionais de configuração do sistema foram preservados:

```bash
sudo cp -a /etc/fstab /mnt/usb/configs/
sudo cp -a /etc/hostname /mnt/usb/configs/
sudo cp -a /etc/hosts /mnt/usb/configs/
```

A finalidade não era copiar cegamente todas as configurações para a VM.

Em vez disso, esses arquivos foram mantidos como referências e utilizados seletivamente ao reconstruir o ambiente.

---

## 7. Registro dos pacotes instalados

Os pacotes instalados no servidor físico foram registrados:

```bash
dpkg-query -W | sort > /mnt/usb/packages.txt
```

Isso gerou uma lista contendo aproximadamente 515 pacotes.

Ela foi útil para identificar o ambiente original e determinar quais pacotes eram necessários na nova VM.

Os pacotes importantes de monitoramento incluíam:

```text
zabbix-agent
zabbix-apache-conf
zabbix-frontend-php
zabbix-server-mysql
zabbix-sql-scripts
zabbix-release
```

As versões originais incluíam:

```text
Apache 2.4.68
PHP 8.4.24
MariaDB 11.8.6
Zabbix 7.4.15
```

---

## 8. Criação da máquina virtual

Uma nova VM VirtualBox foi criada.

```text
Name: Debian-Monitoring-Server
RAM: 4 GB
CPU: 2 vCPUs
Disk: 32 GB dynamically allocated VDI
Network: Bridged Adapter
OS: Debian 13.7.0 amd64 netinst
```

A VM foi intencionalmente dedicada ao monitoramento.

Posteriormente, o servidor Debian físico estaria disponível para serviços de infraestrutura, como:

```text
DNS
DHCP
Samba
NFS
SSH
VPN
Docker
```

Isso cria uma separação mais clara de responsabilidades.

---

## 9. Instalação do novo sistema Debian

O Debian 13 foi instalado normalmente no VirtualBox.

Inicialmente, a VM recebeu seu endereço IP por DHCP:

```text
192.168.1.219
```

O usuário `kayo` foi adicionado ao sudo.

O SSH foi instalado para que a VM pudesse ser administrada remotamente.

O hostname foi configurado como:

```text
debian-monitoring-server
```

O sistema foi atualizado:

```bash
sudo apt update
sudo apt full-upgrade -y
```

---

## 10. Conexão da unidade USB de migração à VM

A unidade USB estava originalmente disponível ao servidor físico.

O VirtualBox foi configurado para repassar o dispositivo USB à nova VM.

Dentro da VM:

```bash
lsblk -f
```

mostrou:

```text
/dev/sdb1
```

como a unidade USB de migração.

Ela foi montada:

```bash
sudo mkdir -p /mnt/usb
sudo mount /dev/sdb1 /mnt/usb
```

O conteúdo foi confirmado:

```bash
ls -lah /mnt/usb
```

Resultado:

```text
configs/
lost+found/
packages.txt
zabbix.sql.gz
```

O backup do banco de dados foi testado novamente:

```bash
gzip -t /mnt/usb/zabbix.sql.gz
```

Nenhum erro foi retornado.

---

## 11. Instalação do MariaDB

O MariaDB foi instalado na VM:

```bash
sudo apt install mariadb-server mariadb-client -y
```

O MariaDB foi iniciado automaticamente e verificado.

Inicialmente, o servidor de banco de dados continha apenas seus bancos padrão.

Um novo banco Zabbix foi criado:

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

Um usuário do banco foi criado:

```sql
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '<INITIAL_DB_PASSWORD>';
```

Privilégios foram concedidos:

```sql
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
```

A senha temporária foi posteriormente substituída pela senha original do banco Zabbix.

---

## 12. Restauração do banco de dados Zabbix

O banco Zabbix original foi restaurado no banco recém-criado:

```bash
sudo mariadb zabbix < <(gzip -dc /mnt/usb/zabbix.sql.gz)
```

O comando funciona em duas etapas:

```text
gzip -dc
     ↓
decompress zabbix.sql.gz
     ↓
mariadb zabbix
     ↓
restore SQL into the zabbix database
```

O banco de dados foi então inspecionado:

```sql
SHOW TABLES;
```

Muitas tabelas Zabbix existentes apareceram, incluindo tabelas como:

```text
acknowledges
actions
alerts
auditlog
autoreg_host
changelog
conditions
config_autoreg_tls
connector
```

Isso indicou que o banco Zabbix original havia sido restaurado; a validação da aplicação ocorreu posteriormente.

---

## 13. Instalação do repositório Zabbix

A VM precisava da mesma versão Zabbix utilizada pelo servidor original.

O repositório Zabbix para Debian 13 foi instalado utilizando:

```bash
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
```

Em seguida:

```bash
sudo apt update
```

A versão disponível foi verificada:

```bash
apt-cache policy zabbix-server-mysql
```

A versão candidata era:

```text
7.4.15
```

Isso correspondia ao servidor físico original.

Utilizar a mesma versão principal/secundária reduziu os riscos de compatibilidade durante a migração.

---

## 14. Instalação dos componentes Zabbix

Os pacotes necessários foram instalados:

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

Isso instalou:

```text
Zabbix Server
Zabbix Web
Zabbix Apache configuration
Zabbix SQL scripts
Zabbix Agent
```

A VM, portanto, tinha o software necessário para utilizar o banco restaurado.

---

## 15. Restauração da configuração do Zabbix Server

A configuração recém-instalada foi comparada à configuração original.

Os parâmetros originais importantes eram:

```text
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_server.pid
DBName=zabbix
DBUser=zabbix
Timeout=4
```

A configuração original também continha a senha do banco Zabbix.

Antes de substituir a configuração, foi realizado um backup do novo arquivo:

```bash
sudo cp /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.new
```

A configuração original foi então restaurada:

```bash
sudo cp /mnt/usb/configs/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf
```

---

## 16. Restauração da senha do banco de dados

O usuário do banco Zabbix foi configurado com a senha original:

```sql
ALTER USER 'zabbix'@'localhost' IDENTIFIED BY '<UPDATED_DB_PASSWORD>';
FLUSH PRIVILEGES;
```

O acesso ao banco foi testado:

```bash
mariadb -u zabbix -p zabbix -e "SELECT COUNT(*) FROM users;"
```

A consulta retornou:

```text
COUNT(*)
3
```

Isso confirmou que o usuário Zabbix conseguia acessar o banco restaurado.

---

## 17. Inicialização do Zabbix Server

O servidor Zabbix foi iniciado:

```bash
sudo systemctl start zabbix-server
```

Seu estado foi verificado:

```bash
sudo systemctl status zabbix-server
```

O serviço estava em execução e iniciou seus processos Zabbix normais, incluindo sincronizadores de configuração, sincronizadores de histórico, pollers HTTP e pollers de navegador.

Essa foi uma confirmação importante de que:

```text
Zabbix Server
      ↓
MariaDB
      ↓
restored Zabbix database
```

estava funcionando.

---

## 18. Configuração da interface web Zabbix

O Apache já estava instalado e em execução.

Seu estado foi verificado:

```bash
sudo systemctl status apache2 --no-pager
```

O PHP foi verificado:

```bash
sudo apache2ctl -M | grep -E 'php|rewrite'
```

O módulo PHP estava carregado.

A configuração Apache do Zabbix existia:

```text
/etc/apache2/conf-enabled/zabbix.conf
```

Ela disponibilizava a URL:

```text
/zabbix
```

Inicialmente, acessar:

```text
http://192.168.1.219/zabbix
```

retornava um erro 404 do Apache.

Entretanto, o teste local mostrou que:

```bash
curl -I http://127.0.0.1/zabbix/
```

retornava:

```text
HTTP/1.1 302 Found
Location: setup.php
```

Isso demonstrou que o Apache e a rota do frontend Zabbix respondiam localmente.

---

## 19. Correção da configuração do banco do Zabbix Web

A configuração do frontend Zabbix era um link simbólico:

```text
/usr/share/zabbix/ui/conf/zabbix.conf.php
        ↓
/etc/zabbix/web/zabbix.conf.php
```

O destino não existia.

Portanto, o arquivo de configuração foi criado em:

```text
/etc/zabbix/web/zabbix.conf.php
```

com as informações de conexão do banco de dados:

```php
$DB['TYPE'] = 'MYSQL';
$DB['SERVER'] = 'localhost';
$DB['PORT'] = '0';
$DB['DATABASE'] = 'zabbix';
$DB['USER'] = 'zabbix';
$DB['PASSWORD'] = '<UPDATED_DB_PASSWORD>';
```

O nome do servidor Zabbix foi configurado como:

```php
$ZBX_SERVER_NAME = 'Debian Monitoring Server';
```

O Apache foi recarregado.

A interface Zabbix Web tornou-se então acessível.

---

## 20. Verificação do ambiente Zabbix migrado

O teste mais importante não era simplesmente verificar se a página de login aparecia.

Os dados Zabbix existentes foram verificados.

O ambiente migrado continha os originais:

```text
Hosts
Items
Triggers
History
Templates
Dashboards
Web monitoring
```

A configuração Zabbix existente apareceu na VM.

Isso confirmou que a migração foi bem-sucedida no nível da aplicação e do banco de dados.

Portanto, a migração não exigiu reconstruir manualmente o ambiente de monitoramento.

---

## 21. Instalação do Zabbix Agent e teste do servidor físico

A nova VM também tinha o Zabbix Agent instalado.

O agente da própria VM foi configurado para o servidor Zabbix local.

O servidor Debian físico permanecia em:

```text
192.168.1.117
```

A VM precisava comunicar-se com seu Zabbix Agent em:

```text
192.168.1.117:10050
```

Inicialmente, o teste foi:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

A resposta foi:

```text
ZBX_NOTSUPPORTED: Received empty response from Zabbix Agent at [192.168.1.117].
Assuming that agent dropped connection because of access permissions.
```

Isso indicou que a conexão de rede existia, mas o Zabbix Agent do servidor físico não permitia o IP da VM.

---

## 22. Permissão para o novo servidor Zabbix

No servidor Debian físico, a configuração do Zabbix Agent foi verificada:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Originalmente:

```text
Server=127.0.0.1,192.168.1.117
ServerActive=127.0.0.1
Hostname=Debian Server
```

O endereço da VM foi adicionado:

```text
Server=127.0.0.1,192.168.1.117,192.168.1.219
```

O agente foi reiniciado:

```bash
sudo systemctl restart zabbix-agent
```

O serviço foi verificado como em execução.

---

## 23. Teste de comunicação com o Zabbix Agent

O teste foi repetido a partir da VM:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

O resultado foi:

```text
1
```

Essa é a resposta bem-sucedida esperada de um Zabbix Agent.

O servidor físico tornou-se, portanto, acessível ao novo servidor Zabbix.

Posteriormente, a interface Zabbix Web mostrou o host Debian físico com indicador verde.

A interface continuou exibindo:

```text
192.168.1.117:10050
```

Isso está correto.

O endereço pertence ao **agente do servidor físico**, e não ao novo Zabbix Server.

---

## 24. Habilitação dos serviços na inicialização

Os serviços de monitoramento foram configurados para iniciar automaticamente:

```bash
sudo systemctl enable zabbix-server zabbix-agent apache2 mariadb
```

A arquitetura foi configurada para persistir após reinicializações da VM:

```text
MariaDB
   ↓
Zabbix Server
   ↓
Zabbix Web
   ↓
Zabbix Agent
```

---

## 25. Por que um IP estático foi necessário

Nesse momento, a VM funcionava com:

```text
192.168.1.219
```

Entretanto, esse endereço havia sido atribuído inicialmente por DHCP.

Isso cria um problema para um servidor de monitoramento.

Por exemplo:

```text
Today:
VM → 192.168.1.219

After DHCP lease changes:
VM → 192.168.1.150
```

O ambiente Zabbix e outras configurações poderiam então referenciar um endereço incorreto.

Para um servidor cuja função é monitorar outros sistemas, um endereço estável é importante.

Portanto, a VM foi configurada com um endereço IPv4 estático.

---

## 26. Primeira tentativa de configuração de IP estático

A primeira abordagem foi configurar o endereço em:

```text
/etc/network/interfaces
```

A configuração tornou-se:

```text
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.219
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.1
```

O IPv6 permaneceu:

```text
iface enp0s3 inet6 auto
```

Entretanto, o sistema também executava `dhcpcd`.

Isso criou dois possíveis mecanismos de configuração de rede:

```text
ifupdown
     +
dhcpcd
```

O processo `dhcpcd` em execução foi confirmado com:

```bash
ps aux | grep '[d]hcpcd'
```

O processo estava em execução como PID 661 com PID pai 1.

Não havia um:

```text
dhcpcd.service
```

visível, nem um hook `dhcpcd` evidente dentro de:

```text
/etc/network/if-*.d/
```

Portanto, utilizar ambos os mecanismos de configuração simultaneamente era indesejável.

---

## 27. Configuração final de IP estático

A abordagem final foi permitir que o `dhcpcd` gerenciasse a interface, atribuindo explicitamente um endereço estático.

O seguinte foi adicionado a:

```text
/etc/dhcpcd.conf
```

```text
interface enp0s3
static ip_address=192.168.1.219/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

A configuração IPv4 em:

```text
/etc/network/interfaces
```

foi alterada para:

```text
auto enp0s3
iface enp0s3 inet manual

iface enp0s3 inet6 auto
```

Isso impede que o `ifupdown` tente atribuir outra configuração IPv4.

---

## 28. Aplicação da configuração de rede

A interface foi reiniciada:

```bash
sudo ifdown enp0s3 && sudo ifup enp0s3
```

Como a operação estava sendo realizada por SSH, havia a possibilidade de a conexão ser temporariamente interrompida.

A VM foi então acessada novamente por:

```bash
ssh kayo@192.168.1.219
```

A conexão foi bem-sucedida.

---

## 29. Verificação do endereço estático

O endereço foi verificado:

```bash
ip -4 addr show enp0s3
```

O resultado incluía:

```text
inet 192.168.1.219/24
```

e, de maneira importante:

```text
valid_lft forever
preferred_lft forever
```

Isso, em conjunto com a configuração estática explícita, sustenta que o endereço não era uma concessão DHCP temporária.

A configuração final do `dhcpcd` também foi verificada:

```bash
grep -E '^(interface enp0s3|static ip_address|static routers|static domain_name_servers)' /etc/dhcpcd.conf
```

Resultado:

```text
interface enp0s3
static ip_address=192.168.1.219/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

A tabela de roteamento final continha:

```text
default via 192.168.1.1 dev enp0s3 src 192.168.1.219
192.168.1.0/24 dev enp0s3 proto dhcp scope link src 192.168.1.219
```

Embora a rota local ainda exibisse `proto dhcp`, o endereço efetivo foi verificado como:

```text
valid_lft forever
preferred_lft forever
```

e a configuração estática explícita existia em `dhcpcd.conf`.

Portanto, a VM não dependia mais de receber outro endereço IPv4 do roteador.

---

## 30. Arquitetura final

A migração alterou a arquitetura de:

```text
Before

Physical Debian Server
192.168.1.117
│
├── Zabbix Server
├── Zabbix Agent
├── Zabbix Web
├── Apache
├── MariaDB
└── Monitoring data
```

para:

```text
After

Ubuntu Host
│
├── Debian Physical Server
│   192.168.1.117
│   │
│   └── Zabbix Agent :10050
│
└── Debian Monitoring Server VM
    192.168.1.219
    │
    ├── Zabbix Server :10051
    ├── Zabbix Agent :10050
    ├── Zabbix Web :80
    ├── Apache
    └── MariaDB
```

O caminho de comunicação nesta etapa era:

```text
Debian Physical Server
        │
        │ Zabbix Agent
        │ 192.168.1.117:10050
        ▼
Debian Monitoring Server
        │
        ├── Zabbix Server
        ├── MariaDB
        └── Zabbix Web
```

---

## 31. Resultado

O ambiente de monitoramento foi migrado com sucesso do servidor Debian físico para a máquina virtual Debian dedicada.

A migração preservou o estado existente da aplicação Zabbix por meio de backup e restauração do banco de dados, em vez de reconstruir manualmente o ambiente.

O servidor Debian físico permaneceu intacto e tornou-se um host monitorado.

A nova VM de monitoramento passou a ter um endereço fixo:

```text
192.168.1.219
```

O servidor físico permaneceu em:

```text
192.168.1.117
```

A comunicação foi verificada com:

```bash
zabbix_get -s 192.168.1.117 -k agent.ping
```

retornando:

```text
1
```

A interface Zabbix também mostrou o servidor Debian físico com estado saudável.

Nesse momento, a arquitetura pode ser considerada:

```text
              ┌──────────────────────────────┐
              │ Debian Monitoring Server VM  │
              │ 192.168.1.219                │
              │                              │
              │ Zabbix Server                │
              │ Zabbix Web                   │
              │ MariaDB                      │
              │ Apache                       │
              └──────────────┬───────────────┘
                             │
                         Zabbix Agent
                         TCP 10050
                             │
                             ▼
              ┌──────────────────────────────┐
              │ Debian Physical Server      │
              │ 192.168.1.117               │
              │                              │
              │ Infrastructure services     │
              │ + Zabbix Agent              │
              └──────────────────────────────┘
```

A principal mudança arquitetural é que **o servidor Debian físico deixou de ser responsável pela plataforma de monitoramento. A máquina virtual tornou-se o servidor dedicado de monitoramento Zabbix, enquanto a máquina Debian física tornou-se um dos sistemas monitorados.**

Essa separação também deixa o servidor físico disponível para futuros projetos de infraestrutura, sem misturar esses serviços com a plataforma de monitoramento.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
