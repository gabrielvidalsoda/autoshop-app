# Visão do sistema

**Status:** Rascunho para validação · **Etapa:** E0 · **Atualizado em:** 09/10/2026

## Contexto

A oficina mecânica expandiu para múltiplas unidades e a base de clientes cresce de forma constante. O sistema das fases 1 e 2 (cadastros, ordens de serviço, orçamento e acompanhamento) já resolve a operação. Agora a direção quer levá-lo a um nível de operação corporativa:

- controlar acessos e autenticações com segurança;
- monitorar o ambiente e detectar gargalos em tempo real;
- usar serverless para autenticação (e, se houver tempo, notificações);
- separar a aplicação em repositórios com CI/CD completo;
- documentar e melhorar a modelagem do banco.

## Objetivo da Fase 3

Rodar o sistema de ordens de serviço na nuvem, com:

- autenticação por CPF via função serverless;
- API Gateway na frente de tudo;
- Kubernetes com escalabilidade;
- banco gerenciado;
- infraestrutura em Terraform;
- deploy automático em homologação e produção;
- observabilidade no New Relic.

## Escopo

| Dentro | Fora |
|---|---|
| Cadastro de clientes, veículos, serviços e peças/estoque | Frontend (a API é consumida via Swagger, Postman e curl) |
| Ciclo completo da OS: abertura, diagnóstico, orçamento, aprovação, execução, finalização e entrega | Pagamento e faturamento |
| Autenticação por CPF (Lambda + JWT) e autorização por perfil | Gestão de funcionários via API (são criados por migração/seed) |
| Indicadores de OS: volume diário e tempo médio por status | Modelagem de múltiplas unidades (ver Q6) |
| Deploy em homolog e prod, observabilidade, documentação arquitetural | Login de clientes PJ (ver Q3) |

## Atores e perfis

| Ator | Perfil no JWT | O que faz |
|---|---|---|
| Cliente | `CLIENTE` | Acompanha as próprias OS e aprova ou recusa orçamentos |
| Funcionário | `FUNCIONARIO` | Cadastra clientes e veículos; abre, diagnostica, executa, finaliza e entrega OS |
| Administrador | `ADMIN` | Tudo o que o funcionário faz, mais catálogo de serviços, estoque de peças e indicadores |
| Função de autenticação | — | Valida o CPF, consulta a base e emite o JWT |

## Glossário

| Termo | Definição |
|---|---|
| OS | Ordem de serviço: pedido de trabalho sobre um veículo de um cliente |
| Orçamento | Soma dos serviços e peças da OS, com os preços congelados no momento da inclusão |
| Tempo por status | Intervalo entre a entrada da OS em um status e a saída dele, registrado no histórico de status |
| Homolog / Prod | Ambientes do sistema, cada um com seu namespace no EKS e seu database no RDS |

### Status da OS

| Status | Significado | Entra no painel "tempo médio por status" como |
|---|---|---|
| `RECEBIDA` | OS aberta, aguardando o mecânico | — |
| `EM_DIAGNOSTICO` | Mecânico avaliando o veículo | **Diagnóstico** |
| `AGUARDANDO_APROVACAO` | Orçamento enviado ao cliente | — |
| `EM_EXECUCAO` | Serviço aprovado e em andamento | **Execução** |
| `FINALIZADA` | Serviço concluído, veículo aguardando retirada | **Finalização** |
| `ENTREGUE` | Veículo devolvido ao cliente (terminal) | — |
| `CANCELADA` | Orçamento recusado (terminal) — ver Q1 | — |

## Decisões a validar

Pontos em que precisei assumir algo. Cada um afeta um caso de uso ou o modelo de dados.

| # | Suposição | Impacto se mudar |
|---|---|---|
| Q1 | Orçamento recusado leva a OS para o novo status `CANCELADA`, terminal. Não existia nas fases 1 e 2; entra como ajuste do modelo. | UC-09 e a máquina de estados |
| Q2 | Funcionários e administradores também entram por CPF. A Lambda consulta `funcionarios` e `clientes`, e o perfil vai no JWT. | UC-01 e RFC-003 |
| Q3 | Clientes PJ (CNPJ) continuam cadastráveis, mas não têm login, porque a autenticação do enunciado é por CPF. | UC-01 e UC-03 |
| Q4 | Listagem de OS ativas herdada da fase 2: ordem `EM_EXECUCAO` > `AGUARDANDO_APROVACAO` > `EM_DIAGNOSTICO` > `RECEBIDA`, mais antigas primeiro, sem `FINALIZADA`, `ENTREGUE` e `CANCELADA`. **Confirmar com o enunciado da fase 2.** | UC-11 |
| Q5 | Notificações serverless (citadas no desafio, mas fora dos requisitos obrigatórios) ficam como opcional na E6. | Nenhum obrigatório |
| Q6 | A OS não é vinculada a uma unidade da oficina. "Múltiplas unidades" é tratado como motivação de escala, não como entidade. | Modelo ER (E4) |
| Q7 | A baixa de estoque das peças acontece quando a OS entra em `EM_EXECUCAO`. Sem estoque suficiente, a transição é recusada. | UC-06, UC-09 e UC-10 |
