# Projeto de Migração de Servidores CentOS 7 (HostGator) para AlmaLinux

## 1. Visão Geral do Projeto

**Objetivo:** Migrar servidores CentOS 7 hospedados na HostGator para AlmaLinux, garantindo a funcionalidade e o suporte do cPanel, com o mínimo de tempo de inatividade.

- **Servidores Envolvidos:** [Listar aqui a quantidade e, se possível, os nomes/funções dos servidores]
- **Ferramentas Principais:** cPanel/WHM (backup/restauração), SSH, rsync, ferramentas de virtualização (se aplicável, para snapshots/clones).
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

### Fase 1: Planejamento e Preparação (Semana 1)

**Objetivo:** Coletar informações, planejar a estratégia e preparar o ambiente.

#### Tarefa 1.1: Inventário Completo dos Servidores Atuais

Listar todos os servidores a serem migrados.

Para cada servidor:
- Nome do servidor / IP.
- Função (web, banco de dados, e-mail, etc.).
- Lista de domínios hospedados.
- Versão do CentOS 7.
- Versão do cPanel/WHM.
- Tamanho do disco utilizado.
- Uso de RAM e CPU.
- Lista de softwares/serviços adicionais instalados (Apache, Nginx, PHP, MySQL/MariaDB, Postfix, Dovecot, etc.).
- Versões de PHP e MySQL/MariaDB utilizadas pelos sites.
- Configurações personalizadas (mod_security, httpd.conf, php.ini, my.cnf, etc.).
- Scripts cron agendados.
- Certificados SSL (anotar paths e nomes).
- Contas de e-mail e tamanhos de caixas postais.

#### Tarefa 1.2: Avaliação da Infraestrutura na HostGator

- Confirmar se a HostGator oferece ou permite a instalação de AlmaLinux em novas instâncias (VPS ou Dedicado).
- Verificar os planos disponíveis que atendam aos requisitos de hardware dos novos servidores AlmaLinux.
- Discutir com o suporte da HostGator o processo de "upgrade" ou migração, se eles oferecem alguma ferramenta ou serviço para isso.

#### Tarefa 1.3: Definição da Estratégia de Migração

- **Opção 1: Criação de Nova Instância:** A mais recomendada. Provisionar um novo servidor com AlmaLinux e cPanel na HostGator e migrar os dados para ele. Isso minimiza o risco de quebrar o ambiente atual.
- **Opção 2: In-place (Se possível e suportado pela HostGator):** Tentar um upgrade direto do CentOS 7 para AlmaLinux. Altamente não recomendado, especialmente com cPanel, devido à complexidade e alto risco de falha. Não consideraremos essa opção como padrão.

#### Tarefa 1.4: Preparação de Ambiente de Testes (Se aplicável)

Se for um ambiente crítico, considerar criar um ambiente de homologação idêntico para testar o processo de backup/restauração antes da migração em produção. Pode ser uma VM local ou um servidor temporário na HostGator.

#### Tarefa 1.5: Documentação de Credenciais

Organizar todas as credenciais de acesso (root, cPanel/WHM, FTP, bancos de dados) em um local seguro.

---

### Fase 2: Backup e Provisionamento (Semana 2)

**Objetivo:** Realizar backups completos e provisionar os novos servidores AlmaLinux.

#### Tarefa 2.1: Realizar Backups Completos dos Servidores Atuais

**Backup do cPanel/WHM:**
- Via WHM: "Backup Configuration" -> "Backup All Accounts". Gerar um backup completo para cada conta.
- Baixar os arquivos de backup para um local seguro (máquina local, armazenamento externo).

**Backup Adicional (Manual, se necessário):**
- rsync de diretórios críticos: `/etc`, `/var/www`, `/var/lib/mysql` (se não estiverem inclusos no backup do cPanel), `/home`.
- Exportar bancos de dados MySQL/MariaDB via `mysqldump`.
- Baixar todos os arquivos de configuração personalizados que foram identificados na Fase 1.

**Verificação de Integridade dos Backups:**
- Descompactar alguns arquivos de backup para garantir que não estão corrompidos.
- Verificar os tamanhos dos arquivos de backup.

#### Tarefa 2.2: Contratar Novos Servidores na HostGator com AlmaLinux

- Provisionar novas instâncias (VPS ou Dedicado) com AlmaLinux (preferencialmente AlmaLinux 9) e cPanel/WHM pré-instalado (se disponível).
- Certificar-se de que as especificações de hardware (CPU, RAM, Disco) são iguais ou superiores às dos servidores atuais.

#### Tarefa 2.3: Configuração Inicial dos Novos Servidores AlmaLinux

- Acessar os novos servidores via SSH como root.
- Atualizar o sistema: `dnf update -y`
- Definir um hostname apropriado.
- Configurar fusos horários.
- Instalar pacotes essenciais que não vêm por padrão, mas que eram utilizados no CentOS 7 (ex: git, htop, firewalld, etc.).
- Configurar firewalld com as regras necessárias (portas 22, 80, 443, 2087, 2083, etc.).
- Garantir que o cPanel esteja atualizado: `/usr/local/cpanel/scripts/upcp --force`

---

### Fase 3: Migração de Dados (Semana 3)

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

