# Lab 04 — Servidor web Nginx no Debian

[English](../../../../docs/labs/04-nginx-web-server/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../03-firewall-e-controle-de-acesso/README.md) · [Próximo laboratório](../05-dashboard-personalizado-zabbix/README.md)

## Visão geral

Este laboratório consiste em implantar e configurar o **Nginx como servidor web no Debian GNU/Linux**.

O objetivo foi compreender o funcionamento básico de um servidor web em um ambiente Linux real, incluindo:

* Instalação do Nginx
* Gerenciamento de serviços com systemd
* Estrutura de diretórios do servidor web
* Configuração de blocos de servidor
* Alteração da porta de escuta
* Hospedagem de uma página HTML personalizada
* Validação da configuração do Nginx
* Testes de requisições HTTP
* Logs de acesso e de erros do Nginx
* Nginx Stub Status
* Diagnóstico de problemas de configuração e serviços

O laboratório foi realizado em um ambiente Debian Server leve.

---

## Objetivos

* Instalar o Nginx utilizando o gerenciador de pacotes do Debian.
* Compreender onde o Nginx armazena suas configurações e arquivos do site.
* Configurar um diretório raiz web personalizado.
* Executar o Nginx em uma porta HTTP não padrão.
* Criar e servir uma página HTML personalizada.
* Validar a configuração antes de aplicar alterações.
* Testar o servidor utilizando `curl`.
* Inspecionar os logs do Nginx.
* Habilitar o endpoint `stub_status`.
* Praticar o diagnóstico de um serviço Linux real.

---

## Ambiente

| Componente | Configuração |
| --- | --- |
| Sistema operacional | Debian GNU/Linux 13 (Trixie) |
| Arquitetura | x86_64 |
| Servidor web | Nginx |
| Porta HTTP | 8080 |
| Diretório raiz web | `/var/www/nginx-site` |
| Configuração principal | `/etc/nginx/nginx.conf` |
| Configuração do servidor | `/etc/nginx/sites-enabled/default` |
| Log de acesso | `/var/log/nginx/access.log` |
| Log de erros | `/var/log/nginx/error.log` |
| Gerenciador de serviços | systemd |

O pacote Nginx está disponível diretamente nos repositórios do Debian 13.

---

## 1. Instalação do Nginx

A primeira etapa foi atualizar o índice de pacotes:

```bash
sudo apt update
```

O Nginx foi então instalado com:

```bash
sudo apt install nginx
```

Após a instalação, o serviço foi verificado com:

```bash
systemctl status nginx
```

O resultado esperado é um serviço ativo:

```text
Active: active (running)
```

O Nginx também pode ser gerenciado pelo systemd:

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
```

---

## 2. Compreensão da estrutura do Nginx

O arquivo de configuração principal é:

```text
/etc/nginx/nginx.conf
```

O Debian também fornece uma estrutura de configuração de sites:

```text
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/
```

A configuração padrão de servidor ativa neste laboratório era:

```text
/etc/nginx/sites-enabled/default
```

O próprio site foi armazenado em:

```text
/var/www/nginx-site
```

A estrutura resultante era aproximadamente:

```text
/etc/nginx/
├── nginx.conf
├── sites-available/
│   └── default
└── sites-enabled/
    └── default

/var/www/
└── nginx-site/
    └── index.html

/var/log/nginx/
├── access.log
└── error.log
```

---

## 3. Criação do diretório raiz web

Um diretório dedicado foi criado para o site de laboratório:

```bash
sudo mkdir -p /var/www/nginx-site
```

A página principal foi criada como:

```text
/var/www/nginx-site/index.html
```

A página foi intencionalmente criada como uma página técnica simples de laboratório, e não como um portfólio pessoal.

Sua finalidade era fornecer um recurso HTTP real que o Nginx pudesse servir.

---

## 4. Configuração do servidor

A configuração padrão de servidor do Nginx foi editada:

```bash
sudo nano /etc/nginx/sites-enabled/default
```

O servidor foi configurado para utilizar a porta `8080` em vez da porta HTTP padrão.

A configuração relevante tornou-se, conceitualmente:

```nginx
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;

    server_name _;

    root /var/www/nginx-site;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Por que a porta 8080?

A porta 80 já estava sendo utilizada por outro serviço no servidor.

Em vez de remover ou reconfigurar esse serviço existente, o Nginx foi transferido para a porta `8080`.

Isso permitiu que múltiplos serviços relacionados a HTTP coexistissem na mesma máquina:

```text
Port 80
└── Existing web service

Port 8080
└── Nginx
```

Essa foi uma decisão importante de diagnóstico e configuração durante o laboratório.

---

## 5. Compreensão de `server`

A configuração do Nginx utiliza um bloco `server` para definir como as requisições devem ser tratadas.

