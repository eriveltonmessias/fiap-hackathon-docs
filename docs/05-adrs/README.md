# Architecture Decision Records

Os ADRs abaixo registram retrospectivamente decisões já refletidas na
implementação. As justificativas recompõem o contexto técnico de cada decisão.

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](adr-001-eks-e-terraform.md) | EKS provisionado com Terraform | Aceita |
| [ADR-002](adr-002-state-e-lock-no-s3.md) | State remoto e lock nativo no S3 | Aceita |
| [ADR-003](adr-003-kong-dbless.md) | Kong DB-less como entrada única | Aceita |
| [ADR-004](adr-004-processamento-assincrono-kafka.md) | Processamento assíncrono com Kafka | Aceita |
| [ADR-005](adr-005-outbox-e-idempotencia.md) | Outbox e consumidores idempotentes | Aceita |
| [ADR-006](adr-006-persistencia-poliglota.md) | PostgreSQL e MongoDB por responsabilidade | Aceita |
| [ADR-007](adr-007-storage-s3.md) | S3 para entrada e resultado | Aceita |
| [ADR-008](adr-008-autenticacao-e-comunicacao-interna.md) | JWT para usuários e API key interna | Aceita |
| [ADR-009](adr-009-observabilidade.md) | Prometheus, Grafana, Loki, Alloy e Jaeger | Aceita |
| [ADR-010](adr-010-notificacao-com-link.md) | Notificação com link autenticado do Kong | Aceita |

Formato adotado: contexto, decisão, alternativas e consequências.
