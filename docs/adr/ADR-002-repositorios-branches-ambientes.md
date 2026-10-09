# ADR-002 — Quatro repositórios, fluxo de branches e ambientes

- **Status:** Aceita
- **Data:** 09/10/2026
- **Origem:** RFC-001

## Contexto

O enunciado exige:

- quatro repositórios separados, cada um com CI/CD e deploy automático;
- `main` protegida, com Pull Request obrigatório;
- deploy automático das branches de homologação e de produção.

O crédito do Learner Lab não comporta duplicar cluster e banco por ambiente.

## Decisão

### Repositórios

| Repositório | Conteúdo | Deploy |
|---|---|---|
| `autoshop-infra-k8s` | VPC, EKS, ECR, namespaces, metrics-server, New Relic no cluster | Terraform |
| `autoshop-infra-db` | RDS PostgreSQL | Terraform |
| `autoshop-auth-lambda` | Lambdas `auth` e `authorizer`, API Gateway | Terraform + zip |
| `autoshop-app` | API FastAPI, manifests Kubernetes, documentação transversal | Imagem no ECR + Kustomize |

A documentação transversal (RFCs, ADRs, diagramas, ER) fica em `autoshop-app/docs/`. Os outros READMEs apontam para lá.

### Branches

`feature/*` → PR para `develop` → PR para `main`. Commits diretos são bloqueados em `develop` e em `main`. Os checks do CI precisam passar para o merge.

### Ambientes

A infraestrutura base é compartilhada; os ambientes são separados logicamente.

| Item | Homolog | Prod |
|---|---|---|
| Namespace no EKS | `homolog` | `prod` |
| Database no RDS | `autoshop_homolog` | `autoshop_prod` |
| Lambdas e stage do API Gateway | sufixo `-homolog` | sufixo `-prod` |
| GitHub Environment (secrets) | `homolog` | `prod` |

### Gatilhos de deploy

| Repositório | PR | Push em `develop` | Push em `main` |
|---|---|---|---|
| app e lambda | lint + testes | deploy homolog | deploy prod |
| infra-k8s e infra-db | `fmt`, `validate`, `tflint`, `plan` | `plan` | `apply` |

Os repos de infra têm um único `apply`, na `main`, porque a infraestrutura é compartilhada.

## Alternativas descartadas

- **Monorepo:** contraria o enunciado.
- **Um cluster e um RDS por ambiente:** dobra o custo, que é o principal limite do lab.
- **Gitflow completo (release/hotfix):** cerimônia demais para um projeto solo de três semanas.

## Consequências

- **Positivas:** homolog e prod existem de verdade (isolados por namespace e database) com o custo de uma infra só.
- **Negativas / custos aceitos:** uma falha na infra compartilhada derruba os dois ambientes; o isolamento é lógico, não físico.
- **Como verificar:** regras de proteção de branch configuradas no GitHub (print como evidência) e os workflows com os gatilhos da tabela acima.
