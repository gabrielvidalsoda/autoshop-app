# Matriz de rastreabilidade

**Atualizado em:** 09/10/2026 (E0)

Cada requisito do enunciado da Fase 3 aponta para onde é atendido e para a evidência que comprova. Toda etapa atualiza esta tabela; na E8 nenhuma linha pode ter evidência pendente.

**Legenda de status:** ⬜ não iniciado · 🟨 documentado · 🟦 implementado · ✅ evidenciado

## Autenticação e API Gateway

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-01 | API Gateway para controle e roteamento | RFC-003, ADR-003, Terraform em autoshop-auth-lambda | E6 | 🟨 | — |
| REQ-02 | Rotas sensíveis protegidas por autenticação via CPF | UC-02, Lambda authorizer | E6 | 🟨 | — |
| REQ-03 | Função serverless valida o CPF | UC-01, RN-01 | E6 | 🟨 | — |
| REQ-04 | Função consulta existência e status do cliente | UC-01, RN-05 | E6 | 🟨 | — |
| REQ-05 | Função gera e devolve JWT válido | UC-01, RFC-003 | E6 | 🟨 | — |

## Repositórios e CI/CD

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-06 | 4 repositórios separados (Lambda, infra K8s, infra DB, app) | ADR-002 | E0–E6 | 🟨 | — |
| REQ-07 | CI/CD com deploy automático para a nuvem em cada repo | ADR-002, workflows | E2–E6 | 🟨 | — |
| REQ-08 | Branch `main` protegida (sem commit direto) | ADR-002, proteção no GitHub | E0 | 🟨 | — |
| REQ-09 | Pull Request obrigatório para merge | ADR-002, proteção no GitHub | E0 | 🟨 | — |
| REQ-10 | Deploy automático das branches de homolog e prod | ADR-002 | E5–E6 | 🟨 | — |

## Infraestrutura

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-11 | Banco de dados gerenciado | RFC-002, autoshop-infra-db | E3 | 🟨 | — |
| REQ-12 | Cluster Kubernetes com escalabilidade | ADR-004 (HPA), autoshop-infra-k8s | E2, E5 | ⬜ | — |
| REQ-13 | Terraform para provisionamento | infra-k8s, infra-db, auth-lambda | E2, E3, E6 | ⬜ | — |

> API Gateway e função serverless também aparecem na lista de infraestrutura obrigatória do enunciado; estão cobertos por REQ-01 e REQ-03.

## Monitoramento e observabilidade

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-14 | Integração com Datadog ou New Relic | RFC-004 | E7 | 🟨 | — |
| REQ-15 | Monitorar latência das APIs | `docs/observabilidade.md` | E7 | ⬜ | — |
| REQ-16 | Monitorar CPU e memória do Kubernetes | `docs/observabilidade.md` | E7 | ⬜ | — |
| REQ-17 | Healthchecks e uptime | `/health`, monitor sintético | E4, E7 | ⬜ | — |
| REQ-18 | Alertas para falhas no processamento de OS | RN-12, alerta New Relic | E7 | 🟨 | — |
| REQ-19 | Logs estruturados (JSON) com correlação entre requisições | ADR-003 (`X-Request-Id`), `docs/padroes/logging.md` | E4 | 🟨 | — |
| REQ-20 | Dashboard: volume diário de OS | UC-12 | E7 | 🟨 | — |
| REQ-21 | Dashboard: tempo médio por status (Diagnóstico, Execução, Finalização) | UC-12, RN-07, glossário da visão | E7 | 🟨 | — |
| REQ-22 | Dashboard: erros e falhas nas integrações | `docs/observabilidade.md` | E7 | ⬜ | — |

## Documentação da arquitetura

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-23 | Diagrama de componentes (nuvem, APIs, banco, monitoramento) | `docs/arquitetura/componentes.md` | E0 → E8 | 🟨 | — |
| REQ-24 | Diagrama de sequência: autenticação e abertura de OS | `docs/arquitetura/sequencia-*.md` | E6 | ⬜ | — |
| REQ-25 | RFCs para decisões técnicas (nuvem, banco, autenticação) | RFC-001 a RFC-004 | E0 → E7 | 🟨 | — |
| REQ-26 | ADRs para decisões arquiteturais permanentes | ADR-001 a ADR-011 | E0 → E7 | 🟨 | — |
| REQ-27 | Justificativa do banco, ajustes do modelo, ER e relacionamentos | RFC-002, `docs/banco/modelo-er.md` | E3, E4 | 🟨 | — |

## Entregáveis

| ID | Requisito | Atendido por | Etapa | Status | Evidência |
|---|---|---|---|---|---|
| REQ-28 | README em cada repo: propósito, tecnologias, execução/deploy, diagrama e link do Swagger/Postman | README de cada repo | E2–E8 | ⬜ | — |
| REQ-29 | Dockerfiles (quando aplicável) | `autoshop-app/Dockerfile` | E4 | ⬜ | — |
| REQ-30 | Pipelines de CI/CD funcionais | workflows dos 4 repos | E2–E6 | ⬜ | — |
| REQ-31 | Links para os deploys ativos | READMEs | E8 | ⬜ | — |
| REQ-32 | Vídeo ≤ 15 min: CPF, pipeline, deploy, API protegida, dashboard ao vivo, logs e traces | `docs/entrega/roteiro-video.md` | E8 | ⬜ | — |
| REQ-33 | PDF com links dos repos, do vídeo e da documentação | PDF de entrega | E8 | ⬜ | — |
| REQ-34 | Usuário `soat-architecture` em todos os repos | Configuração no GitHub | E0–E6 | ⬜ | — |

## Desejáveis (citados no desafio, fora dos requisitos obrigatórios)

| ID | Item | Decisão | Etapa |
|---|---|---|---|
| DES-01 | Notificações serverless | Opcional, se houver tempo (visão, Q5) | E6 |
| DES-02 | Alta disponibilidade | App: mínimo de 2 réplicas + HPA, nós em 2 AZs (ADR-004). Banco: single-AZ por custo (RFC-002) | E2, E5 |
| DES-03 | Múltiplas unidades | Tratado como motivação de escala, não como entidade (visão, Q6) | — |
