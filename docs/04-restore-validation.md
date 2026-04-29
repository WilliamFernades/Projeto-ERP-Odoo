# Validação de Restore

## Objetivo

Garantir que os backups são válidos e utilizáveis.

---

## Estratégia

Após cada backup:

- Executar restore automático
- Restaurar banco em ambiente isolado

---

## Banco de Teste

test_restore

---

## Processo

1. Criar banco test_restore
2. Restaurar dump
3. Validar integridade

---

## Problema Encontrado

Erro ao restaurar:

"database is being accessed by other users"

Causa:
- Conexão ativa do Odoo no banco de teste

---

## Solução

- Remover banco test_restore
- Garantir que Odoo não se conecta nele
- Recriar e restaurar

---

## Resultado

- Restore validado com sucesso
- Backup confiável

---

## Importância

Backup sem teste de restore = risco operacional alto
