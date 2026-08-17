# Feature 1 — Data Model Specification

## 1. Purpose

Define the persistence model required by Feature 1:
Importação do Lote de Canetas Estrangeiras.

This specification is derived from:
`specs/features/feature-01-importacao-lote.md`

---

## 2. Entities

### 2.1 LoteImportacao

Representa uma execução de importação de um arquivo CSV.

| Field | Type | Required | Constraints |
|---|---|---:|---|
| id | UUID | yes | PK |
| nome_arquivo | VARCHAR | yes | |
| hash_arquivo | CHAR(64) | yes | UNIQUE |
| importado_por | UUID | yes | FK usuario |
| importado_em | TIMESTAMP | yes | |
| status_processamento | ENUM | yes | PROCESSANDO, CONCLUIDO, FALHOU |
| total_linhas | INTEGER | yes | >= 0 |
| total_sucesso | INTEGER | yes | >= 0 |
| total_erro | INTEGER | yes | >= 0 |

### 2.2 Caneta

Representa uma caneta importada de um lote estrangeiro.

| Field        | Type      | Required | Constraints |
|--------------|-----------|---:|---|
| id           | UUID      | yes | PK |
| numero_serie | VARCHAR   | yes | |
| fabricante   | VARCHAR   | yes | |
| modelo       | VARCHAR   | yes | |
| pais_origem  | VARCHAR   | yes | |
| status       | ENUM      | yes | PENDENTE, ENCONTRADA, NAO_ENCONTRADA, REVISAO_PENDENTE |
| lote_id      | UUID      | yes | FK lote_importacao |
| criado_em    | TIMESTAMP | yes | |
| calibre      | VARCHAR   | yes | |

---

## 3. Relationships

### LoteImportacao → Caneta

One-to-many.

Um `LoteImportacao` possui zero ou várias `Caneta`.

Uma `Caneta` pertence a exatamente um `LoteImportacao`.

```text
LoteImportacao 1 ─────── N Caneta