# Projeto de Migração de Servidores CentOS 7 (HostGator) para AlmaLinux

## 1. Visão Geral do Projeto

**Objetivo:** Migrar servidores CentOS 7 hospedados na HostGator para AlmaLinux, garantindo a funcionalidade e o suporte do cPanel, com o mínimo de tempo de inatividade.

- **Servidores Envolvidos:** server01.oxyalbr.com, server02.yebin.com.br
- **Ferramentas Principais:** cPanel/WHM (backup/restauração), SSH, rsync).
- **Ambiente de Destino:** AlmaLinux (versão mais recente estável, atualmente AlmaLinux 9.x)

## 2. Riscos e Mitigações

| Risco                           | Mitigação                                                                 |
|----------------------------------|---------------------------------------------------------------------------|
| Perda de dados                   | Backups completos e verificados antes da migração.                        |
| Tempo de inatividade prolongado  | Planejamento detalhado, testes em ambiente de homologação, migração em horários de baixo tráfego. |
| Problemas de compatibilidade     | Testes rigorosos dos aplicativos após a migração.                         |
| Falha na restauração do cPanel   | Verificação da integridade dos backups do cPanel.                         |
| Configurações de rede incorretas | Anotação das configurações de rede antigas e validação das novas.         |
| Problemas de DNS                 | Atualização cuidadosa dos registros DNS e atenção ao TTL.                 |
| Suporte HostGator                | Verificar as opções de VPS ou Dedicated Servers que oferecem AlmaLinux e discutir o processo com o suporte deles. |

---

## 3. Fases do Projeto e Tarefas Detalhadas

### Fase 3: Migração de Dados

**Objetivo:** Transferir os dados e configurar os novos servidores.

#### Tarefa 3.1: Transferir os Backups para os Novos Servidores

Utilizar `rsync` ou `scp` para transferir os arquivos de backup do cPanel e quaisquer outros backups manuais para o novo servidor.

Exemplo:
```bash
rsync -avzP /path/to/local/backups user@new_server_ip:/home/backups/
```

#### Tarefa 3.2: Restauração das Contas do cPanel

- Via WHM: "Restore a Full Backup/cpmove File".
- Selecionar os arquivos de backup e iniciar a restauração para cada conta.
- Monitorar o processo de restauração para erros.

#### Tarefa 3.3: Migração de Configurações Personalizadas

- Reaplicar configurações personalizadas de `httpd.conf`, `php.ini`, `my.cnf`, etc.
- Configurar os módulos PHP necessários que podem não ter sido instalados por padrão.
- Reconfigurar certificados SSL para os domínios (pode ser automático com o AutoSSL do cPanel após a restauração ou manual).
- Reconfigurar quaisquer serviços adicionais (nginx como proxy reverso, etc.).

#### Tarefa 3.4: Testes Iniciais nos Novos Servidores (via arquivo hosts)

Modificar o arquivo hosts na sua máquina local (`/etc/hosts` no Linux, `C:\Windows\System32\drivers\etc\hosts` no Windows) para apontar os domínios para o IP do novo servidor.

Acessar os sites e verificar a funcionalidade:
- Navegação geral.
- Funcionalidade de formulários, uploads.
- Conexão com banco de dados.
- Envio/recebimento de e-mails.
- Funcionalidade de scripts cron.
- Verificar logs de erro (Apache, PHP, MySQL).

---

### Fase 4: Cutover e Finalização

**Objetivo:** Direcionar o tráfego para os novos servidores e finalizar a migração.

#### Tarefa 4.1: Ajuste Fino e Otimização (Opcional, mas recomendado)

- Ajustar configurações de PHP (memória, tempo de execução).
- Otimizar MySQL/MariaDB (buffer pool size, etc.).
- Instalar e configurar cache (Redis, Memcached) se necessário e utilizado anteriormente.

#### Tarefa 4.2: Atualização dos Registros DNS

- Importante: Definir um TTL baixo (e.g., 300 segundos ou 5 minutos) para os registros DNS algumas horas antes da migração. Isso minimiza o tempo de propagação após a mudança.
- Alterar os registros A (e CNAME, MX, etc., se aplicável) dos domínios para apontar para o IP dos novos servidores AlmaLinux no provedor de DNS (HostGator ou registrador de domínio).

#### Tarefa 4.3: Monitoramento Pós-Cutover

- Monitorar o tráfego e os logs nos novos servidores após a mudança de DNS.
- Verificar se há erros ou problemas de desempenho.
- Manter os servidores antigos online por um período de contingência (pelo menos 24-48 horas) caso seja necessário reverter.

#### Tarefa 4.4: Descomissionamento dos Servidores Antigos (Apenas após confirmação de sucesso)

