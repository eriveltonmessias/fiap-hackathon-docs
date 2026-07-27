# ADR-008 — JWT para usuários e API key interna

- Status: aceita
- Data: 2026-07-27

## Contexto

As APIs públicas precisam identificar o proprietário de cada vídeo. O Manager
também precisa consultar a preferência de notificação sem agir como um usuário.

## Decisão

Customer Auth emite JWT HS256 com o cliente no subject. Auth e Manager
compartilham secret e issuer. A consulta interna usa uma API key distinta no
header `X-Internal-Api-Key` e não é roteada pelo Kong.

## Alternativas

- Sessão no servidor adicionaria armazenamento e acoplamento ao Auth em cada
  requisição.
- OAuth gerenciado reduziria código próprio, mas adicionaria configuração e
  custo fora do escopo.
- Reutilizar JWT de usuário na chamada interna confundiria identidade do
  cliente com identidade de serviço.

## Consequências

- Validação de requisições públicas é stateless.
- Auth e Manager devem receber exatamente o mesmo secret e issuer.
- Rotação do secret exige coordenação.
- A API key interna protege o contrato de serviço, mas não substitui uma
  identidade de workload em ambientes de produção.
