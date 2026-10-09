# Documentação — Start Management Shop

Documentação arquitetural do sistema de oficina mecânica (Tech Challenge 13SOAT, Fase 3). Ela cobre os quatro repositórios; os outros READMEs apontam para cá.

## Comece por aqui

1. [Visão do sistema](visao.md): contexto, escopo, atores, status da OS e decisões a validar
2. [Casos de uso](requisitos/casos-de-uso.md): UC-01 a UC-12 e regras de negócio
3. [Diagrama de componentes](arquitetura/componentes.md)
4. [Matriz de rastreabilidade](rastreabilidade.md): onde cada requisito do enunciado é atendido

## Índice

| Documento | Status | Etapa |
|---|---|---|
| [Visão](visao.md) | Rascunho | E0 |
| [Casos de uso](requisitos/casos-de-uso.md) | Catálogo; critérios entram nas etapas | E0 → E6 |
| [Rastreabilidade](rastreabilidade.md) | Viva | todas |
| [Componentes](arquitetura/componentes.md) | Rascunho | E0 → E8 |
| Sequência: autenticação | — | E6 |
| Sequência: abertura de OS | — | E6 |
| Máquina de estados da OS | — | E1 |
| Modelo ER e justificativa | — | E4 |
| Padrões (API, logging, configuração) | — | E4 |
| Observabilidade | — | E7 |
| Runbook de operação | — | E2 |
| Roteiro do vídeo | — | E8 |

## Decisões

| ID | Título | Status |
|---|---|---|
| [RFC-001](rfc/RFC-001-nuvem.md) | Escolha da nuvem | Aceita |
| [RFC-002](rfc/RFC-002-banco-de-dados.md) | Escolha do banco de dados | Em revisão |
| [RFC-003](rfc/RFC-003-autenticacao.md) | Estratégia de autenticação | Em revisão |
| [RFC-004](rfc/RFC-004-observabilidade.md) | Ferramenta de observabilidade | Aceita |
| [ADR-001](adr/ADR-001-clean-architecture.md) | Clean Architecture no autoshop-app | Aceita |
| [ADR-002](adr/ADR-002-repositorios-branches-ambientes.md) | Repositórios, branches e ambientes | Aceita |
| [ADR-003](adr/ADR-003-comunicacao-rest-via-gateway.md) | Comunicação REST via API Gateway | Aceita |

Modelos: [RFC](templates/rfc.md) · [ADR](templates/adr.md)

**Quando usar cada um:** RFC para uma escolha técnica com opções a comparar (qual nuvem, qual banco). ADR para uma regra arquitetural permanente que o código deve seguir (como as camadas se organizam, como escalar).
