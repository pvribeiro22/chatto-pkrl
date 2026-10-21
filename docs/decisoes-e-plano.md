# chatto-pkrl — decisões e plano

Consolidado do que foi decidido antes de escrever a primeira linha de código. Cada item
foi escolhido explicitamente; nada aqui é default silencioso.

## Decisões travadas

| # | Decisão |
|---|---|
| Escopo | Uso pessoal, número dedicado em SIM físico (TIM pré-pago, ~R$8/mês; receber SMS não renova) |
| Integração | Baileys (não-oficial) — ToS/ban aceitos como risco, mitigado por não enviar em massa nem entrar em grupos |
| Contatos | Allowlist; estranho fica em silêncio; aviso único ao owner com trecho da mensagem; `add`/`ignora` só do número do owner; `ignora` = bloqueio permanente |
| Grupos | Fora da v1 |
| LLM | Gemini Flash-Lite — fornecedor único, cobre texto+visão+áudio Ogg sem transcodificar; pré-pago US$5, IOF 3,5%, crédito expira em 12 meses |
| Memória | SQLite grava tudo; prompt recebe só as últimas 20 mensagens |
| Figurinha | 512×512 WebP, padding transparente, qualidade decrescente até ≤100 KB; só estática; gatilho determinístico primeiro, LLM depois |
| Lembretes | Linguagem natural, `America/Sao_Paulo`, dispara atrasado ao voltar |
| Personalidade | Arquivo versionado, overrides por contato; sem admin de persona no chat |
| Custos | 30 respostas/h por contato, ≤500 tokens por resposta, teto ~US$1/dia com aviso único |
| Dados | Mídia baixada sob demanda, apagada em 30 dias; backup diário do SQLite, 7 cópias |
| Stack | Node 22 + TypeScript + Baileys + better-sqlite3 + systemd, sem Docker |

## Estrutura

```
whatsbot/
  config.yaml              # owner, allowlist/blocklist inicial, personas, limites
  src/
    index.ts               # boot, reconexão com backoff, /health
    config.ts              # load + validação (zod)
    wa/connection.ts       # socket Baileys, pairing code, auth persistido
    wa/inbound.ts          # normaliza mensagem -> Msg
    pipeline.ts            # allowlist -> persiste -> figurinha -> lembrete -> LLM
    stickers.ts            # sharp, loop de qualidade
    llm.ts                 # Gemini, montagem de prompt, retry
    reminders.ts           # tick de 1 min, catch-up
    db.ts                  # schema + migrations
    media.ts               # download sob demanda, limpeza 30d
    limit.ts               # rate limit + teto diário
  data/                    # session/, media/, bot.sqlite
  test/
```

## Schema do banco

```sql
contacts(jid, push_name, allowlisted, persona)
messages(id, chat_jid, sender_jid, ts, kind, text, media_path)
reminders(id, chat_jid, due_at, text, status, created_at)
personas(name, tom, formalidade, humor, grosseria, max_linhas, emojis, idioma, assuntos_proibidos)
```

## Ordem de construção

1. **Esqueleto** — config + schema + migrations. *Pronto:* banco criado do zero.
2. **Conexão** — pairing code, sessão em disco, `pong`. *Pronto:* reiniciar o processo não
   pede pareamento de novo.
3. **Pipeline + allowlist** — persistência, silêncio para estranho, aviso ao owner,
   `add`/`ignora`. *Pronto:* `add` vale na mensagem seguinte, sem restart.
4. **Figurinha** — determinístico + padding + loop de 100 KB. *Pronto:* foto 4:3 com "fig"
   e reply com "figurinha" viram sticker válido.
5. **Conversa** — persona por contato, 20 mensagens de contexto, limites de custo.
   *Pronto:* dois contatos com tons diferentes; a 31ª mensagem/hora não recebe resposta.
6. **Lembretes** — parse natural, tick, catch-up. *Pronto:* agendar 09h, derrubar o
   processo às 08h59, subir às 09h05 → chega com aviso de atraso.
7. **Mídia** — transcrição de áudio e visão em foto. *Pronto:* áudio de 30 s respondido
   corretamente em texto.
8. **Endurecimento** — `/health` + UptimeRobot, backup diário, log rotacionado, teto de
   gasto testado.

## Riscos explícitos

- Baileys está em RC há meses (v7.0.0-rc14) — congelar a versão.
- Sessão pode cair e exigir novo pareamento; o catch-up de lembretes existe por isso.
- O free tier do Gemini tem limite diário publicado só no console; o teto de US$1 é a
  proteção real.
- Número não-oficial viola o ToS do WhatsApp e pode ser banido; é o custo aceito da rota.

## v2

Grupos, figurinha animada, busca no histórico, TTS, admin de persona no chat.
