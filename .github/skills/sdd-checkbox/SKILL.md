---
name: sdd-checkbox
description: Implementa um único checkbox de tasks.md seguindo SDD e Clean
  Architecture em Java/Quarkus. Use quando o usuário pedir para implementar
  uma task, um checkbox, ou avançar num item de tasks.md.
allowed-tools: shell
---

# Implementar checkbox de SDD

1. Leia /specs/<feature>/spec.md, plan.md e tasks.md.
2. Identifique o PRIMEIRO checkbox não marcado em tasks.md. Implemente
   SOMENTE esse item — nunca avance para o próximo sem confirmação.
3. Verifique em plan.md a camada (domain/application/infrastructure/
   presentation) e as regras de arquitetura aplicáveis.
4. Escreva o código seguindo as regras de Clean Architecture do plan.md
   (injeção por construtor em domain/application, sem dependência de
   framework no domain, etc).
5. Escreva o teste correspondente:
   - domain/application -> JUnit + Mockito (sem @QuarkusTest)
   - infrastructure -> @QuarkusTest + Testcontainers
   - presentation -> REST Assured
6. Rode `mvn test` (inclui ArchUnit). Se falhar, corrija antes de reportar.
7. Marque o checkbox em tasks.md como concluído.
8. Pare e resuma o que foi feito. Aguarde revisão antes de continuar.

## Nunca faça
- Não implemente mais de um checkbox por invocação.
- Não marque um checkbox como feito sem os testes passando.
- Não altere migrations sem confirmação explícita.