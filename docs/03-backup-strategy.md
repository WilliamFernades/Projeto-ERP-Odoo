# Estratégia de Backup e Recuperação

## 1. Objetivo

Proteger os dados necessários para reconstruir o ambiente Odoo em caso de exclusão, corrupção, falha do servidor, erro operacional ou perda do ambiente virtualizado.

Um backup do PostgreSQL isolado não representa uma recuperação completa. O Odoo depende do banco de dados, do filestore e das configurações utilizadas na implantação.

## 2. Escopo de proteção

| Componente | Conteúdo | Método de proteção |
|---|---|---|
| PostgreSQL | Dados estruturados da aplicação | Dump lógico local e cópia ao PBS |
| Filestore | Anexos, documentos e assets | Arquivo `pxar` enviado ao PBS |
| Configuração | Arquivos em `/opt/erp-spark` | Arquivo `pxar` enviado ao PBS |
| Máquina virtual | Sistema operacional e serviços | Backup de VM, quando configurado no Proxmox VE |

## 3. Dados críticos

### 3.1 Banco de dados

Banco principal:

```text
odoo
```

O laboratório possui duas rotinas de dump:

- Dump SQL compactado em `.sql.gz`
- Dump em formato custom do PostgreSQL antes do envio ao PBS

### 3.2 Filestore

Localização lógica no contêiner:

```text
/var/lib/odoo/filestore
```

O caminho do volume no host é identificado dinamicamente pela rotina de backup.

Sem o filestore, podem ocorrer:

- Perda de anexos
- Falhas no carregamento de assets
- Erros HTTP 500
- Interface sem CSS ou JavaScript
- Inconsistência entre banco e arquivos

### 3.3 Configuração

Diretório protegido:

```text
/opt/erp-spark
```

Esse conjunto pode conter arquivos de configuração, Nginx, monitoramento e rotinas operacionais. Segredos não devem ser versionados nem incluídos sem controle de acesso.

## 4. Camadas de backup

### 4.1 Backup lógico local

O script `Scripts/backup_odoo`:

1. Executa `pg_dump` no contêiner PostgreSQL.
2. Compacta o resultado em `.sql.gz`.
3. Armazena o arquivo em `/opt/erp-spark/backup/data`.
4. Verifica se o arquivo foi criado e possui conteúdo.
5. Registra o resultado em `backup.log`.
6. Remove dumps locais com mais de sete dias.

Padrão do arquivo:

```text
db_YYYY-MM-DD_HH-MM-SS.sql.gz
```

A retenção local de sete dias está definida no script. Ela não substitui a retenção do PBS.

### 4.2 Backup de dados para o PBS

O script `Scripts/backup_odoo_pbs`:

1. Carrega a configuração de autenticação do PBS.
2. Descobre o volume do Odoo no host.
3. Gera dump custom do PostgreSQL.
4. Copia o dump para o diretório de backup.
5. Envia banco, filestore e configuração ao PBS.
6. Registra status, duração e horário da execução em métricas.

Conjuntos enviados:

```text
db.pxar
filestore.pxar
config.pxar
```

Identificador utilizado:

```text
odoo-prod-server
```

### 4.3 Backup da máquina virtual

O backup da VM protege o estado geral do servidor, mas não elimina a necessidade do dump lógico e do teste de restauração. A estratégia mais segura combina:

- Backup da VM
- Backup lógico do PostgreSQL
- Proteção do filestore
- Proteção das configurações
- Testes periódicos de recuperação

## 5. Ordem operacional

A execução deve respeitar esta lógica:

1. Gerar o dump local do PostgreSQL.
2. Validar que o arquivo não está vazio.
3. Executar a restauração do dump em banco isolado.
4. Gerar o dump custom utilizado pelo PBS.
5. Enviar banco, filestore e configuração ao PBS.
6. Registrar métricas e logs.
7. Verificar o resultado no PBS e no monitoramento.

A documentação anterior utiliza uma janela entre 02:00 e 02:30. Os horários definitivos devem ser confirmados no `cron` ou no temporizador realmente configurado no servidor.

