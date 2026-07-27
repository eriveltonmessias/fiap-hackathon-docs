# APIs

## Rotas públicas no Kong

| Método | Caminho | Destino | Autenticação |
| --- | --- | --- | --- |
| `POST` | `/customers` | Customer Auth | Pública |
| `POST` | `/auth/login` | Customer Auth | Pública |
| `GET` | `/customers/{customerId}` | Customer Auth | Bearer JWT; somente o próprio cliente |
| `POST` | `/videos` | Video Manager | Bearer JWT |
| `GET` | `/videos` | Video Manager | Bearer JWT |
| `GET` | `/videos/{videoId}` | Video Manager | Bearer JWT e propriedade do vídeo |
| `GET` | `/videos/{videoId}/download` | Video Manager | Bearer JWT, proprietário e status `PROCESSED` |
| Vários | `/grafana/**` | Grafana | Login do Grafana |

Kong aplica correlation ID em `X-Request-ID`, CORS, limite de requisições por
IP, limite de payload e headers defensivos. Rotas não declaradas retornam
`404`.

## Contrato interno

| Método | Caminho | Origem | Destino | Proteção |
| --- | --- | --- | --- | --- |
| `GET` | `/internal/customers/{customerId}/notification-preference` | Video Manager | Customer Auth | `X-Internal-Api-Key` |

Esse endpoint retorna o canal, e-mail e identificador de chat necessários para
a notificação. Ele não é exposto pelo Kong.

## Respostas relevantes

- Upload aceito: `202 Accepted` com `videoId` e `PENDING_PROCESSING`.
- Download antes da conclusão: `409 Conflict`.
- Vídeo inexistente ou pertencente a outro cliente: `404 Not Found`.
- Token ausente ou inválido: `401 Unauthorized`.
- Recurso de outro cliente: a API não revela sua existência.
