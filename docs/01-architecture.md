# Arquitetura da Infraestrutura

## 1. Visão geral

Este laboratório simula a sustentação de uma aplicação corporativa Odoo em infraestrutura Linux, com separação funcional entre aplicação, monitoramento e backup.

A arquitetura utiliza três servidores:

| Servidor | Função principal | Componentes |
|---|---|---|
| `srvodoo-prod01` | Aplicação e banco de dados | Nginx, Odoo 17, PostgreSQL 15, Node Exporter, cAdvisor e Promtail |
| `srvobs-prod01` | Monitoramento e centralização de logs | Prometheus, Grafana, Loki e Blackbox Exporter |
| `srvbkp-prod01` | Backup e recuperação | Proxmox Backup Server e datastore ZFS |

A separação reduz o acoplamento operacional e facilita diagnóstico, manutenção e recuperação. Ela **não representa alta disponibilidade automática**, pois não há cluster, replicação ativa ou failover documentado.

## 2. Objetivos da arquitetura

- Separar aplicação, monitoramento e backup em servidores distintos
- Centralizar métricas e logs para apoiar o troubleshooting
- Proteger banco de dados, filestore e configurações
- Automatizar rotinas de backup e validação de restauração
- Facilitar a documentação e a recuperação do ambiente
- Simular atividades de administração e sustentação de infraestrutura

## 3. Diagrama lógico

```mermaid
flowchart LR
    U[Usuário] -->|HTTP :80| N[Nginx]
    N --> O[Odoo 17]
    O --> P[(PostgreSQL 15)]

    A[Node Exporter] --> PR[Prometheus]
    C[cAdvisor] --> PR
    B[Blackbox Exporter] --> PR
    PR --> G[Grafana]

    L1[Logs do host e serviços] --> PT[Promtail]
    PT --> LK[Loki]
    LK --> G

    P --> BK[Rotina de backup]
    O -->|Filestore| BK
    CFG[/opt/erp-spark] --> BK
    BK --> PBS[Proxmox Backup Server]
```

## 4. Servidor de aplicação — `srvodoo-prod01`

### 4.1 Responsabilidades

- Publicar o Odoo por meio do Nginx
- Executar os contêineres do Odoo e PostgreSQL
- Manter a persistência do banco e do filestore
- Gerar backups lógicos do PostgreSQL
- Enviar banco, filestore e configurações ao PBS
- Exportar métricas e encaminhar logs

### 4.2 Componentes

| Componente | Função |
|---|---|
| Nginx | Reverse proxy e ponto de entrada HTTP |
| Odoo 17 | Aplicação ERP |
| PostgreSQL 15 | Banco de dados da aplicação |
| Node Exporter | Métricas do sistema operacional |
| cAdvisor | Métricas dos contêineres |
| Promtail | Encaminhamento de logs ao Loki |

### 4.3 Dados críticos

| Item | Localização lógica | Observação |
|---|---|---|
| Banco de dados | PostgreSQL, banco `odoo` | Protegido por dump lógico |
| Filestore | `/var/lib/odoo/filestore` dentro do volume do Odoo | Contém anexos e assets |
| Configuração | `/opt/erp-spark` | Contém arquivos operacionais do ambiente |
| Backups locais | `/opt/erp-spark/backup/data` | Armazena dumps compactados |

O caminho real do volume do Odoo no host pode variar. A rotina de backup o identifica dinamicamente com `docker inspect`.

## 5. Servidor de monitoramento — `srvobs-prod01`

### 5.1 Responsabilidades

- Coletar métricas de hosts, contêineres e serviços
- Verificar disponibilidade HTTP da aplicação
- Centralizar logs
- Disponibilizar dashboards operacionais
- Apoiar a identificação de falhas de backup, restauração e disponibilidade

### 5.2 Componentes

| Componente | Função |
|---|---|
| Prometheus | Coleta e armazenamento de métricas |
| Grafana | Dashboards e visualização operacional |
| Loki | Armazenamento centralizado de logs |
| Blackbox Exporter | Testes de disponibilidade HTTP |

## 6. Servidor de backup — `srvbkp-prod01`

### 6.1 Responsabilidades

- Receber backups do banco, filestore e configurações
- Armazenar os dados em datastore ZFS
- Aplicar compressão e deduplicação disponibilizadas pelo PBS
- Permitir recuperação dos dados protegidos
- Apoiar testes periódicos de restauração

### 6.2 Conjuntos protegidos

- `db.pxar`: diretório com o dump do PostgreSQL
- `filestore.pxar`: filestore da aplicação
- `config.pxar`: diretório `/opt/erp-spark`
- Backups de máquinas virtuais, quando configurados no Proxmox VE

## 7. Fluxos operacionais

### 7.1 Acesso à aplicação

1. O usuário acessa o endereço publicado pelo Nginx.
2. O Nginx encaminha a requisição para o contêiner do Odoo.
3. O Odoo consulta e grava dados no PostgreSQL.
4. Arquivos e anexos são armazenados no filestore.

### 7.2 Monitoramento e logs

1. Node Exporter e cAdvisor expõem métricas.
2. Prometheus coleta essas métricas.
3. Blackbox Exporter verifica o endpoint HTTP.
4. Promtail encaminha logs ao Loki.
5. Grafana centraliza a visualização.

### 7.3 Backup e recuperação

1. É gerado um dump lógico do PostgreSQL.
2. O filestore e a configuração são incluídos na proteção.
3. Os dados são enviados ao Proxmox Backup Server.
4. O backup lógico mais recente é restaurado em banco isolado.
5. O resultado da validação é registrado em logs e métricas.

## 8. Dependências e pontos de atenção

- O banco e o filestore devem pertencer ao mesmo ponto lógico de recuperação.
- Restaurar somente o PostgreSQL pode deixar anexos e assets indisponíveis.
- O servidor de monitoramento não deve ser a única fonte de evidência dos backups.
- Credenciais e tokens não devem ser armazenados no repositório.
- A política de retenção do PBS deve ser configurada e documentada.
- A arquitetura atual depende de servidores individuais e não possui failover automático.

## 9. Controles de segurança recomendados

- Armazenar segredos fora do Git e utilizar arquivo `.env.example`
- Utilizar token dedicado e com menor privilégio no PBS
- Restringir portas entre servidores por firewall
- Aplicar atualizações periódicas no Linux, Docker e componentes
- Publicar a aplicação por HTTPS
- Limitar acesso administrativo por rede confiável ou VPN
- Revisar permissões dos diretórios de backup e configuração
- Registrar tentativas de acesso e alterações administrativas

## 10. Escopo do laboratório

O projeto demonstra administração de servidores Linux, contêineres, monitoramento, backup, restauração, troubleshooting e documentação operacional. Não representa, por si só, uma arquitetura de produção com alta disponibilidade, redundância geográfica ou continuidade de negócio completa.
