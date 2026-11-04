# Desafio de Arquitetura: Engenharia Reversa e Views no E-commerce

**O Furo de Caixa no ClassicModels**

**Disciplina:** *Banco de Dados*  
**Aluna:** *Nivea Sofia da Cas*  

---

## 1. Diagnóstico do Problema

### 1.1 Análise da View original

A View criada pelo desenvolvedor foi a seguinte:

```sql
CREATE VIEW view_relatorio_financeiro AS
SELECT
    c.customerName,
    o.orderNumber,
    SUM(od.quantityOrdered * od.priceEach) AS valor_total
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY o.orderNumber;
```

A consulta apresenta os relacionamentos corretos entre as tabelas `customers`, `orders` e `orderdetails`. O problema está na ausência de filtros para selecionar apenas os pedidos que realmente foram enviados e que pertencem ao ano de 2004.

Como não existe uma condição `WHERE`, a consulta considera todos os pedidos, inclusive aqueles com status `Cancelled` e `On Hold`, que não deveriam entrar no cálculo do faturamento.

Além disso, a consulta não limita os resultados ao período solicitado pela diretoria. Dessa forma, os valores apresentados podem ficar maiores do que o esperado, causando o chamado "furo de caixa".

Outro ponto importante é que o cálculo realizado representa a receita bruta dos pedidos, e não o lucro, já que não considera os custos dos produtos.

## 2. Correção da View

### 2.1 Consulta corrigida

Para resolver os problemas encontrados, foi necessário adicionar filtros de status e data, além de ajustar o agrupamento das informações.

```sql
CREATE OR REPLACE VIEW view_relatorio_financeiro AS
SELECT
    c.customerName,
    o.orderNumber,
    SUM(od.quantityOrdered * od.priceEach) AS valor_total
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
WHERE
    o.status = 'Shipped'
    AND o.orderDate >= '2004-01-01'
    AND o.orderDate < '2005-01-01'
GROUP BY
    c.customerNumber,
    c.customerName,
    o.orderNumber;
```

### 2.2 Alterações realizadas

As principais mudanças foram:

- **Filtro de status:** foram selecionados apenas os pedidos com status `Shipped`, excluindo aqueles que foram cancelados ou ficaram retidos.
- **Filtro de data:** a consulta passou a considerar somente os pedidos realizados durante o ano de 2004.
- **Agrupamento:** foram incluídas todas as colunas que não utilizam funções de agregação, evitando possíveis problemas de compatibilidade.
- **CREATE OR REPLACE VIEW:** permite atualizar a View existente sem precisar removê-la e criá-la novamente.

Com essas alterações, o relatório passa a apresentar informações mais próximas da regra de negócio solicitada pela diretoria.

---

## 3. Segurança e Abstração das Views

As Views são tabelas virtuais criadas a partir de consultas SQL. Elas permitem visualizar informações de diferentes tabelas sem precisar repetir toda a consulta sempre que os dados forem necessários (POLSELLI, 2026).

Além disso, apresentam duas vantagens importantes: segurança dos dados e redução da complexidade das consultas.

### 3.1 Segurança dos dados

Uma das principais vantagens das Views é permitir que cada usuário tenha acesso somente às informações necessárias para realizar suas atividades.

No ClassicModels, por exemplo, existem tabelas com informações de clientes, como endereço, telefone e limite de crédito, além de dados financeiros. Entretanto, o gerente financeiro não precisa acessar todas essas informações para consultar o relatório.

Com a View, é possível disponibilizar apenas o nome do cliente, o número do pedido e o valor total, sem conceder acesso direto às demais informações.

Para isso, o administrador pode utilizar o comando:

```sql
GRANT SELECT ON classicmodels.view_relatorio_financeiro
TO 'gerente_financeiro'@'localhost';
```

Dessa forma, a View ajuda a controlar o acesso aos dados e aplica o princípio do menor privilégio, permitindo que cada usuário visualize somente o necessário (HASHTAG TREINAMENTOS, [s. d.]).

### 3.2 Abstração da complexidade

Outra vantagem das Views é facilitar o uso e a manutenção das consultas. Em vez de escrever vários `JOIN`, filtros e agrupamentos sempre que precisar dos dados, o usuário pode simplesmente executar:

```sql
SELECT * FROM view_relatorio_financeiro;
```

Isso facilita o trabalho de quem utiliza o banco, pois não é necessário conhecer todos os relacionamentos entre as tabelas ou os detalhes da consulta.

Além disso, caso alguma regra de negócio precise ser alterada, basta atualizar a View, evitando que a mesma mudança precise ser feita em vários lugares (POLSELLI, 2026).

O problema encontrado no ClassicModels também mostra a importância de revisar e testar as Views antes de disponibilizá-las. Como vários usuários podem depender de uma mesma consulta, qualquer erro na sua construção pode comprometer os resultados apresentados.

---

## 4. Referências

HASHTAG TREINAMENTOS. **Diferença de Tabelas e Views no SQL – Diferenças e Vantagens**. Hashtag Treinamentos, [s. d.]. Disponível em: https://www.hashtagtreinamentos.com/diferenca-tabelas-e-views-no-sql. Acesso em: 3 nov. 2026.

POLSELLI, Alexandre. **Views e Materialized Views em Bancos de Dados**. Data Universe, 23 mar. 2026. Disponível em: https://datauniverse.com.br/views-em-bancos-de-dados/. Acesso em: 3 nov. 2026.