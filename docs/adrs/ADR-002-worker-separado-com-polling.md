# ADR-002 — Worker separado com polling

## Status

Decisão registrada na reunião técnica; ADR preparado para revisão, sem aprovação
formal posterior documentada.

## Contexto

Os eventos persistidos na outbox precisam ser entregues sem acoplar seu ciclo de
vida ao processo HTTP. O MySQL não oferece um mecanismo equivalente ao
`LISTEN/NOTIFY` para acordar um processo externo. O requisito de produto considera
menos de 10 segundos como tempo real, embora isso não constitua garantia sob
backlog ou destino lento.

## Decisão

Executar um worker Node.js em processo separado da API. Ele consulta a cada 2
segundos os eventos pendentes mais antigos, em lote pequeno, e atualiza seu
estado. O processo usa a mesma `DATABASE_URL`, mas cria seu próprio
`PrismaClient`, como exige a separação entre processos.

A proposta de integração é uma nova entry point `src/worker.ts`, com a lógica de
processamento no futuro módulo `src/modules/webhooks/`. Deve haver desligamento
gracioso e recuperação de registros que ficaram em processamento após interrupção.

## Alternativas Consideradas

- **Worker dentro da API:** descartado porque reinícios e escala da API acoplariam
  o processamento de eventos às instâncias HTTP.
- **Trigger MySQL para notificação externa:** descartado porque a trigger executa
  SQL, mas não notifica de forma nativa o processo Node.js.

## Consequências

### Positivas

- Falhas e reinícios do worker não bloqueiam requisições da API.
- Polling de 2 segundos é simples e compatível com a infraestrutura atual.
- Cada processo administra explicitamente sua conexão e seu shutdown.

### Negativas

- Há consultas periódicas mesmo sem trabalho.
- Polling de 2 segundos, isoladamente, não garante entrega abaixo de 10 segundos.
- A ordenação por pedido só é implícita com um worker e pode ser quebrada por
  retries ou por paralelismo futuro; não existe garantia de ordenação global.

Rastreabilidade: `[09:09] Diego`, `[09:09] Bruno`, `[09:10] Larissa`,
`[09:11] Diego/Larissa`, `[09:12] Diego`, `[09:13] Larissa`, `[09:30] Bruno`,
`src/config/database.ts`, `src/server.ts`.
