# PLANO DE BACKUP — TIBURÓN

---

## 1. Objetivo

Definir uma rotina de cópias de segurança do banco de dados Tiburón, reduzindo o risco de perda de informações e permitindo a recuperação dos dados em caso de falhas.

## 2. Dados protegidos

O backup deverá incluir a estrutura e os dados do banco, abrangendo as tabelas:

- **PESSOAS, USUARIOS e REGIOES:** cadastro de pessoas, contas e localização.
- **BENS, FOTOS e CONTATOSCELULARES:** bens cadastrados, imagens e contatos.
- **LIKES, MATCHES, CONTATOSCHAT e MENSAGENS:** interações e conversas entre pessoas.
- **CONDICOES, EMPRESTIMOS e PARCELAS:** condições, empréstimos e pagamentos.
- **INTERACOES, AVALIACOES e AVALIACOESMOTIVOS:** registros de interações e avaliações.
- **PACOTES e USOPACOTES:** pacotes disponíveis e sua utilização.
- **HISTORICO_ALTERACOES:** histórico de alterações registradas pelo banco.

Também deverão ser preservados os objetos do banco necessários ao seu funcionamento, como views e triggers.

## 3. Estratégia de backup

Serão utilizadas três formas de cópia de segurança:

| Tipo | Frequência | Conteúdo |
|---|---|---|
| Backup completo | Diariamente | Estrutura e todos os dados do banco. |
| Backup incremental | A cada 6 horas | Alterações realizadas desde a última cópia de referência. |
| Logs do banco (binlogs) | Contínuo | Registro das operações realizadas no banco, permitindo recuperar alterações posteriores aos backups. |

O backup completo servirá como base para a recuperação. As cópias incrementais e os binlogs deverão ser utilizados para reaplicar as alterações realizadas depois da cópia completa.

A meta de RPO de 1 hora definida no plano de recuperação depende da gravação e conservação contínua dos binlogs. Somente os backups completos diários e incrementais a cada 6 horas não garantem esse objetivo.

## 4. Execução do backup completo

O backup lógico completo poderá ser realizado pelo terminal do servidor utilizando o `mysqldump`:

```bash
mysqldump -u USUARIO -p --databases TIBURON --routines --triggers --events > tiburon_backup.sql
```

O comando gera um arquivo com a estrutura e os dados do banco. Views e triggers também devem ser preservadas para que o sistema possa ser reconstruído corretamente.

## 5. Armazenamento

As cópias de segurança deverão ser armazenadas em um local separado do servidor principal, preferencialmente em um serviço externo ou na nuvem.

Essa separação reduz o risco de que uma falha no servidor principal comprometa também os arquivos de backup.

## 6. Segurança

Os arquivos de backup deverão possuir:

- **Controle de acesso:** somente pessoas autorizadas poderão acessar ou restaurar as cópias.
- **Criptografia:** os arquivos deverão ser protegidos contra acesso não autorizado.
- **Separação do servidor principal:** as cópias deverão permanecer em um ambiente independente.

## 7. Logs e histórico de alterações

O MySQL poderá utilizar os binlogs para registrar continuamente as operações realizadas no banco. A ativação desse recurso depende de configuração no servidor e não é feita apenas pela execução do arquivo `Tiburon.sql`.

A configuração poderá ser conferida com os comandos:

```sql
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'binlog_format';
SHOW BINARY LOGS;
```

O banco também possui a tabela `HISTORICO_ALTERACOES`, alimentada por triggers para operações de inclusão, atualização e exclusão nas tabelas `PESSOAS`, `EMPRESTIMOS` e `PARCELAS`. Esse histórico auxilia na consulta de alterações, mas não substitui os backups nem os binlogs.

## 8. Restauração do backup

Para restaurar um backup completo, o arquivo poderá ser importado pelo terminal:

```bash
mysql -u USUARIO -p < tiburon_backup.sql
```

Quando necessário, as alterações posteriores ao backup deverão ser reaplicadas a partir dos arquivos incrementais e dos binlogs, conforme o procedimento definido no Plano de Recuperação de Desastres.

---

**Tiburón — Plano de Backup**
