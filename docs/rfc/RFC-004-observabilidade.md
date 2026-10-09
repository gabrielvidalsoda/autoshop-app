# RFC-004 — Ferramenta de observabilidade

- **Status:** Aceita (a instrumentação é detalhada na ADR-011, na E7)
- **Data:** 09/10/2026
- **Autor:** Gabriel Vidal
- **Decisões derivadas:** ADR-011

## Contexto

O enunciado pede integração com uma ferramenta como Datadog ou New Relic, cobrindo:

- latência das APIs;
- CPU e memória do Kubernetes;
- healthcheck e uptime;
- alertas de falha em OS;
- logs JSON com correlação;
- dashboards de volume diário, tempo médio por status e erros de integração.

A demonstração ao vivo acontece no vídeo, gravado perto de 27/10.

## Problema

Qual ferramenta usar, considerando custo zero e a janela de uso de 12/10 até a entrega?

## Critérios de avaliação

1. Cobre APM, Kubernetes, logs, uptime, alertas e dashboards num só lugar.
2. Gratuita durante todo o período, inclusive depois da entrega, para a correção.
3. Funciona sem criar IAM roles na AWS.
4. Tem agente para Python/FastAPI e para Lambda.

## Opções consideradas

| Critério | New Relic (free tier) | Datadog (trial) | CloudWatch + Grafana |
|---|---|---|---|
| Tudo num lugar | Sim | Sim | Parcial (exige montar) |
| Gratuito no período | Sim: 100 GB/mês e 1 usuário completo, sem prazo | Trial de 14 dias: venceria antes do vídeo | Dentro do crédito do lab, mas não é "ferramenta como Datadog/New Relic" |
| Sem IAM role | Sim, via license key (agentes e Lambda layer) | A integração AWS completa exige role | Nativo |
| Agente Python e Lambda | Sim | Sim | Parcial |

## Proposta

**New Relic, plano gratuito.**

- **App:** agente Python no FastAPI.
- **Cluster:** `nri-bundle` (métricas de infra, kube-state-metrics e logs).
- **Lambdas:** layer com extension, enviando direto com a license key (sem a integração AWS que exigiria IAM role).
- **Uptime:** monitor sintético no `/health`.

## Consequências

- Dashboards e alertas podem ser versionados com o provider Terraform `newrelic` (ou, no mínimo, exportados em JSON).
- **Risco:** limite de 100 GB/mês de ingestão. Logs em nível INFO de um app com pouco tráfego ficam muito abaixo disso.
- A license key vira mais um secret por repo.

## Questões em aberto

Nenhuma sobre a ferramenta. Detalhes da instrumentação (agente vs. OpenTelemetry, nomes de eventos customizados) ficam para a ADR-011.
