# ADR-004 — Processamento assíncrono com Kafka

- Status: aceita
- Data: 2026-07-27

## Contexto

Extração de frames e compactação podem durar mais que uma requisição HTTP.
Vários usuários devem conseguir enviar vídeos sem manter conexões abertas.

## Decisão

O Manager aceita o upload, retorna `202` e publica o pedido no Kafka. O Worker
processa de forma assíncrona e publica sucesso ou falha. Os tópicos têm três
partições e usam `videoId` como chave.

## Alternativas

- Processamento síncrono simplificaria o fluxo, mas aumentaria timeout e
  acoplamento entre cliente e FFmpeg.
- Chamada HTTP do Manager para o Worker manteria dependência temporal e exigiria
  retry distribuído.
- SQS seria adequado na AWS, mas Kafka permite o mesmo contrato no ambiente
  local e demonstra partições e grupos consumidores.

## Consequências

- Upload e processamento escalam de forma independente.
- Vídeos diferentes podem ser processados em paralelo e eventos do mesmo vídeo
  mantêm ordem por partição.
- A consistência é eventual e o cliente consulta o status.
- Retries, DLQ e idempotência tornam-se obrigatórios.
