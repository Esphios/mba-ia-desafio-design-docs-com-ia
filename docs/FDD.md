# FDD — Sistema de Webhooks de Notificação de Pedidos

## 1. Contexto e motivação técnica

O OMS não possui mecanismo de notificação externa. Clientes B2B consultam pedidos
periodicamente, mas esperam receber mudanças de status em menos de 10 segundos.
O novo fluxo precisa preservar a transação atual de `changeStatus`, desacoplar
falhas externas e produzir um contrato verificável pelo consumidor.

Este documento descreve implementação futura. Nenhum modelo, endpoint, worker ou
teste de webhook existe hoje. Elementos marcados **Decisão** vêm da reunião;
elementos marcados **Proposta** completam lacunas necessárias e exigem revisão.

## 2. Objetivos técnicos

- Persistir o evento atomicamente com a transição do pedido.
- Entregar por HTTPS fora do processo da API, com polling de 2 segundos.
- Aplicar timeout, retry limitado, DLQ e replay auditável.
- Assinar o corpo com HMAC-SHA256 usando secret por endpoint.
- Oferecer at-least-once e uma chave estável de deduplicação.
- Reusar módulos, Prisma, Zod, AppError, Pino, paginação e autorização existentes.
- Medir se eventos elegíveis são entregues abaixo de 10 segundos em cenário
  controlado, sem transformar a meta em SLA universal.

## 3. Escopo e exclusões

### Incluído

- CRUD de endpoints e filtros de status por cliente.
- Rotação de secret com grace period de 24 horas.
- Inserção de snapshot na outbox durante `changeStatus`.
- Worker separado, histórico de entregas, retry, DLQ e replay ADMIN.
- Payload e headers outbound descritos na seção 8.

### Excluído ou adiado

- Webhooks inbound, dashboard visual e alerta por email.
- Rate limiting de saída, até haver dados de uso.
- Escala horizontal do worker e ordenação global.
- Política de retenção/arquivamento; “30 dias ou assim” foi uma sugestão fora de
  escopo, não um requisito.

## 4. Integração com o sistema existente

| Arquivo atual | Integração futura |
| --- | --- |
| `src/modules/orders/order.service.ts` | `changeStatus` chamará `publishWebhookEvent(tx, ...)` após validar a transição e antes do commit, usando o mesmo `Prisma.TransactionClient`. Falha na inserção causa rollback de status, histórico e estoque. |
| `src/modules/orders/order.status.ts` | `OrderStatus` e as transições válidas limitam os filtros cadastráveis e impedem evento para transição rejeitada. |
| `prisma/schema.prisma` | Receberá modelos de endpoint, outbox, entrega e DLQ. O padrão atual usa MySQL e UUID `Char(36)`. Hoje não há esses modelos nem relação `User`→`Customer`. |
| `src/middlewares/auth.middleware.ts` | `authenticate` protege todo o CRUD; `requireRole('ADMIN')` protege replay. O JWT fornece usuário/role, não `customer_id`. |
| `src/middlewares/validate.middleware.ts` | Schemas Zod validam params, body e query, incluindo UUID, statuses e URL HTTPS. |
| `src/middlewares/error.middleware.ts` | AppErrors do domínio usam o envelope atual; Zod continua respondendo `VALIDATION_ERROR`. |
| `src/shared/logger/index.ts` | Pino registra processamento e auditoria. A configuração futura deve impedir logs de secrets; o redactor atual não cobre `secret` genericamente. |
| `src/shared/http/response.ts` | Listas de endpoints e entregas reutilizam `{ data, pagination }`. |
| `src/app.ts` e `src/routes/index.ts` | `buildWebhookRouter` será agregado ao router existente, portanto todos os paths públicos começam com `/api/v1`. |
| `src/config/database.ts` e `src/server.ts` | `src/worker.ts` cria PrismaClient próprio, trata sinais e desconecta no shutdown. |

## 5. Modelagem proposta

Os nomes e campos abaixo são **proposta**; as capacidades e restrições vêm da
reunião.

### `WebhookEndpoint`

