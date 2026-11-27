# Lab 02: Servidor de monitoramento com Zabbix

[English](../../../../docs/labs/02-monitoring-server-with-zabbix/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../01-instalacao-configuracao-ssh/README.md) · [Próximo laboratório](../03-firewall-e-controle-de-acesso/README.md)

Laboratório prático focado na implantação de uma solução completa de monitoramento em um servidor Debian físico com recursos limitados, abrangendo preparação do banco de dados, configuração de repositório, instalação de serviços, configuração da interface web, monitoramento distribuído por agentes e gerenciamento da retenção de dados.

Este laboratório documenta o caminho completo de um servidor Debian sem serviços adicionais até um sistema de monitoramento funcional observando dois hosts pela rede, incluindo os erros encontrados e resolvidos durante o processo.

O objetivo é o mesmo dos laboratórios anteriores: não apenas fazer o sistema funcionar, mas compreender por que cada componente existe, como as partes interagem, como verificar cada etapa e como recuperar-se de falhas.

## Objetivo

Os objetivos deste laboratório foram:

* Compreender o que é um sistema de monitoramento e por que é uma ferramenta fundamental na administração de redes e servidores
* Instalar e configurar uma aplicação web de três camadas: banco de dados, servidor de aplicação e frontend web
* Importar um esquema inicial de banco de dados e compreender seu conteúdo
* Ajustar um serviço para operar com recursos limitados de hardware
* Monitorar um segundo host pela rede utilizando o agente Zabbix
* Verificar a comunicação entre servidor e agente de ponta a ponta
* Configurar a retenção de dados para proteger o espaço limitado em disco
* Praticar a leitura de logs de serviços como ferramenta principal de diagnóstico

---

## 1. O que é um sistema de monitoramento

Um sistema de monitoramento coleta continuamente medições de computadores e equipamentos de rede, armazena-as, avalia regras sobre elas e notifica administradores quando algo falha ou está prestes a falhar.

Sem monitoramento, um administrador pode descobrir uma falha apenas quando um usuário reclama. Com monitoramento, o sistema pode detectar a falha, registrar quando começou e notificar automaticamente as pessoas responsáveis.

O Zabbix é uma solução open source de monitoramento de nível corporativo. Foi escolhido para este laboratório por ser amplamente utilizado no mercado de trabalho, funcionar de maneira aceitável em hardware modesto quando ajustado corretamente e apresentar os conceitos fundamentais de monitoramento: hosts, itens, triggers, eventos, alertas, dashboards e templates.

### Por que isso importa para a administração de redes

O monitoramento é uma atividade cotidiana nas operações de rede. A familiaridade com plataformas de monitoramento é importante em funções de infraestrutura. Os conceitos aprendidos aqui são transferíveis a outras ferramentas, pois a habilidade fundamental está nos conceitos, não no produto.

---

## 2. Ambiente do laboratório

### Hardware do servidor

| Componente | Especificação |
| --- | --- |
| Dispositivo | Positivo Motion |
| CPU | Intel Atom x5-Z8350 |
| RAM | 2 GB DDR3 |
| Armazenamento | Aproximadamente 32 GB |
| Rede | Wi-Fi |
| Sistema operacional | Debian 13 (trixie) |
| Desktop | Nenhum |

O mesmo servidor físico do [Lab 01](../01-instalacao-configuracao-ssh/README.md) foi reutilizado. O hardware limitado é intencional e torna o ajuste de recursos parte obrigatória do exercício, em vez de um refinamento opcional.

### Versões de software

| Componente | Versão |
| --- | --- |
| Debian | 13 (trixie) |
| Kernel | Linux 6.12 (amd64) |
| MariaDB | 11.8 |
| Zabbix | 7.4 (repositório oficial) |
| Servidor web | Apache 2.4 |
| PHP | 8.4 |

A versão do sistema operacional foi confirmada antes da escolha do pacote de repositório com:

```bash
cat /etc/os-release
```

