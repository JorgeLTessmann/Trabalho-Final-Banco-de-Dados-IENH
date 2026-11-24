# PLANO DE RECOVERY — TIBURÓN

---

**Versão:** 1.0  
**Data:** 24/11/2026  
**Responsáveis:** Equipe do projeto

## 1. Objetivo

Definir os procedimentos para recuperar o banco de dados Tiburón em situações de perda de informações, corrupção de dados, erros operacionais ou indisponibilidade do servidor.

## 2. Dados que devem ser recuperados

O processo de recuperação deve preservar todas as tabelas do banco:

- **Cadastro:** REGIOES, PESSOAS e USUARIOS.
- **Bens e contatos:** BENS, FOTOS e CONTATOSCELULARES.
- **Interações e conversas:** LIKES, MATCHES, CONTATOSCHAT, MENSAGENS e INTERACOES.
- **Empréstimos:** CONDICOES, EMPRESTIMOS e PARCELAS.
- **Avaliações:** AVALIACOES e AVALIACOESMOTIVOS.
- **Pacotes:** PACOTES e USOPACOTES.
- **Auditoria:** HISTORICO_ALTERACOES.

## 3. Cenários de falha

Os principais cenários considerados são:

- **UPDATE ou DELETE incorreto:** alteração ou exclusão de registros sem a condição `WHERE` adequada.
- **ALTER TABLE com perda de dados:** mudança na estrutura de uma coluna que provoque truncamento ou perda de informações, como reduzir o tamanho de um campo.
- **Corrupção de dados ou tabelas:** informações danificadas ou inconsistentes.
- **Falha de servidor ou armazenamento:** indisponibilidade ou perda do equipamento principal.
- **Acesso não autorizado:** alteração ou exclusão de informações por acesso indevido.

## 4. Procedimento geral de recuperação

Quando ocorrer uma falha, seguir esta ordem:

1. Identificar o problema, o horário aproximado e as tabelas afetadas.
2. Interromper temporariamente as operações que possam continuar alterando os dados.
3. Preservar os arquivos de backup e logs disponíveis, evitando sobrescrevê-los.
4. Preparar um servidor ou ambiente separado para realizar a recuperação.
5. Restaurar o backup completo mais recente anterior à falha.
6. Aplicar os backups incrementais disponíveis, respeitando a ordem em que foram gerados.
7. Aplicar os logs binários até o momento anterior à operação que causou o problema, quando esse recurso estiver configurado.
8. Conferir a integridade dos dados, das tabelas e dos relacionamentos.
9. Liberar o banco novamente somente após a verificação dos resultados.

## 5. Recuperação de alterações incorretas

### 5.1 UPDATE ou DELETE executado incorretamente

1. Interromper novas alterações no banco.
2. Identificar o horário e, quando possível, o comando responsável pelo problema.
3. Consultar os logs binários para localizar a operação incorreta.
4. Restaurar o backup completo anterior ao erro.
5. Aplicar os backups incrementais e os logs posteriores até imediatamente antes do comando incorreto.
6. Verificar se os registros afetados foram recuperados corretamente.

### 5.2 Erro em ALTER TABLE

Quando uma alteração na estrutura de uma tabela causar perda ou truncamento de dados, não se deve continuar utilizando a estrutura danificada como se estivesse íntegra.

O procedimento será restaurar o backup anterior à alteração e reaplicar somente as operações válidas, corrigindo o comando que causou o problema. Depois, conferir a estrutura das colunas e os registros recuperados.

## 6. Recuperação por logs binários (binlogs)

Os logs binários registram as operações realizadas no banco e podem permitir a recuperação até um ponto específico no tempo.

O procedimento previsto é:

1. Identificar o arquivo de log e a posição ou horário da operação que causou a falha.
2. Restaurar o backup completo mais recente anterior ao problema.
3. Aplicar os backups incrementais disponíveis.
4. Reproduzir os logs binários em ordem cronológica, interrompendo a reprodução antes da operação incorreta.
5. Conferir os dados recuperados.

A leitura e a reprodução dos binlogs são realizadas com ferramentas do MySQL, como `mysqlbinlog`, fora do arquivo SQL principal. A ativação e o armazenamento desses logs dependem de configuração do servidor.

