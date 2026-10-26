# ADR-004 — HMAC e rotação de secret

## Status

Decisão registrada na reunião para algoritmo, isolamento da secret e janela de
rotação. Encoding da assinatura, custódia da chave e rotações sucessivas são
propostas deste documento e ainda requerem revisão de Segurança.

## Contexto

O consumidor precisa verificar origem e integridade do corpo enviado para fora da
infraestrutura. Uma credencial global ampliaria o impacto de vazamento. Clientes
também precisam trocar a credencial sem interrupção abrupta.

## Decisão

Assinar o corpo HTTP exato com HMAC-SHA256 usando uma secret exclusiva por
endpoint e enviar o resultado em `X-Signature`. Na rotação, gerar uma nova secret
e aceitar a anterior por 24 horas; depois da janela, somente a nova permanece
válida. O cadastro aceita apenas HTTPS.

Como proposta sujeita à revisão de Segurança, a implementação deve retornar a
secret apenas na criação/rotação, armazená-la protegida em repouso e usar formato
versionado como `v1=<hex>`. O consumidor deve verificar o corpo bruto recebido.
`X-Timestamp` foi pedido para detecção de replay, mas assinar apenas o corpo não
autentica esse header; ampliar a mensagem assinada é uma questão aberta.

## Alternativas Consideradas

- **Secret global:** discutida implicitamente e rejeitada porque o vazamento
  comprometeria todos os clientes.
- **Troca imediata sem grace period:** alternativa plausível, rejeitada por criar
  indisponibilidade durante a migração do consumidor.
- **Assinar corpo mais timestamp:** alternativa plausível a revisar; protege o
  timestamp, mas muda o contrato explicitamente fechado como HMAC sobre o corpo.

## Consequências

### Positivas

- Comprometimento de uma secret fica limitado a um endpoint.
- Consumidores podem verificar integridade e origem do corpo.
- A janela de 24 horas permite migração gradual.

### Negativas

- Duas secrets válidas elevam a complexidade temporária e exigem expiração.
- HMAC não cifra os dados; HTTPS continua obrigatório.
- Logs e respostas precisam impedir exposição da secret.
- Proteção completa contra replay depende de contrato adicional para timestamp.

Rastreabilidade: `[09:20] Sofia`, `[09:21] Sofia`, `[09:22] Sofia`,
`[09:23] Sofia`, `src/shared/logger/index.ts`.