`id UUID`, `customerId UUID`, `url`, `secretCiphertext`, `previousSecretCiphertext
nullable`, `previousSecretExpiresAt nullable`, `subscribedStatuses JSON`, `active`,
`createdAt`, `updatedAt`. Índices por `customerId` e `active`. A forma de proteção
da secret em repouso depende da revisão de Segurança.

### `WebhookOutbox`

`id UUID`, `eventId UUID`, `webhookId UUID`, `customerId UUID`, `orderId UUID`,
`eventType`, `payload JSON`, `status`, `attemptCount`, `nextAttemptAt`,
`processingStartedAt`, `lastError nullable`, `createdAt`, `deliveredAt nullable`.
Índice composto por `status`, `nextAttemptAt`, `createdAt` e índice único por
`id`. `eventId` é a chave de idempotência do contrato; `id` identifica a entrada
operacional da outbox.

**Proposta:** criar uma linha por endpoint inscrito, pois URL, secret,
`X-Webhook-Id` e histórico pertencem ao destino. Na primeira inserção, gerar
`eventId` e `id`; tentativas e replay preservam `eventId`, enquanto um replay pode
receber novo `id` operacional. Se nenhum endpoint do cliente assina o novo
status, nenhuma linha é criada.

### `WebhookDelivery`

`id UUID`, `outboxId UUID`, `eventId UUID`, `webhookId UUID`, `attemptNumber`,
`startedAt`, `durationMs`, `httpStatus nullable`, `responseExcerpt nullable`,
`success`, `errorCode nullable`. A resposta armazenada deve ser limitada e
sanitizada.

### `WebhookDeadLetter`

`id UUID`, `eventId UUID`, `originalOutboxId UUID`, `webhookId UUID`, `payload
JSON`, `reason`, `failedAt`, `replayedAt nullable`, `replayedByUserId nullable`,
`replayedOutboxId nullable`. Replay cria nova entrada operacional pendente com
novo `outbox.id`, preserva o mesmo `eventId` e registra o usuário ADMIN.

## 6. Fluxos detalhados

### 6.1 Inserção atômica e filtro

1. `changeStatus` lê o pedido e valida a transição.
2. Débito/reposição de estoque, update e histórico continuam no mesmo `tx`.
3. Consultar endpoints ativos do `customerId` que assinam `toStatus`.
4. Para cada endpoint elegível, renderizar snapshot com UUID, dados do pedido e
   instante da transição; não incluir items.
5. Validar tamanho serializado máximo de 64 KB.
6. Inserir as linhas de outbox com `PENDING` e commit único.
7. Qualquer falha até o commit reverte todas as operações.

### 6.2 Claim e entrega pelo worker

1. A cada 2 segundos, o worker busca lote pequeno de `PENDING` cujo
   `nextAttemptAt <= now`, ordenado por `createdAt`.
2. **Proposta:** usar claim transacional/lock para mudar cada linha para
   `PROCESSING`, evitando processamento concorrente acidental.
3. Serializar exatamente o snapshot persistido, calcular HMAC e enviar por HTTPS
   com timeout de 10 segundos.
4. Registrar `WebhookDelivery` para sucesso, resposta não 2xx, timeout ou erro de
   transporte.
5. Em 2xx, marcar `DELIVERED`; nas demais condições, agendar retry ou mover para
   DLQ.

### 6.3 Retry e cardinalidade

Os intervalos decididos são `1m`, `5m`, `30m`, `2h`, `12h`, mas a reunião também
disse “5 tentativas”. Antes da implementação, escolher e testar uma interpretação:

- **Opção A:** cinco envios totais; usam-se somente quatro esperas e o quinto
  intervalo não participa.
- **Opção B:** envio inicial mais cinco retries; são seis envios totais.

Não é permitido implementar silenciosamente uma opção. Depois do limite, copiar
o snapshot e a falha final para `WebhookDeadLetter` e retirar o evento do conjunto
pendente.

### 6.4 Recuperação de processamento interrompido

**Proposta:** `PROCESSING` funciona como lease. No startup e em varredura
periódica, registros cujo `processingStartedAt` exceda o timeout operacional
voltam a `PENDING` sem incrementar tentativa até que haja evidência de envio.
Como o processo pode morrer após a chamada e antes do commit, a recuperação pode
duplicar; `X-Event-Id` permanece igual.

### 6.5 Ordenação

