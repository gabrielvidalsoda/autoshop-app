# autoshop-app

API do sistema de gestão de oficina mecânica: clientes, veículos, catálogo de serviços, estoque de peças e o ciclo completo da ordem de serviço. Roda no Amazon EKS atrás de um API Gateway com autenticação por CPF.

> Tech Challenge 13SOAT, Fase 3. Repositórios do sistema: **autoshop-app** (este) · [autoshop-auth-lambda]() · [autoshop-infra-k8s]() · [autoshop-infra-db]()

## Documentação

A documentação arquitetural dos quatro repositórios está em [`docs/`](docs/README.md).

## Tecnologias

<!-- E1/E4 -->
Python 3.12, FastAPI, SQLAlchemy 2, Alembic, PostgreSQL 16, pytest, ruff, Docker, Kubernetes (Kustomize), GitHub Actions, New Relic.

## Arquitetura deste repositório

<!-- E4: diagrama das camadas + caminho da requisição até o RDS -->
_Em construção._

## Executar localmente

<!-- E4: docker compose up -->
_Em construção._

## Deploy

<!-- E5: fluxo develop → homolog, main → prod -->
_Em construção._

## API

<!-- E5/E6: links do Swagger em homolog e prod -->
_Em construção._

## Fluxo de contribuição

`feature/*` → PR para `develop` (deploy em homolog) → PR para `main` (deploy em prod). Commits diretos em `develop` e `main` são bloqueados. Ver [ADR-002](docs/adr/ADR-002-repositorios-branches-ambientes.md).
