

<img width="1024" height="1024" alt="Sistema_Financeiro_em_SQL_PostgreSQL_PL_pgSQL_" src="https://github.com/user-attachments/assets/49675046-f76c-4dd8-8ca6-146e7d9ca84e" />


# Sistema Financeiro em SQL

Modelagem e implementação de um pequeno **sistema financeiro** em SQL, com foco em:

- **Modelagem relacional** (Clientes, Contas, Transações, Categorias)
- **Integridade referencial** (chaves primárias/estrangeiras e constraints)
- **Geração de massa de dados de teste** via função em **PL/pgSQL**
- **Consultas analíticas** para demonstração e exploração

Exemplo pensado para uso em:

- Laboratórios de Data Warehousing / BI
- Exercícios de SQL analítico
- Testes de performance e tuning
- Demonstrações em aulas e workshops

---

## Tecnologias

- PostgreSQL 13+ (recomendado)
- PL/pgSQL para geração de dados

> O script deve funcionar em versões mais novas do PostgreSQL sem alterações.

---

## Estrutura do Esquema

Tabelas principais:

- `clientes`
- `contas`
- `categorias_transacao`
- `transacoes`

### Diagrama conceitual (resumido)

- Um **cliente** pode ter **várias contas**
- Uma **conta** pode ter **várias transações**
- Cada **transação** pertence a **uma conta** e a **uma categoria**

---

## 1. Como criar o esquema

Execute o arquivo [`schema.sql`](schema.sql) em um banco PostgreSQL:

```bash
psql -U seu_usuario -d seu_banco -f schema.sql
````
Ou dentro de um cliente SQL (psql):

```bash
\i schema.sql;
````
Isso irá:

* Criar o schema financeiro (se ainda não existir)
* Criar as tabelas e índices
* Inserir categorias de transação padrão

 ## 2. Como gerar dados de teste
O arquivo seed_function.sql contém:

* Uma função financeiro.gerar_dados_teste(...) em PL/pgSQL

Carregue a função: 
```bash
psql -U seu_usuario -d seu_banco -f seed_function.sql
````
Depois, chame a função, por exemplo:
```bash
SELECT financeiro.gerar_dados_teste(
    p_qtd_clientes       := 100,
    p_qtd_contas_por_cli := 2,
    p_qtd_transacoes     := 10000
);
````
Isso irá:

* Gerar 100 clientes
* Gerar em média 2 contas por cliente
* Gerar aproximadamente 10.000 transações distribuídas ao longo dos últimos 12 meses
  
Atenção: a função apaga dados anteriores das tabelas transacoes, contas e clientes antes de gerar novos dados. Não use em ambiente de produção.

## 3. Consultas de exemplo
No arquivo queries-exemplos.sql você encontra consultas como:

* Saldo atual por conta
* Saldo consolidado por cliente
* Despesas por categoria e mês
* Top N clientes por volume de movimentação
  
Exemplo simples – saldo por conta:
```bash
SELECT
    c.id_conta,
    cli.nome_cliente,
    c.numero_conta,
    c.agencia,
    c.tipo_conta,
    COALESCE(
        SUM(
            CASE WHEN t.tipo = 'C' THEN t.valor
                 WHEN t.tipo = 'D' THEN -t.valor
            END
        ), 0
    ) AS saldo_atual
FROM financeiro.contas c
JOIN financeiro.clientes cli
  ON cli.id_cliente = c.id_cliente
LEFT JOIN financeiro.transacoes t
  ON t.id_conta = c.id_conta
GROUP BY c.id_conta, cli.nome_cliente, c.numero_conta, c.agencia, c.tipo_conta
ORDER BY saldo_atual DESC;
````

## Organização dos arquivos
* schema.sql
Criação de schema, tabelas, índices e categorias padrão.

* seed_function.sql
Função PL/pgSQL parametrizável para gerar massa de dados de teste.

* queries-exemplos.sql
Consultas SQL para análise e demonstração (arquivo sugerido).

## Próximos passos
Ideias de evolução:

* Criar views analíticas (fatos e dimensões) para cenários de BI.
* Adicionar novas categorias de transação e cenários de teste.
* Incluir scripts de benchmark para comparar índices e estratégias de consulta.
  
## Licença
Projeto disponibilizado para fins de estudo, demonstração e portfólio.
