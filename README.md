# Design Docs — Sistema de Webhooks de Pedidos

## Sobre o desafio

Este repositório transforma uma reunião técnica e o código de um Order Management
System em um pacote de documentação acionável para uma futura feature de webhooks
outbound. A entrega separa produto, proposta arquitetural, decisões e desenho de
implementação, com rastreabilidade até a transcrição ou o código.

O trabalho é exclusivamente documental: a aplicação, o schema atual, os testes e
a transcrição foram preservados. O [enunciado original](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia)
está no repositório-base público.

## Ferramentas de IA utilizadas

- **Codex:** leitura da transcrição e do código, geração dos documentos e revisão
  cruzada de contratos, fontes, links e critérios.
- **OpenSpec:** organização da mudança em proposta, design e tarefas, além do
  acompanhamento e da validação estrutural do workflow.

Git, ripgrep e PowerShell apoiaram verificações de baseline, caminhos, contagens e
consistência. A IA produziu rascunhos; afirmações materiais só permaneceram depois
de confronto com as fontes.

## Workflow adotado

1. Registrei a revisão-base e os arquivos protegidos antes da edição.
2. Classifiquei a reunião em requisitos, decisões, alternativas, exclusões e
   pendências; em paralelo, mapeei os pontos reais de integração no OMS.
3. Escrevi os seis ADRs e iniciei o Tracker com decisões atômicas.
4. Consolidei a arquitetura no RFC, mantendo alternativas e questões abertas.
5. Detalhei fluxos, contratos, erros, resiliência e testes futuros no FDD.
6. Produzi o PRD na altura de produto, usando os documentos técnicos como base.
7. Auditei o Tracker linha a linha e recalculei cobertura e proporção de fontes.
8. Revisei fronteiras, links, paginação do RFC e preservação do repositório.

## Prompts customizados

### Inventário antes da redação

```text
Leia integralmente a transcrição e os pontos reais de integração no OMS. Separe
cada item em requisito, decisão, alternativa, exclusão, pendência ou fato do
código. Para cada item, registre uma fonte precisa. Não promova sugestões a
decisões, inclua as falas finais sobre UUID e snapshot e destaque contradições.
```

### ADRs com fronteiras de evidência

```text
Produza seis ADRs, um por decisão principal, com Status, Contexto, Decisão,
Alternativas Consideradas e Consequências positivas e negativas. Atribua como
decisão somente o que a reunião fechou; rotule alternativas meramente plausíveis
e preserve questões sem resposta. Cite código real apenas após conferir caminho
e símbolo.
```

### FDD acionável sem fabricar decisões

```text
Detalhe modelagem, fluxos, contratos, payload, erros, resiliência,
observabilidade, integração e testes futuros. Marque como proposta todo path,
status HTTP, campo ou algoritmo que a reunião não fechou. Exponha retry,
ordenação, customer_id, assinatura e recuperação interrompida como limites
explícitos; não alegue implementação ou testes existentes.
```

## Iterações e ajustes

Foram nove iterações principais de produção e revisão:

1. **Inventário:** a frase “5 tentativas” parecia compatível com os cinco
   intervalos, mas cinco esperas após o primeiro envio produzem seis tentativas.
   A documentação expõe as duas interpretações e exige decisão antes do código.
2. **ADRs:** detalhes como encoding da assinatura e armazenamento de secrets
   apareciam como decisões. Eles foram reclassificados como propostas sujeitas à
   revisão de Segurança.
3. **RFC:** contratos detalhados duplicavam o FDD. O texto foi reduzido à visão
   arquitetural, alternativas, questões abertas, impactos e riscos.
4. **FDD:** ordenar a leitura por `createdAt` parecia garantir ordem de entrega,
   mas retry pode permitir ultrapassagem. A escolha entre ordem e latência ficou
   registrada como gate de implementação.
5. **PRD:** metas técnicas determinísticas poderiam parecer SLAs comerciais. O
   texto separa a meta de latência dita pelos clientes dos critérios de teste
   propostos.
6. **Tracker:** afirmações compostas sobre HMAC/timestamp e headers foram
   desagregadas; propostas derivadas ganharam linhas próprias sem serem
   apresentadas como decisões da reunião.
7. **README e RFC:** o relato público foi sanitizado e o RFC foi renderizado em
   A4 para confirmar a extensão de duas páginas sem duplicar o FDD.
8. **Consistência final:** métricas, spans e convenções HTTP ainda combinavam
   fontes distintas no Tracker; as linhas foram novamente desagregadas.
9. **Validação independente:** a identidade lógica do replay foi separada do ID
   operacional da outbox, e o inventário do Tracker foi ampliado para incluir
   consequências, impactos, riscos, aceite, testes e dependências antes omitidos.

## Como navegar a entrega

Ordem sugerida de leitura:

1. [PRD](docs/PRD.md) — problema, público, escopo, requisitos e sucesso.
2. [RFC](docs/RFC.md) — proposta arquitetural para revisão e questões abertas.
3. [Índice de ADRs](docs/adrs/README.md) — seis decisões e seus trade-offs.
4. [FDD](docs/FDD.md) — modelagem, fluxos, contratos e estratégia de testes.
5. [Tracker](docs/TRACKER.md) — origem de cada afirmação material.

O RFC está preparado para revisão, não aprovado. A feature descrita ainda não foi
implementada; exemplos e testes do FDD são contratos e cenários futuros.
