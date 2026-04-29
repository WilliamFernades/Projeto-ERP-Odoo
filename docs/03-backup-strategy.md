# Estratégia de Backup

## Objetivo

Garantir recuperação completa da aplicação em caso de falha, corrupção ou perda de dados.

---

## Problema

Backup apenas do banco de dados NÃO é suficiente.

O Odoo depende de:

- Banco de dados (PostgreSQL)
- Filestore (arquivos físicos)

Sem o filestore:
- Assets (CSS/JS) quebram
- Uploads são perdidos
- Interface não carrega corretamente

---

## Dados Críticos

1. Banco de dados (dump PostgreSQL)
2. Filestore (/var/lib/odoo/filestore)
3. Configuração (/opt/erp-spark)

---

## Estratégia Implementada

Backup em duas camadas:

### 1. Backup lógico

- pg_dump (formato custom)
- Permite restore granular

---

### 2. Backup via PBS

Envio para Proxmox Backup Server:

- db (dump)
- filestore (pxar)
- config (pxar)

---

## Fluxo de Backup

02:00 → dump PostgreSQL  
02:05 → coleta filestore  
02:10 → envio para PBS  
02:30 → teste de restore automático  

---

## Tecnologias

- proxmox-backup-client
- ZFS (deduplicação)
- compressão automática

---

## Benefícios

- Backup incremental eficiente
- Redução de armazenamento
- Recuperação rápida
- Validação automática de integridade
