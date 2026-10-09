# RFC-001 — Escolha da nuvem

- **Status:** Aceita
- **Data:** 09/10/2026
- **Autor:** Gabriel Vidal
- **Decisões derivadas:** ADR-002, ADR-005, ADR-006

## Contexto

O enunciado exige API Gateway, função serverless, banco gerenciado, Kubernetes com escalabilidade e Terraform, com "livre escolha de nuvem". O projeto é feito por uma pessoa só, sem orçamento próprio, em cerca de três semanas.

## Problema

Em qual nuvem provisionar toda a infraestrutura obrigatória?

## Critérios de avaliação

1. Tem os 4 serviços gerenciados exigidos (gateway, serverless, banco, Kubernetes).
2. Custo zero para o aluno.
3. Provider Terraform maduro e farto material de referência.
4. Menos restrições de conta que bloqueiem o enunciado.

## Opções consideradas

### Opção A — AWS Academy Learner Lab

- **Prós:** crédito fornecido pela FIAP, sem cartão. Tem API Gateway, Lambda, RDS, EKS e ECR. Provider `hashicorp/aws` é o mais usado do Terraform. É a nuvem usada nas aulas.
- **Contras:** não permite criar IAM roles nem policies (só a `LabRole`); sem OIDC/IRSA; credenciais temporárias de 4h; região fixa `us-east-1`; crédito finito.

### Opção B — Azure for Students

- **Prós:** US$ 100 sem cartão; AKS, Functions, API Management e PostgreSQL Flexible Server.
- **Contras:** o API Management é caro e lento para provisionar; o crédito é da conta pessoal e acaba com o uso de outros projetos; nenhum material de aula na fase.

### Opção C — Google Cloud (trial)

- **Prós:** US$ 300 de crédito; GKE Autopilot simples de operar.
- **Contras:** exige cartão de crédito; API Gateway do GCP é menos flexível com authorizer customizado.

### Opção D — Local (k3d + Kong + PostgreSQL em container)

- **Prós:** custo zero, sem restrição de conta.
- **Contras:** não atende "banco gerenciado" nem "deploy automático para a nuvem".

| Critério | AWS Learner Lab | Azure Students | GCP trial | Local |
|---|---|---|---|---|
| 4 serviços exigidos | Sim | Sim | Sim | Não |
| Custo zero | Sim | Sim (limitado) | Exige cartão | Sim |
| Terraform e referências | Excelente | Bom | Bom | — |
| Restrições de conta | Altas (IAM) | Baixas | Baixas | — |

## Proposta

**AWS Academy Learner Lab, em `us-east-1`.** É a única opção que cumpre os critérios 1 a 3 sem custo nem cartão. As restrições de IAM são contornáveis: todos os serviços usam a `LabRole` existente como role de execução (EKS, nós, Lambda), e o acesso do CI usa as credenciais temporárias da sessão.

## Consequências

- **Fica mais fácil:** reaproveitar material das aulas; um único provider Terraform para os 4 repos.
- **Fica mais difícil:**
  - Os secrets do GitHub (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) precisam ser renovados a cada sessão do lab. Pipelines rodados com credencial vencida falham.
  - Sem IRSA, componentes no cluster que precisariam de role própria (ex.: AWS Load Balancer Controller) ficam de fora; ver ADR-010.
- **Riscos aceitos:** o crédito do lab pode acabar. Mitigação: `terraform destroy` do EKS entre sessões, documentado no runbook.

## Questões em aberto

Nenhuma. A viabilidade de cada serviço com a `LabRole` é testada no spike da E2.
