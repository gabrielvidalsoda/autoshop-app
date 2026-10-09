# RFC-002 — Escolha do banco de dados

- **Status:** Em revisão (versão final com o modelo ER na E3/E4)
- **Data:** 09/10/2026
- **Autor:** Gabriel Vidal
- **Decisões derivadas:** ADR-007, ADR-009

## Contexto

O domínio é fortemente relacional: cliente → veículos → ordens de serviço → itens (serviços e peças) → histórico de status. Há operações que precisam ser atômicas, como aprovar o orçamento e dar baixa no estoque na mesma transação (RN-09). Os indicadores (volume diário, tempo médio por status) são agregações sobre o histórico. O enunciado pede um banco **gerenciado** e uma justificativa formal da escolha.

## Problema

Qual banco gerenciado usar, e em qual configuração, dentro do Learner Lab?

## Critérios de avaliação

1. Garantias ACID e integridade referencial (FKs, unique, check).
2. Agregações temporais simples para os indicadores.
3. Disponível como serviço gerenciado no Learner Lab.
4. Custo dentro do crédito do lab.
5. Bom suporte em Python (SQLAlchemy 2, Alembic, driver usado na Lambda).

## Opções consideradas

### Opção A — PostgreSQL no Amazon RDS

- **Prós:** ACID; FKs e `CHECK` constraints protegem as regras no próprio banco; window functions e `date_trunc` resolvem os indicadores em SQL; `pg8000` (puro Python) funciona na Lambda sem camada nativa.
- **Contras:** instância ligada consome crédito mesmo ociosa.

### Opção B — MySQL no Amazon RDS

- **Prós:** gerenciado e barato como o PostgreSQL.
- **Contras:** `CHECK` constraints e window functions chegaram depois e são menos usadas no ecossistema; nenhuma vantagem para este domínio.

### Opção C — Amazon DynamoDB

- **Prós:** serverless, sem custo ocioso.
- **Contras:** sem joins nem FKs; a baixa de estoque com aprovação exigiria transações com modelagem de chave cuidadosa; agregações para os indicadores teriam de ser mantidas à mão.

### Opção D — Aurora Serverless v2 (PostgreSQL)

- **Prós:** escala automática.
- **Contras:** custo mínimo maior que um `db.t3.micro`; suporte no Learner Lab incerto.

| Critério | RDS PostgreSQL | RDS MySQL | DynamoDB | Aurora Sv2 |
|---|---|---|---|---|
| ACID + integridade | Sim | Sim | Parcial | Sim |
| Indicadores em SQL | Simples | Simples | Manual | Simples |
| No Learner Lab | Sim | Sim | Sim | A confirmar |
| Custo | Baixo | Baixo | Muito baixo | Médio |
| Suporte Python | Excelente | Bom | Bom | Excelente |

## Proposta

**PostgreSQL 16 no Amazon RDS**, `db.t3.micro`, single-AZ, numa única instância com dois databases (`autoshop_homolog` e `autoshop_prod`) — ver ADR-007.

## Consequências

- As regras críticas (documento único, placa única, estoque não negativo) ficam garantidas também por constraints, não só pelo código.
- O tempo por status sai de uma tabela de histórico de transições, sem coluna calculada.
- **Trade-off aceito:** single-AZ não dá alta disponibilidade do banco. Multi-AZ dobra o custo e o crédito do lab é finito. Fica registrado como evolução para um ambiente real.

## Questões em aberto

- Ajustes do modelo relacional em relação às fases 1 e 2 (histórico de status, `CANCELADA`, índices da listagem) são justificados no `docs/banco/modelo-er.md` (E4).
- Confirmar na E3 a versão do PostgreSQL e as classes de instância permitidas no lab.