Essa etapa importa porque os nomes dos pacotes de repositório incluem a versão da distribuição de destino. Instalar um pacote compilado para outra versão do Debian pode causar erros de dependência.

### Rede

A arquitetura de rede de laboratório do [Lab 01](../01-instalacao-configuracao-ssh/README.md) foi mantida sem alterações.

```text
Administration Workstation
            |
            | SSH and HTTP
            |
            v
Debian Server (Zabbix Server, Database, Frontend)
            |
            | SNMP capable network (planned)
            v
Network Devices
```

O servidor Zabbix, o banco de dados e o frontend web são executados no servidor Debian. A estação de administração executa um agente Zabbix e é monitorada pelo servidor como um host gerenciado comum.

Nota: nos comandos abaixo, valores de exemplo substituem endereços e credenciais reais. Substitua-os durante a instalação e nunca publique valores reais em um repositório público.

| Placeholder | Significado |
| --- | --- |
| `<SERVER_IP>` | Endereço IP do servidor Debian |
| `<WORKSTATION_IP>` | Endereço IP da estação Ubuntu monitorada |
| `<STRONG_DB_PASSWORD>` | Senha do usuário do banco Zabbix |
| `<STRONG_UI_PASSWORD>` | Senha do usuário Admin da interface web Zabbix |

---

## 3. Arquitetura do Zabbix

O Zabbix é composto por vários componentes independentes que se comunicam entre si. Compreender essa arquitetura explica praticamente todos os arquivos de configuração utilizados neste laboratório.

```text
                 Web Browser
                      |
                      | HTTP
                      v
  +--------------------------------------+
  |            Debian Server             |
  |                                      |
  |  Apache + PHP  (Frontend)            |
  |         |                            |
  |         | SQL                        |
  |         v                            |
  |  MariaDB  (Historical data)          |
  |         ^                            |
  |         |                            |
  |  Zabbix Server  (Collector/Worker)   |
  |         ^                            |
  |         | TCP 10051                  |
  +---------|----------------------------+
            |
            | TCP 10050
            v
     Zabbix Agent on remote hosts
```

| Componente | Função | Escuta em |
| --- | --- | --- |
| Zabbix Server | Coleta dados, avalia triggers, grava no banco de dados | TCP 10051 |
| Zabbix Agent | Pequeno serviço instalado em cada host monitorado, responde a consultas de métricas | TCP 10050 |
| Banco de dados | Armazena configuração, valores coletados e eventos | Socket local / 3306 |
| Frontend | Aplicação PHP servida pelo Apache, a interface web | HTTP 80 |

O frontend não coleta métricas por si mesmo. Ele lê dados do banco. O servidor coleta e grava. O agente responde a consultas sobre a máquina em que é executado. Quando algo para de funcionar, identificar qual desses quatro componentes falhou é a primeira etapa do diagnóstico.

---

## 4. Conceito importante: usuário do banco versus usuário da aplicação

Este laboratório produziu dois pares de credenciais completamente distintos, e confundi-los causou uma falha real de login documentada na seção de diagnóstico.

| Par de credenciais | Onde reside | Utilizado por | Finalidade |
| --- | --- | --- | --- |
| Usuário do banco | Dentro do MariaDB | Zabbix Server e frontend | Comunicação interna entre aplicação e banco |
| Usuário web | Dentro do banco Zabbix | Pessoa que abre o navegador | Login na interface administrativa |

O usuário do banco é criado pelo administrador por instruções SQL. O usuário web é criado automaticamente pela importação do esquema com uma senha padrão amplamente conhecida, que deve ser alterada imediatamente após o primeiro login. Essa separação entre contas de serviços e contas humanas existe em praticamente toda aplicação web.

---

## 5. Procedimento de instalação

### Etapa 1: confirmar a versão da distribuição

O pacote de repositório depende da versão exata do Debian.

```bash
cat /etc/os-release
```

Resultado esperado: `VERSION_ID="13"` e codinome `trixie`.

A versão determina qual pacote de repositório do Zabbix deverá ser baixado posteriormente.

