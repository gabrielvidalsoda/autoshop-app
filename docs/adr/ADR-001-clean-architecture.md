# ADR-001 — Organizar o autoshop-app em Clean Architecture

- **Status:** Aceita
- **Data:** 09/10/2026
- **Origem:** —

## Contexto

As regras da oficina (máquina de estados da OS, orçamento, estoque) são o que mais precisa de teste e o que menos deve depender de framework. O projeto exige cobertura de 80% em domain e application, e a mesma regra precisa rodar com repositório em memória (E1), PostgreSQL local (E4) e RDS (E5).

## Decisão

O `autoshop-app` terá quatro camadas em `src/autoshop/`, com dependências apontando só para dentro:

| Camada | Contém | Pode importar |
|---|---|---|
| `domain` | Entidades, value objects (CPF, CNPJ, placa), máquina de estados, erros de domínio | Só a biblioteca padrão |
| `application` | Casos de uso (um por UC/ação), portas (interfaces de repositório e de relógio), DTOs | `domain` |
| `infrastructure` | SQLAlchemy, repositórios concretos, config, logging, New Relic | `application`, `domain` |
| `api` | Routers FastAPI, schemas Pydantic, autenticação/autorização, middleware de `request_id` | `application`, `domain` (e `infrastructure` só no ponto de composição) |

A injeção de dependências é feita à mão, num módulo de composição (`api/dependencies.py`), sem container de DI.

## Alternativas descartadas

- **MVC/camadas simples do FastAPI (routers chamando ORM direto):** mais rápido de escrever, mas acopla regra ao SQLAlchemy e torna os testes do domínio dependentes de banco.
- **Hexagonal com container de DI (ex.: `dependency-injector`):** mais uma biblioteca para aprender, sem ganho real num app deste tamanho.

## Consequências

- **Positivas:** testes de domínio e de casos de uso rodam em milissegundos, sem banco; a troca de PostgreSQL local por RDS não toca regra de negócio.
- **Negativas / custos aceitos:** mapeamento entre modelo ORM e entidade de domínio (mais código); mais arquivos por funcionalidade.
- **Como verificar:** regra de import do `ruff` (`flake8-tidy-imports` / `banned-api`) proibindo `fastapi` e `sqlalchemy` em `domain` e `application`; check no CI.
