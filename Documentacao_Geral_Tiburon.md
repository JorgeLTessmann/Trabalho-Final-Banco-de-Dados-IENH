# DOCUMENTAÇÃO GERAL — TIBURÓN

**Projeto:** Tiburón  
**Banco de dados:** MySQL 8.0.16 ou superior  
**Versão do documento:** 1.0  
**Data:** 24/11/2026  
**Responsáveis:** Jorge Luiz Tessmann, Guilherme dos Passos Persch, Mateus Arthur da Silva de Freitas, João Gabriel Guimarães, Leonardo Metz, Francisco Ferreira Telles da Silva

---

## 1. Apresentação do projeto

O Tiburón é um projeto com proposta semelhante a um aplicativo de relacionamento, voltado à conexão entre pessoas que oferecem e procuram empréstimos. O sistema trabalha com dois perfis: **Agiota** e **Otário**.

A proposta prevê cadastro de pessoas, apresentação de condições de empréstimo, interação por likes, criação de matches e conversas, registro de empréstimos e parcelas, avaliações e pacotes de benefícios.

Esta documentação descreve a estrutura que está presente no arquivo `Tiburon_organizado.sql`. Funcionalidades que aparecem apenas na ideia inicial, mas não possuem estrutura implementada no SQL, não são tratadas como recursos concluídos.

## 2. Tecnologias e características

- **SGBD:** MySQL 8.0.16 ou superior.
- **Mecanismo de armazenamento:** InnoDB.
- **Codificação:** utf8mb4.
- **Banco criado pelo script:** `TIBURON`.
- **Objetos no script:** 19 tabelas, 13 triggers, 2 views, 2 procedures e 2 functions.

O banco utiliza chaves primárias, chaves estrangeiras, índices, restrições `CHECK` e valores `ENUM` para estruturar e validar os dados.

## 3. Organização das tabelas

O banco está dividido em grupos de dados:

- **Cadastro:** regiões, pessoas e usuários.
- **Informações pessoais:** bens, fotos e contatos celulares.
- **Interações:** likes, matches, conversas e mensagens.
- **Empréstimos:** condições, empréstimos e parcelas.
- **Avaliações:** interações, avaliações e motivos de avaliação.
- **Pacotes:** pacotes disponíveis e utilização por pessoas.
- **Auditoria:** histórico de alterações.

## 4. Dicionário de dados

A seguir estão as tabelas, seus campos e suas finalidades. `PK` indica chave primária e `FK` indica chave estrangeira.

### 4.1 REGIOES

Armazena as regiões cadastradas no sistema.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da região |
| CIDADE | VARCHAR(100) | Cidade |
| UF | CHAR(2) | Sigla do estado |
| PAIS | VARCHAR(100) | País |

### 4.2 PESSOAS

Armazena os dados das pessoas cadastradas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da pessoa |
| NOME | VARCHAR(255) | Nome |
| TELEFONE | VARCHAR(20) | Telefone |
| IDREGIAO | INT | FK para REGIOES.ID |
| TIPO | ENUM('A','O') | Perfil: Agiota ou Otário |
| STATUS | ENUM('ATIVO','DESATIVADO') | Situação da pessoa; padrão ATIVO |

### 4.3 USUARIOS

Armazena os dados de acesso vinculados a uma pessoa.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do usuário |
| IDPESSOA | INT | FK para PESSOAS.ID; único por pessoa |
| EMAIL | VARCHAR(255) | E-mail único de acesso |
| SENHA | VARCHAR(255) | Hash da senha |
| STATUS | ENUM('ATIVO','INATIVO','BLOQUEADO') | Situação da conta; padrão ATIVO |
| DATA_CADASTRO | DATETIME | Data do cadastro; padrão atual |
| ULTIMO_ACESSO | DATETIME, NULL | Último acesso registrado |

### 4.4 BENS

