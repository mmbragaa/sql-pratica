# Consultas SQL - Contexto de Suporte / Help Desk

Repositório com consultas SQL que recriei para praticar e demonstrar lógica aplicada a um cenário comum no meu dia a dia: **análise de chamados de suporte**.

Os dados são fictícios, criados apenas para fins de estudo e demonstração.

## Estrutura fictícia das tabelas

```sql
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY,
    nome VARCHAR(100),
    empresa VARCHAR(100)
);

CREATE TABLE chamados (
    id_chamado INT PRIMARY KEY,
    id_cliente INT,
    assunto VARCHAR(150),
    status VARCHAR(20),        -- 'aberto', 'em andamento', 'resolvido'
    prioridade VARCHAR(20),    -- 'baixa', 'media', 'alta'
    data_abertura DATE,
    data_resolucao DATE,
    canal VARCHAR(20),         -- 'chat', 'email', 'telefone', 'presencial'
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);
```

## Consultas

### 1. Chamados abertos há mais tempo (fora do SLA)
Lista chamados ainda não resolvidos, ordenados pelos mais antigos — útil para priorizar o que já está estourando o prazo combinado com o cliente.

```sql
SELECT
    c.id_chamado,
    cl.nome AS cliente,
    c.assunto,
    c.prioridade,
    c.data_abertura,
    DATEDIFF(CURDATE(), c.data_abertura) AS dias_em_aberto
FROM chamados c
JOIN clientes cl ON cl.id_cliente = c.id_cliente
WHERE c.status != 'resolvido'
ORDER BY dias_em_aberto DESC;
```

### 2. Volume de chamados por canal de atendimento
Mostra por qual canal os clientes mais entram em contato — ajuda a entender onde vale reforçar a base de conhecimento ou o autoatendimento.

```sql
SELECT
    canal,
    COUNT(*) AS total_chamados
FROM chamados
GROUP BY canal
ORDER BY total_chamados DESC;
```

### 3. Tempo médio de resolução por prioridade
Calcula quanto tempo, em média, leva para resolver um chamado, separado por nível de prioridade — indicador direto de cumprimento de SLA.

```sql
SELECT
    prioridade,
    AVG(DATEDIFF(data_resolucao, data_abertura)) AS media_dias_resolucao
FROM chamados
WHERE status = 'resolvido'
GROUP BY prioridade
ORDER BY FIELD(prioridade, 'alta', 'media', 'baixa');
```

### 4. Clientes com mais chamados abertos no último mês
Identifica clientes recorrentes — pode indicar um problema mais amplo com aquela conta que merece atenção proativa.

```sql
SELECT
    cl.nome AS cliente,
    COUNT(*) AS total_chamados
FROM chamados c
JOIN clientes cl ON cl.id_cliente = c.id_cliente
WHERE c.data_abertura >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
GROUP BY cl.nome
HAVING total_chamados > 1
ORDER BY total_chamados DESC;
```

### 5. Chamados de alta prioridade ainda sem solução
Subquery para isolar rapidamente os casos mais críticos e ainda em aberto — o tipo de consulta que eu rodaria no início do turno para organizar prioridades do dia.

```sql
SELECT *
FROM chamados
WHERE prioridade = 'alta'
AND id_chamado NOT IN (
    SELECT id_chamado FROM chamados WHERE status = 'resolvido'
);
```

---

*Essas consultas refletem o tipo de análise que faço no dia a dia de suporte para priorizar chamados, identificar gargalos e acompanhar cumprimento de SLA, aqui recriadas com dados fictícios para fins de portfólio.*
