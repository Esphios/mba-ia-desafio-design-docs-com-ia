# RFC — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| Autor | Eduardo Lima |
| Status | Em revisão — ainda não submetido nem aprovado |
| Data | 2026-10-26 |
| Revisores propostos | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenharia de Pedidos), Diego (Plataforma), Sofia (Segurança) |

## TL;DR

Propõe-se substituir o polling feito por clientes B2B por webhooks outbound de
mudança de status. A API grava um snapshot do evento em uma outbox MySQL dentro da
mesma transação da mudança do pedido. Um worker separado consulta a outbox a cada
2 segundos e entrega por HTTPS, com retry, DLQ, HMAC-SHA256 e semântica
at-least-once. Configurações são geridas por endpoints autenticados; replay de DLQ
é restrito a ADMIN e auditado.

Esta proposta formaliza o consenso da reunião, mas ainda precisa resolver a
contagem exata de retries, a política diante de ordenação versus disponibilidade e
detalhes do contrato criptográfico.

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo consultam repetidamente `GET
/orders` para descobrir mudanças. O processo é lento e caro para os clientes; eles
consideram uma notificação em menos de 10 segundos como tempo real. A integração é
unidirecional: o OMS envia eventos e não recebe webhooks.

O envio não pode ocorrer dentro de `OrderService.changeStatus`: essa operação já
coordena status, histórico e estoque em uma transação. Latência ou falha externa
não deve bloquear nem reverter a mudança válida do pedido.

## Proposta técnica

1. **Persistência atômica.** Para cada endpoint ativo do cliente interessado no
   novo status, a transação de `changeStatus` insere um evento com UUID e snapshot
   do payload na outbox. Se não houver assinatura para o status, não há inserção.
2. **Processamento desacoplado.** Um único worker, em processo Node.js separado,
   usa PrismaClient próprio e consulta pendentes em lotes a cada 2 segundos.
3. **Entrega resiliente.** Chamadas HTTPS têm timeout de 10 segundos. Falhas entram
   na sequência de backoff acordada e, no limite, são persistidas em DLQ separada.
4. **Segurança.** Cada endpoint tem secret própria. O corpo é assinado com
   HMAC-SHA256; a rotação aceita a chave anterior durante 24 horas.
5. **Semântica de entrega.** A garantia é at-least-once. O UUID estável segue no
   payload e em `X-Event-Id`; deduplicação é responsabilidade do consumidor.
6. **Operação.** CRUD e histórico de entregas usam autenticação existente. Replay
   manual exige role ADMIN e registra quem o executou.

O desenho detalhado de tabelas, estados, payloads, endpoints, erros e testes está
no [FDD](FDD.md), não neste RFC.

## Alternativas consideradas

### Envio HTTP síncrono no fluxo de pedidos

Descartado porque um endpoint lento prolongaria a transação e uma indisponibilidade
externa criaria o dilema incorreto de reverter uma mudança de status já válida.

### Redis Streams/Cluster

Descartado nesta fase. Ofereceria uma infraestrutura dedicada de eventos, mas
adicionaria operação e custo considerados excessivos para o time atual; o MySQL
existente já pode sustentar a outbox.

### Trigger MySQL para acordar o worker

Descartado porque triggers executam SQL e não notificam nativamente um processo
Node.js. Contornar isso com arquivo ou endpoint seria mais frágil que polling.

## Questões em aberto e itens adiados

1. **Cardinalidade do retry:** “5 tentativas” e cinco intervalos não definem se há
   cinco ou seis envios. A implementação aguarda essa escolha.
2. **Ordenação durante retry:** single-worker preserva a leitura por criação, mas
   um evento em backoff pode ser ultrapassado. Bloquear o pedido preserva ordem e
   aumenta latência; permitir avanço favorece disponibilidade.
3. **Assinatura e replay attack:** a reunião decidiu HMAC sobre o corpo e enviou
   `X-Timestamp`; falta decidir se uma versão futura da assinatura cobre também o
   timestamp e qual será seu encoding.
4. **Rate limiting de saída:** será observado em produção e decidido depois; não
   faz parte desta fase.

Também estão adiados: retenção/arquivamento de entregues, alerta por email e
dashboard visual.

## Impacto e riscos

- **Schema e operação:** novas tabelas para configuração, outbox, entregas e DLQ,
  além de um processo implantável separado.
- **API:** novos endpoints sob `/api/v1`; nenhuma mudança no contrato atual de
  pedidos é necessária para consumidores existentes.
- **Latência:** polling de 2 segundos ajuda a meta abaixo de 10 segundos, mas não
  a garante sob backlog, timeout ou destino lento. A validação precisa declarar o
  cenário de carga.
- **Duplicatas:** são esperadas; documentação de integração deve ensinar
  deduplicação por `X-Event-Id`.
- **Segurança:** secrets não podem entrar em logs; Sofia deve revisar HMAC e geração
  de chaves antes do deploy. A reunião reservou dois dias úteis para essa revisão.
- **Isolamento por cliente:** o JWT atual identifica operador, não cliente. O
  `customer_id` precisa vir no path/body e a política de autorização entre ambos
  permanece uma lacuna a resolver antes da implementação.

## Decisões relacionadas

- [ADR-001 — Outbox no MySQL](adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002 — Worker separado com polling](adrs/ADR-002-worker-separado-com-polling.md)
- [ADR-003 — Retry com backoff e DLQ](adrs/ADR-003-retry-com-backoff-e-dlq.md)
- [ADR-004 — HMAC e rotação de secret](adrs/ADR-004-hmac-e-rotacao-de-secret.md)
- [ADR-005 — Entrega at-least-once](adrs/ADR-005-entrega-at-least-once.md)
- [ADR-006 — Reuso dos padrões do OMS](adrs/ADR-006-reuso-dos-padroes-do-oms.md)

## Rastreabilidade principal

Motivação e latência: `[09:00]`–`[09:02] Marcos`; outbox e alternativas:
`[09:03]`–`[09:09]`; worker e ordenação: `[09:09]`–`[09:13]`; retry/DLQ:
`[09:14]`–`[09:19]`; segurança e entrega: `[09:19]`–`[09:26]`; padrões e API:
`[09:27]`–`[09:41]`; payload, prazo e decisões finais: `[09:42]`–`[09:52]`.
