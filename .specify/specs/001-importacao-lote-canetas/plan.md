# Plan — Importação do Lote de Canetas
**Spec**: specs/001-importacao-lote-canetas/feature-1-importacao-lote.md

## Summary
Importar CSV de canetas extraviadas com validação síncrona de formato/
colunas, idempotência via hash SHA-256, e processamento assíncrono
(deduplicação + persistência) rodando em background.

## Technical Context
**Language/Version**: Java 21, Quarkus 3.x
**Primary Dependencies**: Apache Commons CSV, java.security.MessageDigest (SHA-256, JDK puro)
**Storage**: PostgreSQL
**Async mechanism**: ManagedExecutor.runAsync() (worker thread), disparado pelo ImportarLoteUseCase — decisão detalhada em research.md
**Testing**: JUnit5+Mockito (domain/application), @QuarkusTest+Testcontainers (infra), ArchUnit (arquitetura)
**Constraints**: resposta do POST em <2s independente do tamanho do arquivo

## Constitution Check
- [x] domain/ sem dependência de framework — Caneta e LoteImportacao validam
  regras no próprio construtor, sem Jakarta Validation
- [x] injeção por construtor em domain/application
- [x] recuperação de lote travado em PROCESSANDO — limitação conhecida
  e aceita para este MVP (ver research.md), não bloqueia a constitution
  por não haver princípio de durabilidade definido ainda

## Domain (sem dependência de framework)
- `Caneta` — valida numeroSerie/fabricante/calibre no construtor
- `LoteImportacao` — controla transição de status
- `StatusProcessamento` (enum: PROCESSANDO, CONCLUIDO, FALHOU)
- `StatusCaneta` (enum: PENDENTE, ENCONTRADA, NAO_ENCONTRADA, REVISAO_PENDENTE)
- Exceções de domínio: `CanetaInvalidaException`, `ColunaObrigatoriaAusenteException`

## Application (use cases + ports)

- `ImportarLoteUseCase` — cobre A1 (colunas ausentes), A4 (formato inválido),
  A6 (idempotência via hash): valida, calcula hash, verifica duplicidade,
  cria lote PROCESSANDO, dispara processamento via ProcessamentoAssincronoPort
- `ProcessarLoteUseCase` — cobre A2 (dedup dentro do arquivo), A3 (linha
  inválida): parseia via CsvParserPort, aplica regras de negócio linha a
  linha, persiste, atualiza status final e gera relatório

### Ports (interfaces — implementadas em infrastructure)
- `LoteImportacaoRepositoryPort`
- `CanetaRepositoryPort`
- `FileHashPort` (calcula hash do arquivo)
- `CsvParserPort` (só parsing técnico — lê bytes, devolve linhas cruas)
- `ProcessamentoAssincronoPort` (dispara execução em background;
  ImportarLoteUseCase depende só desta interface, não de ManagedExecutor
  diretamente — decisão em research.md)
- `ConsultarStatusLoteUseCase` — atende GET /{id}
- `ListarCanetasDoLoteUseCase` — atende GET /{id}/canetas (paginado)

## Infrastructure (adapters)

- `LoteImportacaoRepositoryPanache`, `CanetaRepositoryPanache`
- `Sha256FileHashAdapter`
- `ApacheCommonsCsvParserAdapter`
- `ManagedExecutorProcessamentoAdapter implements ProcessamentoAssincronoPort`
  — usa ManagedExecutor.runAsync() para chamar ProcessarLoteUseCase em
  worker thread (ver research.md: sem durabilidade nesta versão)

## Presentation

- `ImportacaoResource`
    - `POST /lotes-importacao` -> ImportarLoteUseCase
    - `GET /lotes-importacao/{id}` -> ConsultarStatusLoteUseCase
    - `GET /lotes-importacao/{id}/canetas` -> ListarCanetasDoLoteUseCase
- DTOs de request/response (contratos em contracts/ — pendente de criar)

## Complexity Tracking
(vazio — nenhuma violação da constitution; limitação de durabilidade é
decisão de escopo registrada em research.md, não infração de princípio)