# HLD — High Level Design

```mermaid
flowchart LR
    U["Usuário"] -->|HTTP| K["AWS NLB<br/>Kong Gateway"]
    K -->|rotas públicas| A["Customer Auth API"]
    K -->|rotas públicas| M["Video Manager API"]
    K -->|/grafana| G["Grafana"]

    A --> PA[("PostgreSQL Auth")]
    M --> PV[("PostgreSQL Video")]
    M -->|arquivo original| SI[("S3 Input")]
    M -->|VideoProcessingRequested| KF["Kafka"]
    KF -->|pedido| W["Video Worker API"]
    W --> MW[("MongoDB Worker")]
    W -->|leitura| SI
    W -->|ZIP de frames| SO[("S3 Output")]
    W -->|VideoProcessed ou<br/>VideoProcessingFailed| KF
    KF -->|resultado| M
    M -->|preferência de notificação| A
    M -->|SMTP| GM["Gmail"]
    M -->|download autenticado| SO

    A -. métricas e traces .-> O["Prometheus / Jaeger"]
    M -. métricas e traces .-> O
    W -. métricas e traces .-> O
    A -. stdout .-> L["Alloy / Loki"]
    M -. stdout .-> L
    W -. stdout .-> L
    G --> O
    G --> L
```

## Responsabilidades

- O Kong é a única entrada pública e encaminha tráfego para Services
  Kubernetes, nunca diretamente para IPs de pods.
- O Customer Auth mantém clientes e emite JWT HS256.
- O Manager é a API de negócio e a fonte de verdade do estado público do vídeo.
- O Worker executa o processamento assíncrono e não possui endpoint público.
- Kafka desacopla upload e processamento.
- S3 guarda o vídeo original e o ZIP resultante.
- PostgreSQL mantém dados transacionais; MongoDB mantém os jobs do Worker.
- Grafana consulta Prometheus, Loki e Jaeger.

## Limites de acesso

O Kong publica somente cadastro, login, consulta do próprio cliente, operações
de vídeo e Grafana. Endpoints `/internal`, Actuator, Worker, bancos, Kafka e
MongoDB permanecem internos ao cluster.
