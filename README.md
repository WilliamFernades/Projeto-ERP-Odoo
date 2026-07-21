# Infraestrutura Odoo com Monitoramento e Backup

Laboratório de infraestrutura desenvolvido para implantação e sustentação de um ambiente Odoo em servidores Linux.

O projeto contempla publicação da aplicação, banco de dados, monitoramento centralizado, coleta de logs, backup no Proxmox Backup Server e validação automatizada de restauração.

## Objetivos

- Implantar uma aplicação corporativa em servidores Linux
- Separar aplicação, monitoramento e backup
- Monitorar disponibilidade, utilização de recursos e serviços
- Centralizar logs para apoio ao troubleshooting
- Automatizar backups do banco de dados e arquivos
- Validar periodicamente a recuperação das informações
- Documentar a arquitetura e os procedimentos operacionais

## Arquitetura

O ambiente é composto por três servidores:

| Servidor | Função |
|---|---|
| `srvodoo-prod01` | Aplicação Odoo, PostgreSQL e Nginx |
| `srvobs-prod01` | Monitoramento, logs e verificação de disponibilidade |
| `srvbkp-prod01` | Backup e armazenamento dos dados |

## Servidor de Aplicação

### Componentes

- Nginx como reverse proxy
- Odoo 17
- PostgreSQL 15
- Node Exporter para métricas do servidor
- cAdvisor para acompanhamento dos contêineres
- Promtail para envio de logs

### Responsabilidades

- Publicação da aplicação
- Armazenamento dos dados do Odoo
- Gerenciamento do filestore
- Coleta de métricas do servidor
- Envio de logs para o servidor de monitoramento

## Servidor de Monitoramento

### Componentes

- Prometheus
- Grafana
- Loki
- Blackbox Exporter

### Funções

- Monitoramento de CPU, memória, disco e rede
- Verificação de disponibilidade da aplicação
- Centralização de logs
- Criação de dashboards
- Acompanhamento das rotinas de backup
- Identificação de falhas e indisponibilidades

## Servidor de Backup

### Componentes

- Proxmox Backup Server
- Armazenamento ZFS
- Scripts de backup e restauração

### Funções

- Backup das máquinas virtuais
- Armazenamento dos dumps do PostgreSQL
- Proteção do filestore do Odoo
- Deduplicação e compressão
- Retenção e recuperação dos dados

## Rotina de Backup

| Horário | Atividade |
|---|---|
| 02:00 | Execução do dump do PostgreSQL |
| 02:05 | Cópia do filestore |
| 02:20 | Restauração automática em banco de teste |
| 02:30 | Envio dos dados para o Proxmox Backup Server |
| Após execução | Registro do resultado e geração de métricas |

## Validação de Restauração

Após a execução do backup, o banco de dados é restaurado automaticamente em uma instância de teste.

Essa etapa permite verificar:

- Integridade do arquivo de backup
- Funcionamento do processo de restauração
- Disponibilidade dos dados protegidos
- Tempo necessário para recuperação
- Sucesso ou falha da rotina

Exemplo de resultado:

```text
backup_success 1
restore_success 1
