# ADR-010 — Notificação com link autenticado do Kong

- Status: aceita
- Data: 2026-07-27

## Contexto

O usuário não mantém uma conexão aberta durante o processamento e precisa saber
quando o resultado está disponível.

## Decisão

Depois de confirmar o resultado no banco, o Manager consulta no Auth a
preferência do cliente e envia e-mail ou Telegram. A mensagem de sucesso contém
o endereço público atual do Kong para o download, que continua exigindo JWT.

## Alternativas

- Polling é mantido como possibilidade pela consulta de status, mas exige ação
  contínua do cliente.
- URL pré-assinada permitiria download sem JWT, porém criaria outro modelo de
  autorização e expiração.
- Link direto para o Service do Manager contornaria o gateway.

## Consequências

- O endereço externo não é fixado manualmente; Terraform descobre o NLB.
- O link respeita a entrada única do Kong e a autorização do proprietário.
- A indisponibilidade do SMTP não muda um processamento já concluído; a falha de
  notificação é registrada separadamente.
