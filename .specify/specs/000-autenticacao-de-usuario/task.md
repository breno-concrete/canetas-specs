# Feature: Autenticação (Login JWT) - Quarkus + Clean Architecture

## Contexto
Implementar autenticação usando Quarkus com JWT (SmallRye JWT),
seguindo Clean Architecture.

Separação obrigatória:
- domain → entidades e regras de negócio puras (NÃO depende de nada externo)

- application → casos de uso (orquestram regras do domínio, definem interfaces/ports)

- infrastructure → implementações técnicas (banco, JWT, frameworks, adapters)

- presentation → camada de entrada (HTTP, JAX-RS, controllers/resources)

---

## Tasks

### Domain (NÚCLEO - sem dependência de framework)

- [x] Criar enum Role com valores:
    - AGENTE_INVESTIGACAO
    - DELEGADO

- [x] Criar classe User (domínio puro, sem anotações JPA):
    - id
    - nome
    - email
    - senhaHash
    - role
    - ativo
    - criadoEm

- [x] Garantir que não há dependência de Quarkus/JPA no domínio

---

### Infrastructure - Persistência (Panache)

- [x] Criar entidade JPA UserEntity (separada do domínio)

- [x] Mapear UserEntity com:
    - @Entity
    - campos equivalentes ao User

- [x] Criar mapper entre UserEntity ↔ User (manual ou MapStruct)

- [x] Criar UserRepository usando PanacheRepository<UserEntity>

- [x] Implementar método:
  User findByEmail(String email)
  (convertendo Entity → Domain)

---

### Application - DTOs

- [x] Criar LoginRequest:
    - email
    - senha
    - validações (@Email, @NotBlank)

- [x] Criar LoginResponse:
    - token (String)

---

### Infrastructure - JWT

- [x] Adicionar dependência:
  quarkus-smallrye-jwt

- [x] Configurar application.properties:
    - chave JWT
    - expiração (8h)

- [x] Criar JwtService (infraestrutura)

- [x] Implementar:
  String generateToken(User user)

- [x] Incluir claims:
    - sub (id)
    - groups (role)

---

### Application - Use Case (Regra de Negócio)

- [x] Criar AuthUseCase (ou AuthService) em application layer

- [x] Implementar método:
  LoginResponse login(LoginRequest request)

- [x] Regras:
    - buscar usuário via repository (interface)
    - se não existir → erro 401
    - validar senha com BCrypt
    - se inválida → erro 401 (mensagem genérica)
    - verificar se ativo
    - se inativo → erro 403
    - gerar token (via interface JwtService)
    - retornar LoginResponse

- [x] NÃO usar classes de framework diretamente no use case

---

### 🔌 Application - Ports (Interfaces)

- [x] Criar interface UserRepository (no domínio ou application)

- [x] Criar interface JwtService (porta)

- [x] Garantir inversão de dependência:
  use case depende da interface, não da implementação

---

### Presentation (JAX-RS)

- [x] Criar AuthResource com @Path("/auth")

- [x] Criar endpoint:
  POST /auth/login

- [x] Receber LoginRequest com @Valid

- [x] Chamar AuthUseCase

- [x] Retornar LoginResponse

---

### Tratamento de erros

- [x] Criar ExceptionMapper global (@Provider)

- [x] Tratar:
    - 401 → credenciais inválidas
    - 403 → acesso negado

---

### Autorização

- [x] Configurar JWT no Quarkus

- [PENDENTE] Proteger endpoints com:
  @RolesAllowed("AGENTE_INVESTIGACAO")
  @RolesAllowed("DELEGADO")

- [x] Garantir que roles vêm do claim "groups"
