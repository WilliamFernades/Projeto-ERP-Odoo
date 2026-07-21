# Roadmap de Evolução da Infraestrutura

## 1. Objetivo

Evoluir o laboratório para um ambiente mais seguro, monitorado, documentado e recuperável, mantendo o foco em administração e sustentação de infraestrutura.

## 2. Legenda

- [x] Concluído
- [~] Em andamento ou parcialmente implementado
- [ ] Planejado
- [!] Prioridade crítica

## 3. Situação atual

### Aplicação e infraestrutura

- [x] Servidor de aplicação Linux
- [x] Odoo 17 em contêiner
- [x] PostgreSQL 15 em contêiner
- [x] Nginx como reverse proxy
- [x] Persistência do banco e do filestore
- [x] Separação funcional entre aplicação, monitoramento e backup

### Backup e recuperação

- [x] Dump lógico compactado do PostgreSQL
- [x] Retenção local de sete dias
- [x] Validação automática do dump em `test_restore`
- [x] Encerramento de conexões ativas antes do restore
- [x] Métricas de sucesso e duração do restore
- [x] Envio de banco, filestore e configuração ao PBS
- [x] Métricas de status e duração do backup no PBS
- [~] Documentação completa do procedimento de recuperação
- [ ] Validação integral com banco, filestore e instância temporária do Odoo

### Monitoramento e logs

- [~] Prometheus
- [~] Grafana
- [~] Loki
- [~] Node Exporter
- [~] cAdvisor
- [~] Blackbox Exporter
- [~] Promtail
- [ ] Alertas operacionais revisados e testados
- [ ] Dashboards documentados com capturas e indicadores

## 4. Prioridade crítica — segurança de credenciais

- [!] Remover arquivos `.env` reais do repositório
- [!] Adicionar `.env` ao `.gitignore`
- [!] Criar `.env.example` sem credenciais
- [!] Verificar o histórico do Git e remover segredos publicados
- [!] Rotacionar senhas e tokens que possam ter sido expostos
- [!] Manter a autenticação do PBS fora do projeto
- [ ] Aplicar permissões restritivas aos arquivos de segredo
- [ ] Substituir valores fixos nos scripts por variáveis protegidas

Critério de conclusão: nenhuma credencial real acessível no repositório ou no histórico público.

## 5. Etapa 1 — confiabilidade operacional

- [ ] Padronizar nomes e extensões dos scripts
- [ ] Adicionar cabeçalho, versão e responsável em cada script
- [ ] Aplicar `set -euo pipefail` em todas as rotinas
- [ ] Validar dependências antes da execução
- [ ] Implementar bloqueio contra execuções simultâneas
- [ ] Registrar códigos de saída
- [ ] Centralizar logs em diretório padronizado
- [ ] Configurar rotação de logs
- [ ] Documentar `cron` ou `systemd timers`
- [ ] Criar checklist diário e semanal de operação

Critério de conclusão: rotinas executadas de forma previsível, sem concorrência e com evidência suficiente para diagnóstico.

## 6. Etapa 2 — backup e recuperação

- [ ] Definir RPO e RTO
- [ ] Medir tempo e tamanho de cada backup
- [ ] Documentar política de retenção do PBS
- [ ] Configurar prune e garbage collection
- [ ] Validar restore do filestore
- [ ] Criar ambiente temporário para teste completo do Odoo
- [ ] Testar anexos, assets e autenticação
- [ ] Criar runbook de perda total do servidor
- [ ] Executar simulado de recuperação
- [ ] Registrar evidências e responsáveis
- [ ] Avaliar cópia adicional fora do servidor principal

Critério de conclusão: recuperação completa executada e registrada dentro dos objetivos definidos.

## 7. Etapa 3 — monitoramento e alertas

- [ ] Confirmar coleta de CPU, memória, disco e rede
- [ ] Monitorar espaço dos volumes do Odoo e PostgreSQL
- [ ] Monitorar reinicialização de contêineres
- [ ] Monitorar disponibilidade HTTP pelo Blackbox Exporter
- [ ] Coletar métricas do backup e restore
- [ ] Alertar quando `pbs_backup_success = 0`
- [ ] Alertar quando `restore_success = 0`
- [ ] Alertar quando o último backup estiver atrasado
- [ ] Alertar por baixo espaço no datastore
- [ ] Criar dashboard operacional único
- [ ] Documentar limites e justificativas dos alertas
- [ ] Testar cada alerta por falha controlada

Critério de conclusão: falhas relevantes detectadas e sinalizadas sem depender de verificação manual.

## 8. Etapa 4 — centralização de logs

- [ ] Coletar logs do Odoo
- [ ] Coletar logs do PostgreSQL
- [ ] Coletar logs do Nginx
- [ ] Coletar logs dos scripts de backup e restore
- [ ] Definir retenção no Loki
- [ ] Criar consultas para erros HTTP 500
- [ ] Criar consultas para `FileNotFoundError`
- [ ] Criar consultas para falhas de backup
- [ ] Documentar exemplos de diagnóstico

Critério de conclusão: incidentes principais investigados a partir de uma fonte centralizada de logs.

## 9. Etapa 5 — hardening

- [ ] Publicar o Odoo por HTTPS
- [ ] Restringir portas por firewall
- [ ] Limitar acesso administrativo por VPN ou rede confiável
- [ ] Aplicar atualizações periódicas
- [ ] Revisar usuários e grupos administrativos
- [ ] Aplicar menor privilégio no token do PBS
- [ ] Proteger diretórios de backup
- [ ] Revisar imagens e versões dos contêineres
- [ ] Implementar auditoria de acesso
- [ ] Desabilitar serviços desnecessários
- [ ] Documentar procedimento de correção de vulnerabilidades

Critério de conclusão: superfície de exposição reduzida e controles documentados.

## 10. Etapa 6 — documentação operacional

- [x] Arquitetura inicial
- [x] Setup inicial
- [x] Estratégia de backup
- [x] Validação de restore
- [x] Problemas conhecidos
- [x] Roadmap
- [ ] Inventário de portas e fluxos
- [ ] Matriz de responsabilidades
- [ ] Runbook de indisponibilidade do Odoo
- [ ] Runbook de falha do PostgreSQL
- [ ] Runbook de perda do filestore
- [ ] Runbook de recuperação completa
- [ ] Registro de mudanças
- [ ] Evidências com capturas e resultados reais

Critério de conclusão: outro analista consegue operar e recuperar o ambiente utilizando apenas a documentação.

## 11. Ordem recomendada de execução

1. Corrigir exposição de credenciais.
2. Padronizar e agendar as rotinas.
3. Definir retenção, RPO e RTO.
4. Implementar alertas de backup e restore.
5. Validar recuperação completa do Odoo.
6. Aplicar hardening e HTTPS.
7. Finalizar runbooks e evidências.
8. Executar simulado periódico de recuperação.

## 12. Resultado esperado

Ao concluir o roadmap, o laboratório deverá possuir:

- Aplicação sustentada com procedimentos claros
- Backups completos e retenção definida
- Restauração integral testada
- Métricas, logs e alertas operacionais
- Credenciais protegidas
- Controles básicos de segurança
- Runbooks suficientes para diagnóstico e recuperação
