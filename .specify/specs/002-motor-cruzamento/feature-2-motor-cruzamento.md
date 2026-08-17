# Feature 2: Motor de Cruzamento de Dados (Matching)

**Stack:** Quarkus

## User Story
Como sistema de investigação, preciso cruzar automaticamente os dados das canetas do lote estrangeiro com a base nacional de canetas registradas (dono_da_caneta), para identificar quais canetas extraviadas foram legalizadas indevidamente no Brasil.

## Ator Principal
Sistema (processo automatizado), acionado por um Agente de Investigação via botão "Iniciar Varredura"

## Pré-condições
- Existe um lote com canetas em status `Pendente` (Feature 1 já concluída), ou um lote já varrido anteriormente contendo canetas em status `Não Encontrada` (para re-varredura)
- Sistema tem acesso de leitura à base nacional de canetas via integração simulada (ver Integração Externa)

## Dependência com a Feature 1
O critério de match (RN01) passa a exigir `Calibre`. Esse campo precisa estar disponível nos dados da caneta importada — a Feature 1 deve capturar essa coluna do arquivo de origem, mesmo que continue não sendo motivo de rejeição do arquivo caso venha vazia.

## Entidades Envolvidas
- **`caneta`** — já existe da Feature 1; ganha campos novos nesta feature
- **Base Nacional** — integração externa simulada, somente leitura
- **`alerta_dono_da_caneta`** — nova entidade: a flag de investigação, desacoplada do cadastro original do dono_da_caneta

## Integração Externa
A consulta à base nacional é feita por um cliente HTTP (`canetaNacionalClient`) que simula a API da Base Nacional. A implementação real dessa API está fora do escopo do sistema — o cliente é construído contra um contrato simulado (mock/stub), permitindo substituição futura por uma integração real sem alterar o motor de cruzamento.

## Fluxo Principal
1. Agente aciona "Iniciar Varredura" para um lote específico.
2. Sistema muda o lote para status `EM_VARREDURA` (trava contra execução duplicada).
3. Para cada caneta elegível do lote (`Pendente` ou `Não Encontrada`, desde que o lote não esteja `ENCERRADO`):
   1. Sistema verifica primeiro se a mesma caneta (mesmo `Número de Série` + `Fabricante`) já está `Encontrada` em outro lote/investigação ainda não encerrado (RN06). Se sim, propaga o match sem nova consulta externa.
   2. Caso contrário, consulta a base nacional simulada por correspondência exata de `Número de Série` + `Fabricante` + `Calibre` (RN01).
   3. **Se encontra**: atualiza a caneta para `Encontrada` (RN02); vincula CPF/CNPJ do dono_da_caneta e data de registro nacional (RN03); cria um `alerta_dono_da_caneta` sinalizando a caneta sob investigação (RN04) — sem alterar o cadastro original do dono_da_caneta.
   4. **Se não encontra**: a caneta permanece `Pendente` — a transição para `Não Encontrada` só ocorre no encerramento formal do lote (Feature 3).
4. Sistema muda o lote para status `VARREDURA_CONCLUIDA`. canetas sem match seguem elegíveis para novas varreduras até o lote ser encerrado.

## Máquina de Estados (caneta)

```
Pendente ──(match encontrado)──────────────────────────> Encontrada
Pendente ──(encerramento do lote sem match, Feature 3)──> Não Encontrada
Não Encontrada ──(match propagado de outro lote não encerrado)──> Encontrada
```

`Encontrada` é terminal. A transição para `Não Encontrada` só ocorre no encerramento do lote (Feature 3). Uma vez o lote `ENCERRADO`, o status da caneta também se torna definitivo (ver RN09, Feature 3).

## Fluxos Alternativos / Exceção

| Cenário | Comportamento |
|---|---|
| B1 — Base nacional indisponível durante a varredura | Aborta a execução, lote volta ao status anterior à varredura, erro registrado, permite nova tentativa. RN05 só roda se a varredura terminar com sucesso. |
| B2 — Mais de um registro na base nacional bate os três critérios (Série + Fabricante + Calibre) | Não faz auto-match; fica pendente de revisão manual (fora do escopo desta feature — ver Feature de auditoria) |
| B3 — Varredura acionada num lote que já está `EM_VARREDURA` | Rejeita a segunda chamada com erro de conflito |

## Regras de Negócio

