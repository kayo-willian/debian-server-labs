# Lab 08 - Integração Grafana e Zabbix

[English](../../../../docs/labs/08-grafana-zabbix-integration/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../07-reserva-dhcp-e-persistencia-de-ip/README.md) · [Próximo laboratório](../09-dashboard-de-monitoramento-grafana/README.md)

## Visão geral

Este laboratório documenta a instalação e integração do Grafana com o ambiente de monitoramento Zabbix existente no servidor Debian.

O objetivo foi expandir a infraestrutura de monitoramento conectando o Grafana à API Zabbix, permitindo consultar e visualizar seus dados de monitoramento pelo Grafana.

A integração foi realizada utilizando o repositório oficial de pacotes do Grafana, o plugin Zabbix de Alexander Zobnin e um token de API Zabbix.

O laboratório também incluiu testes da API e diagnóstico da comunicação entre Grafana, Apache e a API Zabbix.

---

## Objetivos

* Instalar o Grafana no servidor Debian.
* Compreender a relação entre Grafana e Zabbix.
* Instalar o plugin Zabbix para Grafana.
* Compreender o endpoint JSON-RPC da API Zabbix.
* Criar um token de API Zabbix.
* Configurar uma fonte de dados Zabbix no Grafana.
* Conectar o Grafana à API Zabbix existente.
* Testar a API independentemente utilizando `curl`.
* Diagnosticar tentativas de conexão malsucedidas do Grafana.
* Validar a integração final.
* Preparar o ambiente para dashboards e visualização de dados no Grafana.

---

## Ambiente

| Item | Informação |
| --- | --- |
| Sistema operacional | Debian Server |
| IP do servidor | `192.168.1.117` |
| Versão do Zabbix | `7.4.15` |
| Versão do Grafana | `13.2.2` |
| Plugin Zabbix para Grafana | Alexander Zobnin `6.8.0` |
| Servidor web | Apache2 |
| Interface web Zabbix | `http://192.168.1.117/zabbix` |
| Interface web Grafana | `http://192.168.1.117:3000` |
| API Zabbix | `http://192.168.1.117/zabbix/api_jsonrpc.php` |
| Porta Zabbix Server | `10051/TCP` |
| Porta Zabbix Agent | `10050/TCP` |
| Porta Apache | `80/TCP` |
| Porta Grafana | `3000/TCP` |
| Banco de dados | MariaDB |

---

## Arquitetura de monitoramento existente

Antes deste laboratório, o servidor Debian já possuía um ambiente de monitoramento Zabbix funcional.

A arquitetura existente era:

```text
                    Debian Server
                         |
        +----------------+----------------+
        |                |                |
     Apache2          Zabbix           MariaDB
      :80             :10051             :3306
        |                |
        |                |
   Zabbix Web       Zabbix Server
        |                |
        +------- Zabbix Data -------+
```

O Grafana foi adicionado como camada de visualização.

A arquitetura final tornou-se:

```text
                     User Browser
                          |
                          |
                    Grafana :3000
                          |
                          |
             Zabbix Grafana Plugin
                          |
                          | HTTP / JSON-RPC
                          |
                    Apache2 :80
                          |
                          |
             /zabbix/api_jsonrpc.php
                          |
                          |
                    Zabbix API
                          |
                          |
                    Zabbix Data
```

O Grafana não substitui o Zabbix.

O Zabbix permanece responsável pelo monitoramento, coleta e armazenamento das informações, enquanto o Grafana oferece uma interface adicional para consultá-las e visualizá-las.

---

## Instalação do Grafana

O Grafana foi instalado no servidor Debian utilizando o repositório APT oficial do Grafana.

Após a instalação, o serviço foi iniciado e habilitado:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

O serviço foi então reiniciado durante o processo de configuração:

```bash
sudo systemctl restart grafana-server
```

O estado do serviço foi verificado com:

```bash
sudo systemctl status grafana-server --no-pager
```

O serviço Grafana foi confirmado como:

```text
Active: active (running)
```

O Grafana foi configurado para escutar em:

```text
TCP 3000
```

A interface web local também foi testada:

```bash
curl http://localhost:3000/
```

A resposta redirecionou para:

```text
/login
```

Isso confirmou que o serviço web Grafana estava respondendo.

---

## Porta do Grafana

O Grafana utiliza a porta TCP `3000` por padrão.