## 6. Agendamento de referência

| Horário | Atividade |
|---|---|
| 02:00 | Geração do dump SQL compactado |
| 02:10 | Validação do dump local |
| 02:20 | Teste de restauração em `test_restore` |
| 02:30 | Envio de banco, filestore e configuração ao PBS |
| Após a execução | Coleta de logs, métricas e verificação do resultado |

Não agende tarefas dependentes no mesmo horário sem margem de execução.

## 7. Retenção

### 7.1 Local

Implementado no script:

```text
7 dias
```

### 7.2 Proxmox Backup Server

A política de retenção do PBS ainda deve ser formalmente documentada. Uma política GFS pode ser adotada após definição da capacidade de armazenamento e da necessidade de recuperação.

Exemplo a ser avaliado:

- Backups diários
- Backups semanais
- Backups mensais
- Prune e garbage collection agendados

Não registre uma política como concluída antes de configurá-la e validá-la no PBS.

## 8. Métricas e evidências

A rotina de PBS gera:

```text
pbs_backup_success
pbs_backup_duration_seconds
pbs_backup_last_run_timestamp_seconds
```

A validação de restauração gera:

```text
restore_success
restore_duration_seconds
restore_rows_res_users
```

Essas métricas permitem acompanhar:

- Sucesso ou falha
- Duração da operação
- Horário da última execução
- Quantidade de registros validada no banco restaurado

Logs operacionais:

```text
/opt/erp-spark/backup/data/backup.log
/opt/erp-spark/backup/data/restore.log
```

## 9. Critérios de sucesso

Um ciclo de proteção somente deve ser considerado concluído quando:

- O dump foi criado e não está vazio
- O teste de restauração do banco foi concluído
- A validação da tabela selecionada retornou registros
- O envio ao PBS terminou sem erro
- O snapshot aparece no datastore esperado
- As métricas indicam sucesso
- Não há erro crítico nos logs

## 10. RPO e RTO

O laboratório ainda não possui RPO e RTO formalmente definidos.

Devem ser registrados após medição:

| Indicador | Definição | Valor |
|---|---|---|
| RPO | Máxima perda de dados aceitável | A definir |
| RTO | Tempo máximo para recuperação | A definir |
| Tempo de dump | Duração do backup lógico | Medir |
| Tempo de restore | Duração da restauração | Coletado pela métrica |
| Tamanho do backup | Espaço utilizado por execução | Medir |

## 11. Cenários de recuperação

### Exclusão ou corrupção do banco

- Restaurar o dump lógico mais recente validado
- Confirmar integridade e acesso à aplicação
- Avaliar a necessidade de restaurar o filestore correspondente

### Perda do filestore

- Recuperar `filestore.pxar` no PBS
- Restaurar no volume correto do Odoo
- Validar permissões e propriedade
- Testar anexos e carregamento de assets

### Perda da configuração

- Recuperar `config.pxar`
- Revisar credenciais e parâmetros antes de iniciar
- Validar Nginx, Docker Compose e integrações

### Perda completa do servidor

- Provisionar um novo servidor Linux
- Instalar os componentes necessários
- Restaurar configuração, banco e filestore
- Validar serviços, permissões e publicação
- Executar checklist funcional

## 12. Segurança

- Utilizar token dedicado no PBS
- Manter o arquivo de autenticação fora do repositório
- Restringir leitura dos diretórios de backup
- Não armazenar senhas em texto claro nos scripts
- Proteger o tráfego entre o servidor e o PBS
- Rotacionar credenciais expostas
- Revisar periodicamente logs de acesso e falha

## 13. Checklist operacional

- [ ] Dump local criado
- [ ] Arquivo possui conteúdo
- [ ] Retenção local aplicada
- [ ] Restauração do banco validada
- [ ] Banco, filestore e configuração enviados ao PBS
- [ ] Snapshot localizado no datastore
- [ ] Métricas atualizadas
- [ ] Logs sem erro crítico
- [ ] Capacidade do datastore verificada
- [ ] Política de retenção do PBS revisada
