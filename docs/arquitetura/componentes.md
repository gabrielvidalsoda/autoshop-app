# Diagrama de componentes

**Status:** Rascunho (E0) · versão final na E8, refletindo o que foi implantado

Visão de nuvem, APIs, banco e monitoramento.

```mermaid
flowchart LR
    user(["Cliente / Funcionário / Admin<br/>Swagger · Postman · curl"])

    subgraph aws["AWS us-east-1 (Learner Lab)"]
        apigw["API Gateway<br/>HTTP API<br/>stages homolog · prod"]

        subgraph lambdas["Serverless"]
            lauth["Lambda auth<br/>POST /auth"]
            lauthz["Lambda authorizer<br/>valida JWT"]
        end

        subgraph vpc["VPC (subnets públicas, sem NAT)"]
            lb["Load Balancer<br/>(Service type LoadBalancer)"]

            subgraph eks["Amazon EKS"]
                subgraph nsh["namespace homolog"]
                    apph["autoshop-app<br/>pods + HPA"]
                end
                subgraph nsp["namespace prod"]
                    appp["autoshop-app<br/>pods + HPA"]
                end
                nri["New Relic<br/>nri-bundle"]
                ms["metrics-server"]
            end

            rds[("RDS PostgreSQL<br/>autoshop_homolog<br/>autoshop_prod")]
        end

        ecr["ECR<br/>imagens do app"]
        s3["S3<br/>state do Terraform"]
    end

    subgraph nr["New Relic"]
        apm["APM · Logs · Infra"]
        dash["Dashboards e alertas"]
        syn["Monitor sintético<br/>/health"]
    end

    gh["GitHub Actions<br/>4 repositórios"]

    user -->|HTTPS| apigw
    apigw -->|POST /auth| lauth
    apigw -.->|autoriza rotas protegidas| lauthz
    apigw -->|/v1/*| lb
    lb --> apph
    lb --> appp
    lauth -->|consulta CPF e status| rds
    apph --> rds
    appp --> rds
    ms -.->|métricas para o HPA| apph
    ms -.-> appp

    apph -. telemetria .-> apm
    appp -. telemetria .-> apm
    nri -. métricas e logs do cluster .-> apm
    lauth -. layer New Relic .-> apm
    lauthz -. layer New Relic .-> apm
    apm --> dash
    syn -->|ping| apigw

    gh -->|push imagem| ecr
    gh -->|kustomize apply| eks
    gh -->|terraform apply| s3
    eks -->|pull| ecr
```

## Componentes

| Componente | Responsabilidade | Repositório |
|---|---|---|
| API Gateway (HTTP API) | Entrada única, roteamento, authorizer, throttling | autoshop-auth-lambda |
| Lambda `auth` | Valida CPF, consulta cadastro e status, emite JWT | autoshop-auth-lambda |
| Lambda `authorizer` | Valida JWT e repassa `sub` e `perfil` | autoshop-auth-lambda |
| EKS + HPA + metrics-server | Executa e escala o app | autoshop-infra-k8s |
| autoshop-app | Regras de negócio e API REST `/v1` | autoshop-app |
| RDS PostgreSQL | Persistência (um database por ambiente) | autoshop-infra-db |
| ECR | Registro das imagens do app | autoshop-infra-k8s |
| S3 | State remoto do Terraform | bootstrap (runbook) |
| New Relic | APM, infra, logs, dashboards, alertas, uptime | app + infra-k8s |
| GitHub Actions | CI/CD dos 4 repositórios | todos |

## Pontos a confirmar no spike da E2

- O `Service type: LoadBalancer` cria o LB usando a `LabRole`? Plano B: NodePort (ADR-010).
- A Lambda `auth` consegue rodar na VPC para alcançar o RDS? Plano B: RDS público restrito por security group.
