# Feature 1: Importação do Lote de canetas Estrangeiras

**Stack:** Quarkus

## User Story
Como agente de investigação, quero importar a lista de canetas extraviadas enviada pelo país de origem, para que o sistema tenha a base de dados inicial para conduzir a investigação.

## Ator Principal
Agente de Investigação (usuário autenticado com perfil apropriado)

## Pré-condições
- Usuário autenticado e autorizado (perfil com permissão de importação)
- Usuário possui o arquivo CSV enviado pelo país de origem

## Fluxo Principal
1. Usuário acessa a tela "Nova Investigação de Lote".
2. Usuário faz upload do arquivo CSV.
3. Sistema valida presença das colunas obrigatórias: `Número de Série`, `Fabricante`, `Modelo`, `País de Origem`.
4. Sistema cria o registro do lote com status `PROCESSANDO` e responde imediatamente ao usuário (processamento é assíncrono).
5. Em background, o sistema processa o arquivo linha a linha e persiste as canetas com status `Pendente`.
6. Ao final, o sistema atualiza o lote para `CONCLUIDO` (ou `FALHOU`) e gera o relatório de importação.

## Fluxos Alternativos / Exceção /  Casos de Borda

| Cenário | Comportamento |
|---|---|
| A1 — Falta coluna obrigatória | Rejeita o arquivo inteiro (nada é persistido). Retorna mensagem indicando qual(is) coluna(s) faltam. |
| A2 — Número de Série + Fabricante duplicado *dentro do arquivo* | Ignora a linha duplicada (mantém a primeira ocorrência). Reporta a duplicidade no relatório final. |
| A3 — Linha com campo obrigatório vazio | Pula a linha, registra no relatório de erros (não derruba o lote inteiro). |
| A4 — Formato de arquivo inválido (não CSV) | Rejeita com erro amigável antes de tentar parsear. |
| A5 — Número de Série + Fabricante já existe no banco (de lote anterior) | Permite — a mesma caneta pode aparecer em mais de uma investigação. |
| A6 — Mesmo arquivo reenviado (mesmo hash) | Não reprocessa; retorna o lote já existente. |

## Critérios de Aceite (Gherkin)

```gherkin
Cenário: Rejeitar arquivo sem coluna obrigatória
  Dado que o usuário faz upload de um arquivo sem a coluna "Número de Série"
  Quando o sistema valida o arquivo
  Então o sistema rejeita a importação
  E nenhuma caneta é persistida
  E o sistema informa qual coluna está faltando

Cenário: Importar lote válido com sucesso
  Dado um arquivo CSV válido com mais de uma linha e todas as colunas obrigatórias
  Quando o usuário faz upload
  Então o sistema persiste as canetas válidas com status "Pendente"
  E o relatório final mostra total lido, total importado e total com erro

Cenário: Linha duplicada dentro do arquivo
  Dado um arquivo com duas linhas com o mesmo par (Número de Série, Fabricante)
  Quando o sistema processa o arquivo
  Então apenas a primeira ocorrência é persistida
  E a duplicidade é reportada no relatório final

Cenário: Reenvio do mesmo arquivo
  Dado um arquivo já importado anteriormente (mesmo hash)
  Quando o usuário faz upload novamente
  Então o sistema não cria um novo lote
  E retorna o lote já existente
  
```

## Requisitos Não Funcionais
- **Processamento assíncrono**: método anotado com `@Asynchronous` (Mutiny), request de upload responde de imediato com o ID do lote; endpoint separado para consultar status/progresso.
- **Idempotência**: hash (SHA-256) do conteúdo do arquivo, verificado antes de reprocessar.
- **Auditoria**: registrar quem importou, quando, nome e hash do arquivo original.
- **Relatório pós-importação**: total lido, total importado, total com erro, detalhamento por linha com motivo da falha.


## Endpoints Propostos
- `POST /lotes-importacao` (multipart, arquivo) → cria o lote e dispara processamento
- `GET /lotes-importacao/{id}` → status e relatório
- `GET /lotes-importacao/{id}/canetas` → lista paginada das canetas do lote
