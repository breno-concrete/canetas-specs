# Constitution — org.breno / hello-quarkus

## Princípios de Arquitetura
1. domain/ não pode depender de framework (sem jakarta.*, sem Panache)
2. Injeção por construtor em domain/ e application/
3. Toda entidade JPA tem um mapper separado da entidade de domínio

## Testes
4. Toda task só é "concluída" com teste da própria unidade passando
5. Regras de arquitetura validadas por ArchUnit no CI