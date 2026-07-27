# ADR-005 — Outbox e consumidores idempotentes

- Status: aceita
- Data: 2026-07-27

## Contexto

Gravar estado no banco e publicar no Kafka são operações independentes. Uma
falha entre elas poderia perder uma solicitação ou publicar o mesmo evento mais
de uma vez.

## Decisão

Persistir eventos em outbox junto com a mudança de estado e publicá-los em
segundo plano. Consumidores registram `eventId` e tratam repetições como
duplicatas.

## Alternativas

- Publicar diretamente após o commit deixaria uma janela de perda.
- Transação distribuída teria complexidade incompatível com Kafka e os bancos
  usados.
- Aceitar duplicatas sem registro poderia repetir processamento e notificação.

## Consequências

- Não há perda silenciosa entre persistência e mensageria.
- A entrega efetiva é pelo menos uma vez, não exatamente uma vez.
- Tabelas e coleções adicionais guardam outbox e eventos processados.
- Dispatchers e limpeza de registros precisam ser monitorados.
