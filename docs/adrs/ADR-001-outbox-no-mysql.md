# ADR-001 — Outbox no MySQL

## Status

Decisão registrada na reunião técnica; ADR preparado para revisão, sem aprovação
formal posterior documentada.

## Contexto

Uma mudança de status hoje atualiza o pedido, registra o histórico e, em algumas
transições, movimenta estoque dentro de `OrderService.changeStatus`. Enviar HTTP
nessa transação faria a disponibilidade e a latência do cliente externo afetarem
uma operação central do OMS. Também seria incorreto reverter uma mudança válida
apenas porque o destino do webhook está indisponível.

## Decisão

Persistir um evento em `webhook_outbox` no MySQL existente, dentro da mesma
transação Prisma usada por `changeStatus`. O evento será um snapshot renderizado
do estado no instante da transição. Se qualquer escrita falhar, a transação
inteira sofre rollback; após commit, um worker independente cuida da entrega.

A integração futura deve aceitar o `Prisma.TransactionClient` já aberto pelo
service, por exemplo em `publishWebhookEvent(tx, order, fromStatus, toStatus)`,
para evitar uma segunda transação.

## Alternativas Consideradas

- **HTTP síncrono no service:** descartado porque um destino lento ou fora do ar
  prolongaria ou comprometeria a mudança de status.
- **Redis Streams/Cluster:** descartado nesta fase porque adicionaria
  infraestrutura operacional para um time pequeno.

## Consequências

### Positivas

- Status, histórico, estoque e registro do evento compartilham atomicidade.
- A API deixa de depender da disponibilidade do consumidor para confirmar a
  mudança do pedido.
- O MySQL e a transação Prisma existentes são reutilizados.

### Negativas

- A outbox exige índices, limpeza, monitoramento e um worker de entrega.
- Atomicidade de persistência não equivale a entrega exactly-once; duplicatas e
  retries continuam possíveis.
- A retenção de eventos entregues não foi definida nesta fase.

Rastreabilidade: `[09:04] Bruno`, `[09:06] Diego`, `[09:07] Larissa/Diego`,
`[09:40] Bruno`, `[09:41] Diego`, `src/modules/orders/order.service.ts`.
