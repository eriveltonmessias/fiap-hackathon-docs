# Infraestrutura

| Documento | Objetivo |
| --- | --- |
| [Ordem de execução](ordem-execucao.md) | Sequência de bootstrap, apply, deploy e destroy. |
| [Terraform state](terraform-state.md) | Backend centralizado e comunicação entre stacks. |

O princípio operacional é: somente a base do backend é criada por bootstrap;
todo recurso de execução da solução é criado e destruído pelas pipelines
Terraform responsáveis.
