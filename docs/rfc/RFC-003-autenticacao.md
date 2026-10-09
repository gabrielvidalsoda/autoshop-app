# RFC-003 — Estratégia de autenticação

- **Status:** Em revisão (versão final na E6)
- **Data:** 09/10/2026
- **Autor:** Gabriel Vidal
- **Decisões derivadas:** ADR-003, ADR-010

## Contexto

O enunciado pede:

- rotas sensíveis protegidas por autenticação via CPF;
- uma função serverless que valida o CPF, consulta a existência e o status do cliente e devolve um JWT;
- um API Gateway fazendo o controle e o roteamento.

## Problema

1. Como emitir e validar o token?
2. Onde fica a autorização por perfil?
3. Como funcionários, que também precisam de rotas protegidas, se autenticam?

## Critérios de avaliação

1. Cumpre o enunciado ao pé da letra (CPF → função → JWT).
2. Não exige criar IAM roles nem provedores de identidade (restrição do Learner Lab).
3. A app não depende do Gateway para ser testável localmente.
4. Pouco código para manter.

## Opções consideradas

### Emissão do token

| Opção | Prós | Contras |
|---|---|---|
| **A. Lambda emite JWT HS256 com segredo compartilhado** | Simples; não precisa de infraestrutura extra | Segredo compartilhado entre a Lambda de auth e o authorizer |
| B. Lambda emite JWT RS256 e publica um JWKS | Permite usar o JWT authorizer nativo do HTTP API | Exige hospedar o JWKS numa URL pública com issuer válido; mais peças |
| C. Amazon Cognito com fluxo customizado | Serviço gerenciado | Cognito não autentica "só com CPF" sem triggers customizados; configuração pesada |

### Validação no Gateway

| Opção | Prós | Contras |
|---|---|---|
| **A. Lambda authorizer (HTTP API, resposta simples)** | Funciona com HS256; o resultado pode ficar em cache por alguns minutos; devolve `sub` e `perfil` no contexto | Mais uma Lambda para manter |
| B. JWT authorizer nativo | Zero código | Só aceita issuer com JWKS (exige a opção B de emissão) |

### Quem se autentica

| Opção | Prós | Contras |
|---|---|---|
| A. Só clientes | Literal ao enunciado | Rotas administrativas ficariam sem proteção |
| **B. Clientes e funcionários, ambos por CPF, com perfil no token** | Um único mecanismo para tudo | A função consulta duas tabelas |

## Proposta

1. **Lambda `auth`** (`POST /auth`, rota pública):
   - valida os dígitos do CPF;
   - procura o CPF em `funcionarios` e depois em `clientes`;
   - exige status `ATIVO`;
   - emite um JWT HS256 com `sub`, `perfil` (`CLIENTE`, `FUNCIONARIO` ou `ADMIN`), `iat` e `exp` (1h).
2. **Lambda `authorizer`** em todas as rotas exceto `POST /auth`, `/docs`, `/openapi.json` e `/health`. Ela valida assinatura e expiração e repassa `sub` e `perfil` ao app.
3. **No app:** a camada `api` faz a autorização por perfil e a regra "cliente só vê as próprias OS" (RN-11). O app também valida o JWT por conta própria (o mesmo segredo), para não confiar cegamente em quem chega pelo load balancer (ver ADR-010).
4. **O segredo do JWT** fica em secrets do GitHub, injetado como variável de ambiente nas Lambdas e como `Secret` no Kubernetes. Há um segredo por ambiente.

## Consequências

- **Risco aceito — autenticação só com CPF é fraca:** CPF não é segredo. É o que o enunciado pede; registrado como limitação acadêmica. Mitigações:
  - token curto (1h);
  - throttling no stage do API Gateway;
  - mesma resposta 401 para "inexistente" e "inativo", para não permitir enumeração de cadastros.
- Clientes PJ (CNPJ) não fazem login (visão, Q3).
- A Lambda `auth` precisa alcançar o RDS. Ela roda dentro da VPC (a validar na E2/E6).

## Questões em aberto

- Confirmar na E6 se o cache do authorizer (TTL) atrapalha o caso "cliente inativado continua acessando até o token expirar". Proposta: aceitar, porque o token dura só 1h.
