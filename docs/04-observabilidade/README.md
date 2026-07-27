# Observabilidade

## Componentes

| Componente | Responsabilidade | Exposição |
| --- | --- | --- |
| Prometheus | Coleta métricas técnicas e de negócio a cada 15 segundos. | Interna |
| Grafana | Dashboard `FIAP X - Fluxo de Vídeo` e exploração dos dados. | Kong em `/grafana` |
| Alloy | Coleta logs de stdout dos pods como DaemonSet. | Interna |
| Loki | Armazena e consulta logs por até 48 horas. | Interna |
| Jaeger | Recebe e consulta traces OTLP enviados pelas aplicações. | Interna; usado como datasource do Grafana |

## Acesso ao Grafana

```bash
KONG_HOST="$(
  kubectl get service kong-proxy \
    --namespace kong \
    --output jsonpath='{.status.loadBalancer.ingress[0].hostname}'
)"

echo "http://${KONG_HOST}/grafana"

kubectl get secret monitoring-grafana \
  --namespace observability \
  --output jsonpath='{.data.admin-password}' |
  base64 --decode
echo
```

Usuário: `admin`.

## Correlação

| Identificador | Uso |
| --- | --- |
| `X-Request-ID` | Correlação da requisição no Kong. |
| `traceId` e `spanId` | Navegação entre traces e logs. |
| `customerId` | Demonstra fluxos simultâneos de clientes diferentes. |
| `videoId` | Acompanha um processamento específico. |
| `eventId` | Identifica a mensagem Kafka e suporta idempotência. |

Os logs das aplicações são texto legível, com pares `chave=valor` nos eventos
de negócio. Verbosidade de OpenTelemetry, Kafka administrativo e bibliotecas é
reduzida para não ocultar o fluxo.

## Métricas de negócio

Entre as métricas expostas pelas aplicações:

- clientes cadastrados e tentativas de login;
- uploads e resultados recebidos pelo Manager;
- notificações enviadas ou com falha;
- jobs iniciados, concluídos e com falha no Worker;
- duração do processamento;
- frames gerados e tentativas de retry.

IDs de cliente, vídeo e evento não são labels de Prometheus. Eles continuam
disponíveis em logs e traces, evitando séries de alta cardinalidade.

## Consultas úteis no Grafana/Loki

```logql
{namespace="fiap-x-apps"}
```

```logql
{namespace="fiap-x-apps"} |= "customerId="
```

```logql
{namespace="fiap-x-apps"} |= "videoId="
```

Para demonstrar paralelismo, filtre pelo intervalo da apresentação e compare
linhas de `video-worker-api` com `customerId` ou `videoId` diferentes.

## Retenção

Logs, métricas e traces usam armazenamento temporário para reduzir custo no
ambiente acadêmico. A recriação dos pods de observabilidade pode apagar o
histórico.