- **RN01 — Critério de Cruzamento**: match exige correspondência exata de `Número de Série`, `Fabricante` e `Calibre`. Os três campos são obrigatórios no critério — evita falso positivo tanto de fabricantes diferentes quanto de calibres diferentes reutilizando a mesma numeração.
- **RN02 — Atualização Automática de Status**: todo match (RN01 ou RN06) muda a caneta para `Encontrada` automaticamente, sem intervenção humana no momento da varredura. Não há etapa de confirmação prévia — a auditoria de eventuais falsos positivos é tratada em feature separada, como revisão *posterior*.
- **RN03 — Vinculação de Posse**: toda caneta `Encontrada` fica obrigatoriamente vinculada ao CPF/CNPJ do dono_da_caneta que a registrou.
- **RN04 — Sinalização de Ilegalidade**: toda caneta `Encontrada` recebe uma flag de alerta (contrabando/investigação), sem alterar o cadastro original do dono_da_caneta.
- **RN05 — Encerramento de Lote**: revogada nesta feature. A transição de `Pendente` para `Não Encontrada` deixou de ser automática ao fim da varredura — ela agora é executada explicitamente pelo agente através do encerramento formal do lote (ver Feature 3, RN08).
- **RN06 — Propagação de Match entre Lotes**: se a mesma caneta (`Número de Série` + `Fabricante`) já está `Encontrada` em outro lote/investigação **não encerrado**, o novo registro herda automaticamente o status `Encontrada` e os dados de vínculo, sem nova consulta à base nacional. Lotes `ENCERRADO` não participam dessa propagação (ver Feature 3, RN09).
- **RN07 — Reabertura de "Não Encontrada"**: o status `Não Encontrada` não é definitivo enquanto o lote não estiver `ENCERRADO`. Uma varredura subsequente (no mesmo lote ou propagação de outro, via RN06) pode atualizar a caneta para `Encontrada`. Após o encerramento do lote, essa reabertura deixa de ocorrer (Feature 3, RN09).

## Critérios de Aceite (Gherkin)

```gherkin
Cenário: Match encontrado corretamente
  Dado uma caneta pendente com Número de Série "X", Fabricante "Y" e Calibre "Z"
  E existe um registro na base nacional com os três campos idênticos
  Quando a varredura roda
  Então a caneta muda para status "Encontrada"
  E fica vinculada ao CPF/CNPJ do dono_da_caneta e à data de registro nacional
  E um alerta de investigação é criado
  E o cadastro original do dono_da_caneta permanece inalterado

Cenário: Falso match evitado por calibre diferente
  Dado uma caneta pendente com Número de Série "123", Fabricante "Faber Castel" e Calibre ".380"
  E existe um registro na base nacional com Número de Série "123", Fabricante "Faber Castel" e Calibre "9mm"
  Quando a varredura roda
  Então a caneta NÃO é marcada como "Encontrada"

Cenário: Propagação de match entre lotes
  Dado que a caneta com Número de Série "X" e Fabricante "Y" já está "Encontrada" em outro lote
  E a mesma caneta aparece em um novo lote com status "Pendente"
  Quando a varredura roda no novo lote
  Então a caneta herda o status "Encontrada" sem nova consulta à base nacional

Cenário: Reabertura de caneta Não Encontrada
  Dado uma caneta com status "Não Encontrada"
  Quando uma varredura futura encontra correspondência para ela
  Então a caneta é atualizada para "Encontrada"

Cenário: Varredura duplicada bloqueada
  Dado um lote que já está com status "EM_VARREDURA"
  Quando o agente aciona "Iniciar Varredura" de novo nesse mesmo lote
  Então o sistema rejeita a segunda execução
```

## Requisitos Não Funcionais
- **Performance**: consulta em lote (bulk) contra a base nacional simulada em vez de uma chamada individual por caneta, sempre que a integração permitir.
- **Concorrência**: status `EM_VARREDURA` trava o lote contra execuções simultâneas.
- **Somente leitura na base nacional**: nenhuma escrita ou alteração no cadastro original do dono_da_caneta.
- **Auditoria**: registrar quando a varredura rodou, quem acionou, quantas canetas processadas, quantas viraram `Encontrada`/`Não Encontrada`.

## Modelo de Dados (extensão da Feature 1)

```
caneta
- ... campos existentes (numero_serie, fabricante, modelo, pais_origem, status, lote_id)
+ calibre
+ cpf_cnpj_dono_da_caneta (nullable, preenchido em caso de match)
+ data_registro_nacional (nullable)

alerta_dono_da_caneta
- id
- caneta_id (FK)
- cpf_cnpj_dono_da_caneta
- motivo ("caneta sob investigação — possível contrabando")
- criado_em
- ativo (boolean)

lote_importacao
- status_processamento passa a incluir: PROCESSANDO / CONCLUIDO / FALHOU / EM_VARREDURA / VARREDURA_CONCLUIDA
```

Cliente de integração (simulado, não é tabela nossa):
```
canetaNacionalClient
- buscarPorNumeroSerieFabricanteCalibre(serie, fabricante, calibre): Optional<canetaNacional>
```

## Endpoints Propostos
- `POST /lotes-importacao/{id}/varredura` → dispara a varredura (assíncrona, mesmo padrão da Feature 1). Funciona tanto para lotes recém-importados quanto para re-varredura de lotes já concluídos.
- `GET /lotes-importacao/{id}/varredura/status` → acompanha progresso e resultado
- `GET /canetas/{id}` → detalhe da caneta, incluindo vínculo dono_da_caneta e alerta, se houver
