# ADR-002 — State remoto e lock nativo no S3

- Status: aceita
- Data: 2026-07-27

## Contexto

As pipelines executam Terraform de máquinas efêmeras e stacks dependentes
precisam consumir outputs de infraestrutura e dados.

## Decisão

Centralizar os states em um bucket S3 versionado e criptografado, usando uma
chave por repositório e `use_lockfile = true`.

## Alternativas

- State local seria perdido no runner e impediria comunicação confiável entre
  stacks.
- Um único state para todo o projeto aumentaria acoplamento e o impacto de
  alterações.
- DynamoDB para lock funcionaria, mas o backend S3 atual já oferece lockfile
  nativo e elimina um recurso adicional.

## Consequências

- Pipelines compartilham outputs sem copiar endpoints manualmente.
- Locks impedem apply concorrente no mesmo state.
- O bucket é o único recurso prévio e persistente.
- A ordem de destroy precisa respeitar as dependências entre states.
