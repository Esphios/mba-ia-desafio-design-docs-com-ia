# Architecture Decision Records (ADRs)

Os ADRs registram decisões arquiteturais isoladas do sistema de webhooks. “Decisão
registrada” indica consenso na reunião transcrita; não significa que o arquivo já
foi formalmente aprovado depois dela. Detalhes ainda abertos aparecem como tal.

| ADR | Decisão |
| --- | --- |
| [ADR-001 — Outbox no MySQL](ADR-001-outbox-no-mysql.md) | Persistência atômica do snapshot junto à mudança de status |
| [ADR-002 — Worker separado com polling](ADR-002-worker-separado-com-polling.md) | Processo próprio, polling de 2s e PrismaClient por processo |
| [ADR-003 — Retry com backoff e DLQ](ADR-003-retry-com-backoff-e-dlq.md) | Retry limitado, cinco intervalos e DLQ separada |
| [ADR-004 — HMAC e rotação de secret](ADR-004-hmac-e-rotacao-de-secret.md) | HMAC-SHA256, secret por endpoint e grace period de 24h |
| [ADR-005 — Entrega at-least-once](ADR-005-entrega-at-least-once.md) | Duplicatas possíveis e deduplicação por `X-Event-Id` |
| [ADR-006 — Reuso dos padrões do OMS](ADR-006-reuso-dos-padroes-do-oms.md) | Estrutura modular, Zod, AppError, Pino e autorização existentes |

Todos usam a variante MADR exigida pelo desafio: Status, Contexto, Decisão,
Alternativas Consideradas e Consequências positivas e negativas.
