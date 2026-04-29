# Setup do Odoo

## Stack

- Odoo 17
- PostgreSQL 15
- Docker / Docker Compose
- Nginx como reverse proxy

---

## Estrutura

Aplicação organizada em:

/opt/erp-spark/

- docker-compose.yml
- nginx/
- promtail/
- backup/

---

## Serviços

### PostgreSQL

- Banco principal: odoo

---

### Odoo

- Conectado ao PostgreSQL via rede Docker

---

### Nginx

- Porta 80
- Reverse proxy para Odoo
- Endpoint de status para monitoramento

---

## Observações

- Toda aplicação roda em containers
- Persistência via volumes Docker
- Configuração centralizada em /opt/erp-spark