### Etapa 2: criar um arquivo de swap

O servidor tem 2 GB de RAM e hospedará simultaneamente um banco de dados, uma aplicação PHP e um servidor de monitoramento. Um arquivo de swap funciona como espaço adicional de emergência e reduz o risco de o mecanismo OOM encerrar serviços.

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

Verificação: `free -h` deve listar aproximadamente 1 GiB de swap total, e a entrada em `/etc/fstab` configura sua ativação após reinicialização.

Por que a swap existe: sob pressão de memória, o sistema pode mover páginas inativas para o disco. Sem swap, o esgotamento de memória pode encerrar processos abruptamente. Com swap, há uma margem adicional, embora ainda possam ocorrer lentidão e encerramento por falta de memória.

### Etapa 3: instalar e verificar o servidor de banco de dados

```bash
sudo apt update
sudo apt install mariadb-server -y
systemctl status mariadb
ss -tulpn | grep 3306
```

Resultado esperado: o serviço mostra `active (running)` e a escuta ocorre em `127.0.0.1:3306`.

O endereço de escuta merece atenção: nesta configuração, o MariaDB vincula-se a `127.0.0.1`, ou seja, aceita conexões TCP apenas da máquina local. Isso é correto para este laboratório porque o servidor Zabbix e o frontend são executados no mesmo host. Expor o banco à rede sem necessidade constitui um problema de segurança.

### Etapa 4: criar o banco de dados e o usuário da aplicação

```bash
sudo mysql
```

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '<STRONG_DB_PASSWORD>';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

Explicação de cada instrução:

| Instrução | Por que existe |
| --- | --- |
| `CREATE DATABASE ... utf8mb4 ... utf8mb4_bin` | O Zabbix utiliza esse conjunto de caracteres e collation binária para armazenamento e comparações corretos |
| `CREATE USER ... @'localhost'` | Cria uma conta dedicada restrita a conexões locais, seguindo o princípio do privilégio mínimo |
| `GRANT ALL PRIVILEGES ON zabbix.*` | Concede direitos somente sobre o banco Zabbix, não sobre todo o servidor de banco de dados |
| `SET GLOBAL log_bin_trust_function_creators = 1` | Permite temporariamente que a importação do esquema crie funções armazenadas |

Detalhe importante: `SET GLOBAL` altera uma variável em tempo de execução que não sobrevive à reinicialização do banco. Ela deve ser definida novamente antes de importar caso o banco tenha sido reiniciado, e retornar a `0` após a importação.

### Etapa 5: adicionar o repositório oficial do Zabbix

Os repositórios da distribuição podem estar defasados ou não conter determinados componentes. O Zabbix publica seu próprio repositório com pacotes atualizados.

