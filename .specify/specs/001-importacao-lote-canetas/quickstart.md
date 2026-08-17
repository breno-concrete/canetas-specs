## Validar a importação de lote

1. Suba a aplicação: `./mvnw quarkus:dev`
2. Envie o arquivo de exemplo:
   `curl -F "file=@lote-exemplo.csv" localhost:8080/lotes-importacao`
3. Resultado esperado: HTTP 202, corpo com `{ "id": "...", "status": "PROCESSANDO" }`
4. Consulte o status: `curl localhost:8080/lotes-importacao/{id}`
5. Resultado esperado (após alguns segundos): `status: "CONCLUIDO"`,
   relatório com total lido/importado/erro batendo com o arquivo de exemplo
6. Liste as canetas do lote: `curl localhost:8080/lotes-importacao/{id}/canetas`
7. Resultado esperado: lista paginada, cada caneta com status `PENDENTE`

## Validar idempotência (A6)

8. Reenvie o mesmo arquivo do passo 2:
   `curl -F "file=@lote-exemplo.csv" localhost:8080/lotes-importacao`
9. Resultado esperado: retorna o **mesmo** `id` do passo 3, não cria lote novo

## Validar busca de caneta por identidade

10. Pegue `numeroSerie` e `fabricante` de uma caneta do passo 6.
11. Busque por ela:
    `curl "localhost:8080/canetas/busca?numeroSerie=X&fabricante=Y"`
12. Resultado esperado: lista com um item — `loteId` (igual ao do passo 3),
    `status` (`PENDENTE`), `criadoEm`

13. Envie um **segundo** arquivo de exemplo contendo a **mesma** caneta
    (mesmo numeroSerie + fabricante, hash de arquivo diferente):
    `curl -F "file=@lote-exemplo-2.csv" localhost:8080/lotes-importacao`
14. Resultado esperado: novo `id` de lote (diferente do passo 3) —
    confirma A5 (mesma caneta permitida em lote diferente)
15. Repita a busca do passo 11
16. Resultado esperado: lista agora com **dois** itens, um por lote,
    cada um com seu próprio `loteId`, `status` e `criadoEm`

## Validar busca sem resultado

17. Busque por uma caneta inexistente:
    `curl "localhost:8080/canetas/busca?numeroSerie=INEXISTENTE&fabricante=X"`
18. Resultado esperado: HTTP 200, lista vazia `[]`