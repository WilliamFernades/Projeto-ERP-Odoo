# Problemas Conhecidos

## Erro: Odoo sem CSS/JS (HTTP 500)

### Sintoma

- Interface quebrada
- Erros 500 ao carregar assets

---

### Causa

Filestore ausente após migração das VM de um ambiente de virtualização para outro.

Erro observado:

FileNotFoundError: /var/lib/odoo/filestore/...

---

### Impacto

- Sistema inacessível visualmente
- Assets não carregam

---

### Solução

- Recriar ambiente Docker
- Restaurar:
  - Banco de dados
  - Filestore

---

### Lições Aprendidas

- Backup do banco isolado é insuficiente
- Filestore é crítico
- Backup deve ser completo (DB + arquivos)