Single-worker e leitura por `createdAt` dão ordem de seleção, não garantia diante
de retry: o evento posterior pode ser entregue enquanto o anterior aguarda.
Política bloqueante por `orderId` preserva ordem e piora latência; política não
bloqueante favorece disponibilidade. Essa escolha permanece gate de implementação
e deve virar teste de integração. Não há ordenação global.

### 6.6 Replay

ADMIN solicita replay de uma DLQ. A API autentica, autoriza, registra `userId` e
cria nova entrada de outbox ligada à original. A entrada recebe novo `outbox.id`,
mas reutiliza o mesmo `eventId` no payload e em `X-Event-Id`. A operação é
at-least-once e pode duplicar processamento já concluído no consumidor; preservar
a chave permite que a deduplicação reconheça o mesmo evento lógico.

## 7. Contratos públicos propostos

Os paths e status abaixo são **proposta v1**, pois a reunião confirmou capacidades
mas não fechou todos os paths/códigos. Todos exigem `Authorization: Bearer <JWT>`.
O `customerId` vem do path; não é derivado do JWT. A política que autoriza um
usuário a operar determinado cliente precisa ser definida antes do código.

### 7.1 Criar endpoint

`POST /api/v1/customers/:customerId/webhooks`

```json
{
  "url": "https://cliente.example.com/hooks/orders",
  "statuses": ["SHIPPED", "DELIVERED"]
}
```

`201 Created` — a secret é gerada pelo servidor e exibida uma única vez:

```json
{
  "id": "7eb8e587-1ac5-4a4e-bda2-704caa6c5485",
  "customerId": "0bea457c-9191-4fb3-a7e0-4c4ae84695af",
  "url": "https://cliente.example.com/hooks/orders",
  "statuses": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "whsec_example_once"
}
```

### 7.2 Listar endpoints

`GET /api/v1/customers/:customerId/webhooks?page=1&pageSize=20`

Request sem corpo. `200 OK`:

```json
{
  "data": [{
    "id": "7eb8e587-1ac5-4a4e-bda2-704caa6c5485",
    "url": "https://cliente.example.com/hooks/orders",
    "statuses": ["SHIPPED", "DELIVERED"],
    "active": true
  }],
  "pagination": { "page": 1, "pageSize": 20, "total": 1, "totalPages": 1 }
}
```

### 7.3 Editar endpoint

`PATCH /api/v1/customers/:customerId/webhooks/:webhookId`

```json
{
  "url": "https://cliente.example.com/hooks/v2/orders",
  "statuses": ["PAID", "SHIPPED", "DELIVERED"],
  "active": true
}
```

`200 OK` devolve a configuração sem secret:

```json
{
  "id": "7eb8e587-1ac5-4a4e-bda2-704caa6c5485",
  "customerId": "0bea457c-9191-4fb3-a7e0-4c4ae84695af",
  "url": "https://cliente.example.com/hooks/v2/orders",
  "statuses": ["PAID", "SHIPPED", "DELIVERED"],
  "active": true
}
```

### 7.4 Remover endpoint

`DELETE /api/v1/customers/:customerId/webhooks/:webhookId`

Request e response sem corpo. `204 No Content`. **Proposta:** desativação lógica
preserva histórico; a política de retenção continua aberta.

### 7.5 Rotacionar secret

`POST /api/v1/customers/:customerId/webhooks/:webhookId/rotate-secret`

Request sem corpo. `200 OK`:

```json
{
  "webhookId": "7eb8e587-1ac5-4a4e-bda2-704caa6c5485",
  "secret": "whsec_new_example_once",
  "previousSecretExpiresAt": "2026-10-27T15:00:00.000Z"
}
```

A data é ilustrativa. O comportamento exato de assinatura com duas chaves durante
as 24 horas deve ser aprovado por Segurança.

### 7.6 Histórico de entregas

`GET /api/v1/customers/:customerId/webhooks/:webhookId/deliveries?page=1&pageSize=100`

Request sem corpo. `200 OK`:

```json
{
  "data": [{
    "eventId": "65755e08-3ad2-4a72-a0f7-713bc97bdf8c",
    "success": false,
    "httpStatus": 503,
    "durationMs": 842,
    "attemptNumber": 2,
    "createdAt": "2026-10-26T14:10:00.000Z"
  }],
  "pagination": { "page": 1, "pageSize": 100, "total": 1, "totalPages": 1 }
}
```

