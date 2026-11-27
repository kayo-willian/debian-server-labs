# Lab 03: Firewall e controle de acesso com UFW

[English](../../../../docs/labs/03-firewall-access-control/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../02-servidor-de-monitoramento-zabbix/README.md) · [Próximo laboratório](../04-servidor-web-nginx/README.md)

Laboratório prático focado na proteção de um servidor Debian físico com um firewall de host, construindo uma política explícita de controle de acesso e verificando a proteção a partir de vários clientes da rede.

Este laboratório dá continuidade diretamente ao [Lab 02](../02-servidor-de-monitoramento-zabbix/README.md). A pilha de monitoramento Zabbix implantada anteriormente serve como objeto de teste: o firewall é configurado para proteger sua interface web, preservando os caminhos legítimos de comunicação do sistema de monitoramento.

O objetivo é passar de "o serviço funciona" para "o serviço funciona e apenas as partes autorizadas podem acessá-lo".

## Objetivo

Os objetivos deste laboratório foram:

* Compreender o que é um firewall de host e como difere da segurança de perímetro de rede
* Aprender o modelo de negação por padrão de um firewall e como construir uma lista de permissões
* Instalar e configurar UFW no Debian Server
* Restringir acesso SSH e HTTP a endereços de origem confiáveis específicos
* Preservar a comunicação necessária aos serviços e bloquear as demais conexões de entrada
* Validar a política a partir de dispositivos permitidos e bloqueados
* Compreender a relação entre endereçamento DHCP e regras de firewall baseadas em IP

---

## 1. Por que um firewall de host?

Anteriormente, o servidor aceitava conexões de dispositivos na sub-rede de laboratório. Qualquer dispositivo conectado à rede sem fio do laboratório podia abrir a interface web do Zabbix simplesmente inserindo o endereço do servidor em um navegador. A página de login de uma interface administrativa não deve estar acessível a todos os dispositivos da rede.

Existem dois níveis complementares de proteção:

| Nível | Mecanismo | Escopo |
| --- | --- | --- |
| Filtragem de perímetro | Regras no roteador separando sub-redes | Controla o que atravessa as redes |
| Firewall de host | Filtra tráfego no próprio servidor | Controla as conexões que chegam à máquina |

O projeto de rede do [Lab 01](../01-instalacao-configuracao-ssh/README.md) já separa a sub-rede de laboratório da rede doméstica. No entanto, dispositivos conectados à rede sem fio do laboratório ainda podiam alcançar os serviços que escutavam na rede. O firewall de host fecha essa lacuna aplicando a política diretamente na máquina.

O UFW (Uncomplicated Firewall) é uma interface para o subsistema de filtragem de pacotes do kernel. Foi escolhido porque expressa regras de filtragem em comandos legíveis, gerenciando automaticamente a complexidade subjacente.

---

## 2. O modelo de controle de acesso

Neste laboratório, o UFW foi configurado com negação por padrão para novas conexões de entrada: conexões que não correspondem a uma regra de permissão são descartadas, respeitadas as regras existentes e o acompanhamento de conexões. Não é necessário criar uma regra de bloqueio para cada origem. O bloqueio é o estado padrão, e o administrador constrói uma lista explícita de comunicações permitidas.

Esse modelo é chamado de lista de permissões e reflete como o controle de acesso é projetado em ambientes de produção: indicar exatamente quem pode conectar-se, com qual finalidade, e negar implicitamente o restante.

A política projetada para este servidor:

| Origem | Porta de destino | Protocolo | Finalidade |
| --- | --- | --- | --- |
| Estação de administração | 22 | TCP | Administração SSH |
| Estação de administração | 80 | TCP | Frontend web Zabbix |
| Dispositivo administrativo secundário | 22 | TCP | Administração SSH pelo terminal móvel |
| Dispositivo administrativo secundário | 80 | TCP | Visualização do frontend Zabbix |
| Qualquer host monitorado | 10051 | TCP | Agentes enviando dados coletados |
| Demais origens | Nenhuma | | Negação implícita |

Dois componentes exigiram exceções e dois deliberadamente não receberam permissões amplas:

* O SSH deve permanecer acessível porque é o canal administrativo do servidor headless
* A porta TCP 10051 deve aceitar conexões dos agentes que enviam dados ao servidor
* O banco não exige regra de entrada porque o MariaDB escuta apenas no loopback e permanece na máquina
* O frontend foi restringido por endereço de origem em vez de aberto à sub-rede inteira