Exemplo:

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/nginx-site;
}
```

As diretivas importantes são:

### `listen`

Define a porta na qual o Nginx aceita conexões HTTP.

```nginx
listen 8080;
```

### `server_name`

Define o nome de host associado ao bloco de servidor.

```nginx
server_name _;
```

O `_` é frequentemente usado como nome convencional em um servidor padrão; a seleção padrão depende da diretiva de escuta, não desse nome.

### `root`

Define onde o Nginx procura os arquivos solicitados.

```nginx
root /var/www/nginx-site;
```

### `index`

Define o arquivo padrão retornado ao solicitar `/`.

```nginx
index index.html;
```

### `location`

Define como requisições correspondentes a uma URI específica devem ser tratadas.

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

---

## 6. Validação da configuração

Um dos comandos mais importantes aprendidos neste laboratório foi:

```bash
sudo nginx -t
```

O comando verifica a sintaxe da configuração do Nginx e tenta verificar se os arquivos referenciados podem ser abertos.

A validação bem-sucedida produziu:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Isso deve ser feito **antes de recarregar o Nginx após alterações de configuração**.

A documentação oficial do Nginx descreve `-t` como a opção de teste de configuração.

---

## 7. Aplicação das alterações de configuração

Após modificar a configuração, o Nginx foi recarregado:

```bash
sudo systemctl reload nginx
```

Uma recarga aplica a nova configuração sem exigir a interrupção completa do serviço.

Isso é diferente de:

```bash
sudo systemctl restart nginx
```

Uma reinicialização interrompe e inicia novamente o serviço, enquanto uma recarga solicita que o Nginx carregue a nova configuração e substitua os workers de maneira gradual.

---

## 8. Teste do servidor web

O servidor foi testado localmente com:

```bash
curl -I http://127.0.0.1:8080/
```

A resposta bem-sucedida foi:

```text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 9475
Connection: keep-alive
```

### O que isso comprova

A resposta confirma que:

1. O Nginx está em execução.
2. O Nginx está escutando na porta `8080`.
3. A requisição HTTP chegou ao Nginx.
4. O Nginx encontrou o recurso solicitado.
5. O Nginx retornou a página HTML com sucesso.

A linha mais importante é:

```text
HTTP/1.1 200 OK
```

Isso representa uma resposta HTTP bem-sucedida.

---

## 9. Teste do servidor sem navegador

O `curl` foi utilizado ao longo do laboratório porque permite testar o comportamento HTTP diretamente pelo terminal.

Requisição básica:

```bash
curl http://127.0.0.1:8080/
```

Somente cabeçalhos:

```bash
curl -I http://127.0.0.1:8080/
```

Isso é útil no diagnóstico porque permite ao administrador investigar se o problema está:

* No próprio Nginx
* Na porta de escuta
* Na configuração do servidor
* No diretório raiz web
* No arquivo solicitado
* Ou no cliente/navegador

---

## 10. Logs do Nginx

O Nginx mantém dois logs importantes.

### Log de acesso

```text
/var/log/nginx/access.log
```

Registra as requisições HTTP recebidas pelo servidor.

Por exemplo:

```bash
sudo tail -f /var/log/nginx/access.log
```

Isso pode ser utilizado enquanto são realizadas requisições:

```bash
curl http://127.0.0.1:8080/
```

A requisição deve aparecer no log de acesso.

### Log de erros

```text
/var/log/nginx/error.log
```

Registra erros e problemas relacionados ao Nginx.

Para monitorá-lo:

```bash
sudo tail -f /var/log/nginx/error.log
```

Esses logs são fundamentais no diagnóstico de um servidor web.

---

## 11. Inspeção da configuração completa

O Nginx fornece:

```bash
sudo nginx -T
```

Esse comando testa a configuração presente nos arquivos e imprime seu conteúdo completo; não é uma consulta direta à configuração em memória do processo ativo.

Foi útil durante o laboratório para conferir:

* Quais arquivos de configuração eram incluídos
* Qual porta estava configurada para escuta
* Qual diretório raiz de documentos estava configurado
* Qual bloco de servidor estava configurado
* Quais locations estavam definidas

---

## 12. Nginx Stub Status

Durante o laboratório, o módulo `stub_status` do Nginx também foi habilitado.

A configuração adicionada ao bloco de servidor foi:

```nginx
location /stub_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

Isso disponibiliza estatísticas básicas de execução do Nginx.

A documentação oficial do Nginx descreve `stub_status` como fornecedor de informações básicas de estado, como conexões ativas, conexões aceitas, conexões tratadas, requisições e conexões em leitura, escrita e espera.

Após modificar a configuração:

```bash
sudo nginx -t
```

Em seguida:

```bash
sudo systemctl reload nginx
```

O endpoint foi testado com:

```bash
curl http://127.0.0.1:8080/stub_status
```

A resposta bem-sucedida foi:

```text
Active connections: 1
server accepts handled requests
 7 7 9
Reading: 0 Writing: 1 Waiting: 0
```

Isso confirmou que o endpoint de status estava funcionando.

---

## 13. Compreensão de `stub_status`

As informações retornadas podem ser interpretadas da seguinte forma:

```text
Active connections: 1
```

Conexões atuais de clientes ativas.

```text
7 7 9
```

Esses valores representam:

```text
accepts
handled
requests
```

Portanto, neste teste:

```text
Accepted connections: 7
Handled connections: 7
Requests: 9
```

A última linha:

```text
Reading: 0
Writing: 1
Waiting: 0
```

