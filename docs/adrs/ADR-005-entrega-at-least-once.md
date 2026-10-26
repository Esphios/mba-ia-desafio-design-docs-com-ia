# ADR-005 — Entrega at-least-once

## Status

Decisão registrada na reunião técnica; ADR preparado para revisão, sem aprovação
formal posterior documentada.

## Contexto

Uma falha pode ocorrer depois que o consumidor processou a chamada, mas antes que
o worker registre sucesso. Repetir é necessário para evitar perda, porém pode
entregar o mesmo evento novamente. Coordenação exactly-once entre organizações é
mais complexa que o escopo desta fase.

## Decisão

Oferecer entrega at-least-once. O evento recebe `eventId` estável quando entra na
outbox, propagado no campo `event_id` e no header `X-Event-Id` em todas as
tentativas e replays. A entrada operacional da outbox possui `id` próprio: replay
pode criar novo `outbox.id`, mas preserva o `eventId` lógico. O consumidor é
responsável por deduplicar por esse valor.

A garantia cobre persistência atômica e repetição conforme a política de retry;
ela não garante que todo destino eventualmente aceitará a entrega, não elimina a
DLQ e não oferece exactly-once.

## Alternativas Consideradas

- **Exactly-once:** descartado porque exigiria coordenação transacional ou
  protocolo adicional entre OMS e cada consumidor.
- **At-most-once:** alternativa plausível descartada porque uma falha transitória
  poderia perder a única tentativa.

## Consequências

### Positivas

- Uma resposta perdida não causa perda silenciosa do evento.
- UUID estável dá ao consumidor uma chave simples de deduplicação.
- O modelo funciona com retry e replay de DLQ.

### Negativas

- Consumidores precisam persistir IDs processados e tornar o handler idempotente.
- Duplicatas continuam observáveis durante retry e replay.
- A documentação pública do contrato deve destacar essa responsabilidade.

Rastreabilidade: `[09:24] Diego`, `[09:25] Diego/Sofia`, `[09:26] Marcos/Larissa`,
`[09:51] Larissa`.