O nome correto do pacote inclui a versão da distribuição. Na época deste laboratório, havia suporte ao Debian 13 a partir do Zabbix 7.4.

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest+debian13_all.deb
sudo dpkg -i zabbix-release_latest+debian13_all.deb
sudo apt update
```

O `apt update` após instalar o pacote de repositório é necessário. Ignorá-lo pode fazer a etapa seguinte falhar com erros de pacote não encontrado, mesmo que os nomes estejam corretos. Isso aconteceu na prática e está documentado na seção de diagnóstico.

Verificação de que o repositório está ativo:

```bash
apt-cache policy zabbix-server-mysql | head -5
```

A versão candidata deve fazer referência a `repo.zabbix.com`.

### Etapa 6: instalar os componentes Zabbix

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

O que cada pacote fornece:

| Pacote | Função |
| --- | --- |
| `zabbix-server-mysql` | Servidor de monitoramento compilado para MySQL/MariaDB |
| `zabbix-frontend-php` | Interface web PHP |
| `zabbix-apache-conf` | Configuração Apache que disponibiliza o frontend |
| `zabbix-sql-scripts` | Esquema SQL e dados usados para inicializar o banco |
| `zabbix-agent` | Agente de monitoramento, utilizado aqui para automonitoramento local |

Durante essa instalação, o gerenciador de pacotes também instalou e habilitou Apache e PHP automaticamente, alterou o Apache para o módulo de multiprocessamento prefork exigido pelo mod_php e habilitou a configuração Apache do Zabbix.

Verificação:

```bash
dpkg -l | grep zabbix
```

Os cinco pacotes Zabbix, além de `zabbix-release`, devem aparecer na lista com estado `ii`.

### Etapa 7: importar o esquema inicial

O arquivo compactado do esquema contém a estrutura completa do banco: mais de 150 tabelas abrangendo hosts, itens, histórico, triggers, usuários e eventos.

Localize o caminho efetivo do arquivo de esquema dentro do pacote instalado:

```bash
dpkg -L zabbix-sql-scripts | grep server.sql.gz
```

Na versão utilizada neste laboratório, o arquivo está localizado em:

```text
/usr/share/zabbix/sql-scripts/mysql/server.sql.gz
```

Documentação antiga frequentemente cita `/usr/share/zabbix-sql-scripts/mysql/server.sql.gz`, que não existia nesta instalação. Quando um tutorial referencia um caminho inexistente, listar os arquivos do pacote proprietário com `dpkg -L` revela a localização correta. Essa é uma técnica de diagnóstico aplicável a outros casos.

Execute a importação com o caminho retornado pelo comando anterior:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

O comando demora vários minutos nesse hardware e parece travado perto do final. Isso foi observado neste laboratório.

Verifique a importação:

```bash
sudo mysql -e "USE zabbix; SHOW TABLES;" | wc -l
```

Uma contagem acima de 150 indica que as tabelas foram criadas, mas não comprova isoladamente a importação integral.

Após importar, restaure a variável em tempo de execução:

```bash
sudo mysql -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

### Etapa 8: configurar o servidor Zabbix

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Garanta que os seguintes parâmetros estejam definidos:

```text
DBName=zabbix
DBUser=zabbix
DBPassword=<STRONG_DB_PASSWORD>
CacheSize=64M
HistoryCacheSize=16M
```

`DBName`, `DBUser` e `DBPassword` conectam o servidor ao banco criado anteriormente. As duas diretivas de cache reduzem a quantidade de RAM reservada pelo servidor para cache, o que é essencial em uma máquina de 2 GB. Com as configurações padrão, o servidor reserva significativamente mais memória e compete com o banco e o servidor web.

### Etapa 9: iniciar e habilitar os serviços

```bash
sudo systemctl restart zabbix-server apache2
sudo systemctl enable zabbix-server apache2
```

Verifique:

```bash
systemctl status zabbix-server --no-pager
systemctl status apache2 --no-pager
```

Ambos devem mostrar `active (running)`.

Se o servidor Zabbix não iniciar, os arquivos de log são uma fonte importante para identificar a causa:

```bash
journalctl -u zabbix-server -n 30 --no-pager
sudo tail -30 /var/log/zabbix/zabbix_server.log
```

### Etapa 10: concluir o assistente web

Abra o frontend a partir da estação de administração:

```text
http://<SERVER_IP>/zabbix/
```

O assistente de instalação solicita os parâmetros de conexão do banco de dados:

| Campo | Valor |
| --- | --- |
| Tipo de banco | MySQL |
| Host do banco | localhost |
| Porta do banco | 0 (padrão) |
| Nome do banco | zabbix |
| Armazenar credenciais em | Texto simples |
| Usuário | zabbix |
| Senha | `<STRONG_DB_PASSWORD>` |
| TLS do banco | Desabilitado |

O TLS do banco foi desabilitado neste laboratório porque a conexão permanece no servidor: ambos os processos são executados na mesma máquina.

A tela seguinte define o nome da instância e o fuso horário padrão. Defina o fuso local real para que os horários de eventos e gráficos correspondam à realidade.

Ao concluir, o assistente grava as configurações de conexão em:

```text
/etc/zabbix/web/zabbix.conf.php
```

