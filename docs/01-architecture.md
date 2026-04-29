# Arquitetura do Ambiente

## Visão Geral

Ambiente distribuído em 3 servidores com separação de responsabilidades:

- Server 1 (srvodoo-prod01): Aplicação (Odoo + PostgreSQL + Nginx)
- Server 2 (srvobs-prod01): Observabilidade (Prometheus, Grafana, Loki)
- Server 3 (srvbkp-prod01): Backup (Proxmox Backup Server)

---

## Componentes

### Server 1 - Aplicação

- Odoo 17 (Docker)
- PostgreSQL 15 (Docker)
- Nginx (Reverse Proxy)
- Node Exporter
- cAdvisor
- Promtail

Diretórios críticos:

---

### Server 2 - Observabilidade

- Prometheus
- Grafana
- Loki
- Blackbox Exporter

---

### Server 3 - Backup

- Proxmox Backup Server (PBS)
- Datastore ZFS

---

## Fluxo de Dados

1. Usuário acessa via Nginx
2. Odoo consome PostgreSQL
3. Logs enviados via Promtail → Loki
4. Métricas coletadas por Prometheus
5. Backups enviados para PBS

---

## Objetivo da Arquitetura

- Separação de responsabilidades
- Alta confiabilidade de backup
- Observabilidade centralizada
- Facilidade de recuperação (Disaster Recovery)