Armazena os bens pertencentes às pessoas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do bem |
| IDPESSOA | INT | FK para PESSOAS.ID |
| TIPO | VARCHAR(50) | Tipo do bem |
| VALORESTIMADO | DECIMAL(12,2) | Valor estimado |
| DESCRICAO | TEXT, NULL | Descrição do bem |
| STATUSGARANTIA | ENUM('S','N'), NULL | Indica se possui garantia |
| VENCIMENTOGARANTIA | DATE, NULL | Vencimento da garantia |

### 4.5 FOTOS

Armazena as imagens associadas às pessoas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da foto |
| IDPESSOA | INT | FK para PESSOAS.ID |
| FINALIDADE | ENUM('PERFIL','CARD','OUTROS') | Finalidade da imagem |
| FORMATO | VARCHAR(20) | Formato do arquivo |
| TAMANHO | INT | Tamanho em bytes |
| CONTEUDO | LONGBLOB | Conteúdo binário da imagem |

### 4.6 CONTATOSCELULARES

Armazena contatos celulares associados a uma pessoa.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do contato |
| IDPESSOA | INT | FK para PESSOAS.ID |
| NOMECONTATO | VARCHAR(255) | Nome do contato |
| TELEFONECONTATO | VARCHAR(20) | Telefone do contato |
| IDFOTO | INT, NULL | FK para FOTOS.ID; foto opcional |

### 4.7 LIKES

Registra os likes enviados entre pessoas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do like |
| IDPESSOA_LIKE | INT | FK para PESSOAS.ID; pessoa que enviou |
| IDPESSOA_RECEBEU | INT | FK para PESSOAS.ID; pessoa que recebeu |
| DATA_LIKE | DATETIME | Data e hora do like; padrão atual |

O par de pessoas possui restrição de unicidade e não pode apontar para a mesma pessoa.

### 4.8 MATCHES

Armazena os matches formados por likes recíprocos.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do match |
| IDPESSOA_1 | INT | FK para PESSOAS.ID |
| IDPESSOA_2 | INT | FK para PESSOAS.ID |
| IDFOTO | INT, NULL | FK para FOTOS.ID; foto opcional |
| DATA_MATCH | DATETIME | Data e hora da criação; padrão atual |

O par de pessoas é único e o script mantém `IDPESSOA_1` menor que `IDPESSOA_2`.

### 4.9 CONTATOSCHAT

Armazena conversas vinculadas aos matches.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da conversa |
| IDMATCH | INT | FK para MATCHES.ID; único por match |
| DATA_CRIACAO | DATETIME | Data de criação; padrão atual |
| STATUS | ENUM('ATIVO','ENCERRADO') | Situação da conversa; padrão ATIVO |

### 4.10 MENSAGENS

Armazena as mensagens enviadas nas conversas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da mensagem |
| IDCONTATOCHAT | INT | FK para CONTATOSCHAT.ID |
| IDREMETENTE | INT | FK para PESSOAS.ID |
| CONTEUDO | TEXT | Texto ou referência ao conteúdo enviado |
| TIPO | ENUM('TEXTO','IMAGEM','AUDIO','ARQUIVO') | Tipo da mensagem |
| DATAENVIO | DATETIME | Data e hora do envio; padrão atual |
| DATALEITURA | DATETIME, NULL | Data e hora da leitura |
| STATUS | ENUM('ENVIADA','ENTREGUE','LIDA','EXCLUIDA') | Situação da mensagem; padrão ENVIADA |

### 4.11 CONDICOES

Armazena as condições de empréstimo oferecidas pelos agiotas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador das condições |
| IDAGIOTA | INT | FK para PESSOAS.ID |
| CONDICAODESCRICAO | TEXT | Descrição das condições |
| PORCENTAGEMJUROSMINIMO | DECIMAL(5,2) | Percentual mínimo de juros |
| PORCENTAGEMJUROSMAXIMO | DECIMAL(5,2) | Percentual máximo de juros |
| VALORMINIMO | DECIMAL(12,2) | Valor mínimo do empréstimo |
| VALORMAXIMO | DECIMAL(12,2) | Valor máximo do empréstimo |
| GRAUAGRESSIVIDADE | ENUM('BAIXO','MEDIO','ALTO') | Grau de agressividade informado |