Esse arquivo permite ao frontend lembrar os parâmetros do banco. Consultá-lo com acesso autorizado após a instalação permite verificar quais credenciais são utilizadas pelo frontend; seu conteúdo deve permanecer protegido.

### Etapa 11: primeiro login e hardening imediato

Credenciais padrão de uma instância Zabbix recém-importada:

```text
Username: Admin
Password: zabbix
```

O nome de usuário tem A maiúsculo. A forma minúscula não autentica.

Essa senha é de conhecimento público porque é idêntica em novas instalações padrão do Zabbix. Alterá-la não é opcional. Navegue até Users, Users, selecione Admin e altere a senha para um valor armazenado em um gerenciador de senhas.

Se houver repetidas falhas de login, o Zabbix pode bloquear temporariamente a conta. Aguardar o período de bloqueio antes de tentar novamente permite retomar a autenticação.

---

## 6. Adição do segundo host monitorado

Nesse momento, o servidor monitora apenas a si mesmo. Um sistema de monitoramento observando uma única máquina ensina pouco sobre operação distribuída. O segundo host é a estação de administração.

### Etapa 12: instalar e configurar o agente na estação

```bash
sudo apt install zabbix-agent -y
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Defina os seguintes parâmetros:

```text
Server=<SERVER_IP>
ServerActive=<SERVER_IP>
Hostname=<HOST_NAME>
ListenIP=0.0.0.0
```

Significados dos campos:

| Parâmetro | Significado |
| --- | --- |
| `Server` | Endereços autorizados a consultar o agente (verificações passivas) |
| `ServerActive` | Endereço para o qual o agente envia dados (verificações ativas) |
| `Hostname` | Nome pelo qual o agente se identifica; deve corresponder exatamente ao frontend para verificações ativas |
| `ListenIP` | Endereço de rede ao qual o agente se vincula |

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
systemctl status zabbix-agent --no-pager
```

Verificação de que o agente está escutando em todas as interfaces IPv4:

```bash
ss -tulpn | grep 10050
```

A saída esperada neste laboratório é `0.0.0.0:10050`. Se mostrar apenas `127.0.0.1:10050`, o agente está restrito ao loopback e o servidor remoto não consegue alcançá-lo por essa porta.

### Etapa 13: registrar o host no frontend

Navegue até Data collection, Hosts e Create host.

| Campo | Valor |
| --- | --- |
| Nome do host | `<HOSTNAME>` (deve corresponder ao `Hostname` do agente para verificações ativas) |
| Templates | Linux by Zabbix agent |
| Grupos de hosts | Linux servers |
| Interface | Agent, `<WORKSTATION_IP>`, porta 10050 |
| Descrição | Texto livre identificando a função da máquina e a rede |

O campo de template é a parte poderosa. Um template é um conjunto predefinido de itens, gráficos e triggers. Associar `Linux by Zabbix agent` disponibiliza dezenas de métricas de CPU, memória, disco e rede sem definição manual. Em produção, templates permitem escalar o monitoramento consistente para centenas de hosts.

O valor `Hostname` na configuração do agente e o nome do host no frontend são comparados literalmente nas verificações ativas. Uma diferença de um caractere pode impedir o processamento desses dados.

### Etapa 14: verificar a comunicação servidor-agente

Instale o utilitário de teste no servidor:

```bash
sudo apt install zabbix-get -y
zabbix_get -s <WORKSTATION_IP> -k system.hostname
```

O retorno de um hostname demonstra que o utilitário consegue consultar o agente pela rede, com firewall e configuração do agente permitindo a consulta. Isso ajuda a separar problemas de rede de problemas de configuração do servidor Zabbix.

No frontend, a coluna Availability do novo host apresenta o indicador ZBX verde após as verificações bem-sucedidas, observado dentro de um ou dois minutos neste laboratório.

Os valores coletados são visíveis em Monitoring, Hosts e Latest data. Gráficos prontos estão em Monitoring, Hosts e Graphs.

---

## 7. Retenção de dados

