# chatto-pkrl

Bot pessoal de WhatsApp: número dedicado, uso próprio, sem multi-tenant.

Estado: **planejamento fechado, nenhum código escrito.** O próximo passo é o item 1 da
ordem de construção.

## Onde olhar

- `docs/decisoes-e-plano.md` — decisões travadas, estrutura de arquivos, ordem de
  construção com critério de pronto. Fonte da verdade do escopo.
- `docs/pesquisa.md` — fatos externos verificados (rotas de integração, preços, SIM,
  cobrança de API). Cheque aqui antes de pesquisar de novo.
- `.skills/whatsapp-media/SKILL.md` — spec de figurinha e áudio mais pegadinhas do
  Baileys. Leia antes de mexer em mídia.
- `.skills/README.md` — convenção das skills locais do projeto.

## Regras

- Node 22 + TypeScript, `better-sqlite3`, Baileys, systemd na VPS. Sem Docker.
- Baileys fica pinado na versão exata; subir versão só com teste de pareamento e de
  envio de figurinha.
- `data/` (sessão, mídia, banco) nunca entra no git.
- Decisão de escopo nova atualiza `docs/decisoes-e-plano.md` no mesmo commit.

## Fluxo de cada mensagem

allowlist → persiste → figurinha (determinístico) → lembrete → LLM.
Detalhes em `docs/decisoes-e-plano.md`.
