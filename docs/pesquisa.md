# Fatos externos verificados

Levantado em outubro/2026. Serve para não re-pesquisar o que já foi checado — e para saber
o que **não** foi confirmado.

## Rota de integração

- **Cloud API (oficial)** não tem tier pessoal: é produto business, com janela de 24h,
  templates fora dela e número que não esteja em uso no WhatsApp. Número de teste grátis
  fala só com 5 destinatários. Verificação de empresa não é exigida para começar (250
  destinatários únicos/24h, 2 números), só para escalar.
  - [phone numbers](https://developers.facebook.com/docs/whatsapp/cloud-api/phone-numbers/),
    [limits](https://developers.facebook.com/docs/whatsapp/messaging-limits/)
  - Webhook exige HTTPS público com CA válida; assinatura em `X-Hub-Signature-256`.
- **Não-oficial**: Baileys (MIT, TS socket multi-device) — último tag `v7.0.0-rc14`
  (29/jul/2026), ativo. `whatsapp-web.js` (Chromium headless) e `venom` também mantidos.
  - ToS proíbe uso não pessoal sem autorização; as Messaging Guidelines citam "uso de
    clientes não-oficiais" como comportamento adversarial, com suspensão temporária ou
    permanente. [ToS](https://www.whatsapp.com/legal/terms-of-service),
    [guidelines](https://www.whatsapp.com/legal/messaging-guidelines)
  - Não existe estatística pública de taxa de ban. Relatos se concentram em envio em
    massa, status e grupos.
- **BSPs** (Twilio, 360dialog, Gupshup): reduzem atrito de onboarding, não a exposição a
  ban. Custam por número/mês (€49–99) ou por mensagem. Z-API, Evolution API e WPPConnect
  **não** são BSPs — são wrappers de Baileys/WhatsApp Web.

## Preços WhatsApp (desde 1/out/2026)

- Cobrança por mensagem entregue (por conversa está depreciada). Entrada é grátis.
- Cada número tem **1.000 mensagens de serviço grátis/mês**, depois tarifa de utility.
  Templates de utility dentro da janela também são cobrados.
- Brasil: ~**R$0,035** service/utility e ~**R$0,32** marketing por mensagem.
  [pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing)
- "Serviço é grátis" era verdade até 30/set/2026 — tutoriais antigos repetem a regra velha.

## Cobrança de LLM

- **Nenhuma assinatura de consumidor dá acesso à API**: ChatGPT Plus/Pro e Google AI Pro
  são cobranças separadas; só Claude Max/Team incluem créditos de API.
- OpenAI e Google: **pré-pago**, mínimo **US$5**, não reembolsável, expira em **12 meses**.
  Anthropic: pós-pago mensal.
- IOF de **3,5%** em compra com cartão internacional (Decreto 12.499/2025, art. 15-B VII).
- Custo no volume do projeto (300 in + 150 out tokens/turno): 1.000 turnos ≈
  US$0,08 (GPT-5-nano), US$0,11–0,14 (Haiku 5.5), US$0,47 (Gemini Flash-Lite).
  100 imagens 1024² ≈ US$0,01–0,03. 100 áudios de 30 s ≈ US$0,02 (Gemini) contra
  US$0,15–0,75 (OpenAI); **Claude não aceita áudio**.
- Free tier do Gemini Flash-Lite ainda existe; o RPD exato só aparece no console
  (⚠ não verificado publicamente).

## SIM pré-pago (Brasil)

- Anatel (Res. 765/2023, art. 51): crédito mínimo de 30 dias; operadoras oferecem 90 e 180.
  Vencido: 15 dias de aviso → 30 dias suspensão parcial (não envia SMS) → 30 dias total →
  cancelamento em **75 dias**. Receber SMS **não** renova.
- TIM: R$50 ≈ 180 dias (~R$8,30/mês) — a opção mais barata das três grandes.
- WhatsApp recusa número VoIP/virtual, 0800 e premium. eSIM pré-pago existe (Claro, TIM,
  Vivo), mas precisa ser linha móvel de operadora real.
- Oi não tem mais pré-pago móvel (vendido para Claro/TIM/Vivo em 2022).
