# Feature 4: Dashboard de Investigação e Auditoria

**Stack:** Quarkus

## User Story
Como agente de investigação ou delegado, quero visualizar as estatísticas de um lote e auditar os CACs envolvidos, para reunir os elementos necessários à expedição de mandados de busca.

## Ator Principal
Agente de Investigação / Delegado

## Pré-condições
- O lote já teve pelo menos uma varredura concluída (Feature 2). Não precisa estar `ENCERRADO` para ser consultado, mas normalmente é acessado depois do encerramento (Feature 3).

## Fluxo Principal
1. Usuário acessa o Painel da Investigação de um lote.
2. Sistema exibe as estatísticas: total importado, quantidade `Encontrada`, quantidade `Não Encontrada` (e `Pendente`, se o lote ainda não foi encerrado).
3. Usuário clica no número de canetas `Encontradas` (ou aplica um filtro de status).
4. Sistema exibe uma grid paginada com: Modelo da Arma, CPF do CAC, Data de Registro Nacional.
5. Usuário seleciona "Exportar Dossiê" para gerar um PDF com os dados da lista atualmente filtrada.
6. Sistema gera o PDF e disponibiliza para download.

## Fluxos Alternativos / Exceção

| Cenário | Comportamento |
|---|---|
| D1 — Lote sem nenhuma varredura executada | Painel exibe contagens zeradas, sem erro |
| D2 — Filtro aplicado sem resultados | Grid exibe estado vazio; exportação de dossiê fica desabilitada |
| D3 — Exportação de dossiê para uma lista grande (ex: todas as `Encontradas` de um lote de 4.000) | Geração de PDF é assíncrona, mesmo padrão de outras operações demoradas do sistema, com endpoint de status/download |

## Regras de Negócio
- **RN11 — Filtro por Status**: o Painel permite filtrar as canetas por qualquer status (`Pendente`, `Encontrada`, `Não Encontrada`).
- **RN12 — Escopo do Dossiê**: o PDF exportado reflete exatamente o filtro ativo no momento da exportação.
- **RN13 — Conteúdo Mínimo do Dossiê**: cada caneta no PDF traz Número de Série, Fabricante, Modelo, Calibre, CPF/CNPJ do CAC e Data de Registro Nacional — dados suficientes para embasar uma ação policial.
- **RN14 — Somente Leitura**: o Painel e a exportação não alteram nenhum dado de caneta, lote ou CAC; são operações estritamente de consulta/relatório.

## Critérios de Aceite (Gherkin)

```gherkin
Cenário: Exibir estatísticas do lote
  Dado um lote com 4000 canetas importadas, sendo 120 "Encontrada" e 3880 "Não Encontrada"
  Quando o usuário acessa o Painel da Investigação desse lote
  Então o sistema exibe total importado = 4000, Encontrada = 120, Não Encontrada = 3880

Cenário: Filtrar canetas por status
  Dado o Painel de um lote
  Quando o usuário filtra por status "Encontrada"
  Então a grid exibe apenas as canetas com esse status

Cenário: Exportar dossiê
  Dado a grid filtrada por "Encontrada"
  Quando o usuário seleciona "Exportar Dossiê"
  Então o sistema gera um PDF contendo Número de Série, Fabricante, Modelo, Calibre, CPF/CNPJ do CAC, Nome do CAC e Data de Registro Nacional de cada caneta da lista filtrada

Cenário: Painel sem varredura
  Dado um lote que ainda não teve nenhuma varredura concluída
  Quando o usuário acessa o Painel
  Então o sistema exibe as contagens zeradas, sem erro
```

## Requisitos Não Funcionais
- **Performance**: contagens agregadas (total, por status) calculadas via query de agregação no banco, sem carregar as 4.000 canetas em memória.
- **Paginação**: a grid de canetas é paginada no backend.
- **Assíncrono**: geração de PDF para listas grandes segue o mesmo padrão de job assíncrono das demais features, com endpoint de acompanhamento.
- **Auditoria**: registrar quem exportou um dossiê e quando — rastreabilidade de acesso a dados sensíveis de CACs.

## Modelo de Dados (extensão)
```
caneta
+ nome_cac (nullable, preenchido no momento do match — Feature 2)

dossie_exportado
- id
- lote_id (FK)
- filtro_aplicado (status usado na exportação)
- exportado_por (FK usuário)
- exportado_em
```

## Endpoints Propostos
- `GET /lotes-importacao/{id}/estatisticas` → totais por status
- `GET /lotes-importacao/{id}/canetas?status=ENCONTRADA&page=...` → grid paginada e filtrável
- `POST /lotes-importacao/{id}/dossie` → dispara geração assíncrona do PDF (recebe o filtro de status)
- `GET /dossies/{id}` → status/download do PDF gerado