Por padrão, o Zabbix armazena o histórico coletado por um período longo. Em um disco de 32 GB, isso pode eventualmente causar um incidente de armazenamento. A retenção é configurada em Administration, Housekeeping.

| Configuração | Valor recomendado | Motivo |
| --- | --- | --- |
| Sobrescrita da retenção de histórico | Habilitada, 7d | Valores brutos coletados ocupam mais espaço |
| Sobrescrita da retenção de tendências | Habilitada, 30d | Agregados horários são compactos e ainda alimentam gráficos de longo prazo |

Histórico e tendências são dados diferentes. O histórico contém cada medição individual. Tendências são agregados horários com médias, mínimos e máximos calculados pelo Zabbix para métricas numéricas. Reduzir o histórico mantém a visão agregada no período retido pelas tendências, mas perde o detalhe das medições brutas descartadas.

---

## 8. Observação de alertas reais

Imediatamente após a instalação, apareceu um problema de nível de aviso no host do servidor Zabbix:

```text
Linux: Number of installed packages has been changed
```

Essa trigger compara o número de pacotes instalados entre avaliações. Ela ajuda a detectar alterações de software potencialmente não autorizadas nos sistemas monitorados. Neste laboratório, disparou porque dezenas de pacotes foram instalados minutos antes, o que era esperado e inofensivo.

Esse evento demonstra vários conceitos:

| Conceito | Observação |
| --- | --- |
| Níveis de severidade | A cor de aviso reflete a severidade registrada, de caráter informativo em vez de crítico |
| Finalidade da trigger | Triggers codificam expectativas operacionais, como o software permanecer inalterado |
| Reconhecimento | Registrar o motivo ao reconhecer ou fechar um evento documenta por que ocorreu, prática padrão de operações |

Em ambientes de produção, avisos de baixa severidade podem acumular ruído. Administradores ajustam ou desabilitam triggers irrelevantes para manter alertas significativos visíveis. A fadiga de alertas é um problema operacional real.

---

## 9. Resumo do diagnóstico

Cada falha encontrada durante este laboratório, sua causa e resolução.

| Problema | Sintoma | Causa | Resolução |
| --- | --- | --- | --- |
| Pacotes Zabbix não encontrados | `apt install` falha para `zabbix-apache-conf` e `zabbix-sql-scripts` | `apt update` não foi executado após instalar o pacote de repositório | Executar `sudo apt update` e repetir a instalação |
| Falha na importação do esquema | `server.sql.gz: No such file or directory` | Caminho do esquema mudou em versões mais recentes do Zabbix | Localizar o arquivo com `dpkg -L zabbix-sql-scripts \| grep server.sql.gz` |
| Login do frontend rejeitado | Mensagem de usuário ou senha incorretos | Confusão entre senha do banco e senha web inicial | O login web inicial padrão é Admin com senha zabbix, independente das credenciais do banco |
| Confusão na localização da tela de hosts | Dificuldade em localizar a tela de criação de host na interface traduzida | Rótulos dos menus variam entre traduções | Acessada por Data collection, Hosts e Create host |
| Ferramenta de teste ausente | `zabbix_get: command not found` | Utilitário distribuído em pacote separado | `sudo apt install zabbix-get` |
| Teste de conexão do banco impossível | `curl: command not found` | Instalação mínima não contém curl | `sudo apt install curl` |
| zabbix_server não inicia | Serviço inativo | Possível credencial incorreta do banco na configuração do servidor | Verificar `journalctl -u zabbix-server` e o log do servidor, conferir DBPassword |

A lição recorrente: quando um pacote ou arquivo não está onde a documentação indica, consulte o gerenciador de pacotes para localizar seus arquivos. A documentação de pacotes envelhece; os dados do pacote instalado ajudam a verificar a situação real.

---

## 10. Lista de verificação