representa os estados atuais das conexões.

A documentação oficial do Nginx define esses contadores e estados em detalhes.

---

## 14. Diagnóstico realizado

### Problema 1 — Serviço HTTP existente

#### Sintoma

A porta 80 já estava sendo utilizada por outro serviço web.

#### Problema

O Nginx não poderia simplesmente assumir a porta HTTP padrão sem interferir no serviço existente.

#### Resolução

O Nginx foi configurado para escutar em:

```text
8080
```

Isso permitiu a coexistência dos serviços.

---

### Problema 2 — Verificar se o Nginx realmente servia a página

Em vez de presumir que o serviço estava funcionando, o servidor foi testado diretamente:

```bash
curl -I http://127.0.0.1:8080/
```

Resultado:

```text
HTTP/1.1 200 OK
Server: nginx
```

#### Resolução

Foi confirmado que o serviço HTTP estava operacional.

---

### Problema 3 — Alterações de configuração

Após modificar a configuração do Nginx, foi necessário determinar se a sintaxe era válida.

O seguinte comando foi utilizado:

```bash
sudo nginx -t
```

Resultado:

```text
syntax is ok
test is successful
```

Somente após a validação bem-sucedida a configuração foi recarregada:

```bash
sudo systemctl reload nginx
```

---

### Problema 4 — Verificação do endpoint de status

O endpoint `stub_status` foi adicionado e testado independentemente antes de considerá-lo parte da arquitetura de monitoramento.

Teste:

```bash
curl http://127.0.0.1:8080/stub_status
```

Resultado:

```text
Active connections: 1
server accepts handled requests
 7 7 9
Reading: 0 Writing: 1 Waiting: 0
```

Isso demonstrou que o módulo e o endpoint do Nginx estavam funcionando.

---

## 15. Comandos úteis de diagnóstico

### Verificar o estado do serviço

```bash
systemctl status nginx
```

### Verificar a versão do Nginx

```bash
nginx -v
```

### Validar a configuração

```bash
sudo nginx -t
```

### Exibir a configuração completa

```bash
sudo nginx -T
```

### Verificar portas de escuta

```bash
sudo ss -ltnp | grep nginx
```

### Testar HTTP

```bash
curl -I http://127.0.0.1:8080/
```

### Testar Stub Status

```bash
curl http://127.0.0.1:8080/stub_status
```

### Acompanhar o log de acesso

```bash
sudo tail -f /var/log/nginx/access.log
```

### Acompanhar o log de erros

```bash
sudo tail -f /var/log/nginx/error.log
```

### Recarregar a configuração

```bash
sudo systemctl reload nginx
```

### Reiniciar o serviço

```bash
sudo systemctl restart nginx
```

### Interromper o serviço

```bash
sudo systemctl stop nginx
```

### Iniciar o serviço

```bash
sudo systemctl start nginx
```

---

## 16. Arquitetura final

A arquitetura final do laboratório é:

```text
                    Debian Server
                         │
                         │
                    ┌────▼────┐
                    │  Nginx  │
                    └────┬────┘
                         │
                  HTTP : 8080
                         │
                  ┌──────▼──────┐
                  │ nginx-site  │
                  │  index.html │
                  └─────────────┘
                         │
                         │
                /stub_status
                         │
                  Runtime metrics
```

---

## 17. O que foi aprendido

Este laboratório proporcionou experiência prática com um servidor web Linux real.

Os principais conceitos praticados foram:

* Instalação de software com APT.
* Gerenciamento de serviços com systemd.
* Compreensão dos arquivos de configuração do Nginx.
* Compreensão dos blocos de servidor.
* Configuração de portas de escuta.
* Configuração de um diretório raiz web.
* Serviço de conteúdo HTML estático.
* Testes HTTP utilizando `curl`.
* Validação da configuração antes de aplicar alterações.
* Recarga segura de serviços.
* Leitura de logs de serviços.
* Inspeção de sockets de escuta.
* Diagnóstico de conflitos de portas.
* Disponibilização de informações de execução do Nginx com `stub_status`.

A lição prática mais importante foi o fluxo de diagnóstico:

```text
Change configuration
        ↓
nginx -t
        ↓
Configuration valid?
        ↓
       YES
        ↓
systemctl reload nginx
        ↓
curl
        ↓
HTTP response
        ↓
Inspect logs if necessary
```

Esse fluxo é mais importante que memorizar comandos individuais do Nginx.

---

## 18. Referências

* Documentação oficial do Nginx — instalação e pacotes
* Documentação oficial do Nginx — módulo HTTP Stub Status
* Repositório de pacotes Debian — pacote Nginx para Debian 13

A documentação oficial do Nginx confirma os métodos de instalação disponíveis para Linux e o comportamento da configuração.

O repositório de pacotes Debian lista o Nginx como pacote disponível para Debian 13 (Trixie).

---

## Estado

**Concluído**

Nesta etapa histórica, o servidor Debian executava o Nginx como servidor HTTP funcional na porta `8080`, servindo um site de laboratório personalizado e disponibilizando um endpoint local `stub_status` para informações de execução.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
