# ADR-007 — S3 para entrada e resultado

- Status: aceita
- Data: 2026-07-27

## Contexto

Vídeos e arquivos ZIP são binários grandes e não devem trafegar pelo Kafka nem
ser armazenados em bancos transacionais.

## Decisão

Armazenar originais no bucket de entrada e ZIPs no bucket de saída. Eventos
transportam somente as chaves dos objetos. Localmente, MinIO implementa a mesma
API compatível com S3.

## Alternativas

- Banco de dados aumentaria volume, backup e custo de consultas.
- Volume compartilhado no Kubernetes acoplaria pods à infraestrutura de
  arquivos.
- Enviar binários no Kafka aumentaria mensagens e retenção do broker.

## Consequências

- Manager e Worker transferem arquivos por streaming.
- O contrato de eventos permanece pequeno.
- Buckets possuem ciclo de vida separado dos bancos.
- Credenciais e nomes de bucket precisam ser consistentes entre as aplicações.
