# Spec-Driven Development — Project Skeleton

Repositório base para desenvolvimento orientado a especificações (**Spec-Driven Development — SDD**).

O projeto mantém regras, especificações e tarefas separadas da implementação. A implementação deve ser guiada pelas specs e respeitar as regras definidas em `memory/constitution.md`.

## Estrutura

```text
.
├── AGENTS.md
├── memory
│   └── constitution.md
└── specs
    ├── 000-autenticacao-de-usuario
    │   ├── feature-0-autenticacao.md
    │   └── task.md
    │
    ├── 001-importacao-lote-canetas
    │   ├── data-model.md
    │   ├── feature-1-importacao-lote.md
    │   ├── plan.md
    │   ├── quickstart.md
    │   ├── research.md
    │   └── task.md
    │
    ├── 002-motor-cruzamento
    │   └── feature-2-motor-cruzamento.md
    │
    ├── 003-encerramento-lote
    │   └── feature-3-encerramento-lote.md
    │
    └── 004-dashboard-auditoria
        └── feature-4-dashboard-auditoria.md
```

> **Nota:** a estrutura das specs ainda está em evolução. A `001-importacao-lote-canetas` é a referência de estrutura e nível de detalhamento para as próximas features.

## Como usar

### 1. Regras do projeto

Antes de trabalhar em qualquer feature, leia:

```text
memory/constitution.md
```

A `constitution.md` define as regras gerais que devem ser respeitadas durante o desenvolvimento.

### 2. Especificação da feature

Escolha a feature que será desenvolvida e leia sua especificação:

```text
specs/<feature>/feature-*.md
```

A especificação define **o que o sistema deve fazer**, incluindo regras de negócio, fluxos e critérios de aceitação.

### 3. Documentos da feature

Quando disponíveis:

| Documento | Responsabilidade |
|---|---|
| `feature-*.md` | Especificação funcional |
| `research.md` | Pesquisas e decisões técnicas |
| `data-model.md` | Modelo de dados |
| `plan.md` | Planejamento técnico |
| `task.md` | Tasks de implementação |
| `quickstart.md` | Validação da feature |

A **Feature 001** deve ser utilizada como modelo para estruturar e detalhar as próximas features.

## Fluxo de implementação

O desenvolvimento segue:

```text
Feature
   ↓
Research
   ↓
Data Model
   ↓
Plan
   ↓
Tasks
   ↓
Implementação
   ↓
Testes
   ↓
Validação
```

A implementação deve ser feita **uma task por vez**.

```text
Selecionar task
      ↓
Implementar
      ↓
Executar testes
      ↓
Revisar implementação
      ↓
Validar critérios de aceitação
      ↓
Concluir task
      ↓
Próxima task
```

Não avance para a próxima task antes de revisar a atual.

## Uso de IA

O repositório possui um `AGENTS.md` com as instruções para os agentes de IA.

O agente deve:

- consultar `memory/constitution.md`;
- consultar a especificação da feature;
- utilizar os documentos da feature como contexto;
- trabalhar em **uma task por vez**;
- não inventar regras de negócio;
- respeitar a arquitetura e as regras do projeto;
- executar os testes após a implementação;
- revisar o resultado antes de concluir a task.

A IA deve ser utilizada para **implementar o que foi especificado**, e não para definir sozinha o comportamento do sistema.

## Princípio

```text
Constitution
     ↓
Regras do projeto

Specs
     ↓
O que deve ser feito

Tasks
     ↓
O que implementar

Código
     ↓
Como implementar

Testes
     ↓
Se está correto
```

> **A especificação define o que o sistema deve fazer. A implementação define como fazê-lo.**
