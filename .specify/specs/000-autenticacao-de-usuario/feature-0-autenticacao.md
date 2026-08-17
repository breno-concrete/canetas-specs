# Feature 0: Autenticação e Controle de Acesso (Login Básico)

**Stack:** Quarkus

## User Story
Como usuário do sistema (Agente de Investigação ou Delegado), preciso me autenticar para acessar as funcionalidades do sistema, de forma que minhas ações fiquem vinculadas à minha identidade nos registros de auditoria.

## Por que essa feature é necessária
As Features 1 a 4 pressupõem um usuário autenticado e dependem de rastrear "quem fez o quê" (`importado_por`, `encerrado_por`, `exportado_por`) e de diferenciar perfis (Agente de Investigação, Delegado). Esta feature cobre essa base, sendo pré-requisito das demais.

## Ator Principal
Qualquer usuário do sistema

## Pré-condições
- Usuário possui uma conta previamente cadastrada no sistema (cadastro/gestão de usuários está fora do escopo desta feature — ver Observação de Escopo)

## Fluxo Principal
1. Usuário acessa a tela de login.
2. Usuário informa e-mail e senha.
3. Sistema valida as credenciais.
4. Sistema gera um token de acesso (JWT) contendo o identificador do usuário e seu perfil (role).
5. Usuário passa a usar esse token nas chamadas seguintes (header `Authorization: Bearer`).

## Fluxos Alternativos / Exceção

| Cenário | Comportamento |
|---|---|
| E1 — Credenciais inválidas | Rejeita com 401 e mensagem genérica ("usuário ou senha inválidos"), sem indicar qual dos dois está errado |
| E2 — Conta desativada | Rejeita com 403 |
| E3 — Token expirado numa chamada autenticada | Rejeita com 401, exige novo login |
| E4 — Usuário sem o perfil necessário acessa funcionalidade restrita | Rejeita com 403 |

## Regras de Negócio
- **RN00.1 — Perfis (Roles)**: o sistema reconhece `AGENTE_INVESTIGACAO` e `DELEGADO`. Features 1 a 3 exigem `AGENTE_INVESTIGACAO`; a Feature 4 é acessível por ambos os perfis.
- **RN00.2 — Senha nunca em texto puro**: senhas são armazenadas com hash (BCrypt).
- **RN00.3 — Token com Expiração**: o token tem tempo de vida limitado (ex: 8h), forçando reautenticação periódica.
- **RN00.4 — Auditoria a partir do Token**: os campos de auditoria das outras features (`importado_por`, `encerrado_por`, `exportado_por`) são preenchidos a partir do usuário autenticado no token, nunca informados manualmente pelo cliente da API.

## Critérios de Aceite (Gherkin)

```gherkin
Cenário: Login bem-sucedido
  Dado um usuário cadastrado com e-mail e senha válidos
  Quando ele envia essas credenciais para o endpoint de login
  Então o sistema retorna um token JWT válido
  E o token contém o perfil do usuário

Cenário: Login com senha incorreta
  Dado um usuário cadastrado
  Quando ele envia a senha errada
  Então o sistema rejeita com 401
  E não informa se o erro foi no e-mail ou na senha

Cenário: Acesso negado por perfil insuficiente
  Dado um usuário autenticado com perfil "DELEGADO"
  Quando ele tenta acionar "Iniciar Varredura" (Feature 2, restrita a AGENTE_INVESTIGACAO)
  Então o sistema rejeita com 403
```

## Requisitos Não Funcionais
- Hash de senha com BCrypt.
- JWT assinado (`quarkus-smallrye-jwt`), com claims de `sub` (id do usuário) e `role`.
- Rate limiting básico no endpoint de login, para mitigar força bruta (pode ser simplificado dado o escopo do curso).

## Modelo de Dados
```
usuario
- id
- nome
- email (unique)
- senha_hash
- role (AGENTE_INVESTIGACAO / DELEGADO)
- ativo (boolean)
- criado_em
```

## Endpoints Propostos
- `POST /auth/login` → recebe e-mail + senha, retorna token JWT

## Observação de Escopo
Esta spec cobre login e autorização básica por perfil. Cadastro e gestão de usuários (criar, editar, desativar contas) ficam fora do escopo — podem virar uma feature administrativa separada, se necessário.