Os serviços em escuta foram inspecionados com:

```bash
ss -tulpn
```

O resultado mostrou o Grafana escutando na porta `3000` e o Apache na porta `80`.

A distinção entre os dois serviços é importante:

```text
Apache
192.168.1.117:80
        |
        +-- Zabbix Web
        +-- Zabbix API

Grafana
192.168.1.117:3000
        |
        +-- Grafana Web Interface
```

---

## Instalação do plugin Zabbix

O Grafana utiliza um plugin para comunicar-se com o Zabbix nesta integração.

O plugin Zabbix de Alexander Zobnin foi instalado utilizando:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
```

Os plugins instalados foram verificados com:

```bash
sudo grafana cli plugins ls
```

O plugin Zabbix foi identificado como:

```text
alexanderzobnin-zabbix-app @ 6.8.0
```

O serviço Grafana foi então reiniciado:

```bash
sudo systemctl restart grafana-server
```

---

## Compreensão da API Zabbix

A interface web Zabbix e a API Zabbix são endpoints distintos.

A interface web normal do Zabbix é:

```text
http://192.168.1.117/zabbix
```

O endpoint JSON-RPC da API é:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

O endpoint da API é utilizado por aplicações como o Grafana para comunicar-se programaticamente com o Zabbix.

A API utiliza requisições JSON-RPC.

Ela espera requisições HTTP `POST` contendo dados JSON.

---

## Teste da API

Um primeiro teste foi realizado utilizando uma requisição HTTP comum:

```bash
curl -i http://localhost/zabbix/api_jsonrpc.php
```

O servidor retornou:

```text
HTTP/1.0 412 Precondition Failed
```

Isso não era evidência de que a API estava com defeito.

A API espera uma requisição JSON-RPC `POST`, em vez de uma requisição `GET` comum de navegador.

Uma requisição válida à API foi então realizada:

```bash
curl -i -X POST http://localhost/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

A API retornou:

```json
{
  "jsonrpc": "2.0",
  "result": "7.4.15",
  "id": 1
}
```

Isso confirmou que a API Zabbix estava operacional.

O mesmo teste foi realizado utilizando o endereço de rede do servidor:

```bash
curl -i -X POST http://192.168.1.117/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

O resultado retornou novamente:

```text
7.4.15
```

Isso confirmou que a API também estava acessível pela interface de rede do servidor.

---

## Token de API Zabbix

O Grafana foi configurado para autenticar-se no Zabbix utilizando um token de API.

O token foi criado no Zabbix e utilizado como credencial de autenticação para a fonte de dados do Grafana.

O token em si foi intencionalmente omitido deste repositório.

Credenciais sensíveis nunca devem ser registradas no Git.

---

## Fonte de dados Grafana

Uma nova fonte de dados Zabbix foi criada no Grafana.

A URL da API configurada no Grafana era:

```text
http://192.168.1.117/zabbix/api_jsonrpc.php
```

O método geral de autenticação da fonte de dados foi configurado como:

```text
No Authentication
```

O método de autenticação da conexão Zabbix foi configurado como:

```text
API Token
```

O token de API Zabbix gerado anteriormente foi então inserido na configuração do plugin.

---

## Diagnóstico

A primeira tentativa de conexão do Grafana falhou.

O Grafana informou:

```text
Could not connect to given url
```

Os logs do serviço Grafana foram inspecionados com:

```bash
sudo journalctl -u grafana-server -n 50 --no-pager
```

Os logs continham erros como:

```text
Error querying Zabbix version
invalid character '<' looking for beginning of value
```

e:

```text
Error connecting Zabbix server
invalid character '<' looking for beginning of value
```

O erro indicava que o plugin recebia uma resposta que não era JSON válido.

Outro erro apareceu durante a investigação:

```text
basic auth is not supported for Zabbix v7.2 and later
```

Isso ajudou a identificar que a configuração de autenticação precisava ser corrigida para a versão da API Zabbix utilizada.

O plugin Zabbix foi posteriormente removido e reinstalado para reiniciar a configuração a partir de um estado limpo.

O plugin foi removido com:

```bash
sudo grafana-cli plugins remove alexanderzobnin-zabbix-app
```

Em seguida, foi instalado novamente:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
```

A instalação do plugin foi verificada:

```bash
sudo grafana cli plugins ls
```

