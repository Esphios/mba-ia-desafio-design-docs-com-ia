# PRD — Sistema de Webhooks de Notificação de Pedidos

## 1. Resumo e contexto

Atlas Comercial, MaxDistribuição e Nova Cargo precisam ser avisadas quando pedidos
mudam de status. Hoje elas consultam `GET /orders` repetidamente, o que torna a
integração lenta e cara. A feature oferece webhooks outbound configuráveis para
substituir esse polling e considera abaixo de 10 segundos uma experiência de
“tempo real” no cenário de validação.

A entrega de produto inclui APIs de configuração e consulta, notificações
assinadas, histórico, recuperação de falhas e operação administrativa. A solução
detalhada está no [FDD](FDD.md); decisões arquiteturais estão nos
[ADRs](adrs/README.md).

## 2. Problema e motivação

- Clientes não recebem mudanças proativamente e mantêm polling periódico.
- Polling aumenta custo e atraso de integração nos dois lados.
- A Atlas indicou risco de migração para concorrente se a capacidade não chegar
  até o fim do trimestre; a reunião não registrou data nem ano desse trimestre.
- Falha do sistema de um cliente não pode impedir a operação de pedidos no OMS.

## 3. Público-alvo e cenários de uso

### Integradores B2B

- Cadastram um endpoint HTTPS para um cliente.
- Escolhem quais mudanças de status desejam receber.
- Validam a assinatura, deduplicam eventos e consultam detalhes do pedido quando
  necessário.
- Rotacionam credenciais e investigam o histórico de entregas.

### Operação e administradores do OMS

- Operadores autenticados mantêm configurações nesta fase.
- Administradores inspecionam falhas e disparam replay auditado da DLQ.
- Engenharia e Segurança observam latência, backlog, duplicatas e uso de secrets.

## 4. Objetivos e métricas de sucesso

| Objetivo | Meta | Como validar |
| --- | --- | --- |
| Reduzir atraso percebido | Evento elegível entregue em menos de 10 segundos no cenário de aceite | Medir do commit da mudança ao primeiro 2xx, declarando carga, backlog e percentil observado |
| Evitar perda entre pedido e notificação | 100% das mudanças elegíveis confirmadas devem possuir registro de outbox no teste de atomicidade | Testes de commit e rollback na mesma transação |
| Permitir recuperação | 100% dos eventos que esgotarem a política escolhida devem aparecer na DLQ e aceitar replay ADMIN no teste | Teste de falhas, DLQ e auditoria |
| Tornar duplicatas tratáveis | 100% das tentativas do mesmo evento usam o mesmo `X-Event-Id` | Teste de retry e replay |

A única meta de latência pedida pelos clientes é “abaixo de 10 segundos”. O PRD
não inventa SLA, disponibilidade ou percentil. As demais porcentagens são
critérios determinísticos propostos para os testes de integridade da feature, não
metas comerciais atribuídas à reunião.

## 5. Escopo

### Incluído

- Webhooks outbound para mudanças de status de pedidos.
- Configuração por cliente e filtro por lista de statuses.
- Secret exclusiva por endpoint, HMAC-SHA256 e rotação.
- Histórico das últimas 100 entregas.
- Retry limitado, DLQ separada e replay administrativo.
- Entrega at-least-once com identificação estável do evento.

### Fora de escopo

- **Alerta por email** após falhas; adiado para uma fase futura.
- **Dashboard visual**; projeto separado do time de frontend.
- **Rate limiting de saída**; será observado e decidido se virar problema.
- **Webhooks inbound**, escala horizontal do worker e garantia de ordenação global.
- **Retenção de 30 dias**; a reunião citou “30 dias ou assim” como ideia fora da
  feature, sem fechar política.

## 6. Requisitos funcionais

- **RF-01 — Cadastrar endpoint:** usuário autenticado informa `customer_id`, URL
  HTTPS e statuses desejados; o OMS gera e devolve a secret na criação.
- **RF-02 — Listar endpoints:** usuário autenticado lista as configurações de um
  cliente sem expor secrets.
- **RF-03 — Editar endpoint:** usuário autenticado altera URL, filtros e estado.
- **RF-04 — Remover endpoint:** usuário autenticado remove/desativa uma
  configuração.
- **RF-05 — Rotacionar secret:** uma nova secret é emitida e a anterior permanece
  válida por 24 horas.
- **RF-06 — Filtrar na origem:** somente endpoints ativos inscritos no novo status
  geram linha na outbox.
- **RF-07 — Registrar atomicamente:** snapshot do evento é persistido na mesma
  transação da mudança de status.
- **RF-08 — Entregar notificação:** worker separado envia o snapshot aos destinos
  por HTTPS.
- **RF-09 — Consultar histórico:** cliente consulta até os últimos 100 envios com
  resultado, payload, resposta e duração, sujeitos a sanitização.
- **RF-10 — Retentar falhas:** timeout ou resposta malsucedida agenda backoff;
  a cardinalidade exata precisa ser resolvida antes da implementação.
- **RF-11 — Encaminhar à DLQ:** falha permanente é preservada em tabela separada.
- **RF-12 — Reprocessar DLQ:** somente ADMIN dispara replay, com identidade do
  executor registrada.
- **RF-13 — Identificar duplicatas:** todas as tentativas carregam o mesmo UUID em
  `X-Event-Id`; consumidor deduplica por ele.
