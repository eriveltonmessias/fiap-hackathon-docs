# LLD — Low Level Design

```mermaid
flowchart TB
    subgraph AWS["AWS us-east-1"]
        S3T[("S3 Terraform state<br/>lockfile nativo")]
        VPC["VPC e 3 subnets públicas"]
        EKS["EKS + managed node group"]
        ECR["ECR<br/>3 imagens"]
        NLB["Network Load Balancer"]

        subgraph KONG["namespace kong"]
            KG["Kong DB-less<br/>Service LoadBalancer"]
        end

        subgraph APP["namespace fiap-x-apps"]
            AUTH["customer-auth-api<br/>HTTP 8081"]
            MAN["video-manager-api<br/>HTTP 8082"]
            WORK["video-worker-api<br/>8083 interno"]
        end

        subgraph DATA["namespace data-plataform"]
            P1[("postgres-auth")]
            P2[("postgres-video")]
            MO[("mongo-worker")]
            KA["Kafka<br/>3 tópicos / 3 partições"]
        end

        subgraph OBS["namespace observability"]
            PR["Prometheus"]
            LO["Loki"]
            AL["Alloy DaemonSet"]
            JA["Jaeger"]
            GR["Grafana"]
        end

        B1[("S3 vídeos input")]
        B2[("S3 vídeos output")]
    end

    S3T -. remote state .-> EKS
    VPC --> EKS
    EKS --> KONG
    EKS --> APP
    EKS --> DATA
    EKS --> OBS
    ECR --> APP
    NLB --> KG
    KG --> AUTH
    KG --> MAN
    KG --> GR
    MAN --> KA
    KA --> WORK
    KA --> MAN
    MAN --> B1
    WORK --> B1
    WORK --> B2
    MAN --> B2
    AUTH --> P1
    MAN --> P2
    WORK --> MO
    AL --> LO
    APP -. scrape .-> PR
    APP -. OTLP/HTTP .-> JA
```

## Stacks e states

| Stack | State |
| --- | --- |
| Infraestrutura | `infra/terraform.tfstate` |
| Plataforma de dados | `data-platform/terraform.tfstate` |
| Customer Auth | `apps/customer-auth-api/terraform.tfstate` |
| Video Worker | `apps/video-worker-api/terraform.tfstate` |
| Video Manager | `apps/video-manager-api/terraform.tfstate` |

As stacks dependentes leem os outputs remotos de infraestrutura e dados para
descobrir cluster, namespace, ECR e buckets. Isso elimina cópia manual desses
endereços.

## Protocolos

| Origem | Destino | Protocolo |
| --- | --- | --- |
| Usuário | Kong/NLB | HTTP |
| Kong | Auth, Manager e Grafana | HTTP via DNS de Service Kubernetes |
| Auth/Manager | PostgreSQL | JDBC |
| Worker | MongoDB | MongoDB |
| Manager/Worker | Kafka | protocolo Kafka |
| Manager/Worker | S3 | HTTPS / API compatível com S3 |
| Manager | Auth interno | HTTP + `X-Internal-Api-Key` |
| Manager | Gmail | SMTP com STARTTLS |
| Prometheus | aplicações | HTTP `/actuator/prometheus` |
| Aplicações | Jaeger | OTLP HTTP |
| Alloy | Loki | Loki Push API |

## Descoberta dos pods

Kong, aplicações e componentes de dados usam nomes de Services Kubernetes.
Quando um pod é substituído, o controlador do Service atualiza seus endpoints.
O consumidor não precisa conhecer IP, nome ou quantidade de pods.
