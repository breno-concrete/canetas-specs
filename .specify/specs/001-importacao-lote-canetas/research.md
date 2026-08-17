## Mecanismo de processamento assíncrono

**Pergunta**: como processar o CSV em background sem bloquear o POST,
respeitando o NFR de resposta <2s?
**Decisão**: `ManagedExecutor.runAsync()` disparado a partir do
`ImportarLoteUseCase`, processamento roda em worker thread.
**Justificativa**: projeto é single-instance nesta fase, já tem Postgres
mas não tem broker de mensageria configurado. Subir Kafka só pra essa
feature seria complexidade desproporcional ao estágio atual (constitution
não exige durabilidade de infraestrutura). ManagedExecutor é builtin do
Quarkus/MicroProfile, sem dependência nova.
**Alternativas consideradas**:
- SmallRye Reactive Messaging (Kafka) — rejeitada: exige broker novo,
  fora de escopo do MVP desta feature.
- Quartz persistido — rejeitada por ora: resolveria a durabilidade, mas
  adiciona tabela/infra de jobs só pra um caso de recuperação que ainda
  não é requisito confirmado da spec. Revisar se durabilidade virar
  requisito explícito.
  **Referências**: quarkus.io/guides (ManagedExecutor / MicroProfile Context Propagation)

## Recuperação de lote travado em PROCESSANDO após restart

**Pergunta**: o que acontece com um lote se a aplicação reiniciar no meio
do processamento?
**Decisão**: aceito como limitação conhecida do MVP — lote fica em
PROCESSANDO indefinidamente; não há retomada automática nesta versão.
**Justificativa**: spec não define requisito de durabilidade; resolver
isso agora empurraria a decisão anterior para Quartz/fila, aumentando
escopo sem pedido explícito do usuário da feature.
**Alternativas consideradas**:
- Job persistido com retomada automática (Quartz) — adiado: vira débito
  técnico registrado aqui, não decisão silenciosa.
  **Ação de acompanhamento**: se durabilidade virar requisito, revisar
  esta decisão e a anterior juntas — estão acopladas.