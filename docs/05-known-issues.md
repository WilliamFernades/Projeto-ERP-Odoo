# Troubleshooting e Problemas Conhecidos

Este documento registra incidentes observados no laboratório, suas causas, correções e medidas preventivas.

## 1. Odoo sem CSS/JavaScript após migração

### Status

Resolvido.

### Sintomas

- Interface exibida sem formatação
- Falha no carregamento de CSS e JavaScript
- Erros HTTP 500 ao solicitar assets
- Aplicação parcialmente inacessível pelo navegador

### Impacto

- Usuários não conseguem utilizar a interface normalmente
- Recursos visuais e scripts deixam de carregar
- O ambiente pode parecer disponível no nível de processo, mas permanece indisponível funcionalmente

### Evidência observada

```text
FileNotFoundError: /var/lib/odoo/filestore/...
```

### Contexto

O incidente ocorreu após a migração das máquinas virtuais entre ambientes de virtualização.

### Causa raiz

O banco de dados foi mantido, mas o filestore correspondente não estava presente no novo ambiente.

O Odoo mantém referências aos arquivos no PostgreSQL, enquanto o conteúdo físico permanece no filestore. A restauração isolada do banco gerou inconsistência entre metadados e arquivos.

### Diagnóstico

1. Confirmar que os contêineres estão em execução:

```bash
docker compose ps
```

2. Consultar os logs do Odoo:

```bash
docker compose logs --tail=200 odoo
```

3. Identificar o volume montado em `/var/lib/odoo`:

```bash
docker inspect -f '{{ range .Mounts }}{{ if eq .Destination "/var/lib/odoo" }}{{ .Source }}{{ end }}{{ end }}' odoo
```

4. Confirmar a existência do filestore:

```bash
find /CAMINHO_DO_VOLUME/filestore -maxdepth 2 -type d
```

5. Comparar o nome do banco com o diretório correspondente no filestore.

### Correção aplicada

- Recriação controlada do ambiente Docker
- Restauração do banco de dados
- Restauração do filestore correspondente
- Validação do volume montado no contêiner
- Reinicialização dos serviços
- Teste do carregamento da interface e dos assets

### Validação

- Página carregada com CSS
- JavaScript carregado sem erro
- Ausência de `FileNotFoundError` nos logs
- Anexos acessíveis
- Aplicação funcional pelo Nginx

### Ação preventiva

- Proteger banco e filestore no mesmo ciclo de backup
- Registrar a correspondência entre banco e filestore
- Validar restauração antes de migrações
- Não excluir volumes sem confirmar o backup
- Incluir filestore no runbook de recuperação
- Testar assets e anexos após qualquer restauração

### Lição aprendida

O backup do PostgreSQL isolado não é suficiente para recuperar integralmente o Odoo.

---

## 2. Falha ao excluir banco de teste por conexões ativas

### Status

Resolvido.

### Sintoma

Erro durante a validação de restauração:

```text
database is being accessed by other users
```

### Impacto

- O banco `test_restore` não pode ser excluído
- A rotina de restauração é interrompida
- A métrica de validação não é atualizada com sucesso

### Causa raiz

Existiam sessões ativas conectadas ao banco de teste, possivelmente abertas por execução anterior ou por conexão indevida da aplicação.

### Diagnóstico

Listar sessões conectadas:

```sql
SELECT pid, usename, application_name, client_addr, state
FROM pg_stat_activity
WHERE datname = 'test_restore';
```

### Correção aplicada

Encerrar conexões antes da exclusão:

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'test_restore'
  AND pid <> pg_backend_pid();
```

Depois:

1. Executar `DROP DATABASE IF EXISTS test_restore`.
2. Recriar o banco.
3. Restaurar o dump.
4. Consultar `res_users`.
5. Registrar métricas e logs.

### Validação

- Banco excluído e recriado
- Dump restaurado
- Consulta concluída
- `restore_success 1` gravado no arquivo de métricas

### Ação preventiva

- Não configurar o Odoo para utilizar o banco de teste
- Encerrar conexões de forma automática
- Registrar as sessões encontradas
- Utilizar nome exclusivo para o banco temporário
- Remover o banco após o teste, quando apropriado

---

## 3. Padrão para novos incidentes

Utilize a estrutura abaixo:

```markdown
## Título do incidente

### Status
Aberto, em análise ou resolvido.

### Sintomas
O que foi observado.

### Impacto
Serviços, usuários ou dados afetados.

### Evidências
Logs, mensagens de erro e métricas.

### Diagnóstico
Etapas utilizadas para investigar.

### Causa raiz
Motivo confirmado do incidente.

### Correção aplicada
Mudanças realizadas.

### Validação
Como a recuperação foi comprovada.

### Ação preventiva
Medidas para evitar recorrência.

### Lição aprendida
Conclusão técnica e operacional.
```

## 4. Boas práticas de troubleshooting

- Registrar horário e sequência dos eventos
- Preservar logs antes de reiniciar serviços
- Alterar uma variável por vez
- Validar dependências: banco, filestore, rede, proxy e volumes
- Diferenciar disponibilidade de processo de disponibilidade funcional
- Documentar a causa confirmada, não apenas a solução aplicada
- Atualizar procedimentos de backup e restauração após incidentes
