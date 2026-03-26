# Treinamento-de-dados-mysql

Passo a passo como executar transações no MySQL (cliente CLI e em scripts), desabilitando o autocommit e usando COMMIT/ROLLBACK corretamente.

1) O que é autocommit
- Por padrão o MySQL executa cada instrução DML (INSERT/UPDATE/DELETE) em sua própria transação e confirma automaticamente (autocommit = ON).
- Para agrupar várias instruções numa única transação você precisa desabilitar o autocommit ou iniciar explicitamente uma transação.

2) Desabilitar autocommit (sessão)
- No cliente MySQL (mysql CLI) ou em uma conexão, rode:
  SET autocommit = 0;
- Isso afeta somente a sessão/conexão atual. Enquanto estiver 0, instruções DML não serão confirmadas automaticamente e deverão ser confirmadas com COMMIT.

3) Iniciar transação explicitamente (recomendado)
- Em vez de depender de autocommit, a forma clara é:
  START TRANSACTION;
  -- ou BEGIN;
- Após START TRANSACTION, todas as instruções DML fazem parte da mesma transação até COMMIT ou ROLLBACK.

4) Exemplos práticos

- Usando autocommit = 0:
  SET autocommit = 0;
  UPDATE contas SET saldo = saldo - 100 WHERE id = 1;
  UPDATE contas SET saldo = saldo + 100 WHERE id = 2;
  -- verificar se tudo ok
  COMMIT;
  -- se detectar erro:
  -- ROLLBACK;
  -- ao final, se quiser voltar ao comportamento padrão:
  SET autocommit = 1;

- Usando START TRANSACTION:
  START TRANSACTION;
  INSERT INTO pedidos (...) VALUES (...);
  UPDATE estoque SET qtd = qtd - 1 WHERE produto_id = ...;
  -- se tudo OK:
  COMMIT;
  -- se erro:
  ROLLBACK;

5) Controle de erros (exemplo em pseudocódigo)
- Em aplicações é comum:
  START TRANSACTION;
  try {
    executar várias instruções;
    COMMIT;
  } catch (erro) {
    ROLLBACK;
    tratar erro;
  }

6) Locks e isolamento
- Enquanto a transação não for confirmada, mudanças podem ficar visíveis apenas para a própria transação (dependendo do nível de isolamento).
- Pode haver locks que bloqueiam outras transações; mantenha transações curtas.

7) Dicas e boas práticas
- Use START TRANSACTION / COMMIT explicitamente em vez de depender de autocommit = 0, fica mais claro.
- Confirme (COMMIT) o mais rápido possível para evitar contenção de locks.
- Em scripts SQL automatizados, defina explicitamente autocommit se necessário.
- Verifique o nível de isolamento (e.g., REPEATABLE READ é padrão no MySQL) e ajuste se necessário:
  SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
  START TRANSACTION;

8) Comandos úteis de verificação
- Ver autocommit atual:
  SELECT @@autocommit;
- Ver nível de isolamento:
  SELECT @@transaction_isolation;
- Voltar autocommit:
  SET autocommit = 1;