* [x] MariaDB instalado e em execução
* [x] Banco Zabbix criado com utf8mb4 e collation binária
* [x] Usuário dedicado do banco com privilégios limitados ao banco Zabbix
* [x] Repositório oficial Zabbix instalado para Debian 13
* [x] Servidor, frontend, agente e scripts SQL instalados
* [x] Esquema inicial importado, contagem de tabelas verificada
* [x] Variável de criação de funções em tempo de execução restaurada após importação
* [x] Configuração do servidor editada com credenciais do banco e caches reduzidos
* [x] Serviço Zabbix Server habilitado e em execução
* [x] Apache servindo o frontend em /zabbix
* [x] Assistente web concluído com parâmetros do banco, nome e fuso horário
* [x] Senha padrão Admin alterada e armazenada em um gerenciador de senhas
* [x] Host local coletando dados com ZBX verde
* [x] Agente instalado e configurado na estação de administração
* [x] Comunicação servidor-agente verificada com zabbix_get
* [x] Segundo host registrado com template e ZBX verde
* [x] Housekeeping configurado para histórico de 7 dias e tendências de 30 dias
* [x] Trigger de nível de aviso observada e interpretada

---

## 11. Notas sobre recursos

O servidor executa a pilha completa dentro de 2 GB de RAM por meio de reduções deliberadas:

| Medida | Efeito |
| --- | --- |
| Sem ambiente de desktop | Economiza centenas de megabytes de RAM |
| `CacheSize=64M` | Limita o cache de configuração do Zabbix |
| `HistoryCacheSize=16M` | Limita o buffer de histórico em memória |
| Arquivo swap de 1 GB | Margem adicional quando todas as camadas atingem picos simultâneos |
| Períodos curtos de retenção | Mantém o banco pequeno em um disco de 32 GB |

Monitorar o consumo de recursos do próprio servidor de monitoramento, utilizando seus dados, fecha o ciclo: o sistema observa a máquina que o executa.

O cuidado físico importa tanto quanto a configuração. Na época deste registro, a máquina operava 24 horas por dia conectada à tomada, com uma bateria envelhecida. Inspeção física periódica e atenção ao risco de inchaço da bateria fazem parte da operação desse hardware como servidor.

---

## 12. Principais aprendizados

Uma pilha de monitoramento é uma aplicação web de três camadas, e instalá-la é um exercício completo de administração de serviços: banco de dados, servidor de aplicação e interface devem ser configurados e verificados de maneira independente antes que o conjunto funcione.

Dois domínios de credenciais existem nesse tipo de sistema. O usuário do banco e o usuário da aplicação são independentes, e tratá-los como iguais produz confusão evitável durante a configuração.

O índice de pacotes do apt é atualizado por repositório. Adicionar um repositório sem atualizar o índice produz erros de pacote não encontrado que parecem indicar pacotes ausentes, mas podem ser apenas metadados desatualizados.

Documentação de software envelhece. Os caminhos de arquivos deste laboratório diferiam de guias antigos amplamente copiados. Localizar caminhos reais com ferramentas de pacotes é mais confiável do que confiar cegamente em tutoriais.

Um sistema de monitoramento não está pronto apenas porque sua interface abre. É importante demonstrar que consegue detectar falhas. O teste de parada e início do agente, os indicadores de disponibilidade e a observação de alertas são o que transforma um programa instalado em uma capacidade operacional de monitoramento.

---

## 13. Próximas etapas

Continuações planejadas deste laboratório:

```text
Dashboard construction for presentation
        |
Notification channels (email on trigger)
        |
Custom trigger authoring
        |
SNMP based monitoring of network devices
        |
Database backup with mysqldump and cron
        |
Host inventory and automation
```

O SNMP merece destaque. Equipamentos de rede como switches, roteadores, pontos de acesso e impressoras geralmente não podem executar um agente Zabbix. SNMP é a família de protocolos para monitorá-los, o que o torna essencial para a administração de redes. Um laboratório futuro habilitaria o daemon SNMP neste servidor e o utilizaria pelo Zabbix com o template Linux by SNMP, praticando configuração de comunidade, porta 161 e coleta de dados por OID.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