## 7. Falha do servidor principal

Como medida de continuidade, é proposta a utilização de um segundo servidor, configurado como réplica do servidor principal.

Em caso de falha do servidor principal:

1. Confirmar a indisponibilidade do servidor principal.
2. Verificar se o servidor de réplica está atualizado e íntegro.
3. Caso esteja adequado, promover a réplica para assumir o atendimento.
4. Se a réplica não estiver disponível ou estiver inconsistente, preparar outro servidor e restaurar o backup completo, os incrementais e os logs disponíveis.
5. Após o restabelecimento, verificar as tabelas, os relacionamentos e as consultas principais.

A replicação deve ser usada junto com os backups. Uma operação incorreta também pode ser replicada para o segundo servidor, por isso a réplica não substitui uma cópia de segurança independente.

## 8. Histórico de alterações

O banco possui a tabela `HISTORICO_ALTERACOES`, alimentada por triggers para registrar operações de INSERT, UPDATE e DELETE nas tabelas PESSOAS, EMPRESTIMOS e PARCELAS.

Esse histórico pode ajudar a identificar alterações nesses registros. Entretanto, ele não registra todas as tabelas do banco e não substitui os backups nem os logs binários.

## 9. Prioridade de recuperação

A recuperação deve respeitar as dependências entre as tabelas e suas chaves estrangeiras. A ordem de referência é:

1. **REGIOES, PESSOAS e USUARIOS:** cadastro e acesso dos usuários.
2. **BENS, FOTOS e CONTATOSCELULARES:** bens, imagens e contatos associados.
3. **LIKES, MATCHES, CONTATOSCHAT, MENSAGENS e INTERACOES:** interações e conversas.
4. **CONDICOES, EMPRESTIMOS e PARCELAS:** condições e registros de empréstimos e pagamentos.
5. **AVALIACOESMOTIVOS e AVALIACOES:** motivos e avaliações.
6. **PACOTES e USOPACOTES:** pacotes e seus registros de utilização.
7. **HISTORICO_ALTERACOES:** registros de auditoria.

Quando for restaurado um backup completo, todas as tabelas devem ser recuperadas. A ordem acima serve como referência para conferir as dependências e priorizar verificações.

## 10. Metas de recuperação (RTO e RPO)

- **RTO — Recovery Time Objective:** meta de até 4 horas para restabelecer o funcionamento do banco.
- **RPO — Recovery Point Objective:** meta de até 1 hora de perda máxima de dados, considerando o último ponto recuperável.

Esses valores são metas definidas para o projeto e precisam ser confirmados por testes de restauração. Para atingir o RPO de 1 hora, os logs binários precisam estar habilitados, armazenados e disponíveis; o backup incremental a cada 6 horas, sozinho, não garante essa meta.

## 11. Testes e manutenção

O plano deverá ser revisado e testado periodicamente, sempre em ambiente separado do banco principal.

- **Semanalmente:** verificar se os arquivos de backup e logs estão disponíveis e íntegros.
- **Mensalmente:** realizar uma restauração de teste e conferir tabelas e registros.
- **Trimestralmente:** simular uma falha de servidor e verificar o procedimento de recuperação.
- **Semestralmente:** revisar este plano e atualizar os procedimentos quando houver mudanças no banco ou na infraestrutura.

Após cada teste, registrar a data, o resultado e o tempo gasto na recuperação.

## 12. Responsabilidades

- **Administrador do banco:** localizar os backups, executar a restauração e verificar a integridade dos dados.
- **Desenvolvedor:** conferir as consultas, views, procedures, functions, triggers e o funcionamento da aplicação após a recuperação.
- **Responsável pela infraestrutura:** preparar os servidores e manter o armazenamento dos backups e logs.
- **Responsável pela segurança:** investigar acessos indevidos e apoiar a análise de incidentes.

## 13. Considerações finais

Este plano estabelece os procedimentos propostos para recuperação do banco de dados Tiburón. A execução efetiva depende da existência de backups válidos, da configuração dos logs binários e, caso adotada, da implantação da replicação em um segundo servidor.

Os procedimentos e as metas descritos devem ser tratados como planejamento até que sejam implementados e validados em testes.

---

**Tiburón — Plano de Recovery**
