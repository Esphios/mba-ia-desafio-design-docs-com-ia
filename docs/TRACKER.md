# Tracker de Rastreabilidade

Cada linha representa uma unidade material verificável do pacote. O inventário
considera cada requisito, decisão, alternativa, consequência, risco, critério,
dependência ou contrato coerente como uma unidade; campos de uma mesma entidade e
passos inseparáveis de um mesmo fluxo são agrupados. A mesma afirmação pode ter
uma linha por documento quando cumpre função distinta em cada artefato. Decisões
de contrato não fechadas na reunião são identificadas como proposta e rastreadas
à capacidade que as motiva; não são apresentadas como fatos já aprovados.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR1-DEC-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Outbox participa da mesma transação da mudança de status | TRANSCRICAO | `[09:06] Diego` |
| ADR1-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa descartada | HTTP síncrono bloquearia a transação e acoplaria falhas externas | TRANSCRICAO | `[09:04] Bruno` |
| ADR1-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa descartada | Redis adicionaria infraestrutura considerada excessiva | TRANSCRICAO | `[09:07] Diego` |
| ADR1-COD-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Integração | `changeStatus` já usa transação Prisma para status, histórico e estoque | CODIGO | `src/modules/orders/order.service.ts` |
| ADR2-DEC-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Worker roda em processo separado | TRANSCRICAO | `[09:11] Diego` |
| ADR2-DEC-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Polling ocorre a cada 2 segundos | TRANSCRICAO | `[09:09] Diego` |
| ADR2-DEC-03 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Cada processo abre seu próprio PrismaClient | TRANSCRICAO | `[09:30] Bruno` |
| ADR2-LIM-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Limitação | Ordenação por pedido depende de single-worker e não é global | TRANSCRICAO | `[09:13] Larissa` |
| ADR2-COD-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Integração | Projeto expõe factory de PrismaClient | CODIGO | `src/config/database.ts` |
| ADR3-DEC-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Backoff usa 1m, 5m, 30m, 2h e 12h | TRANSCRICAO | `[09:17] Diego` |
| ADR3-PEN-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Questão aberta | “5 tentativas” conflita com cinco esperas após envio inicial | TRANSCRICAO | `[09:17] Larissa` — “5 tentativas” |
| ADR3-DEC-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Falha permanente vai para tabela de DLQ separada | TRANSCRICAO | `[09:18] Diego` |
| ADR3-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa descartada | Retry indefinido manteria evento pendente para sempre | TRANSCRICAO | `[09:15] Diego` |
| ADR3-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa descartada | Três tentativas cobrem mal indisponibilidade de horas | TRANSCRICAO | `[09:16] Diego` |
| ADR4-DEC-01 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Decisão | Corpo é assinado com HMAC-SHA256 | TRANSCRICAO | `[09:20] Sofia` |
| ADR4-DEC-02 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Decisão | Cada endpoint tem uma secret exclusiva | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-DEC-03 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Decisão | Secret anterior permanece válida por 24 horas | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-RST-01 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Restrição | URL de destino deve usar HTTPS | TRANSCRICAO | `[09:23] Sofia` |
| ADR4-COD-01 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Risco | Redaction atual não cobre genericamente campos `secret` | CODIGO | `src/shared/logger/index.ts` |
| ADR5-DEC-01 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Decisão | Garantia de entrega é at-least-once | TRANSCRICAO | `[09:24] Diego` |
| ADR5-DEC-02 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Decisão | UUID estável é enviado em `X-Event-Id` | TRANSCRICAO | `[09:25] Diego` |
| ADR5-ALT-01 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Alternativa descartada | Exactly-once exigiria coordenação entre os dois lados | TRANSCRICAO | `[09:25] Diego` |
| ADR6-DEC-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Decisão | Módulo segue controller/service/repository/routes/schemas | TRANSCRICAO | `[09:27] Bruno` |
| ADR6-DEC-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Decisão | Erros próprios usam prefixo `WEBHOOK_` | TRANSCRICAO | `[09:29] Larissa` |
| ADR6-COD-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Integração | `requireRole` aceita roles ADMIN e OPERATOR | CODIGO | `src/middlewares/auth.middleware.ts` |
| ADR6-COD-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Integração | Middleware central trata AppError, Zod e Prisma | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR6-COD-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Integração | API é montada sob `/api/v1` | CODIGO | `src/app.ts` |
| ADR6-COD-04 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Limitação | User não possui vínculo com Customer no schema atual | CODIGO | `prisma/schema.prisma` |
| RFC-MOT-01 | `docs/RFC.md` | Motivação | Três clientes B2B pediram notificação e polling é lento/caro | TRANSCRICAO | `[09:00] Marcos` |
| RFC-MET-01 | `docs/RFC.md` | Meta | Menos de 10 segundos é considerado tempo real pelos clientes | TRANSCRICAO | `[09:02] Marcos` |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa descartada | Envio síncrono acoplaria mudança de status ao destino externo | TRANSCRICAO | `[09:04] Bruno` |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa descartada | Redis adicionaria infraestrutura considerada excessiva | TRANSCRICAO | `[09:07] Diego` |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa descartada | Trigger MySQL não notifica o processo externo | TRANSCRICAO | `[09:09] Diego` |
| RFC-OQ-01 | `docs/RFC.md` | Questão aberta | Contagem exata de envios e intervalos de retry | TRANSCRICAO | `[09:17] Larissa` |
| RFC-OQ-02 | `docs/RFC.md` | Questão aberta | Retry pode romper ordenação por pedido | TRANSCRICAO | `[09:12] Diego` |
| RFC-OQ-03A | `docs/RFC.md` | Questão aberta | A decisão de HMAC cobre o corpo | TRANSCRICAO | `[09:22] Sofia` |
| RFC-OQ-03B | `docs/RFC.md` | Questão aberta | X-Timestamp é enviado para eventual detecção de replay | TRANSCRICAO | `[09:44] Diego` |
| RFC-OQ-04 | `docs/RFC.md` | Item adiado | Rate limiting de saída será observado | TRANSCRICAO | `[09:39] Larissa` |
| RFC-RSK-01 | `docs/RFC.md` | Risco | JWT atual não fornece customer_id | TRANSCRICAO | `[09:32] Larissa` |
| FDD-FLOW-01 | `docs/FDD.md` | Fluxo | Evento entra na outbox na transação de mudança de status | TRANSCRICAO | `[09:40] Bruno` |
| FDD-FLOW-02 | `docs/FDD.md` | Fluxo | Filtro de status é aplicado antes da inserção | TRANSCRICAO | `[09:34] Bruno` |
| FDD-FLOW-03 | `docs/FDD.md` | Decisão | Outbox guarda snapshot renderizado na inserção | TRANSCRICAO | `[09:52] Larissa` |
| FDD-MOD-01 | `docs/FDD.md` | Decisão | ID da outbox usa UUID | TRANSCRICAO | `[09:51] Larissa` |
| FDD-MOD-02 | `docs/FDD.md` | Proposta | Uma linha de outbox por endpoint inscrito | TRANSCRICAO | `[09:34] Bruno` |
| FDD-WRK-01 | `docs/FDD.md` | Decisão | Worker busca pendentes antigos em lote pequeno | TRANSCRICAO | `[09:08] Diego` |
| FDD-WRK-02 | `docs/FDD.md` | Restrição | Chamada outbound tem timeout de 10 segundos | TRANSCRICAO | `[09:42] Diego` |
| FDD-WRK-03 | `docs/FDD.md` | Proposta | Lease recupera registros PROCESSING interrompidos | TRANSCRICAO | `[09:11] Diego` |
| FDD-ORD-01 | `docs/FDD.md` | Limitação | Retry pode exigir escolha entre ordem por pedido e latência | TRANSCRICAO | `[09:12] Diego`; `[09:15] Diego` |
| FDD-API-01 | `docs/FDD.md` | Requisito funcional | Criar endpoint com URL, secret gerada e filtros | TRANSCRICAO | `[09:31] Marcos` |
| FDD-API-02 | `docs/FDD.md` | Requisito funcional | Listar, editar e remover endpoints | TRANSCRICAO | `[09:33] Bruno` |
| FDD-API-03 | `docs/FDD.md` | Requisito funcional | Rotacionar secret com grace period de 24h | TRANSCRICAO | `[09:21] Sofia` |
| FDD-API-04 | `docs/FDD.md` | Requisito funcional | Consultar últimos 100 envios e seus resultados | TRANSCRICAO | `[09:34] Marcos` |
| FDD-API-05 | `docs/FDD.md` | Requisito funcional | Replay recoloca entrada da DLQ como pendente | TRANSCRICAO | `[09:18] Diego` |
| FDD-AUT-01 | `docs/FDD.md` | Autorização | Replay exige ADMIN e auditoria do executor | TRANSCRICAO | `[09:36] Sofia` |
| FDD-AUT-02 | `docs/FDD.md` | Autorização | CRUD aceita qualquer role autenticada nesta fase | TRANSCRICAO | `[09:37] Sofia` |
| FDD-PAY-01 | `docs/FDD.md` | Contrato | Payload contém IDs, tipo, timestamp, statuses, customer e total | TRANSCRICAO | `[09:43] Diego` |
| FDD-PAY-02 | `docs/FDD.md` | Restrição | Payload não inclui items | TRANSCRICAO | `[09:43] Diego` |
| FDD-PAY-03 | `docs/FDD.md` | Restrição | Payload máximo é 64 KB e não deve ser truncado | TRANSCRICAO | `[09:24] Larissa` |
| FDD-HDR-01 | `docs/FDD.md` | Contrato | Headers incluem Event, Signature, Timestamp e Content-Type | TRANSCRICAO | `[09:44] Diego` |
| FDD-HDR-02 | `docs/FDD.md` | Contrato | Header X-Webhook-Id identifica o endpoint | TRANSCRICAO | `[09:44] Sofia` |
| FDD-RES-01 | `docs/FDD.md` | Resiliência | Email não é fallback desta fase | TRANSCRICAO | `[09:37] Larissa` |
| FDD-OBS-01 | `docs/FDD.md` | Observabilidade | Replay deve registrar usuário para auditoria | TRANSCRICAO | `[09:36] Sofia` |
| FDD-COD-01 | `docs/FDD.md` | Integração | Transação de pedidos coordena status, histórico e estoque | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-COD-02 | `docs/FDD.md` | Integração | Regras de transição e estoque são centralizadas | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-COD-03 | `docs/FDD.md` | Integração | Modelos atuais usam UUID e não possuem webhook | CODIGO | `prisma/schema.prisma` |
| FDD-COD-04 | `docs/FDD.md` | Integração | Autenticação fornece user e role; replay pode usar requireRole | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-COD-05 | `docs/FDD.md` | Integração | Zod valida body, query e params | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-COD-06 | `docs/FDD.md` | Integração | AppError define status, código e detalhes | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-COD-07 | `docs/FDD.md` | Integração | Listas atuais usam data e pagination | CODIGO | `src/shared/http/response.ts` |
| FDD-COD-08 | `docs/FDD.md` | Integração | Router central agrega módulos sob `/api/v1` | CODIGO | `src/routes/index.ts`; `src/app.ts` |
| PRD-MOT-01 | `docs/PRD.md` | Motivação | Três clientes querem evitar polling de pedidos | TRANSCRICAO | `[09:00] Marcos` |
| PRD-MET-01 | `docs/PRD.md` | Meta | Entrega abaixo de 10 segundos é percebida como tempo real | TRANSCRICAO | `[09:02] Marcos` |
| PRD-SCP-01 | `docs/PRD.md` | Escopo | Webhooks são somente outbound | TRANSCRICAO | `[09:02] Marcos` |
| PRD-RF-01 | `docs/PRD.md` | Requisito funcional | Cadastrar endpoint e receber secret gerada | TRANSCRICAO | `[09:31] Marcos` |
| PRD-RF-02 | `docs/PRD.md` | Requisito funcional | Listar, editar e remover configurações | TRANSCRICAO | `[09:33] Bruno` |
| PRD-RF-03 | `docs/PRD.md` | Requisito funcional | Rotacionar secret com janela de 24h | TRANSCRICAO | `[09:21] Sofia` |
| PRD-RF-04 | `docs/PRD.md` | Requisito funcional | Filtrar statuses antes da outbox | TRANSCRICAO | `[09:34] Bruno` |
| PRD-RF-05 | `docs/PRD.md` | Requisito funcional | Persistir evento junto à mudança de status | TRANSCRICAO | `[09:40] Bruno` |
| PRD-RF-06 | `docs/PRD.md` | Requisito funcional | Worker entrega em processo separado | TRANSCRICAO | `[09:11] Diego` |
| PRD-RF-07 | `docs/PRD.md` | Requisito funcional | Histórico mostra os últimos 100 envios | TRANSCRICAO | `[09:34] Marcos` |
| PRD-RF-08 | `docs/PRD.md` | Requisito funcional | Falhas seguem backoff e limite | TRANSCRICAO | `[09:15] Diego` |
| PRD-RF-09 | `docs/PRD.md` | Requisito funcional | Falha permanente vai para DLQ separada | TRANSCRICAO | `[09:18] Diego` |
| PRD-RF-10 | `docs/PRD.md` | Requisito funcional | ADMIN pode reprocessar DLQ com auditoria | TRANSCRICAO | `[09:36] Sofia` |
| PRD-RF-11 | `docs/PRD.md` | Requisito funcional | Consumidor deduplica usando X-Event-Id | TRANSCRICAO | `[09:25] Diego` |
| PRD-RF-12 | `docs/PRD.md` | Requisito funcional | Payload leva mudança de status sem items | TRANSCRICAO | `[09:43] Diego` |
| PRD-RNF-01 | `docs/PRD.md` | Requisito não funcional | Destino deve usar HTTPS | TRANSCRICAO | `[09:23] Sofia` |
| PRD-RNF-02 | `docs/PRD.md` | Requisito não funcional | Payload máximo é 64 KB | TRANSCRICAO | `[09:24] Larissa` |
| PRD-RNF-03 | `docs/PRD.md` | Requisito não funcional | Timeout HTTP é 10 segundos | TRANSCRICAO | `[09:42] Diego` |
| PRD-OUT-01 | `docs/PRD.md` | Fora de escopo | Email fica para fase futura | TRANSCRICAO | `[09:37] Larissa` |
| PRD-OUT-02 | `docs/PRD.md` | Fora de escopo | Dashboard visual é projeto separado | TRANSCRICAO | `[09:40] Larissa` |
| PRD-OUT-03 | `docs/PRD.md` | Item adiado | Rate limiting será observado | TRANSCRICAO | `[09:39] Larissa` |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | Segurança precisa de dois dias úteis para revisão | TRANSCRICAO | `[09:46] Sofia` |
| PRD-PLN-01 | `docs/PRD.md` | Estimativa | Implementação foi estimada em três sprints | TRANSCRICAO | `[09:46] Larissa` |
| FDD-PROP-01 | `docs/FDD.md` | Proposta derivada | Índice composto permite consultar pendentes por agenda e criação | TRANSCRICAO | `[09:08] Diego` |
| FDD-PROP-02 | `docs/FDD.md` | Proposta derivada | Claim transacional evita seleção concorrente do mesmo evento | TRANSCRICAO | `[09:08] Diego` |
| FDD-PROP-03 | `docs/FDD.md` | Proposta derivada | PROCESSING funciona como lease recuperável após queda | TRANSCRICAO | `[09:11] Diego` |
| FDD-PROP-04 | `docs/FDD.md` | Proposta derivada | Desativação lógica preserva histórico após DELETE | TRANSCRICAO | `[09:34] Marcos` |
| FDD-PROP-05 | `docs/FDD.md` | Proposta derivada | Formato versionado `v1=<hex>` é candidato, não decisão | TRANSCRICAO | `[09:20] Sofia` |
| FDD-PROP-06 | `docs/FDD.md` | Proposta derivada | Secret deve ser protegida em repouso e exibida só quando emitida | TRANSCRICAO | `[09:21] Sofia` |
| FDD-PROP-07 | `docs/FDD.md` | Proposta derivada | Resposta de destino armazenada deve ser limitada e sanitizada | TRANSCRICAO | `[09:34] Marcos` |
| FDD-PROP-08A | `docs/FDD.md` | Proposta derivada | Métricas medem backlog e latência ponta a ponta | TRANSCRICAO | `[09:02] Marcos` |
| FDD-PROP-08B | `docs/FDD.md` | Proposta derivada | Métricas contam resultados, retries e DLQ | TRANSCRICAO | `[09:15] Diego` |
| FDD-PROP-09 | `docs/FDD.md` | Proposta derivada | Event ID correlaciona transação, worker e entrega | TRANSCRICAO | `[09:25] Diego` |
| FDD-PROP-10A | `docs/FDD.md` | Proposta derivada | Instrumentação futura pode criar span de enqueue | TRANSCRICAO | `[09:06] Diego` |
| FDD-PROP-10B | `docs/FDD.md` | Proposta derivada | Instrumentação futura pode criar span de delivery | TRANSCRICAO | `[09:11] Diego` |
| FDD-PROP-11A | `docs/FDD.md` | Proposta derivada | Paths completos herdam o prefixo atual `/api/v1` | CODIGO | `src/app.ts` |
| FDD-PROP-11B | `docs/FDD.md` | Proposta derivada | Status HTTP propostos seguem o padrão de controllers atuais | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-PROP-12 | `docs/FDD.md` | Proposta derivada | Novas listas reutilizam limite de pageSize até 100 | CODIGO | `src/modules/orders/order.schemas.ts` |
| FDD-COD-09 | `docs/FDD.md` | Integração | Controller atual define status e encaminha erro ao middleware | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-COD-10 | `docs/FDD.md` | Integração | Rotas atuais autenticam antes de validar e chamar controller | CODIGO | `src/modules/orders/order.routes.ts` |
| ADR1-CON-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | Status, histórico, estoque e evento compartilham atomicidade | TRANSCRICAO | `[09:06] Diego` |
| ADR1-CON-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | API não depende da disponibilidade do consumidor | TRANSCRICAO | `[09:04] Bruno` |
| ADR1-CON-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | Solução reutiliza MySQL e transação Prisma | TRANSCRICAO | `[09:07] Diego` |
| ADR1-CON-04 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | Outbox exige índices, limpeza, monitoramento e worker | TRANSCRICAO | `[09:08] Diego` |
| ADR1-CON-05 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | Atomicidade não produz exactly-once | TRANSCRICAO | `[09:25] Diego` |
| ADR1-CON-06 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | Retenção de eventos entregues permanece indefinida | TRANSCRICAO | `[09:08] Diego` |
| ADR2-ALT-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa descartada | Worker dentro da API acoplaria processamento ao processo HTTP | TRANSCRICAO | `[09:11] Diego` |
| ADR2-ALT-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa descartada | Trigger MySQL não notifica processo externo | TRANSCRICAO | `[09:09] Diego` |
| ADR2-CON-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência positiva | Reinício do worker não bloqueia requisições da API | TRANSCRICAO | `[09:11] Diego` |
| ADR2-CON-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência positiva | Polling de 2 segundos usa infraestrutura atual | TRANSCRICAO | `[09:09] Diego` |
| ADR2-CON-03 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência positiva | Cada processo administra PrismaClient próprio | TRANSCRICAO | `[09:30] Bruno` |
| ADR2-CON-04 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência negativa | Polling consulta o banco mesmo sem trabalho | TRANSCRICAO | `[09:09] Diego` |
| ADR2-CON-05 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência negativa | Polling sozinho não garante entrega abaixo de 10 segundos | TRANSCRICAO | `[09:02] Marcos` |
| ADR2-CON-06 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência negativa | Retry ou paralelismo podem romper ordenação | TRANSCRICAO | `[09:13] Larissa` |
| ADR3-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa descartada | Manter falhas na outbox foi descartado por DLQ separada | TRANSCRICAO | `[09:18] Diego` |
| ADR3-CON-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência positiva | Backoff cobre indisponibilidades transitórias longas | TRANSCRICAO | `[09:16] Diego` |
| ADR3-CON-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência positiva | DLQ isola falhas e preserva reprocessamento | TRANSCRICAO | `[09:18] Diego` |
| ADR3-CON-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência positiva | Entregas ficam limitadas após decidir cardinalidade | TRANSCRICAO | `[09:17] Larissa` |
| ADR3-CON-04 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência negativa | Cinco tentativas e cinco intervalos exigem decisão | TRANSCRICAO | `[09:17] Larissa` |
| ADR3-CON-05 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência negativa | Replay pode produzir duplicata e exige auditoria | TRANSCRICAO | `[09:24] Diego`; `[09:36] Sofia` |
| ADR3-CON-06 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência negativa | Evento posterior pode ultrapassar evento em retry | TRANSCRICAO | `[09:12] Diego`; `[09:15] Diego` |
| ADR4-ALT-01 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Alternativa descartada | Secret global ampliaria impacto de vazamento | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-ALT-02 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Alternativa plausível | Troca imediata foi rejeitada em favor da janela de 24h | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-ALT-03 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Alternativa plausível | Assinar corpo e timestamp protegeria o header, mas muda o contrato | TRANSCRICAO | `[09:22] Sofia`; `[09:44] Diego` |
| ADR4-CON-01 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência positiva | Vazamento fica limitado a um endpoint | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-CON-02 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência positiva | Consumidor verifica integridade e origem do corpo | TRANSCRICAO | `[09:19] Sofia`; `[09:20] Sofia` |
| ADR4-CON-03 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência positiva | Janela de 24h permite migração gradual | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-CON-04 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência negativa | Duas secrets válidas exigem expiração | TRANSCRICAO | `[09:21] Sofia` |
| ADR4-CON-05 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência negativa | HMAC não substitui HTTPS | TRANSCRICAO | `[09:23] Sofia` |
| ADR4-CON-06 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência negativa | Logs e respostas precisam evitar exposição da secret | TRANSCRICAO | `[09:22] Diego` |
| ADR4-CON-07 | `docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md` | Consequência negativa | Proteção contra replay depende de contrato adicional | TRANSCRICAO | `[09:44] Diego` |
| ADR5-ALT-02 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Alternativa plausível | At-most-once perderia eventos em falha transitória | TRANSCRICAO | `[09:24] Diego` |
| ADR5-CON-01 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência positiva | Retry evita perda silenciosa após resposta perdida | TRANSCRICAO | `[09:24] Diego` |
| ADR5-CON-02 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência positiva | Event ID estável fornece chave de deduplicação | TRANSCRICAO | `[09:25] Diego` |
| ADR5-CON-03 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência positiva | Modelo funciona com retry e replay da DLQ | TRANSCRICAO | `[09:18] Diego`; `[09:24] Diego` |
| ADR5-CON-04 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência negativa | Consumidor precisa tornar processamento idempotente | TRANSCRICAO | `[09:25] Diego` |
| ADR5-CON-05 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência negativa | Duplicatas continuam possíveis em retry e replay | TRANSCRICAO | `[09:24] Diego` |
| ADR5-CON-06 | `docs/adrs/ADR-005-entrega-at-least-once.md` | Consequência negativa | Contrato público precisa destacar deduplicação | TRANSCRICAO | `[09:26] Marcos` |
| ADR6-ALT-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Alternativa plausível | Arquitetura exclusiva divergiria do padrão modular atual | CODIGO | `src/routes/index.ts` |
| ADR6-ALT-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Alternativa plausível | Novo framework duplicaria Pino e Zod existentes | CODIGO | `src/shared/logger/index.ts`; `src/middlewares/validate.middleware.ts` |
| ADR6-CON-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência positiva | Reuso reduz curva de aprendizado | CODIGO | `src/routes/index.ts` |
| ADR6-CON-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência positiva | Contratos HTTP permanecem consistentes | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR6-CON-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência positiva | Novo módulo segue estrutura reconhecível | TRANSCRICAO | `[09:27] Bruno` |
| ADR6-CON-04 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência negativa | Zod usa VALIDATION_ERROR em vez de WEBHOOK_ | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR6-CON-05 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência negativa | Redactor atual não mascara secret genericamente | CODIGO | `src/shared/logger/index.ts` |
| ADR6-CON-06 | `docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md` | Consequência negativa | JWT não resolve autorização por cliente | CODIGO | `prisma/schema.prisma`; `src/middlewares/auth.middleware.ts` |
| PRD-RSK-01 | `docs/PRD.md` | Risco avaliado | Backlog ou destino lento pode romper meta de latência | TRANSCRICAO | `[09:02] Marcos`; `[09:42] Diego` |
| PRD-RSK-02 | `docs/PRD.md` | Risco avaliado | Consumidor pode não deduplicar eventos | TRANSCRICAO | `[09:25] Sofia`; `[09:26] Marcos` |
| PRD-RSK-03 | `docs/PRD.md` | Risco avaliado | Secret pode vazar em logs | TRANSCRICAO | `[09:22] Diego` |
| PRD-RSK-04 | `docs/PRD.md` | Risco avaliado | Ambiguidade de retry pode gerar envios a mais ou menos | TRANSCRICAO | `[09:17] Larissa` |
| PRD-RSK-05 | `docs/PRD.md` | Risco avaliado | Operador pode acessar cliente indevido sem política | TRANSCRICAO | `[09:32] Larissa` |
| PRD-RSK-06 | `docs/PRD.md` | Risco avaliado | Evento posterior pode ultrapassar anterior | TRANSCRICAO | `[09:12] Diego`; `[09:15] Diego` |
| PRD-ACC-01 | `docs/PRD.md` | Critério de aceitação | CRUD e rotação exigem autenticação, customer e HTTPS | TRANSCRICAO | `[09:31] Marcos`; `[09:23] Sofia` |
| PRD-ACC-02 | `docs/PRD.md` | Critério de aceitação | Outbox e mudança de status compartilham commit ou rollback | TRANSCRICAO | `[09:40] Bruno` |
| PRD-ACC-03 | `docs/PRD.md` | Critério de aceitação | Worker entrega JSON com headers e HMAC | TRANSCRICAO | `[09:43] Diego`; `[09:44] Diego` |
| PRD-ACC-04 | `docs/PRD.md` | Critério de aceitação | Payload acima de 64 KB falha sem truncamento | TRANSCRICAO | `[09:24] Larissa` |
| PRD-ACC-05 | `docs/PRD.md` | Critério de aceitação | Falha segue retry e termina em DLQ | TRANSCRICAO | `[09:17] Larissa`; `[09:18] Diego` |
| PRD-ACC-06 | `docs/PRD.md` | Critério de aceitação | Histórico tem 100 entregas e replay é auditado | TRANSCRICAO | `[09:34] Marcos`; `[09:36] Sofia` |
| PRD-ACC-07 | `docs/PRD.md` | Critério de aceitação | Retry e replay preservam X-Event-Id | TRANSCRICAO | `[09:18] Diego`; `[09:25] Diego` |
| PRD-ACC-08 | `docs/PRD.md` | Critério de aceitação | Rotação mantém secret anterior por 24h | TRANSCRICAO | `[09:21] Sofia` |
| PRD-ACC-09 | `docs/PRD.md` | Critério de aceitação | Teste mede latência com cenário declarado | TRANSCRICAO | `[09:02] Marcos` |
| PRD-ACC-10 | `docs/PRD.md` | Critério de aceitação | Email, dashboard e rate limiting não entram nesta fase | TRANSCRICAO | `[09:37] Larissa`; `[09:39] Larissa`; `[09:40] Larissa` |
| PRD-TST-01 | `docs/PRD.md` | Estratégia de teste | Unidade cobre filtros, tamanho, backoff, HMAC e autorização | TRANSCRICAO | `[09:23] Sofia`; `[09:42] Diego` |
| PRD-TST-02 | `docs/PRD.md` | Estratégia de teste | Integração cobre transação, lease, DLQ, replay e snapshot | TRANSCRICAO | `[09:40] Bruno`; `[09:52] Larissa` |
| PRD-TST-03 | `docs/PRD.md` | Estratégia de teste | HTTP cobre respostas, timeout, headers, duplicatas e HTTPS | TRANSCRICAO | `[09:23] Sofia`; `[09:42] Diego`; `[09:44] Diego` |
| PRD-TST-04 | `docs/PRD.md` | Estratégia de teste | Autorização cobre CRUD e replay por role | TRANSCRICAO | `[09:36] Sofia`; `[09:37] Sofia` |
| PRD-TST-05 | `docs/PRD.md` | Estratégia de teste | Desempenho mede latência com carga e backlog | TRANSCRICAO | `[09:02] Marcos` |
| PRD-TST-06 | `docs/PRD.md` | Estratégia de teste | Segurança cobre HMAC, HTTPS e exposição em logs | TRANSCRICAO | `[09:22] Diego`; `[09:23] Sofia` |
| FDD-MOD-03 | `docs/FDD.md` | Proposta de modelagem | Outbox separa id operacional de eventId estável | TRANSCRICAO | `[09:25] Diego`; `[09:51] Larissa` |
| FDD-MOD-04 | `docs/FDD.md` | Proposta de modelagem | Delivery registra tentativa, resposta e duração | TRANSCRICAO | `[09:34] Marcos` |
| FDD-MOD-05 | `docs/FDD.md` | Proposta de modelagem | DLQ preserva eventId e referência da outbox | TRANSCRICAO | `[09:18] Diego`; `[09:25] Diego` |
| FDD-RPL-01 | `docs/FDD.md` | Proposta de replay | Replay cria nova entrada operacional com o mesmo eventId | TRANSCRICAO | `[09:18] Diego`; `[09:25] Diego` |
| FDD-ERR-01 | `docs/FDD.md` | Erro proposto | WEBHOOK_NOT_FOUND representa recurso de webhook ausente | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-ERR-02 | `docs/FDD.md` | Erro proposto | WEBHOOK_INVALID_URL representa URL sem HTTPS | TRANSCRICAO | `[09:23] Sofia` |
| FDD-ERR-03 | `docs/FDD.md` | Erro proposto | WEBHOOK_INVALID_STATUS_FILTER representa filtro inválido | TRANSCRICAO | `[09:33] Marcos` |
| FDD-ERR-04 | `docs/FDD.md` | Erro proposto | WEBHOOK_PAYLOAD_TOO_LARGE representa limite de 64 KB | TRANSCRICAO | `[09:24] Larissa` |
| FDD-ERR-05 | `docs/FDD.md` | Erro proposto | WEBHOOK_SECRET_REQUIRED representa material de chave ausente | TRANSCRICAO | `[09:21] Sofia` |
| FDD-ERR-06 | `docs/FDD.md` | Erro proposto | WEBHOOK_REPLAY_NOT_ALLOWED representa replay inelegível | TRANSCRICAO | `[09:18] Diego` |
| FDD-ERR-07 | `docs/FDD.md` | Erro proposto | WEBHOOK_DELIVERY_TIMEOUT representa timeout do worker | TRANSCRICAO | `[09:42] Diego` |
| FDD-RSK-01 | `docs/FDD.md` | Risco | Cardinalidade errada muda número de entregas | TRANSCRICAO | `[09:17] Larissa` |
| FDD-RSK-02 | `docs/FDD.md` | Risco | Retry pode produzir estados fora de ordem | TRANSCRICAO | `[09:12] Diego`; `[09:15] Diego` |
| FDD-RSK-03 | `docs/FDD.md` | Risco | Backlog pode romper meta de latência | TRANSCRICAO | `[09:02] Marcos`; `[09:42] Diego` |
| FDD-RSK-04 | `docs/FDD.md` | Risco | Secret em log compromete endpoint | TRANSCRICAO | `[09:22] Diego` |
| FDD-RSK-05 | `docs/FDD.md` | Risco | Ausência de vínculo permite acesso a cliente indevido | CODIGO | `prisma/schema.prisma` |
| FDD-RSK-06 | `docs/FDD.md` | Risco | Queda após envio e antes do commit gera duplicata | TRANSCRICAO | `[09:24] Diego` |
| FDD-ACC-01 | `docs/FDD.md` | Critério técnico | Falha da outbox reverte a transação inteira | TRANSCRICAO | `[09:40] Bruno` |
| FDD-ACC-02 | `docs/FDD.md` | Critério técnico | Transição inválida ou filtro ausente não gera outbox | TRANSCRICAO | `[09:34] Bruno` |
| FDD-ACC-03 | `docs/FDD.md` | Critério técnico | Snapshot permanece imutável após mudança posterior | TRANSCRICAO | `[09:52] Larissa` |
| FDD-ACC-04 | `docs/FDD.md` | Critério técnico | EventId permanece estável em retry e replay | TRANSCRICAO | `[09:18] Diego`; `[09:25] Diego` |
| FDD-ACC-05 | `docs/FDD.md` | Critério técnico | Timeout, retry escolhido e DLQ são testados | TRANSCRICAO | `[09:17] Larissa`; `[09:42] Diego` |
| FDD-ACC-06 | `docs/FDD.md` | Critério técnico | PROCESSING interrompido é recuperado sem perda | TRANSCRICAO | `[09:11] Diego` |
| FDD-ACC-07 | `docs/FDD.md` | Critério técnico | Política de ordenação durante retry é testada | TRANSCRICAO | `[09:12] Diego`; `[09:15] Diego` |
| FDD-ACC-08 | `docs/FDD.md` | Critério técnico | HTTP e payload maior que 64 KB são rejeitados | TRANSCRICAO | `[09:23] Sofia`; `[09:24] Larissa` |
| FDD-ACC-09 | `docs/FDD.md` | Critério técnico | HMAC e rotação de 24h são verificáveis | TRANSCRICAO | `[09:20] Sofia`; `[09:21] Sofia` |
| FDD-ACC-10 | `docs/FDD.md` | Critério técnico | CRUD autenticado e replay ADMIN são verificados | TRANSCRICAO | `[09:36] Sofia`; `[09:37] Sofia` |
| FDD-ACC-11 | `docs/FDD.md` | Critério técnico | Latência é medida com cenário e percentil declarados | TRANSCRICAO | `[09:02] Marcos` |
| FDD-ACC-12 | `docs/FDD.md` | Critério técnico | Secrets não aparecem em logs ou respostas comuns | TRANSCRICAO | `[09:22] Diego` |
| RFC-IMP-01 | `docs/RFC.md` | Impacto | Novas tabelas e worker aumentam operação e exigem limpeza e monitoramento | TRANSCRICAO | `[09:08] Diego`; `[09:11] Diego` |
| RFC-IMP-02 | `docs/RFC.md` | Impacto | Endpoints novos herdam `/api/v1` sem alterar contrato atual de pedidos | CODIGO | `src/app.ts`; `src/modules/orders/order.routes.ts` |
| RFC-IMP-03 | `docs/RFC.md` | Risco | Polling de 2s ajuda, mas não garante a meta abaixo de 10s sob backlog | TRANSCRICAO | `[09:02] Marcos`; `[09:09] Diego` |
| RFC-IMP-04 | `docs/RFC.md` | Risco | Duplicatas são esperadas e consumidores precisam deduplicar por event ID | TRANSCRICAO | `[09:24] Diego`; `[09:25] Diego`; `[09:26] Marcos` |
| RFC-IMP-05 | `docs/RFC.md` | Risco | Secrets não podem aparecer em logs e HMAC exige revisão de Segurança | TRANSCRICAO | `[09:22] Diego`; `[09:46] Sofia` |
| RFC-IMP-06 | `docs/RFC.md` | Risco | JWT atual não define isolamento por cliente | TRANSCRICAO | `[09:32] Bruno`; `[09:32] Larissa` |
| PRD-MET-02 | `docs/PRD.md` | Métrica de sucesso | Entregas finais terminam em sucesso ou DLQ, sem órfãos | TRANSCRICAO | `[09:15] Diego`; `[09:18] Diego`; `[09:40] Bruno` |
| PRD-MET-03 | `docs/PRD.md` | Métrica de sucesso | Duplicatas permanecem deduplicáveis pelo mesmo X-Event-Id | TRANSCRICAO | `[09:24] Diego`; `[09:25] Diego` |
| PRD-MET-04 | `docs/PRD.md` | Métrica de sucesso | Operação consegue consultar 100 entregas e auditar replay | TRANSCRICAO | `[09:34] Marcos`; `[09:36] Sofia` |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Modelagem e migração Prisma devem incluir endpoint, outbox, histórico e DLQ | TRANSCRICAO | `[09:08] Diego`; `[09:18] Diego`; `[09:34] Marcos` |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Worker precisa ser implantável como processo separado no mesmo banco | TRANSCRICAO | `[09:11] Diego`; `[09:30] Bruno` |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Cardinalidade de retry e política de ordenação precisam ser fechadas | TRANSCRICAO | `[09:13] Larissa`; `[09:17] Larissa` |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | Autorização usuário-cliente precisa ser definida | TRANSCRICAO | `[09:32] Bruno`; `[09:32] Larissa` |
| PRD-DEP-06 | `docs/PRD.md` | Dependência | Consumidores precisam receber documentação de HMAC e deduplicação | TRANSCRICAO | `[09:26] Marcos`; `[09:40] Marcos` |
| FDD-MOD-06 | `docs/FDD.md` | Proposta de modelagem | Endpoint guarda cliente, URL HTTPS, filtros, estado e ciclo de secrets | TRANSCRICAO | `[09:21] Bruno`; `[09:21] Sofia`; `[09:31] Marcos` |
| FDD-RES-02 | `docs/FDD.md` | Resiliência | Timeout HTTP de 10s conta como falha e agenda retry | TRANSCRICAO | `[09:42] Diego` |
| FDD-RES-03 | `docs/FDD.md` | Resiliência | Retry usa intervalos decididos, com cardinalidade ainda pendente | TRANSCRICAO | `[09:17] Diego`; `[09:17] Larissa` |
| FDD-RES-04 | `docs/FDD.md` | Resiliência | Falha permanente vai à DLQ e replay é manual por ADMIN | TRANSCRICAO | `[09:18] Diego`; `[09:36] Sofia` |
| FDD-RES-05 | `docs/FDD.md` | Resiliência | Duplicatas são aceitas e deduplicadas por X-Event-Id | TRANSCRICAO | `[09:24] Diego`; `[09:25] Diego` |
| FDD-RES-06 | `docs/FDD.md` | Resiliência | Falha antes do commit não deixa evento órfão | TRANSCRICAO | `[09:06] Diego`; `[09:40] Bruno` |
| FDD-RES-07 | `docs/FDD.md` | Proposta de resiliência | Lease vencida permite recuperar worker interrompido, com risco de duplicata | TRANSCRICAO | `[09:11] Diego`; `[09:24] Diego` |
| FDD-RES-08 | `docs/FDD.md` | Resiliência | Falha do consumidor não reverte pedido e segue para retry ou DLQ | TRANSCRICAO | `[09:04] Bruno`; `[09:15] Diego`; `[09:18] Diego` |
| FDD-DEP-01 | `docs/FDD.md` | Dependência | Migração aditiva reutiliza MySQL e Prisma sem alterar endpoints existentes | CODIGO | `prisma/schema.prisma`; `src/modules/orders/order.routes.ts` |
| FDD-DEP-02 | `docs/FDD.md` | Dependência | Implementação reutiliza Node.js, TypeScript, Express, Zod e Pino | CODIGO | `package.json`; `src/app.ts`; `src/shared/logger/index.ts` |
| FDD-DEP-03 | `docs/FDD.md` | Pendência técnica | Cliente HTTP e proteção de secret ainda precisam ser escolhidos | TRANSCRICAO | `[09:20] Sofia`; `[09:21] Sofia` |
| FDD-DEP-04 | `docs/FDD.md` | Dependência | Worker usa DATABASE_URL comum e PrismaClient próprio | TRANSCRICAO | `[09:30] Bruno` |
| FDD-DEP-05 | `docs/FDD.md` | Dependência externa | Consumidores precisam aceitar JSON, HTTPS, HMAC-SHA256 e duplicatas | TRANSCRICAO | `[09:20] Sofia`; `[09:23] Sofia`; `[09:24] Diego`; `[09:43] Diego` |
| FDD-DEP-06 | `docs/FDD.md` | Dependência | Vínculo de autorização usuário-cliente não existe no desenho atual | CODIGO | `prisma/schema.prisma`; `src/middlewares/auth.middleware.ts` |
| FDD-DEP-07 | `docs/FDD.md` | Dependência | Revisão de Segurança requer dois dias úteis antes do deploy | TRANSCRICAO | `[09:46] Sofia` |
