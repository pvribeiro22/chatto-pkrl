---
name: whatsapp-media
description: Specs e pegadinhas de figurinha (WebP 512×512, limite de 100 KB) e de áudio (Ogg/Opus) no WhatsApp com Baileys. Ler antes de implementar stickers.ts, media.ts ou transcrição.
---

# Mídia no WhatsApp com Baileys

Fatos verificados em outubro/2026. O que não tem fonte aqui é marcado como ⚠.

## Figurinha — spec dura

- **512×512 WebP com transparência**. Estática **≤100 KB**; animada **≤500 KB**.
- Animada: cada frame ≥8 ms, total ≤10 s, primeiro frame precisa ser a imagem completa.
- Animada usa chunks `ANIM`/`ANMF` com frames VP8/VP8L. VP9 não é codec de WebP.
  [spec](https://developers.google.com/speed/webp/docs/riff_container#animation)
- Pacote de figurinhas: 3–30, nunca misturado; tray icon 96×96 PNG ≤50 KB.
  [sticker README](https://github.com/WhatsApp/stickers/blob/main/Android/README.md)

## Conversão estática (o caminho da v1)

```ts
sharp(input)
  .autoOrient()                       // sem isso o EXIF roda a foto
  .resize(512, 512, { fit: 'contain', background: { r: 0, g: 0, b: 0, alpha: 0 } })
  .webp({ quality: 80, effort: 6 })
```

- `autoOrient()` vem antes do resize, sempre — sharp descarta metadados por default.
- `contain` = padding transparente (não corta); `cover` = crop central. A v1 usa `contain`.
- Nunca esticar. JPEG não tem alfa: comece de canvas transparente, não achate em branco.
- **O limite de 100 KB é o problema real**: 512² de foto estoura com facilidade em q80.
  Loop de qualidade/effort para baixo até caber; se não couber, avisar em vez de mandar
  arquivo inválido.

## Animada (v2)

- ffmpeg de vídeo/GIF: `-c:v libwebp_anim`. `libwebp` puro gera still ou animação quebrada.
  [ticket](https://trac.ffmpeg.org/ticket/10200)
- sharp também reencoda animado com `{ animated: true }` + `webp({ loop: 0, delay })`.
- `wa-sticker-formatter` funciona para protótipo, mas está parado desde 2023 (sharp 0.30).

## Envio pelo Baileys

```ts
await sock.sendMessage(jid, { sticker: webpBuffer, isAnimated: false })
```

- `WAMediaUpload` aceita `Buffer`, `{ url }` ou `{ stream }`; MIME default `image/webp`.
- **Baileys não infere `isAnimated`** — setar na mão para animada.
- Baileys envia os bytes como estão; não há reencode escondido.

## Áudio que chega

- Voz do WhatsApp é **Ogg/Opus, mono**. `audioMessage` traz `mimetype:
  'audio/ogg; codecs=opus'`, `ptt: true`, `seconds`, `waveform`.
- Áudio completo exige `downloadMediaMessage`. Mídia expira: passe
  `reuploadRequest: sock.updateMediaMessage` para o Baileys pedir de novo (senão 404/410).
- Limite duro: Ogg até **16 MB**; o ícone de play só aparece em **≤512 KB**.
  [Meta audio](https://developers.facebook.com/docs/whatsapp/cloud-api/messages/audio-messages)
- Bitrate típico 12–16 kbps / 16 kHz ⚠ (fonte secundária); duração máxima não é documentada
  para consumidor — trate 16 MB como o limite.

## Transcrição

- **Gemini aceita `audio/ogg` e `audio/opus` direto**, 20 MB inline, até 9,5 h por prompt.
  [docs](https://ai.google.dev/gemini-api/docs/audio) — é por isso que o projeto usa
  fornecedor único.
- **OpenAI Transcriptions não lista Ogg/Opus** (mp3, mp4, mpeg, mpga, m4a, wav, webm):
  exigiria transcodificar. `whisper-1` está depreciado (desligamento 26/fev/2027).
- `faster-whisper` dispensa ffmpeg do sistema (usa PyAV). `whisper.cpp` CLI só aceita WAV
  16-bit (`ffmpeg -i in.ogg -ar 16000 -ac 1 -c:a pcm_s16le out.wav`).

## Áudio de saída (v2)

```bash
ffmpeg -i tts.mp3 -c:a libopus -ac 1 -ar 48000 -b:a 32k -avoid_negative_ts make_zero out.ogg
```

Enviar `{ audio: buf, mimetype: 'audio/ogg; codecs=opus', ptt: true }`. Sem waveform
explícita o Baileys calcula uma de 64 bytes (precisa de `audio-decode`); sem isso a bolha
pode sair com waveform achatada. Manter ≤512 KB para virar bolha de voz.

## Imagem que chega

- `imageMessage` traz `jpegThumbnail` (base64, só preview) + `url`/`directPath`. Visão e OCR
  precisam do arquivo completo via `downloadMediaMessage`.
- Limite Cloud API: imagem 5 MB, áudio 16 MB. Para Baileys, 16 MB inline é ⚠ não documentado.