### 4.12 EMPRESTIMOS

Registra os empréstimos concedidos.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do empréstimo |
| IDDEVEDOR | INT | FK para PESSOAS.ID; pessoa que recebeu |
| IDAGIOTA | INT | FK para PESSOAS.ID; pessoa que concedeu |
| IDCONDICAO | INT | FK para CONDICOES.ID |
| STATUSPAGAMENTO | ENUM('PENDENTE','EM_ANDAMENTO','QUITADO','ATRASADO') | Situação do empréstimo |
| QUANTIDADEPARCELAS | INT | Quantidade de parcelas |
| VALORTOTAL | DECIMAL(12,2) | Valor total do empréstimo |
| TAXAJUROS | DECIMAL(5,2) | Percentual de juros aplicado |
| DATAINICIO | DATETIME | Data de concessão |
| DATAFIM | DATETIME | Data prevista para quitação |

### 4.13 PARCELAS

Armazena as parcelas vinculadas aos empréstimos.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da parcela |
| IDEMPRESTIMO | INT | FK para EMPRESTIMOS.ID |
| STATUSPAGAMENTO | ENUM('PENDENTE','PAGO','ATRASADO') | Situação da parcela |
| VALORPARCELA | DECIMAL(12,2) | Valor da parcela |
| DATAVENCIMENTO | DATE | Data de vencimento |
| DATAPAGAMENTO | DATETIME, NULL | Data e hora do pagamento |

### 4.14 INTERACOES

Registra interações realizadas entre pessoas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da interação |
| IDENVIADOR | INT | FK para PESSOAS.ID; quem enviou |
| IDRECEBEDOR | INT | FK para PESSOAS.ID; quem recebeu |
| REACAO | VARCHAR(50) | Tipo de reação |
| DATA_INTERACAO | DATETIME | Data e hora; padrão atual |

### 4.15 AVALIACOESMOTIVOS

Armazena os motivos disponíveis para as avaliações.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do motivo |
| TIPO | VARCHAR(30) | Categoria do motivo |
| TITULO | VARCHAR(100) | Nome do motivo |

### 4.16 AVALIACOES

Armazena avaliações entre pessoas relacionadas a um empréstimo.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da avaliação |
| IDEMPRESTIMO | INT | FK para EMPRESTIMOS.ID |
| IDAVALIADO | INT | FK para PESSOAS.ID; pessoa avaliada |
| IDAVALIANTE | INT | FK para PESSOAS.ID; pessoa que avaliou |
| IDMOTIVO | INT | FK para AVALIACOESMOTIVOS.ID |
| NOTA | INT | Nota atribuída |
| OBSERVACAO | TEXT, NULL | Comentário adicional |

### 4.17 PACOTES

Armazena os pacotes de benefícios disponíveis.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador do pacote |
| TIPO | VARCHAR(50) | Modalidade do pacote |
| BENEFICIOS | TEXT, NULL | Descrição dos benefícios |
| VALOR | DECIMAL(12,2) | Preço do pacote |
| LIKES | INT | Quantidade de likes disponibilizada |
| SUPERLIKES | INT | Quantidade de superlikes disponibilizada |
| BOOSTES | INT | Quantidade de boostes disponibilizada |

### 4.18 USOPACOTES

Registra os pacotes utilizados pelas pessoas.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | INT, AUTO_INCREMENT | PK; identificador da utilização |
| IDPESSOA | INT | FK para PESSOAS.ID |
| IDTIPOPACOTE | INT | FK para PACOTES.ID |
| LIKESUSADO | INT | Likes utilizados; padrão 0 |
| SUPERLIKESUSADO | INT | Superlikes utilizados; padrão 0 |
| BOOSTESUSADOS | INT | Boostes utilizados; padrão 0 |
| STATUS | ENUM('ATIVO','EXPIRADO','CANCELADO') | Situação do pacote; padrão ATIVO |