Modificar o arquivo hosts na sua máquina local (`/etc/hosts` no Linux/macOS, `C:\Windows\System32\drivers\etc\hosts` no Windows) para apontar os domínios para o IP do novo servidor.

Acessar os sites e verificar a funcionalidade:
- Navegação geral.
- Funcionalidade de formulários, uploads.
- Conexão com banco de dados.
- Envio/recebimento de e-mails.
- Funcionalidade de scripts cron.
- Verificar logs de erro (Apache, PHP, MySQL).

---

### Fase 4: Cutover e Finalização (Semana 4)

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

Este é um cronograma estimado e pode variar dependendo da quantidade de servidores, complexidade das aplicações e sua experiência.

| Fase / Semana                       | Dias      | Tarefas Principais                                                                 |
|--------------------------------------|-----------|------------------------------------------------------------------------------------|
| Semana 1: Planejamento e Preparação  | Dia 1     | Reunião de kickoff, inventário inicial, avaliação HostGator.                       |
|                                      | Dia 2     | Detalhamento do inventário, análise de riscos.                                     |
|                                      | Dia 3     | Definição da estratégia, revisão de recursos.                                      |
|                                      | Dia 4     | Preparação de ambiente de testes (se aplicável).                                   |
|                                      | Dia 5     | Documentação de credenciais, planejamento de backups.                              |
| Semana 2: Backup e Provisionamento   | Dia 6     | Realização de backups completos dos servidores.                                    |
|                                      | Dia 7     | Transferência de backups para local seguro, verificação.                           |
|                                      | Dia 8     | Contratação e provisionamento de novos servidores AlmaLinux.                       |
|                                      | Dia 9     | Configuração inicial dos novos servidores (updates, hostname, firewall).           |
|                                      | Dia 10    | Instalação de pacotes essenciais, verificação cPanel.                              |
| Semana 3: Migração de Dados          | Dia 11    | Transferência de backups para os novos servidores.                                 |
|                                      | Dia 12    | Restauração de contas cPanel (Servidor 1).                                        |
|                                      | Dia 13    | Restauração de contas cPanel (Servidor 2, se houver).                             |
|                                      | Dia 14    | Migração de configurações personalizadas.                                          |
|                                      | Dia 15    | Testes iniciais via arquivo hosts (Servidor 1).                                   |
| Semana 4: Cutover e Finalização      | Dia 16    | Testes iniciais via arquivo hosts (Servidor 2, se houver).                        |
|                                      | Dia 17    | Ajuste fino e otimização (se necessário).                                         |
|                                      | Dia 18    | Preparação para Cutover: Redução do TTL de DNS.                                   |
|                                      | Dia 19    | Dia do Cutover: Atualização de registros DNS, monitoramento intensivo.            |
|                                      | Dia 20    | Monitoramento contínuo, validação final.                                          |
| Pós-Migração                         | Dias 21-25| Monitoramento da estabilidade, resolução de possíveis problemas.                   |
|                                      | Dia 26    | Descomissionamento gradual dos servidores antigos (após 1 semana de estabilidade). |

---

## 5. Ao que se Atentar Durante a Migração

- **Comunicação com a HostGator:** Mantenha contato com o suporte da HostGator para entender as opções de migração e o que eles podem oferecer. Eles podem ter ferramentas ou recomendações específicas para o ambiente deles.
- **Backups:** Nunca é demais enfatizar. Faça backups completos e verificados. Tenha cópias em locais diferentes.
- **Tempo de Inatividade:** Planeje a migração para horários de menor tráfego para minimizar o impacto nos usuários. Comunique-se com os usuários ou clientes sobre a possível interrupção.
- **Teste, Teste, Teste:** Teste exaustivamente os sites e aplicativos nos novos servidores antes de mudar o DNS.
- **Versões de Software:** Verifique as versões de PHP, MySQL/MariaDB, Apache, etc., e suas configurações. O AlmaLinux 9.x vem com versões mais recentes de software (PHP 8.x, MySQL 8.x ou MariaDB 10.x), o que pode exigir que seus aplicativos sejam compatíveis. Se necessário, use o seletor de PHP do cPanel para definir as versões adequadas.
- **Permissões e Propriedade de Arquivos:** Após a restauração, verifique as permissões e a propriedade dos arquivos e diretórios, especialmente para sites e aplicações.
- **Certificados SSL:** Garanta que os certificados SSL sejam migrados ou reemitidos corretamente. O AutoSSL do cPanel deve facilitar isso.
- **Crons:** Verifique se todos os jobs cron foram migrados e estão funcionando corretamente.
- **Registros DNS:** A atenção ao TTL é crucial. Um TTL baixo permite que a mudança de DNS se propague mais rapidamente.
- **Monitoramento:** Use ferramentas de monitoramento para acompanhar o desempenho e a saúde dos novos servidores após a migração.
- **Paciência:** Migrações de servidor podem ser complexas. Mantenha a calma, siga o plano e resolva os problemas um de cada vez.
- **Rollback Plan:** Sempre tenha um plano de rollback caso algo dê muito errado. Isso geralmente significa manter os servidores antigos online e funcionais por um tempo após o cutover.

---

Este projeto detalhado deve fornecer um guia robusto para a sua migração. Se precisar de mais detalhes em alguma etapa ou tiver outras dúvidas, me diga!