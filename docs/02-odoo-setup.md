# Implantação e Sustentação do Odoo

## 1. Objetivo

Documentar a estrutura do servidor de aplicação, os componentes utilizados e os procedimentos básicos de implantação, validação e operação do ambiente Odoo.

## 2. Stack do ambiente

- Odoo 17
- PostgreSQL 15
- Docker
- Docker Compose
- Nginx como reverse proxy
- Linux
- Node Exporter
- cAdvisor
- Promtail

## 3. Estrutura de diretórios

A configuração principal do ambiente está centralizada em:

```text
/opt/erp-spark/
├── docker-compose.yml
├── nginx/
├── promtail/
└── backup/
    ├── data/
    ├── backup.log
    ├── restore.log
    └── restore.prom
```

A estrutura pode variar conforme a versão do laboratório. Antes de executar scripts, confirme os caminhos configurados nos próprios arquivos.

## 4. Pré-requisitos

- Servidor Linux atualizado
- Docker instalado e em execução
- Docker Compose disponível
- Nginx instalado ou definido no ambiente
- Acesso administrativo ao host
- Espaço em disco para banco, filestore e backups
- Resolução de nomes ou endereço IP definido
- Comunicação com o servidor de monitoramento e com o PBS
- Horário do sistema sincronizado

## 5. Gerenciamento de credenciais

Credenciais não devem ser armazenadas diretamente no repositório.

Utilize:

```text
.env.example
```

O arquivo real de variáveis deve permanecer apenas no servidor:

```text
.env
```

Exemplo de estrutura sem valores reais:

```dotenv
POSTGRES_DB=odoo
POSTGRES_USER=odoo
POSTGRES_PASSWORD=ALTERAR
ODOO_DB_HOST=postgres
ODOO_DB_PORT=5432
```

Recomendações:

- Adicionar `.env` ao `.gitignore`
- Não registrar senhas ou tokens em scripts
- Restringir a leitura do arquivo no servidor
- Rotacionar qualquer credencial que tenha sido publicada

## 6. Serviços

### 6.1 PostgreSQL

Responsável pelo armazenamento dos dados estruturados da aplicação.

Configuração lógica documentada:

- Banco principal: `odoo`
- Execução em contêiner
- Comunicação com o Odoo pela rede interna do Docker
- Persistência por volume

### 6.2 Odoo

Responsável pela execução do ERP.

Características:

- Versão 17
- Execução em contêiner
- Comunicação com o PostgreSQL pela rede Docker
- Persistência do diretório `/var/lib/odoo`
- Filestore localizado abaixo de `/var/lib/odoo/filestore`

### 6.3 Nginx

Responsável pela publicação da aplicação.

Configuração documentada:

- Escuta HTTP na porta 80
- Encaminha as requisições ao Odoo
- Disponibiliza endpoint para verificação de disponibilidade

Em um ambiente exposto externamente, a publicação deve utilizar HTTPS.

## 7. Validação da configuração

No diretório da aplicação:

```bash
cd /opt/erp-spark
docker compose config
```

O comando deve concluir sem erro de sintaxe ou variável ausente.

## 8. Inicialização do ambiente

```bash
cd /opt/erp-spark
docker compose up -d
```

Verifique os serviços:

```bash
docker compose ps
```

Resultado esperado:

- Contêiner do PostgreSQL em execução
- Contêiner do Odoo em execução
- Serviços sem reinicializações contínuas
- Porta da aplicação acessível pelo Nginx

## 9. Validação pós-implantação

### 9.1 Contêineres

```bash
docker compose ps
docker stats --no-stream
```

### 9.2 Logs do Odoo

```bash
docker compose logs --tail=100 odoo
```

### 9.3 Logs do PostgreSQL

```bash
docker compose logs --tail=100 postgres
```

### 9.4 Conectividade com o banco

```bash
docker exec postgres pg_isready
```

### 9.5 Publicação pelo Nginx

```bash
curl -I http://127.0.0.1
```

### 9.6 Persistência

Confirme que os volumes do Odoo e PostgreSQL estão montados:

```bash
docker inspect odoo
docker inspect postgres
```

A remoção e recriação dos contêineres não deve eliminar os dados persistentes.

## 10. Comandos operacionais

### Consultar status

```bash
docker compose ps
```

### Acompanhar logs

```bash
docker compose logs -f
```

### Reiniciar um serviço

```bash
docker compose restart odoo
```

### Parar o ambiente

```bash
docker compose down
```

Não utilize `docker compose down -v` sem análise prévia, pois a opção remove volumes e pode causar perda de dados.

### Iniciar novamente

```bash
docker compose up -d
```

## 11. Atualização controlada

Antes de atualizar:

1. Gerar backup do PostgreSQL.
2. Confirmar proteção do filestore.
3. Validar que o backup foi concluído.
4. Registrar as versões atuais.
5. Definir procedimento de retorno.

Fluxo recomendado:

```bash
cd /opt/erp-spark
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

A atualização somente deve ser considerada concluída após validação da aplicação, autenticação, anexos e integrações utilizadas.

## 12. Troubleshooting inicial

### Odoo não inicia

- Consultar logs do contêiner
- Validar variáveis de conexão com o banco
- Confirmar que o PostgreSQL está pronto
- Verificar permissões do volume
- Conferir espaço em disco

### Erro de conexão com PostgreSQL

- Confirmar o nome do serviço ou host
- Validar usuário, banco e senha
- Executar `pg_isready`
- Verificar se os serviços compartilham a mesma rede Docker

### Interface sem CSS ou JavaScript

- Verificar logs do Odoo
- Confirmar a existência do filestore
- Validar o volume montado em `/var/lib/odoo`
- Comparar banco e filestore do mesmo ponto de recuperação

### Nginx retorna erro 502

- Confirmar que o Odoo está em execução
- Validar endereço e porta do upstream
- Consultar os logs do Nginx
- Testar acesso direto ao serviço do Odoo

## 13. Checklist de implantação

- [ ] Docker e Docker Compose instalados
- [ ] Estrutura criada em `/opt/erp-spark`
- [ ] Credenciais mantidas fora do Git
- [ ] Volumes persistentes configurados
- [ ] PostgreSQL iniciado e respondendo
- [ ] Odoo iniciado e conectado ao banco
- [ ] Nginx encaminhando requisições
- [ ] Aplicação acessível
- [ ] Filestore validado
- [ ] Backup lógico executado
- [ ] Envio ao PBS validado
- [ ] Teste de restauração executado
- [ ] Métricas e logs disponíveis
