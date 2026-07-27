# Eventos Kafka

## Tópicos

| Tópico | Produtor | Consumidor | Partições |
| --- | --- | --- | ---: |
| `video.processing.requested` | Video Manager | Video Worker | 3 |
| `video.processing.completed` | Video Worker | Video Manager | 3 |
| `video.processing.failed` | Video Worker | Video Manager | 3 |

As mensagens usam `videoId` como chave Kafka para preservar a ordem dos eventos
do mesmo vídeo. Os três consumidores podem processar vídeos de clientes
diferentes em paralelo.

## VideoProcessingRequested

```json
{
  "eventId": "5fbe33b1-bc03-4cd2-9798-3297d50c2100",
  "eventType": "VideoProcessingRequested",
  "occurredAt": "2026-07-27T18:00:00Z",
  "videoId": "9a7cb4c3-fe03-4a9c-a377-30a8843b2331",
  "customerId": "f12d504c-8fac-476d-8edb-3cf56cf902f4",
  "originalFilename": "video.mp4",
  "inputObjectKey": "customers/f12d504c/videos/9a7cb4c3/video.mp4"
}
```

## VideoProcessed

```json
{
  "eventId": "8f728013-762a-42a2-91bf-73fbb3b3c1c4",
  "eventType": "VideoProcessed",
  "occurredAt": "2026-07-27T18:01:00Z",
  "videoId": "9a7cb4c3-fe03-4a9c-a377-30a8843b2331",
  "outputObjectKey": "customers/f12d504c/videos/9a7cb4c3/frames.zip"
}
```

## VideoProcessingFailed

```json
{
  "eventId": "f555d92b-e82e-4a9e-a683-95b75303295e",
  "eventType": "VideoProcessingFailed",
  "occurredAt": "2026-07-27T18:01:00Z",
  "videoId": "9a7cb4c3-fe03-4a9c-a377-30a8843b2331",
  "failureReason": "Unable to extract frames"
}
```

## Confiabilidade

- O Manager grava o vídeo e o pedido na mesma transação de banco usando
  Transactional Outbox.
- O Worker grava o estado do job e o resultado a publicar no MongoDB.
- `eventId` identifica duplicatas; consumidores devem ser idempotentes.
- A publicação pode ser repetida quando o ACK do Kafka e a confirmação local
  não terminam juntos.
- Mensagens que esgotam as tentativas seguem para uma DLQ com sufixo `.dlq`.
- IDs aparecem em logs e traces para correlação, mas não como labels de
  métricas para evitar alta cardinalidade.
