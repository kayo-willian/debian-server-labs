# Lab 10 - Remoção do Grafana e otimização de recursos

[English](../../../../docs/labs/10-grafana-removal-and-resource-optimization/README.md) · [Índice dos laboratórios](../../README.md) · [Laboratório anterior](../09-dashboard-de-monitoramento-grafana/README.md) · [Próximo laboratório](../11-migracao-do-servidor-zabbix-para-maquina-virtual/README.md)

## Visão geral

Este laboratório documenta a remoção do Grafana do ambiente de laboratório Debian Server após concluir a integração Grafana e Zabbix documentada no [Lab 09](../09-dashboard-de-monitoramento-grafana/README.md).

O dashboard Grafana foi criado com sucesso e utilizado para visualizar os dados de monitoramento coletados pelo Zabbix.

Após concluir os objetivos práticos do laboratório Grafana, decidiu-se remover o Grafana do servidor Debian físico.

A decisão baseou-se principalmente nos recursos limitados de hardware disponíveis no laboratório e na necessidade de preservá-los para futuros projetos.

---

## Por que o Grafana foi removido

O servidor Debian utilizado neste laboratório funciona com hardware limitado.

Os recursos disponíveis são restritos em:

* RAM
* Capacidade de processamento da CPU
* Armazenamento em disco

O servidor é utilizado como ambiente de laboratório prático no qual vários serviços e projetos futuros serão testados.

Manter todos os serviços instalados permanentemente consumiria gradualmente armazenamento e recursos do sistema, mesmo quando esses serviços já não estivessem em uso ativo.

Por isso, o Grafana foi removido após concluir seus objetivos de aprendizado.

Isso não foi resultado de um problema com o Grafana ou com a integração Zabbix.

A integração funcionou com sucesso e forneceu o dashboard de monitoramento esperado.

A remoção foi uma decisão de gerenciamento de recursos.

---

## Limitações de hardware

Executar múltiplos serviços de infraestrutura em uma máquina com poucos recursos exige gerenciamento cuidadoso dos recursos disponíveis.

O servidor de laboratório já contém serviços necessários ao ambiente de monitoramento, incluindo:

* Zabbix Server
* Zabbix Agent
* MariaDB
* Apache2
* Zabbix Web

Adicionar o Grafana introduziu outra aplicação e suas dependências associadas.

Embora o Grafana tenha sido útil durante o laboratório de dashboard, mantê-lo permanentemente na mesma máquina não era necessário após a conclusão dos objetivos práticos.

Removê-lo reduz a quantidade de software instalado permanentemente no servidor e preserva recursos para outros experimentos.

---

## Priorização dos laboratórios futuros

O principal motivo para remover o Grafana é preparar o ambiente para os projetos seguintes.

Este servidor não foi concebido como plataforma de monitoramento de produção. É um ambiente de aprendizado que mudará continuamente à medida que novas tecnologias e serviços forem testados.

Portanto, armazenamento e recursos do sistema precisam ser tratados como recursos limitados do laboratório.

Em vez de manter software instalado simplesmente por ter sido utilizado anteriormente, os componentes de experimentos concluídos podem ser removidos quando deixarem de ser necessários.

Isso abre espaço para:

* Novos serviços
* Novos experimentos Linux
* Novos projetos de redes
* Novas configurações de servidor
* Futuros experimentos de monitoramento
* Laboratórios adicionais de infraestrutura

O objetivo é manter o ambiente suficientemente flexível para suportar as próximas etapas do laboratório.

---

## O Grafana continua fazendo parte do aprendizado

Remover o Grafana do servidor físico não significa perder o laboratório Grafana.

O laboratório anterior contém a documentação do dashboard e a configuração exportada.

A configuração do dashboard também pode ser reutilizada em outro ambiente se o Grafana for instalado novamente no futuro.

Uma máquina virtual separada também pode ser utilizada para futuros experimentos Grafana, permitindo atribuir recursos adicionais de RAM e disco sem afetar o servidor Debian principal.

Isso oferece um ambiente mais adequado para continuar os estudos de Grafana enquanto o servidor físico permanece focado em seus serviços principais de laboratório.

---

## Processo de remoção

A remoção foi realizada por meio do sistema de gerenciamento de pacotes Debian.

Primeiramente, o serviço Grafana foi interrompido e desabilitado.

```bash
sudo systemctl stop grafana-server
sudo systemctl disable grafana-server
```

Em seguida, o pacote Grafana instalado foi removido:

```bash
sudo apt remove grafana
```

Para remover arquivos de configuração associados ao pacote:

```bash
sudo apt purge grafana
```

As dependências não utilizadas foram então removidas:

```bash
sudo apt autoremove
```

O cache de pacotes também poderia ser limpo para recuperar espaço adicional em disco:

```bash
sudo apt clean
```

---

## Verificação

Após a remoção, o sistema foi verificado para confirmar que o Grafana não estava mais em execução.

O estado do serviço pode ser verificado com:

```bash
systemctl status grafana-server
```

A porta do Grafana também pode ser verificada:

```bash
ss -tulpn | grep :3000
```

Se o Grafana tiver sido removido com sucesso e nenhuma outra aplicação estiver utilizando a porta, não deverá haver serviço Grafana escutando na porta TCP `3000`.

Também é possível pesquisar os pacotes instalados com:

```bash
dpkg -l | grep grafana
```

Essas verificações ajudam a confirmar que o serviço e o pacote foram removidos do ambiente.

---

## Gerenciamento de recursos

Este laboratório demonstra que a administração de sistemas não envolve apenas instalar e configurar software.

Também envolve decidir quais serviços devem permanecer instalados e quais podem ser removidos quando deixam de ser necessários.

Em um laboratório com recursos limitados, espaço em disco, memória e capacidade de processamento são recursos valiosos.

Remover componentes de laboratórios concluídos ajuda a manter um ambiente mais limpo e permite continuar construindo novos projetos sem acumular desnecessariamente software e dependências.

---

## Resultado

O Grafana foi removido com sucesso do ambiente Debian Server após concluir o laboratório de dashboard Grafana.

O ambiente de monitoramento Zabbix permaneceu disponível para as necessidades de monitoramento do servidor.

Os recursos anteriormente alocados ao Grafana puderam ser utilizados por futuros laboratórios e experimentos.

A configuração Grafana e a documentação do [Lab 09](../09-dashboard-de-monitoramento-grafana/README.md) permanecem disponíveis como registro da integração concluída.

---

## O que este laboratório demonstrou

* Gerenciamento de serviços Linux
* `systemctl`
* Gerenciamento de pacotes Debian
* Remoção de pacotes
* Remoção de pacotes e arquivos de configuração
* Limpeza de dependências
* Gerenciamento de espaço em disco
* Gerenciamento de recursos
* Manutenção de infraestrutura
* Gerenciamento de ambiente de laboratório
* Planejamento de projetos futuros

---

## Conclusão

O Grafana cumpriu sua finalidade no laboratório proporcionando experiência prática com integração Zabbix e dashboards de monitoramento.

Após concluir essa etapa, manter o Grafana permanentemente no servidor físico já não era necessário.

Como o hardware do laboratório possui RAM, capacidade de processamento e armazenamento limitados, os recursos disponíveis são mais bem reservados para futuros experimentos.

A remoção do Grafana, portanto, representa a transição de uma etapa concluída do laboratório para a próxima.

O ambiente de laboratório permanece focado, leve e preparado para os novos projetos que se seguirão.

[Índice dos laboratórios](../../README.md) · [Visão geral do repositório](../../../README.md)