- **RF-14 — Expor contrato de entrega:** payload traz mudança de status e dados
  básicos, sem items, junto aos headers de evento, webhook, timestamp e assinatura.

## 7. Requisitos não funcionais

- **RNF-01 — Latência:** buscar entrega abaixo de 10 segundos no cenário de aceite;
  backlog e destino lento são limites explícitos.
- **RNF-02 — Segurança:** somente HTTPS; HMAC-SHA256 por endpoint; secret nunca em
  logs e exibida apenas quando emitida, conforme proposta do FDD.
- **RNF-03 — Tamanho:** corpo serializado até 64 KB; excedente falha sem truncar.
- **RNF-04 — Resiliência:** timeout de 10 segundos, retry limitado e DLQ.
- **RNF-05 — Semântica:** at-least-once; exactly-once não é prometido.
- **RNF-06 — Compatibilidade:** reutilizar MySQL, Prisma, Node/TypeScript, Express,
  Zod, AppError e Pino atuais.
- **RNF-07 — Observabilidade:** medir backlog, latência, resultados, retries e DLQ;
  correlacionar logs sem credenciais.

## 8. Decisões e trade-offs principais

- [Outbox no MySQL](adrs/ADR-001-outbox-no-mysql.md) troca infraestrutura nova por
  operação de tabela/worker e mantém atomicidade local.
- [Worker separado com polling](adrs/ADR-002-worker-separado-com-polling.md) reduz
  acoplamento à API, mas consulta o banco periodicamente.
- [Retry e DLQ](adrs/ADR-003-retry-com-backoff-e-dlq.md) limitam eventos pendentes,
  mas exigem fechar a contagem de tentativas e aceitar possível quebra de ordem.
- [HMAC e rotação](adrs/ADR-004-hmac-e-rotacao-de-secret.md) isolam credenciais,
  mas adicionam custódia e janela com duas chaves.
- [At-least-once](adrs/ADR-005-entrega-at-least-once.md) evita perda silenciosa ao
  custo de deduplicação pelo consumidor.

## 9. Dependências

- Modelagem e migração Prisma para endpoint, outbox, histórico e DLQ.
- Worker implantável como processo separado no mesmo banco.
- Escolha da cardinalidade de retry e política de ordenação por pedido.
- Definição da autorização entre usuário autenticado e `customer_id`, ausente no
  schema atual.
- Revisão de Segurança para HMAC, geração, armazenamento e rotação de secrets; a
  reunião reservou pelo menos dois dias úteis antes do deploy.
- Documentação para consumidores sobre HMAC e deduplicação.

## 10. Riscos e mitigação

Probabilidade e impacto são avaliações deste PRD, não valores declarados na
reunião.

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Backlog ou destino lento impede meta abaixo de 10s | Média | Alto | Medir fila/latência, testar sob carga e dimensionar lote/worker |
| Consumidor não deduplica | Média | Alto | Destacar at-least-once, fornecer exemplo e manter UUID estável |
| Secret vaza em log | Baixa | Alto | Redaction explícita, respostas restritas e revisão de Segurança |
| Retry contraditório gera envio a mais/menos | Alta até decisão | Médio | Fechar cinco envios ou cinco retries e cobrir com teste |
| Operador acessa cliente indevido | Média até política | Alto | Definir vínculo/autorização de `customer_id` antes de implementar |
| Evento posterior ultrapassa um anterior | Média | Médio | Escolher política de ordem por pedido e documentar o efeito na latência |

## 11. Critérios de aceitação

1. CRUD autenticado permite criar, listar, editar, remover e rotacionar endpoint
   com `customer_id` explícito e URL HTTPS.
2. Transição elegível e inscrita grava snapshot na mesma transação; rollback não
   deixa pedido ou outbox parcialmente persistidos.
3. Worker separado entrega JSON e todos os headers definidos, com HMAC verificável.
4. Payload maior que 64 KB falha sem truncamento.
5. Falha e timeout seguem a política aprovada; esgotamento cria DLQ.
6. Histórico expõe até 100 entregas sem secret e replay é ADMIN/auditado.
7. Retry/replay preserva `X-Event-Id`; documentação alerta para duplicatas.
8. Rotação funciona por 24 horas conforme contrato aprovado por Segurança.
9. Cenário de latência mede e reporta se o resultado ficou abaixo de 10 segundos,
   com carga e percentil descritos.
10. Email, dashboard e rate limiting não aparecem como capacidades desta fase.

## 12. Estratégia de testes e validação

- **Unidade:** filtros, tamanho, geração de evento, backoff escolhido, HMAC,
  rotação, autorização e mapeamento de erros.
- **Integração com banco:** commit/rollback, claim, lease vencida, DLQ, replay e
  snapshot imutável.
- **Integração HTTP:** 2xx, não 2xx, timeout, headers, duplicatas e HTTPS.
- **Autorização:** CRUD autenticado, replay negado a OPERATOR e permitido a ADMIN,
  além da política futura de usuário por cliente.
- **Desempenho:** medir latência ponta a ponta com carga e backlog declarados.
- **Segurança:** revisão de secret/HMAC, tentativa de URL HTTP e inspeção de logs.

Estes são testes planejados. A feature ainda não existe e nenhum desses cenários
foi executado nesta entrega documental.