---

## 3. Instalação

```bash
sudo apt install ufw -y
```

As regras são criadas enquanto o firewall ainda está inativo. Essa ordem é crítica, como explicado na seção de diagnóstico: ativar um firewall com negação por padrão antes de adicionar a regra SSH pode impedir novos acessos administrativos.

---

## 4. Construção do conjunto de regras

Cada regra concede a um endereço de origem específico acesso a uma porta de destino específica, exceto quando a origem é deliberadamente irrestrita.

### Permitir SSH apenas a partir da estação de administração

```bash
sudo ufw allow from <WORKSTATION_IP> to any port 22 proto tcp comment 'SSH from administration workstation'
```

Sem essa regra, ativar o firewall poderia impedir o acesso administrativo remoto; não se deve depender da continuidade da sessão existente.

### Restringir o frontend web

O frontend Zabbix escuta na porta 80. Abertura somente para a estação de administração e o dispositivo administrativo secundário:

```bash
sudo ufw allow from <WORKSTATION_IP> to any port 80 proto tcp comment 'Zabbix frontend from workstation'
sudo ufw allow from <SECONDARY_DEVICE_IP> to any port 80 proto tcp comment 'Zabbix frontend from secondary admin device'
```

Os demais endereços da rede deixam de alcançar a interface web pela política configurada.

### Permitir tráfego dos agentes

Agentes em modo ativo enviam dados coletados ao servidor pela porta TCP 10051:

```bash
sudo ufw allow 10051/tcp comment 'Zabbix agent data'
```

Essa regra não restringe a origem porque os agentes podem ser executados em vários hosts. Assim, a porta está aberta também a origens não monitoradas que consigam alcançá-la.

### Ativar o firewall

```bash
sudo ufw enable
sudo ufw status numbered
```

`status numbered` exibe o conjunto de regras com índices, utilizados para referenciá-las ao remover ou inspecionar regras.

---

## 5. Regras por escopo

As regras UFW podem conceder acesso com diferentes granularidades:

```bash
# Everything from one address
sudo ufw allow from <IP>

# One port from one address
sudo ufw allow from <IP> to any port <PORT> proto tcp

# One port from an entire subnet
sudo ufw allow from 192.168.2.0/24 to any port <PORT> proto tcp
```

Uma regra para toda a sub-rede concede acesso a cada dispositivo naquele intervalo e contraria a finalidade de uma restrição por host. O valor desse projeto está na concessão restrita de acesso: por endereço, porta e finalidade.

---

## 6. Verificação

A validação foi realizada a partir de três origens distintas:

| Teste | Origem | Expectativa | Resultado |
| --- | --- | --- | --- |
| Frontend a partir da estação | `http://<SERVER_IP>/zabbix/` | Carrega normalmente | Confirmado |
| Frontend a partir do dispositivo secundário | Mesmo endereço | Carrega normalmente | Confirmado |
| Frontend a partir de dispositivo não listado | Mesmo endereço | Conexão expira, página não carrega | Confirmado |
| SSH a partir de origens permitidas | Porta 22 | Sessão estabelecida | Confirmado |
| SSH a partir de dispositivo não listado | Porta 22 | Conexão bloqueada | Confirmado |
| Fluxo de dados dos agentes | Indicador ZBX | Permanece verde | Confirmado |

O teste do dispositivo móvel é a demonstração mais clara: antes da regra, a página não termina de carregar, pois o firewall descarta silenciosamente os pacotes; após adicionar a regra, o dashboard carrega normalmente.

### Verificação da política ativa

```bash
sudo ufw status numbered
```

A saída lista cada regra com seu número de posição. As regras podem ser removidas pelo número com:

```bash
sudo ufw delete <NUMBER>
```

Também podem ser removidas repetindo-as com `delete`, e o campo de comentário ajuda a documentar cada regra nas listagens.

---

## 7. Defesa em profundidade além do firewall

O firewall controla quais endereços podem abrir uma conexão. Duas camadas adicionais complementam o modelo de proteção.

### Qualidade da autenticação

Regras de firewall concedem alcançabilidade de rede, não autorização. Quem passa pelo firewall ainda precisa de credenciais válidas. Senhas fortes armazenadas em um gerenciador de senhas e, preferencialmente, autenticação por chaves em vez de senhas são controles que protegem o acesso a uma porta SSH aberta.

### Proteção contra força bruta

