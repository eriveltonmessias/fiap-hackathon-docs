# ADR-009 — Prometheus, Grafana, Loki, Alloy e Jaeger

- Status: aceita
- Data: 2026-07-27

## Contexto

O fluxo atravessa HTTP, Kafka, storage e três aplicações. A apresentação precisa
demonstrar processamento paralelo e localizar falhas sem adicionar uma
plataforma operacional complexa.

## Decisão

Prometheus coleta métricas; Alloy coleta stdout e envia ao Loki; as aplicações
enviam traces OTLP diretamente ao Jaeger; Grafana centraliza dashboards e
consultas. A retenção é temporária.

## Alternativas

- Somente `kubectl logs` não correlacionaria o fluxo completo.
- OpenTelemetry Collector centralizaria exportadores, mas adicionaria um salto e
  configuração sem benefício necessário neste ambiente.
- Serviços gerenciados de observabilidade aumentariam custo e credenciais.

## Consequências

- Métricas, logs e traces ficam acessíveis em uma interface.
- Logs usam texto legível e IDs no contexto.
- Métricas evitam IDs como labels.
- A recriação dos pods pode apagar o histórico.
