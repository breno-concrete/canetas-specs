# Feature 3: Encerramento de Lote (Triagem de Extravio)

**Stack:** Quarkus

## User Story
Como agente de investigação, quero encerrar oficialmente um lote após a varredura, para declarar as canetas não localizadas como extraviadas/ilegais não rastreadas.

## Ator Principal
Agente de Investigação

## Pré-condições
- O lote já teve pelo menos uma execução do Motor de Cruzamento (Feature 2) concluída (status `VARREDURA_CONCLUIDA`)
- O lote ainda não está `ENCERRADO`

## Fluxo Principal
1. Usuário clica em "Encerrar Lote".
2. Sistema exibe aviso de confirmação: "Tem certeza? Armas pendentes serão marcadas como não encontradas."
3. Usuário confirma.
4. Sistema localiza todas as canetas do lote que ainda estão com status `Pendente`.
5. Sistema atualiza essas canetas em massa para `Não Encontrada (Extraviada)`.
6. Sistema muda o status do lote para `ENCERRADO`.

## Fluxos Alternativos / Exceção

| Cenário | Comportamento |
|---|---|
| C1 — Usuário cancela a confirmação | Nenhuma alteração é feita; lote permanece como estava |
| C2 — Encerramento acionado num lote sem varredura concluída | Rejeita a ação — pré-condição não satisfeita |
| C3 — Encerramento acionado num lote já `ENCERRADO` | Rejeita a ação |
| C4 — Encerramento acionado enquanto o lote está `EM_VARREDURA` | Rejeita a ação até a varredura em andamento terminar |

## Regras de Negócio
- **RN08 — Encerramento é Ato Deliberado**: a transição de canetas `Pendente` para `Não Encontrada` só ocorre através desta ação explícita do agente, nunca automaticamente ao fim de uma varredura.
- **RN09 — Bloqueio Pós-Encerramento**: um lote `ENCERRADO` não aceita mais varreduras — nem manuais, nem propagação automática de match vinda de outro lote (RN06/RN07 da Feature 2 deixam de valer para canetas de um lote encerrado).
- **RN10 — Cobertura Total**: o encerramento só é considerado concluído quando 100% das canetas do lote estão em `Encontrada` ou `Não Encontrada` — nenhuma pode restar `Pendente`.

## Nota de Consistência com a Feature 2
Este encerramento é o único ponto do sistema onde a transição `Pendente → Não Encontrada` acontece. A Feature 2 (Motor de Cruzamento) não aplica mais essa transição automaticamente ao fim de uma varredura — ela só resolve o que consegue casar (`Encontrada`) e deixa o restante em `Pendente`, disponível para novas tentativas de varredura até que o agente decida encerrar o lote.

## Máquina de Estados (lote)
```
PROCESSANDO → CONCLUIDO → EM_VARREDURA ⇄ VARREDURA_CONCLUIDA → ENCERRADO
```
(`EM_VARREDURA ⇄ VARREDURA_CONCLUIDA` indica que o lote pode ser varrido mais de uma vez antes do encerramento.)

## Critérios de Aceite (Gherkin)

```gherkin
Cenário: Encerramento bem-sucedido
  Dado um lote com status "VARREDURA_CONCLUIDA"
  E existem canetas nesse lote ainda com status "Pendente"
  Quando o agente confirma "Encerrar Lote"
  Então todas as canetas "Pendente" desse lote mudam para "Não Encontrada"
  E o lote muda para status "ENCERRADO"
  E nenhuma caneta do lote permanece "Pendente"

Cenário: Encerramento sem varredura prévia
  Dado um lote que nunca teve uma varredura concluída
  Quando o agente tenta encerrar o lote
  Então o sistema rejeita a ação

Cenário: Encerramento duplicado
  Dado um lote já "ENCERRADO"
  Quando o agente tenta encerrar esse lote novamente
  Então o sistema rejeita a ação

Cenário: Lote encerrado não aceita nova varredura
  Dado um lote "ENCERRADO"
  Quando alguém tenta acionar "Iniciar Varredura" nesse lote
  Então o sistema rejeita a execução

Cenário: Usuário cancela a confirmação
  Dado um lote com status "VARREDURA_CONCLUIDA"
  Quando o agente clica em "Encerrar Lote" mas cancela a confirmação
  Então nenhum status de caneta ou de lote é alterado
```

## Requisitos Não Funcionais
- **Transacionalidade**: a atualização em massa das canetas e a mudança de status do lote ocorrem na mesma transação — ou tudo é persistido, ou nada é.
- **Auditoria**: registrar quem encerrou o lote e quando.

## Modelo de Dados (extensão)
```
lote_importacao
- status_processamento passa a incluir: ENCERRADO
+ encerrado_por (FK usuário, nullable)
+ encerrado_em (nullable)
```

## Endpoints Propostos
- `POST /lotes-importacao/{id}/encerramento` → executa o encerramento (confirmação já tratada no frontend antes da chamada)
- `GET /lotes-importacao/{id}` → passa a refletir o status `ENCERRADO` quando aplicável