Um firewall controla quem pode conectar-se. Um serviço complementar chamado fail2ban monitora logs de autenticação e bloqueia temporariamente origens que falham repetidamente na autenticação. Ele acrescenta uma camada dinâmica sobre as regras estáticas de firewall e foi planejado como laboratório separado.

### Um princípio fundamental

Defesa em profundidade: nenhum controle isolado é considerado suficiente. O firewall restringe a alcançabilidade, a autenticação forte restringe o acesso e o monitoramento de logs detecta abusos. As camadas ajudam a cobrir falhas umas das outras.

---

## 8. Limitações encontradas

Duas restrições influenciaram essa configuração e merecem registro.

### O roteador não suporta reservas DHCP na sub-rede de laboratório

A sub-rede de laboratório é fornecida por um recurso de sub-rede SSID cujo firmware disponibiliza associações DHCP estáticas apenas para a LAN principal. A atribuição de reserva ao endereço do servidor foi, portanto, rejeitada pela interface do roteador com um erro de validação de sub-rede.

Consequências:

* Os endereços do servidor e da estação permanecem atribuídos dinamicamente
* As regras de firewall referenciam endereços de origem específicos; se o DHCP reatribuir o endereço de uma estação, seu acesso deixa de funcionar até a atualização da regra
* A probabilidade de mudança foi considerada baixa, mas não nula, e essa compensação foi documentada conscientemente

Um esquema alternativo de endereçamento foi considerado e intencionalmente adiado: configurar um endereço estático no servidor fora do pool DHCP. O procedimento foi analisado, incluindo uma sequência de migração com adição de endereço secundário antes de remover o DHCP, mas foi adiado para manter o ambiente estável.

### Regras baseadas em IP dependem de endereçamento estável

Toda a política depende da previsibilidade dos endereços. Este laboratório documentou explicitamente essa limitação em vez de ocultá-la. Em ambientes maiores, as soluções usuais são reservas DHCP, infraestrutura com endereços estáticos ou ambas.

---

## 9. Resumo do diagnóstico

| Situação | Detecção | Resolução |
| --- | --- | --- |
| Risco de perder acesso antes da ativação | Primeira ativação do firewall sem regra SSH | Criar a permissão SSH antes de `ufw enable` |
| Dispositivo legítimo bloqueado | Página não carrega no celular confiável | Adicionar regra de permissão específica para seu endereço |
| Regra sem efeito | Revisão com `ufw status numbered` | Verificar ordem das regras e correção do endereço de origem |

Lição de projeto: altere a configuração do firewall em uma ordem segura. Primeiro regras permissivas necessárias, ativação por último e verificação imediatamente depois.

---

## 10. Lista de verificação

* [x] UFW instalado no servidor Debian
* [x] Regra SSH restrita ao endereço da estação de administração
* [x] Regra SSH restrita ao dispositivo administrativo secundário
* [x] Regra HTTP restrita ao endereço da estação de administração
* [x] Regra HTTP restrita ao dispositivo administrativo secundário
* [x] Porta de dados dos agentes aberta para conexões de entrada
* [x] Firewall habilitado com política de negação de entrada por padrão
* [x] Frontend carrega em ambos os dispositivos autorizados
* [x] Frontend inacessível a dispositivos não autorizados
* [x] Sessões SSH dos dispositivos autorizados não afetadas
* [x] Disponibilidade dos agentes observada pela continuidade dos indicadores verdes

---

## 11. Principais aprendizados

Um firewall de host transforma um servidor aberto em um servidor com política explícita de acesso. Antes do firewall, dispositivos na rede de laboratório podiam alcançar os serviços que escutavam na rede. Depois do firewall, a alcançabilidade tornou-se uma decisão deliberada registrada como regra.

A negação por padrão é a propriedade que torna isso possível. Sem ela, novos serviços que escutassem na rede poderiam ficar acessíveis sem uma decisão explícita. Com ela, disponibilizar algo novo exige uma decisão explícita e revisada.

Firewall e autenticação resolvem problemas diferentes. O firewall decide se um pacote pode chegar ao serviço; a autenticação decide se o solicitante pode utilizá-lo. As camadas são complementares, não redundantes.

Regras de alcançabilidade de rede são tão estáveis quanto o endereçamento que referenciam. Endereços dinâmicos criam uma dependência permanente entre comportamento DHCP e correção do firewall, uma compensação operacional real que vale compreender em vez de ignorar.

---

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