Após reiniciar o Grafana, a fonte de dados foi configurada novamente utilizando o endpoint correto da API e autenticação por token de API.

---

## Conexão bem-sucedida

Após reconfigurar a fonte de dados, o Grafana conectou-se com sucesso ao Zabbix.

O teste da fonte de dados do Grafana retornou:

```text
Zabbix API version 7.4.15
```

Essa foi a confirmação final de que a integração funcionava.

O caminho de comunicação foi, portanto, estabelecido com sucesso:

```text
Grafana
   |
   v
Zabbix Plugin
   |
   v
Zabbix API
   |
   v
Zabbix 7.4.15
```

---

## Funções do Grafana e do Zabbix

O laboratório demonstrou que Grafana e Zabbix possuem responsabilidades distintas.

### Zabbix

O Zabbix permanece responsável por:

* Monitorar hosts
* Coletar métricas
* Processar dados de monitoramento
* Gerar problemas
* Armazenar informações de monitoramento
* Disponibilizar a API

### Grafana

O Grafana oferece:

* Visualização de dados
* Dashboards
* Gráficos
* Painéis
* Consultas
* Opções adicionais de visualização

O plugin Zabbix para Grafana atua como conexão entre os dois sistemas.

---

## Verificação

O ambiente final foi verificado por múltiplos testes.

### Serviço Grafana

```bash
sudo systemctl status grafana-server --no-pager
```

Resultado esperado:

```text
Active: active (running)
```

### Serviço HTTP Grafana

```bash
curl http://localhost:3000/
```

Comportamento esperado:

```text
/login
```

### API Zabbix

```bash
curl -i -X POST http://192.168.1.117/zabbix/api_jsonrpc.php \
-H 'Content-Type: application/json-rpc' \
-d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

Resultado esperado:

```text
"result":"7.4.15"
```

### Fonte de dados Grafana

O teste da fonte de dados Grafana retornou com sucesso:

```text
Zabbix API version 7.4.15
```

---

## Arquitetura final

```text
                         Administration PC
                                |
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
              Zabbix Web                 Grafana
              TCP 80                   TCP 3000
                    |                       |
                    |                       |
                    v                       v
              Zabbix API             Zabbix Plugin
                    |                       |
                    +-----------+-----------+
                                |
                                v
                         Zabbix Data
                                |
                                v
                           Zabbix Server
                           TCP 10051
                                |
                                v
                         Zabbix Monitored
                              Host
```

O ponto importante é que o Grafana acessa o Zabbix por sua API web. O processo Zabbix Server e a API Zabbix são componentes separados com funções distintas.

---

## O que aprendi

* O Grafana pode ser integrado a uma instalação Zabbix existente.
* O Grafana não substitui o Zabbix.
* O plugin Zabbix para Grafana fornece a camada de integração.
* A API Zabbix é disponibilizada pelo frontend web Zabbix.
* A API JSON-RPC espera requisições `POST`.
* Uma requisição `GET` comum ao endpoint da API não constitui uma consulta válida.
* `curl` pode ser utilizado para verificar independentemente se a API Zabbix funciona.
* A API Zabbix retornou a versão `7.4.15`.
* O Grafana utiliza a porta TCP `3000` por padrão.
* O Apache continua servindo o frontend e a API Zabbix na porta TCP `80`.
* O Zabbix Server continua operando independentemente na porta TCP `10051`.
* Tokens de API podem ser utilizados em vez de autenticação por usuário e senha.
* Os logs do Grafana são úteis para identificar problemas de comunicação da fonte de dados e do plugin.
* Uma resposta HTML recebida quando JSON era esperado pode indicar endpoint ou configuração incorretos.
* Testar cada componente independentemente facilita o diagnóstico.

---

## Resultado

A integração Grafana e Zabbix foi concluída com sucesso.

Nesta etapa histórica, o Grafana `13.2.2` estava em execução no servidor Debian e comunicava-se com o Zabbix `7.4.15` pela API JSON-RPC, utilizando o plugin Zabbix de Alexander Zobnin `6.8.0` e um token de API.

O ambiente estava então preparado para a etapa seguinte: consultar dados Zabbix no Grafana e criar painéis e dashboards personalizados.

---

## Estado do projeto

Concluído

A fonte de dados Grafana conectou-se com sucesso à API Zabbix e retornou a versão esperada.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
