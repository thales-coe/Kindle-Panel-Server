# Security Policy

## Escopo
Este projeto roda em ambiente doméstico/local e pode consumir dados pessoais, como calendários e identificadores de rede. Por isso, qualquer publicação do código deve tratar segredos e identificadores com cuidado.

## Nunca commitar
- `credentials.json`
- `token.json`
- `.env`
- logs com dados pessoais
- IDs sensíveis, prints ou exports de agenda

## Divulgação responsável
Se você identificar um risco de segurança neste projeto, trate primeiro a rotação dos segredos afetados e só depois publique correções.

## Boas práticas mínimas
- usar `.gitignore` desde o primeiro commit;
- revisar `git diff` antes do push;
- manter dependências atualizadas;
- evitar reuso de credenciais entre ambientes;
- preferir aliases a e-mails pessoais em metadados públicos.
