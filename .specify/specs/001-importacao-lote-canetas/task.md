# Task - Importação de Lote

## Domain

### LoteImportacao

- [x] Criar modelo de domínio `LoteImportacao`
  - `id`
  - `nomeArquivo`
  - `hashArquivo`
  - `importadoPor`
  - `importadoEm`
  - `statusProcessamento`
  - `totalLinhas`
  - `totalSucesso`
  - `totalErro`

### caneta

- [x] Criar modelo de domínio `caneta`
  - `id`
  - `numeroSerie`
  - `fabricante`
  - `modelo`
  - `paisOrigem`
  - `status`
  - `loteId`
  - `criadoEm`

### Enums

- [x] Criar `StatusProcessamento`
  - `PROCESSANDO`
  - `CONCLUIDO`
  - `FALHOU`

- [x] Criar `Statuscaneta`
  - `PENDENTE`
  - `ENCONTRADA`
  - `NAO_ENCONTRADA`
  - `REVISAO_PENDENTE`

## Application
  
### Use Cases

- [x] Criar `ImportarLoteUseCase`
  - [x] Implementar `executar(ArquivoImportacao arquivo, UUID usuarioId)`
  - [x] Calcular hash do arquivo
  - [x] Verificar idempotência pelo hash
  - [x] Retornar lote existente quando o hash já estiver cadastrado
  - [x] Criar novo lote quando o hash não existir
  - [x] Disparar processamento do lote

- [x] Implementar `criarLote(...)`
  - [x] Criar lote com status `PROCESSANDO`
  - [x] Persistir lote
  - [x] Disparar processamento assíncrono
  - [x] Retornar ID do lote

### Ports

- [x] Criar `LoteImportacaoRepositoryPort`
- [x] Criar `canetaRepositoryPort`
- [x] Criar `FileHashPort`
- [x] Criar `CsvParserPort`

### DTOs



- [x] Criar DTO de resposta do lote
- [x] Criar DTO de resposta da caneta
- [x] Criar DTO do relatório de importação

### Exceptions

- [x] Criar exceção para arquivo inválido
- [x] Criar exceção para colunas obrigatórias ausentes
- [x] Criar exceção para lote não encontrado

## Infrastructure

### Persistence

- [x] Criar `LoteImportacaoEntity`
- [x] Criar `canetaEntity`
- [x] Criar repositories
- [x] Criar mappers
- [x] Criar migration
- [x] Adicionar constraint de unicidade do hash do arquivo
  (`lote_importacao.hash_arquivo`)
- [x] Adicionar constraint única composta `(lote_id, numero_serie, fabricante)`
  na tabela `caneta` — rede de segurança pro A2, sem violar A5
  (mesma caneta pode repetir entre lotes diferentes)
- - [x] Implementar `buscarPorNumeroSerieEFabricante` no
    `CanetaRepositoryPanache`


### CSV

- [X] Implementar `CsvParserPort`
- [ ] Validar colunas obrigatórias
- [ ] Processar arquivo linha a linha
- [ ] Identificar campos obrigatórios vazios
- [ ] Identificar duplicidade `(numeroSerie, fabricante)` dentro do arquivo
- [ ] Manter primeira ocorrência da duplicidade
- [ ] Registrar erros por linha

### Hash

- [ ] Implementar `FileHashPort` com SHA-256

### Async Processing

- [ ] Implementar `ImportacaoProcessorPort`
- [ ] Implementar processamento assíncrono
- [ ] Persistir canetas válidas com status `PENDENTE`
- [ ] Atualizar contadores do lote
- [ ] Atualizar status para `CONCLUIDO`
- [ ] Atualizar status para `FALHOU` em falha não recuperável

### Report

- [ ] Implementar geração do relatório
- [ ] Registrar total lido
- [ ] Registrar total importado
- [ ] Registrar total com erro
- [ ] Registrar erros por linha

## Presentation

- [ ] Criar `ImportacaoResource`
- [ ] Implementar `POST /lotes-importacao`
- [ ] Implementar `GET /lotes-importacao/{id}`
- [ ] Implementar `GET /lotes-importacao/{id}/canetas`
- [ ] Implementar paginação das canetas
- [ ] Integrar tratamento de exceções

## Tests

### Unit

- [ ] Testar `ImportarLoteUseCase`
- [ ] Testar cálculo/verificação de idempotência
- [ ] Testar criação do lote
- [ ] Testar duplicidade dentro do arquivo
- [ ] Testar linha com campo obrigatório vazio
- [ ] Testar transições de status

### Integration

- [ ] Testar fluxo completo de importação
- [ ] Testar `POST /lotes-importacao`
- [ ] Testar `GET /lotes-importacao/{id}`
- [ ] Testar `GET /lotes-importacao/{id}/canetas`
- [ ] Testar persistência no banco
- [ ] Testar reenvio do mesmo arquivo
- [ ] Testar busca com caneta em múltiplos lotes
- [ ] Testar busca sem resultado (lista vazia)
- [ ] Testar busca com caneta em um único lote
- [ ] Testar `GET /canetas/busca` (integration)

## Validation

- [ ] Executar testes
- [ ] Executar testes de arquitetura (ArchUnit)
- [ ] Verificar todos os critérios de aceite da SPEC
- [ ] Verificar alterações fora do escopo