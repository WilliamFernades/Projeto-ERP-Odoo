# Validação de Restauração

## 1. Objetivo

Confirmar que o backup lógico do PostgreSQL pode ser restaurado e consultado antes que seja necessário utilizá-lo em uma recuperação real.

A existência de um arquivo de backup não comprova que ele é recuperável. A validação reduz o risco de descobrir corrupção, incompatibilidade ou falha operacional apenas durante um incidente.

## 2. Escopo atual

A rotina `Scripts/restore_test` valida:

- Existência de um dump `.sql.gz`
- Restauração em banco isolado
- Abertura e consulta do banco restaurado
- Presença de registros na tabela `res_users`
- Tempo de execução
- Geração de métrica de sucesso ou falha

A rotina atual **valida somente o banco de dados**. Ela não comprova, por si só:

- Integridade do filestore
- Correspondência entre banco e filestore
- Inicialização completa do Odoo
- Carregamento de anexos e assets
- Funcionamento do Nginx
- Recuperação integral do servidor

## 3. Banco de validação

Banco utilizado:

```text
test_restore
```

Esse banco deve permanecer isolado da aplicação principal e não deve ser selecionado pelo Odoo.

## 4. Arquivos e registros

Diretório de backups:

```text
/opt/erp-spark/backup/data
```

Log da validação:

```text
/opt/erp-spark/backup/data/restore.log
```

Arquivo de métricas:

```text
/opt/erp-spark/backup/restore.prom
```

## 5. Processo executado

1. Identificar o arquivo `.sql.gz` mais recente.
2. Encerrar conexões ativas no banco `test_restore`.
3. Excluir o banco de teste, caso exista.
4. Criar novamente o banco `test_restore`.
5. Descompactar o dump.
6. Restaurar o conteúdo no banco de teste.
7. Consultar a tabela `res_users`.
8. Registrar o total de linhas retornado.
9. Calcular a duração da operação.
10. Gravar o resultado em log e métricas.

## 6. Execução manual

Antes da execução, confirme que existe ao menos um arquivo:

```bash
ls -lh /opt/erp-spark/backup/data/*.sql.gz
```

Execute:

```bash
chmod +x Scripts/restore_test
sudo Scripts/restore_test
```

Consulte o resultado:

```bash
tail -n 100 /opt/erp-spark/backup/data/restore.log
cat /opt/erp-spark/backup/restore.prom
```

## 7. Critérios de sucesso

A validação é considerada aprovada quando:

- Existe um arquivo de backup válido
- O banco de teste é criado
- O dump é restaurado sem erro
- A consulta a `res_users` retorna valor maior que zero
- A métrica `restore_success` recebe valor `1`
- O log registra a conclusão da rotina

Exemplo:

```text
restore_success 1
restore_duration_seconds 42
restore_rows_res_users 8
```

Os valores acima são apenas exemplos de formato. Os valores reais dependem do ambiente.

## 8. Métricas

### `restore_success`

```text
1 = restauração e validação concluídas
0 = falha durante o processo
```

### `restore_duration_seconds`

Tempo total utilizado para recriar o banco, restaurar o dump e executar a validação.

### `restore_rows_res_users`

Quantidade de registros encontrada na tabela `res_users`.

A presença de usuários comprova que o banco foi restaurado e consultado, mas não substitui uma validação funcional completa.

## 9. Problema encontrado: conexões ativas

### Sintoma

Erro durante a exclusão ou recriação do banco:

```text
database is being accessed by other users
```

### Causa

Existiam sessões ativas conectadas ao banco `test_restore`.

### Correção aplicada

A rotina encerra as sessões do banco antes de executar `DROP DATABASE`:

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'test_restore'
  AND pid <> pg_backend_pid();
```

Depois disso, o processo:

1. Exclui o banco de teste.
2. Recria o banco.
3. Executa a restauração.
4. Realiza a consulta de validação.

### Prevenção

- Não apontar o Odoo para `test_restore`
- Manter o banco de teste fora da lista de bancos utilizados pela aplicação
- Encerrar conexões antes de excluir o banco
- Registrar falhas e sessões ativas no log

## 10. Falhas tratadas

### Nenhum backup encontrado

Resultado:

```text
restore_success 0
```

Ação:

- Verificar se a rotina de backup foi executada
- Confirmar o diretório configurado
- Revisar permissões e padrão do nome do arquivo

### Falha no restore

Ação:

- Validar se o arquivo está íntegro
- Consultar o log do PostgreSQL
- Confirmar versão e formato do dump
- Verificar espaço em disco
- Confirmar usuário e permissões

### Consulta retorna zero

Ação:

- Verificar se o dump pertence ao banco esperado
- Consultar outras tabelas críticas
- Comparar com o banco principal
- Não considerar o teste aprovado

## 11. Evolução necessária: validação integral

Para validar a recuperação completa do Odoo, o processo deve evoluir para:

1. Restaurar o banco em ambiente isolado.
2. Restaurar uma cópia correspondente do filestore.
3. Subir uma instância temporária do Odoo.
4. Validar autenticação.
5. Abrir registros com anexos.
6. Confirmar carregamento de CSS e JavaScript.
7. Testar o endpoint por meio do Nginx.
8. Registrar duração e resultado.
9. Remover o ambiente temporário.

## 12. Registro de evidências

Para cada teste, registre:

| Campo | Valor |
|---|---|
| Data e hora | Registrar |
| Arquivo utilizado | Registrar |
| Tamanho do dump | Registrar |
| Duração | Métrica automática |
| Registros validados | Métrica automática |
| Resultado | Sucesso ou falha |
| Responsável | Registrar |
| Observações | Registrar |

## 13. Checklist

- [ ] Backup mais recente localizado
- [ ] Banco de teste isolado
- [ ] Sessões encerradas
- [ ] Banco recriado
- [ ] Dump restaurado
- [ ] Consulta de validação aprovada
- [ ] Métricas atualizadas
- [ ] Log revisado
- [ ] Resultado registrado
- [ ] Validação de filestore planejada ou executada
