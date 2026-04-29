# 🚀 Ambiente Odoo com Observabilidade e Backup

## 📌 Visão Geral

Este projeto implementa um ambiente distribuído com foco em **disponibilidade, monitoramento e recuperação de dados**.

A arquitetura é composta por três servidores:

* **Server 1 — Aplicação**: Odoo + PostgreSQL + Nginx
* **Server 2 — Observabilidade**: Prometheus + Grafana + Loki + Exporters
* **Server 3 — Backup**: Proxmox Backup Server (PBS)

Objetivos principais:

* Visibilidade completa do ambiente (métricas e logs)
* Monitoramento ativo (health checks)
* Backup automatizado com validação
* Recuperação confiável

---

## 🏗️ Arquitetura

![Arquitetura](assets/architecture.png)

---

## 🖥️ Estrutura dos Servidores

### 🔹 Server 1 — Aplicação (`srvodoo-prod01`)

**Stack:**

* Nginx (reverse proxy)
* Odoo 17 (Docker)
* PostgreSQL 15 (Docker)

**Coleta de dados:**

* Node Exporter (métricas do host)
* cAdvisor (métricas de containers)
* Promtail (logs)

---

### 🔹 Server 2 — Observabilidade (`srvobs-prod01`)

**Stack:**

* Prometheus (métricas)
* Grafana (dashboards e alertas)
* Loki (logs)
* Blackbox Exporter (health checks HTTP)

**Fluxos:**

* Métricas → Prometheus → Grafana
* Logs → Loki → Grafana
* HTTP probes → Prometheus

---

### 🔹 Server 3 — Backup (`srvbkp-prod01`)

**Stack:**

* Proxmox Backup Server (PBS)
* Armazenamento ZFS

**Funções:**

* Backup de VMs
* Backup de dados críticos
* Deduplicação e compressão

---

## 💾 Pipeline de Backup

Rotina automatizada:

| Horário | Etapa                             |
| ------- | --------------------------------- |
| 02:00   | Dump do PostgreSQL (`pg_dump`)    |
| 02:05   | Coleta do filestore               |
| 02:20   | Restore de teste (`test_restore`) |
| 02:30   | Envio para PBS                    |
| 02:30+  | Geração de métricas               |

---

## 🔁 Validação de Backup

Após o backup:

* O banco é restaurado automaticamente em `test_restore`
* Validação garante integridade dos dados
* Resultado pode ser usado para monitoramento

Exemplo:

```
backup_success 1
restore_success 1
```

---

## 📊 Monitoramento do Backup

O processo de backup gera métricas como:

* Status de execução
* Tempo de duração
* Sucesso ou falha

Essas informações podem ser coletadas pelo Prometheus e visualizadas no Grafana.


## ⚙️ Scripts Principais

### Backup

* Dump do banco
* Coleta do filestore
* Envio para PBS

### Restore Test

* Restauração automatizada
* Validação do banco

### Métricas

* Geração de arquivos `.prom`
* Integração com Prometheus

---

## 🔐 Segurança

* Autenticação via token no PBS
* Isolamento por rede Docker
* Separação de funções por servidor

---

## 📈 Evoluções Planejadas

* Alertas no Grafana (falha de backup)
* Políticas de retenção (GFS)
* Automação adicional
* Hardening dos servidores

---

## 🎯 Objetivo

Construir um ambiente confiável, monitorado e com capacidade de recuperação rápida, simulando cenários reais de operação.

---

## 📬 Contato

Sugestões e melhorias podem ser feitas via issues ou pull requests.