### 4.19 HISTORICO_ALTERACOES

Armazena registros de auditoria das alterações monitoradas por triggers.

| Campo | Tipo | Chave / finalidade |
|---|---|---|
| ID | BIGINT, AUTO_INCREMENT | PK; identificador do histórico |
| TABELA | VARCHAR(64) | Tabela afetada |
| ID_REGISTRO | INT | Identificador do registro afetado |
| OPERACAO | ENUM('INSERT','UPDATE','DELETE') | Operação realizada |
| DATA_HORA | DATETIME | Data e hora; padrão atual |
| USUARIO_BD | VARCHAR(288) | Usuário conectado ao banco |
| DADOS_ANTERIORES | JSON, NULL | Dados anteriores, quando aplicável |
| DADOS_NOVOS | JSON, NULL | Dados novos, quando aplicável |

A tabela de histórico não possui chave estrangeira para as tabelas auditadas, permitindo manter o registro mesmo quando o dado original é excluído.

## 5. Relacionamentos principais

Os relacionamentos são realizados por chaves estrangeiras:

- `PESSOAS.IDREGIAO` referencia `REGIOES.ID`.
- `USUARIOS.IDPESSOA`, `BENS.IDPESSOA` e `FOTOS.IDPESSOA` referenciam `PESSOAS.ID`.
- `CONTATOSCELULARES` referencia `PESSOAS` e, opcionalmente, `FOTOS`.
- `LIKES` e `MATCHES` referenciam as pessoas envolvidas nas interações.
- `CONTATOSCHAT.IDMATCH` referencia `MATCHES.ID`; `MENSAGENS` referencia a conversa e a pessoa remetente.
- `CONDICOES.IDAGIOTA` referencia `PESSOAS.ID`.
- `EMPRESTIMOS` referencia o devedor, o agiota e as condições utilizadas.
- `PARCELAS.IDEMPRESTIMO` referencia `EMPRESTIMOS.ID`.
- `INTERACOES` referencia as pessoas que enviam e recebem a interação.
- `AVALIACOES` referencia o empréstimo, as pessoas avaliadas e o motivo da avaliação.
- `USOPACOTES` referencia a pessoa e o pacote utilizado.

O script também define regras `ON UPDATE CASCADE`, `ON DELETE RESTRICT` e `ON DELETE SET NULL` conforme a relação, além de restrições `CHECK` para impedir valores inválidos em alguns campos.

## 6. Views

As views disponibilizam consultas consolidadas para utilização por outros módulos ou sistemas.

### VW_PERFIL_PESSOAS

Apresenta os dados básicos das pessoas junto com cidade, estado, país e a quantidade de bens e fotos cadastrados para cada pessoa.

Consulta de exemplo:

```sql
SELECT * FROM VW_PERFIL_PESSOAS;
```

### VW_EMPRESTIMOS_DETALHADOS

Apresenta informações do empréstimo, nomes do devedor e do agiota, condições utilizadas, quantidade de parcelas cadastradas e valor total já pago.

Consulta de exemplo:

```sql
SELECT * FROM VW_EMPRESTIMOS_DETALHADOS;
```

## 7. Triggers

O script possui 13 triggers:

