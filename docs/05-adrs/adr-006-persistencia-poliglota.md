# ADR-006 — PostgreSQL e MongoDB por responsabilidade

- Status: aceita
- Data: 2026-07-27

## Contexto

Clientes e o ciclo público dos vídeos exigem restrições relacionais e
transações. Jobs do Worker acumulam estados técnicos e metadados de
processamento com estrutura própria.

## Decisão

Usar PostgreSQL separado para Auth e Manager, e MongoDB para os jobs e a outbox
do Worker.

## Alternativas

- Um PostgreSQL compartilhado facilitaria a operação, mas acoplaria schemas e
  ciclos de implantação.
- MongoDB para tudo reduziria tecnologias, porém perderia a modelagem relacional
  natural de clientes e outbox do Manager.
- Banco gerenciado reduziria administração, mas elevaria o custo do ambiente
  acadêmico temporário.

## Consequências

- Cada serviço possui seus dados e evolui o schema de forma independente.
- O Worker persiste documentos sem compartilhar tabelas.
- A plataforma precisa operar três datastores no cluster.
- Não existem joins ou transações entre microsserviços.
