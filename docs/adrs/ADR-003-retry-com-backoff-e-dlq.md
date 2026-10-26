# ADR-003 — Retry com backoff e DLQ

## Status

Decisão parcialmente fechada na reunião. A política, os intervalos e a DLQ foram
confirmados; a cardinalidade exata entre tentativa inicial e cinco intervalos
permanece aberta e bloqueia a implementação literal do agendamento.

## Contexto

Destinos podem ficar indisponíveis por horas. Retry indefinido manteria eventos
pendentes para sempre, enquanto apenas três tentativas poderiam encerrar a
entrega durante uma manutenção conhecida. Falhas permanentes precisam sair do
caminho normal sem perder evidência nem impedir replay manual.

## Decisão

Aplicar retry com os intervalos `1m`, `5m`, `30m`, `2h` e `12h`; ao atingir o
limite, mover o evento para uma tabela `webhook_dead_letter` separada, preservando
payload, motivo e timestamp. Um endpoint administrativo poderá recolocar uma
entrada na outbox como pendente.

A reunião também fixou “5 tentativas”. Cinco intervalos após a primeira tentativa
produzem seis envios; cinco envios totais usam apenas quatro intervalos. O time
deve escolher uma das duas semânticas antes de codificar. O FDD apresenta ambas
e os testes que devem congelar a decisão.

## Alternativas Consideradas

- **Retry indefinido:** descartado por manter eventos eternamente pendentes quando
  um cliente abandona o endpoint.
- **Três tentativas:** descartado por cobrir mal indisponibilidades de algumas
  horas.
- **Marcar `failed` apenas na outbox:** descartado em favor de DLQ separada, que
  mantém a leitura de pendentes mais clara e oferece evidência para diagnóstico.

## Consequências

### Positivas

- Falhas transitórias têm uma janela longa para recuperação.
- Falhas permanentes ficam isoladas e ainda podem ser auditadas e reprocessadas.
- O número de envios é limitado depois que a cardinalidade for decidida.

### Negativas

- A ambiguidade de cinco tentativas versus cinco intervalos exige decisão antes
  da implementação.
- Replay pode gerar nova duplicata e precisa de autorização e auditoria.
- Um evento em retry pode ser ultrapassado por evento posterior do mesmo pedido.

Rastreabilidade: `[09:15] Diego`, `[09:16] Bruno/Diego`, `[09:17] Diego/Larissa`,
`[09:18] Diego`, `[09:48] Larissa`.
