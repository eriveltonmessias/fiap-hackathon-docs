# FIAP X — Documentação

Documentação central da solução de processamento assíncrono de vídeos do
Hackathon FIAP X. Cada repositório mantém os detalhes de implementação do seu
componente; este repositório concentra a visão integrada, as dependências e as
decisões arquiteturais.

## Repositórios em ordem de execução

| Ordem | Repositório | Responsabilidade |
| ---: | --- | --- |
| 1 | [fiap-hackathon-infra](https://github.com/eriveltonmessias/fiap-hackathon-infra) | Faz o bootstrap do backend Terraform e cria rede, EKS, ECR, Kong e observabilidade. O bootstrap do bucket ocorre uma única vez; apply e destroy da infraestrutura são automatizados. |
| 2 | [fiap-hackathon-data-platform](https://github.com/eriveltonmessias/fiap-hackathon-data-platform) | Cria namespaces, PostgreSQL, MongoDB, Kafka, tópicos e buckets S3 de entrada e saída. Consome o state da infraestrutura. |
| 3 | [fiap-hackathon-customer-auth-api](https://github.com/eriveltonmessias/fiap-hackathon-customer-auth-api) | Cadastra e autentica clientes, emite JWT e fornece internamente a preferência de notificação. |
| 4 | [fiap-hackathon-video-worker-api](https://github.com/eriveltonmessias/fiap-hackathon-video-worker-api) | Consome pedidos Kafka, extrai frames com FFmpeg, gera o ZIP e publica o resultado do processamento. |
| 5 | [fiap-hackathon-video-manager-api](https://github.com/eriveltonmessias/fiap-hackathon-video-manager-api) | Recebe uploads, controla o estado do vídeo, publica pedidos, recebe resultados e envia notificações com o link de download. |

Os três microsserviços podem ser implantados na mesma fase depois da plataforma
de dados. A ordem acima é a recomendada para que as dependências do fluxo
completo já estejam disponíveis quando o Manager for publicado.

## Documentação

| Área | Conteúdo |
| --- | --- |
| [Arquitetura](docs/01-arquitetura/README.md) | HLD, LLD, fluxo ponta a ponta e estados. |
| [Integrações](docs/02-integracoes/README.md) | APIs públicas e internas, eventos Kafka e regras de idempotência. |
| [Infraestrutura](docs/03-infraestrutura/README.md) | Ordem operacional, Terraform state e dependências entre stacks. |
| [Observabilidade](docs/04-observabilidade/README.md) | Métricas, logs, traces, dashboards e acesso ao Grafana. |
| [ADRs](docs/05-adrs/README.md) | Decisões arquiteturais, alternativas e consequências. |

## Escopo

Esta documentação descreve somente a arquitetura técnica implementada.
