# Fluxo de processamento

## Sequência principal

```mermaid
sequenceDiagram
    actor U as Usuário
    participant K as Kong
    participant A as Customer Auth
    participant M as Video Manager
    participant S1 as S3 Input
    participant Q as Kafka
    participant W as Video Worker
    participant DB as MongoDB
    participant S2 as S3 Output
    participant SMTP as Gmail

    U->>K: POST /customers
    K->>A: cadastro
    U->>K: POST /auth/login
    K->>A: autenticação
    A-->>U: JWT
    U->>K: POST /videos + Bearer JWT
    K->>M: upload
    M->>S1: grava vídeo original
    M->>M: grava vídeo + outbox
    M-->>U: 202 PENDING_PROCESSING
    M->>Q: VideoProcessingRequested
    Q->>W: entrega pedido
    W->>DB: registra job idempotente
    W->>S1: baixa vídeo
    W->>W: FFmpeg extrai frames
    W->>W: compacta ZIP
    W->>S2: grava resultado
    W->>DB: conclui job + outbox
    W->>Q: VideoProcessed
    Q->>M: entrega resultado
    M->>M: atualiza para PROCESSED
    M->>A: consulta preferência interna
    M->>SMTP: envia link do Kong
    U->>K: GET /videos/{id}/download
    K->>M: download autenticado
    M->>S2: lê ZIP
    M-->>U: frames.zip
```

Em uma falha terminal, o Worker publica `VideoProcessingFailed`; o Manager muda
o vídeo para `FAILED` e envia a notificação de falha sem disponibilizar ZIP.

## Estado público do vídeo

```mermaid
stateDiagram-v2
    [*] --> RECEIVED
    RECEIVED --> STORED: upload salvo
    STORED --> PENDING_PROCESSING: pedido persistido
    PENDING_PROCESSING --> PROCESSING: processamento em andamento
    PROCESSING --> PROCESSED: VideoProcessed
    PENDING_PROCESSING --> PROCESSED: VideoProcessed
    RECEIVED --> FAILED
    STORED --> FAILED
    PENDING_PROCESSING --> FAILED: VideoProcessingFailed
    PROCESSING --> FAILED: VideoProcessingFailed
    PROCESSED --> [*]
    FAILED --> [*]
```

O estado `PROCESSING` existe no domínio, mas o contrato atual do Worker não
publica um evento de início. Por isso, no fluxo implementado, o resultado pode
levar o vídeo diretamente de `PENDING_PROCESSING` para `PROCESSED`.

## Estado interno do Worker

```mermaid
stateDiagram-v2
    [*] --> RECEIVED
    RECEIVED --> PROCESSING
    PROCESSING --> GENERATING_FRAMES
    GENERATING_FRAMES --> COMPRESSING
    COMPRESSING --> UPLOADING_RESULT
    UPLOADING_RESULT --> COMPLETED
    RECEIVED --> FAILED
    PROCESSING --> FAILED
    GENERATING_FRAMES --> FAILED
    COMPRESSING --> FAILED
    UPLOADING_RESULT --> FAILED
```
