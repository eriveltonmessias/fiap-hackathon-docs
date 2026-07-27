# ADR-001 — EKS provisionado com Terraform

- Status: aceita
- Data: 2026-07-27

## Contexto

A solução possui aplicações, bancos, mensageria, gateway e observabilidade
containerizados. O ambiente precisa ser recriado e destruído com pouca operação
manual.

## Decisão

Executar os workloads em Amazon EKS e declarar a infraestrutura em Terraform.
Cada repositório é responsável pelos recursos de sua camada.

## Alternativas

- ECS reduziria a administração de Kubernetes, mas exigiria adaptar manifests,
  descoberta e charts de observabilidade.
- Máquinas EC2 com Docker Compose seriam simples, porém teriam menor isolamento,
  descoberta e automação de rollout.
- Criação manual do EKS não seria reproduzível.

## Consequências

- O ambiente completo pode ser recriado pelas pipelines.
- A equipe reutiliza Kubernetes Services, probes, Secrets e charts Helm.
- EKS aumenta o tempo de provisionamento e exige controle da ordem entre
  stacks.
