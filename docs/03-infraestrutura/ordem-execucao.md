# Ordem de execução

## Criação

| Etapa | Repositório | Ação |
| ---: | --- | --- |
| 0 | Infra | Executar uma única vez `bootstrap/bootstrap-terraform-backend.sh` para criar o bucket do backend. |
| 1 | Infra | Pipeline Terraform cria VPC, EKS, ECR, Kong e observabilidade. |
| 2 | Data Platform | Pipeline lê o state da Infra e cria dados, mensageria, namespaces e buckets de vídeo. |
| 3 | Customer Auth | Pipeline testa, publica imagem ECR e aplica recursos Kubernetes. |
| 4 | Video Worker | Pipeline testa, publica imagem ECR e aplica o Worker interno. |
| 5 | Video Manager | Pipeline testa, publica imagem ECR, aplica a API e descobre o hostname atual do Kong para notificações. |

## Validação mínima

```bash
aws eks update-kubeconfig --region us-east-1 --name fiapx-eks
kubectl get nodes
kubectl get pods -n data-plataform
kubectl get pods -n fiap-x-apps
kubectl get pods -n observability
kubectl get service kong-proxy -n kong
```

## Destruição

Execute na ordem inversa:

1. Video Manager.
2. Video Worker.
3. Customer Auth.
4. Data Platform.
5. Infra.

O bucket de backend e seu histórico de state permanecem para permitir novas
criações sem preparação manual. Os buckets de vídeo pertencem à Data Platform e
são recriados com essa stack.