| Trigger | Funcionamento |
|---|---|
| `TPESSOAS_DESATIVAR_USUARIO` | Ao desativar uma pessoa, altera o status da conta vinculada para INATIVO. |
| `TPESSOAS_AUDITORIA_AI` | Registra a inclusão de uma pessoa no histórico. |
| `TPESSOAS_AUDITORIA_AU` | Registra alterações de uma pessoa, incluindo dados anteriores e novos. |
| `TPESSOAS_AUDITORIA_AD` | Registra a exclusão de uma pessoa. |
| `TEMPRESTIMOS_AUDITORIA_AI` | Registra a inclusão de um empréstimo. |
| `TEMPRESTIMOS_AUDITORIA_AU` | Registra alterações de um empréstimo. |
| `TEMPRESTIMOS_AUDITORIA_AD` | Registra a exclusão de um empréstimo. |
| `TPARCELAS_AUDITORIA_AI` | Registra a inclusão de uma parcela. |
| `TPARCELAS_AUDITORIA_AU` | Registra alterações de uma parcela. |
| `TPARCELAS_AUDITORIA_AD` | Registra a exclusão de uma parcela. |
| `TLIKES_GERAR_MATCH` | Cria um match quando é identificado um like recíproco. |
| `TMATCHES_CRIAR_CHAT` | Cria uma conversa quando um match é inserido. |
| `TMENSAGENS_VALIDAR_REMETENTE` | Impede que uma pessoa fora do match envie mensagem na conversa. |

A auditoria automática está implementada para as tabelas `PESSOAS`, `EMPRESTIMOS` e `PARCELAS`. Ela não registra automaticamente alterações de todas as tabelas do banco.

## 8. Procedures e functions

O script possui duas procedures e duas functions.

### Procedures

**`SP_LISTAR_PARCELAS_EMPRESTIMO(P_IDEMPRESTIMO)`**  
Lista as parcelas de um empréstimo específico, ordenadas pela data de vencimento.

```sql
CALL SP_LISTAR_PARCELAS_EMPRESTIMO(1);
```

**`SP_RANKING_PAGADORES_POR_AGIOTA()`**  
Apresenta, para cada agiota, os devedores associados aos empréstimos, com indicadores de pagamento. O resultado inclui parcelas avaliadas, parcelas pagas no prazo, parcelas pagas com atraso, parcelas vencidas ainda não pagas, valor pago e taxa de pontualidade. A classificação considera primeiro a taxa de pontualidade, depois a quantidade de parcelas em atraso e, por fim, o valor pago.

```sql
CALL SP_RANKING_PAGADORES_POR_AGIOTA();
```

### Functions

**`FN_VALOR_RESTANTE_EMPRESTIMO(P_IDEMPRESTIMO)`**  
Retorna a soma das parcelas do empréstimo cujo status seja diferente de `PAGO`.

```sql
SELECT FN_VALOR_RESTANTE_EMPRESTIMO(1) AS VALOR_RESTANTE;
```

**`FN_MAIOR_EMPRESTIMO_MES(P_DATA_REFERENCIA)`**  
Recebe uma data de referência e retorna o maior `VALORTOTAL` entre os empréstimos cuja `DATAINICIO` esteja no mesmo mês e ano. Se não houver empréstimos no período, retorna `0.00`.

```sql
SELECT FN_MAIOR_EMPRESTIMO_MES('2026-11-01') AS MAIOR_EMPRESTIMO;
```

## 9. Execução do script

1. Abrir o MySQL Workbench ou outro cliente MySQL.
2. Conectar ao servidor com um usuário que tenha permissão para criar bancos, tabelas, views, triggers, procedures e functions.
3. Abrir o arquivo `Tiburon_organizado.sql`.
4. Executar o script completo.
5. Conferir se o banco `TIBURON` e seus objetos foram criados.

**Atenção:** o script contém comandos `DROP TABLE`, `DROP VIEW`, `DROP TRIGGER`, `DROP PROCEDURE` e `DROP FUNCTION`. Ao ser executado novamente, ele recria os objetos e apaga os dados que estejam nas tabelas listadas. Deve ser executado inicialmente em um ambiente de teste ou em uma base sem dados importantes.

## 10. Considerações finais

O arquivo `Tiburon.sql` concentra a estrutura do banco de dados, as relações entre as tabelas, as regras automatizadas, as consultas consolidadas e as rotinas de consulta e cálculo. Esta documentação serve como referência para entender e utilizar os objetos implementados no script.

A documentação descreve o conteúdo do SQL, mas não substitui a execução e a validação do script em um servidor MySQL.