### 7.7 Replay de DLQ

`POST /api/v1/admin/webhooks/dead-letter/:id/replay`

Request sem corpo, `requireRole('ADMIN')`. `202 Accepted`:

```json
{
  "deadLetterId": "15562d80-1642-4907-a7fb-727e7a6323d5",
  "outboxId": "6d2f0f15-a11e-43d0-831e-2bb246b5dbb8",
  "eventId": "65755e08-3ad2-4a72-a0f7-713bc97bdf8c",
  "status": "PENDING"
}
```

O path verbalizado na reunião não continha prefixo; `/api/v1` decorre da montagem
atual em `src/app.ts`.

## 8. Contrato outbound

### Payload

```json
{
  "event_id": "65755e08-3ad2-4a72-a0f7-713bc97bdf8c",
  "event_type": "order.status_changed",
  "timestamp": "2026-10-26T14:10:00.000Z",
  "order_id": "f041e180-c540-4b88-82b8-b8327fdd6b92",
  "order_number": "ORD-000123",
  "from_status": "PROCESSING",
  "to_status": "SHIPPED",
  "customer_id": "0bea457c-9191-4fb3-a7e0-4c4ae84695af",
  "total_cents": 12990
}
```

`items` não é enviado. O consumidor consulta `GET /orders/:id` para detalhes. O
JSON serializado precisa ter no máximo 64 KB; acima disso, o evento falha com erro
explícito, sem truncamento.

### Headers

```http
Content-Type: application/json
X-Event-Id: 65755e08-3ad2-4a72-a0f7-713bc97bdf8c
X-Webhook-Id: 7eb8e587-1ac5-4a4e-bda2-704caa6c5485
X-Timestamp: 2026-10-26T14:10:00.000Z
X-Signature: v1=<hex-hmac-sha256>
```

**Decisão:** HMAC-SHA256 cobre o corpo, com secret por endpoint. **Proposta:**
`v1=<hex>` é apenas um encoding candidato. Como `X-Timestamp` não faz parte do
corpo, ele não está autenticado por essa decisão. O encoding, a mensagem assinada,
a distribuição de secrets e o envio durante a janela com duas chaves precisam de
revisão de Segurança. Transporte HTTPS é obrigatório.

## 9. Matriz de erros

| Código | HTTP proposto | Situação | Origem |
| --- | ---: | --- | --- |
| `WEBHOOK_NOT_FOUND` | 404 | Endpoint ou DLQ do domínio não encontrado | Domínio futuro |
| `WEBHOOK_INVALID_URL` | 400 | URL não usa HTTPS ou é inválida | Domínio futuro/Zod |
| `WEBHOOK_INVALID_STATUS_FILTER` | 400 | Filtro vazio ou status desconhecido | Domínio futuro/Zod |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | 422 | Snapshot serializado excede 64 KB | Domínio futuro |
| `WEBHOOK_SECRET_REQUIRED` | 422 | Material de chave esperado está indisponível | Domínio futuro |
| `WEBHOOK_REPLAY_NOT_ALLOWED` | 409 | Entrada não está elegível para replay | Domínio futuro |
| `WEBHOOK_DELIVERY_TIMEOUT` | N/A | Worker excede 10s; registra tentativa e retry | Processamento interno |

Erros compartilhados permanecem `VALIDATION_ERROR` (Zod), `UNAUTHORIZED`,
`FORBIDDEN`, `CONFLICT` e erros Prisma conforme middleware atual. “Prefixo
`WEBHOOK_`” aplica-se aos erros próprios do domínio; mudar códigos compartilhados
exigiria um mapper novo e não foi decidido.

## 10. Resiliência e fallback

- Timeout HTTP: 10 segundos; timeout conta como falha de entrega.
- Retry: intervalos decididos, cardinalidade pendente conforme seção 6.3.
- DLQ: tabela separada e replay manual ADMIN.
- Duplicatas: aceitas; consumidor deduplica por `X-Event-Id`.
- Falha da API: transação sem commit não deixa evento órfão.
- Falha do worker: lease vencida recupera processamento, podendo duplicar.
- Falha do consumidor: não reverte pedido e segue retry/DLQ.
- Fallback por email não integra esta fase; não há fallback obrigatório além da
  persistência e do replay.

