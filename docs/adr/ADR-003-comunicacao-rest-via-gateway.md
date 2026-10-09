# ADR-003 — Comunicação REST síncrona através do API Gateway

- **Status:** Aceita
- **Data:** 09/10/2026
- **Origem:** RFC-003

## Contexto

Os consumidores são pessoas usando Swagger, Postman ou curl, e os fluxos são requisição-resposta: cadastrar, abrir OS, aprovar orçamento, consultar status. Nada no enunciado exige processamento assíncrono entre serviços. Todo tráfego externo deve passar pelo API Gateway, que concentra roteamento, autenticação e throttling.

## Decisão

- **Estilo:** REST com JSON, síncrono, sobre HTTPS.
- **Entrada única:** Amazon API Gateway do tipo **HTTP API**. É mais barato e simples que o REST API, e suporta Lambda authorizer com resposta simples.
- **Versionamento:** prefixo `/v1` nas rotas do app. `POST /auth` fica fora do prefixo.
- **Erros:** formato único `{"erro": {"codigo", "mensagem", "request_id"}}`, detalhado em `docs/padroes/api.md` (E4).
- **Correlação:** o Gateway gera ou repassa `X-Request-Id`, e app e Lambdas o gravam em todo log.
- **Comunicação interna:** o app e a Lambda `auth` não conversam entre si; ambos leem o mesmo banco.

## Alternativas descartadas

- **Mensageria (SQS/SNS) entre componentes:** não há fluxo que a exija; adicionaria componente, custo e pontos de falha. Pode entrar só para notificações opcionais (visão, Q5).
- **gRPC:** sem suporte direto no API Gateway e pior para demonstrar no Swagger.
- **API Gateway REST API:** oferece usage plans, mas custa mais e é mais verboso no Terraform; o throttling por stage do HTTP API basta.

## Consequências

- **Positivas:** um só padrão para documentar (OpenAPI), testar e monitorar (latência por rota no New Relic).
- **Negativas / custos aceitos:** chamadas síncronas acoplam disponibilidade; se o RDS cair, a API inteira responde 5xx. Aceito pelo porte do sistema.
- **Como verificar:** `openapi.json` versionado e check no CI (E4); smoke test pelo Gateway (E6).