- Após um período de estabilidade e confiança nos novos servidores (ex: 1 semana), você pode considerar o cancelamento ou desligamento dos servidores CentOS 7 antigos na HostGator.
- Certifique-se de ter um último backup dos servidores antigos antes de desativá-los completamente.

---

## 4. Cronograma Detalhado

Este é um cronograma estimado e pode variar por varios fatores, imprevistos na migração, trobleshooting de erros, complexidade das aplicações e experiência dos participantes.

| Fase / Semana                       | Dias      | Tarefas Principais                                                                 |
|--------------------------------------|-----------|------------------------------------------------------------------------------------|
|Planejamento e Preparação             | Semana 1  | Reunião de kickoff, inventário inicial, avaliação HostGator.                       |
|                                      |           | Detalhamento do inventário, análise de riscos.                                     |
|                                      | Semana 2  | Definição da estratégia, revisão de recursos.                                      |
|                                      |           | Preparação de ambiente.                                   |
|                                      | Semana 3  | Documentação de credenciais, planejamento de backups.                              |
|Backup e Provisionamento              |           | Realização de backups completos dos servidores.                                    |
|                                      |           | Transferência de backups para local seguro, verificação.                           |
|                                      | Semana 4  | Provisionamento de novos servidores AlmaLinux.                       |
|                                      |           | Configuração inicial dos novos servidores (updates, hostname, firewall).           |
|                                      |           | Instalação de pacotes essenciais, verificação cPanel.                              |
| Semana 3: Migração de Dados          | Semana 5  | Transferência de backups para os novos servidores.                                 |
|                                      |           | Restauração de contas cPanel (Servidor 1).                                        |
|                                      |           | Restauração de contas cPanel (Servidor 2).                             |
|                                      | Semana 6  | Migração de configurações personalizadas.                                          |
|                                      |           | Testes iniciais via arquivo hosts (Servidor 2).                                   |
|Cutover e Finalização                 | Semana 7  | Testes iniciais via arquivo hosts (Servidor 1).                        |
|                                      |           | Ajuste fino e otimização (se necessário).                                         |
|                                      | Semana 8  | Preparação para Cutover: Redução do TTL de DNS.                                   |
|                                      | Em diante | Dia do Cutover: Atualização de registros DNS, monitoramento intensivo.            |
|                                      |           | Monitoramento contínuo, validação final.                                          |
| Pós-Migração                         |           | Monitoramento da estabilidade, resolução de possíveis problemas.                   |
|                                      |           | Descomissionamento gradual dos servidores antigos (após 1 semana de estabilidade). |

---

## 5. Ao que se Atentar Durante a Migração

- **Comunicação com a HostGator:** Manter o contato com o suporte da HostGator para entender as opções de migração e o que eles podem oferecer. Eles podem ter ferramentas ou recomendações específicas para o ambiente deles.
- **Backups:** Nunca é demais enfatizar. Fazer backups completos e verificados. Tenha cópias em locais diferentes.
- **Tempo de Inatividade:** Planejar a migração para horários de menor tráfego para minimizar o impacto nos usuários. Comunique-se com os usuários ou clientes sobre a possível interrupção.
- **Teste, Teste, Teste:** Testar exaustivamente os sites e aplicativos nos novos servidores antes de mudar o DNS.
- **Versões de Software:** Verifique as versões de PHP, MySQL/MariaDB, Apache, etc., e suas configurações. O AlmaLinux 9.x vem com versões mais recentes de software (PHP 8.x, MySQL 8.x ou MariaDB 10.x), o que pode exigir que seus aplicativos sejam compatíveis. Se necessário, use o seletor de PHP do cPanel para definir as versões adequadas.
- **Permissões e Propriedade de Arquivos:** Após a restauração, verificar as permissões e a propriedade dos arquivos e diretórios, especialmente para sites e aplicações.
- **Certificados SSL:** Garanta que os certificados SSL sejam migrados ou reemitidos corretamente. O AutoSSL do cPanel deve facilitar isso.
- **Crons:** Verificar se todos os jobs cron foram migrados e estão funcionando corretamente.
- **Registros DNS:** A atenção ao TTL é crucial. Um TTL baixo permite que a mudança de DNS se propague mais rapidamente.
- **Monitoramento:** Use ferramentas de monitoramento para acompanhar o desempenho e a saúde dos novos servidores após a migração.
- **Paciência:** Migrações de servidor podem ser complexas. Mantenha a calma, siga o plano e resolva os problemas um de cada vez.
- **Rollback Plan:** Os servidores antigos se manteram funcionais por um tempo após o cutover como plano de contingência, caso aconteça algum problema.

---