## 11. Observabilidade

Os nomes abaixo são **proposta de instrumentação**; Pino existe hoje, mas não há
infraestrutura de métricas ou tracing comprovada no repositório.

### Métricas

- `webhook_outbox_pending` (gauge) e idade do pendente mais antigo.
- `webhook_delivery_total{outcome,status_class}` (counter).
- `webhook_delivery_duration_ms` (histogram).
- `webhook_retry_scheduled_total` e `webhook_dlq_total` (counters).
- `webhook_end_to_end_latency_ms` do status confirmado ao 2xx.

### Logs

Eventos estruturados: claim, tentativa, retry agendado, entrega, ida à DLQ, replay
e recuperação de lease. Campos permitidos incluem `eventId`, `webhookId`,
`orderId`, tentativa, duração, status HTTP e `requestId`. Nunca registrar secret,
assinatura completa, Authorization ou payload/resposta sem sanitização. Replay
inclui `userId` para auditoria.

### Tracing

**Proposta:** propagar `eventId` como correlação entre transação, worker e
delivery; criar spans `webhook.enqueue` e `webhook.deliver` quando uma biblioteca
de tracing for escolhida. Não há tracing instalado hoje e este FDD não seleciona
fornecedor.

## 12. Dependências e compatibilidade

- MySQL e Prisma atuais; migração aditiva, sem alterar endpoints existentes.
- Node.js/TypeScript, Express, Zod e Pino já presentes.
- Cliente HTTP e mecanismo de proteção de secret ainda devem ser escolhidos.
- O worker usa a mesma `DATABASE_URL`, mas PrismaClient próprio.
- Consumidores precisam aceitar JSON, HTTPS, HMAC-SHA256 e duplicatas.
- O vínculo de autorização usuário→cliente não existe; é dependência de desenho.
- Revisão de Segurança de HMAC e geração de secret requer dois dias úteis antes
  do deploy, dentro da estimativa de três sprints.

## 13. Critérios de aceite técnicos e testes futuros

Nenhum teste de webhook foi executado, pois a feature não está implementada. A
implementação futura deverá provar:

1. rollback conjunto quando a inserção da outbox falha;
2. ausência de outbox para transição inválida ou filtro não inscrito;
3. snapshot preservado mesmo se o pedido mudar depois;
4. `eventId` estável em todas as tentativas e replays, mesmo quando o replay cria
   novo `outbox.id`, permitindo deduplicação;
5. timeout em 10s, sequência de retry escolhida e transferência para DLQ;
6. recuperação de `PROCESSING` interrompido sem perder evento;
7. política escolhida para ordenação durante retry;
8. rejeição de HTTP e payload acima de 64 KB;
9. HMAC validável, rotação durante 24h e expiração da secret anterior;
10. CRUD e deliveries para usuário autenticado; replay negado a OPERATOR e
    permitido/auditado para ADMIN;
11. medição da meta abaixo de 10 segundos com carga, backlog e percentil
    declarados pelo teste, sem extrapolar para SLA universal;
12. ausência de secrets em logs e respostas fora de criação/rotação.

## 14. Riscos e mitigação

| Risco | Efeito | Mitigação proposta |
| --- | --- | --- |
| Retry implementado com cardinalidade errada | Entregas a mais ou janela menor | Fechar Opção A/B e congelar com teste de agendamento |
| Evento posterior ultrapassa anterior | Consumidor observa estados fora de ordem | Escolher política por pedido e documentar limite |
| Backlog rompe meta de latência | Cliente volta a fazer polling | Métricas de fila, teste sob carga e capacidade do lote |
| Secret aparece em log | Comprometimento de endpoint | Redaction explícita, revisão de Segurança e testes de log |
| Usuário opera cliente indevido | Violação de autorização | Definir vínculo/política antes da implementação |
| Worker morre após envio e antes do commit | Duplicata | Event ID estável e deduplicação do consumidor |

## 15. Referências

- [RFC](RFC.md)
- [Índice de ADRs](adrs/README.md)
- [Tracker](TRACKER.md)
