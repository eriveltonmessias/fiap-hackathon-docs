# ADR-003 — Kong DB-less como entrada única

- Status: aceita
- Data: 2026-07-27

## Contexto

Somente Auth, Manager e Grafana precisam de acesso externo. Worker, Actuator,
endpoints internos e componentes de dados não devem ganhar Load Balancers
próprios.

## Decisão

Usar Kong em modo DB-less, com configuração declarativa versionada, publicado
por um único Service `LoadBalancer` do tipo NLB.

## Alternativas

- Um Load Balancer por aplicação aumentaria custo e espalharia regras de borda.
- Ingress nativo atenderia roteamento básico, mas exigiria componentes
  adicionais para rate limit, correlation ID e transformação de headers.
- Kong com PostgreSQL permitiria configuração dinâmica, porém adicionaria banco,
  migrations e ciclo de vida desnecessários.

## Consequências

- Rotas, rate limits, CORS e limites de payload são revisados em Git.
- Não existe interface administrativa pública.
- Alterações de rota exigem novo apply da configuração.
- Um único gateway concentra a disponibilidade da entrada externa.
