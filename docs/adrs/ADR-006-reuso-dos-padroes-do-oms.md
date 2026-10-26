# ADR-006 — Reuso dos padrões do OMS

## Status

Decisão registrada na reunião técnica; ADR preparado para revisão, sem aprovação
formal posterior documentada.

## Contexto

O OMS já organiza domínios em módulos, valida entradas com Zod, usa AppError para
erros HTTP e Pino para logs. Introduzir convenções paralelas elevaria custo de
manutenção e tornaria o módulo inconsistente com o restante da API.

## Decisão

Criar futuramente `src/modules/webhooks/` com controller, service, repository,
routes e schemas. Montar suas rotas por `src/routes/index.ts` sob o prefixo
`/api/v1` definido em `src/app.ts`. Reusar:

- `authenticate` e `requireRole` de `src/middlewares/auth.middleware.ts`;
- `validate` com Zod e o envelope de `src/middlewares/error.middleware.ts`;
- `AppError` e subclasses de `src/shared/errors/`;
- Pino de `src/shared/logger/index.ts`;
- paginação de `src/shared/http/response.ts`;
- `createPrismaClient` de `src/config/database.ts` por processo.

Erros próprios do domínio usam prefixo `WEBHOOK_`. Erros compartilhados mantêm
seus códigos atuais, como `VALIDATION_ERROR`, `UNAUTHORIZED` e `FORBIDDEN`; o
prefixo não exige renomear toda falha de infraestrutura ou validação.

## Alternativas Consideradas

- **Criar uma arquitetura exclusiva para webhooks:** alternativa plausível
  descartada por aumentar a divergência sem requisito que a justifique.
- **Novo framework de log/validação:** alternativa plausível descartada porque
  Pino e Zod já cumprem esses papéis.

## Consequências

### Positivas

- Menor curva de aprendizado e contratos HTTP consistentes.
- Autorização, validação e tratamento de erros reutilizam componentes testados.
- A nova feature permanece reconhecível dentro da estrutura atual.

### Negativas

- O middleware atual converte falhas Zod em `VALIDATION_ERROR`, não em código
  `WEBHOOK_*`; qualquer mapper diferente será código novo.
- O redactor Pino atual não mascara genericamente campos `secret`; a implementação
  deve evitar esses campos ou ampliar a configuração.
- O schema atual não relaciona `User` e `Customer`, portanto reutilizar JWT não
  resolve autorização por cliente.

Rastreabilidade: `[09:27] Bruno`, `[09:28] Bruno`, `[09:29] Bruno/Larissa`,
`[09:30] Larissa`, `src/app.ts`, `src/routes/index.ts`,
`src/middlewares/auth.middleware.ts`, `src/middlewares/error.middleware.ts`,
`src/shared/logger/index.ts`, `prisma/schema.prisma`.
