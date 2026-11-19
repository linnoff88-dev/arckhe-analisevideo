# analisevideo

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

AnÃ¡lise **visual e cinematogrÃ¡fica** de vÃ­deo com o Gemini, arquivada num banco
local pesquisÃ¡vel. NÃ£o Ã© transcriÃ§Ã£o de fala: o que sai daqui Ã© o que um diretor
de fotografia, um montador e um produtor musical enxergam no material.

Roda como skill do [OpenPCBot](https://github.com/inematds/openpcbotv2) sob o
comando `/analisevideo` â€” de propÃ³sito **nÃ£o dispara por vibe** (link solto ou
"analisa esse vÃ­deo" continuam indo pro inemaVOX).

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/analisevideo/guia/**

## Uso

```bash
S=~/projetos/analisevideo/analisevideo.sh

bash $S analisa <url|arquivo> [slug] [--keep-src] [--prompt "..."]
bash $S ver     <slug>       # relatÃ³rio markdown
bash $S json    <slug>       # anÃ¡lise crua
bash $S prompts <slug>       # 1 prompt por cena (5â€“10), pronto pra Kling/Veo/Seedance
bash $S list    [N]          # Ãºltimas anÃ¡lises
bash $S search  "<termo>"    # busca no banco
bash $S stats                # o que mais aparece no banco
bash $S reindex              # reconstrÃ³i o Ã­ndice a partir das pastas
```

Precisa de `yt-dlp`, `ffmpeg`/`ffprobe`, `jq`, `python3` e uma `GOOGLE_API_KEY`
(lida de `~/projetos/openpcbotv2/.env` ou `~/projetos/wifi/.env`).

## O que a anÃ¡lise cobre

- **Prompts por cena** â€” 5 a 10 prompts em inglÃªs (+ negative prompt), um por
  cena com timecode, cobrindo o vÃ­deo inteiro.
- **Fotografia** â€” paleta em hex, esquema de luz, temperatura, contraste, look,
  lente aparente, profundidade de campo, textura/grain.
- **CÃ¢mera bloco a bloco** com timecode â€” plano, Ã¢ngulo, movimento (pan, dolly,
  gimbal, whip-pan, drone, orbital...), velocidade e nota.
- **ComposiÃ§Ã£o** â€” enquadramento, regras, camadas de profundidade.
- **Montagem** â€” cortes estimados, cortes por minuto, ritmo, transiÃ§Ãµes, match
  cut, jump cut, corte no beat, slow-mo/speed ramp.
- **Trilha e som** â€” gÃªnero, BPM aproximado, instrumentaÃ§Ã£o, mood, SFX, mixagem
  e sincronia com a imagem.
- **Texto e grafismo** â€” tipografia, animaÃ§Ã£o, legendas, motion graphics.
- **PÃ³s** â€” efeitos, VFX, LUT sugerida, sound design.
- **Narrativa** â€” gancho dos 3 primeiros segundos, estrutura, arco, CTA.
- **Como refazer** â€” equipamento mÃ­nimo, passos prÃ¡ticos e um prompt pronto para
  Kling/Veo/Seedance reproduzindo o look e o movimento do trecho mais forte.

## Banco

Cada anÃ¡lise vira uma pasta em `~/projetos/output/analisevideo/<slug>/` com
`meta.json`, `analise.json` e `analise.md`. Uma linha por vÃ­deo Ã© acrescentada a
`index.jsonl` â€” Ã© o que `list`, `search` e `stats` leem. O vÃ­deo baixado Ã©
apagado no fim (guarde com `--keep-src`).

## Detalhes

- Download em atÃ© 480p de propÃ³sito: cor, luz e movimento se leem bem e o upload
  fica rÃ¡pido. Acima de 18 MB o arquivo Ã© reencodado para 360p/12fps sÃ³ para a
  anÃ¡lise.
- Link que o yt-dlp nÃ£o pega (site logado): baixe por fora e passe o caminho.
- VÃ­deo longo (>15 min) fica caro e vago â€” prefira analisar um trecho.
- 429/500/503 do Gemini sÃ£o reprocessados automaticamente (4 tentativas).

## Chaves do Gemini â€” trÃªs, e ele troca sozinho

O `analisa.py` tenta as chaves nesta ordem, parando na primeira que funcionar:

| variÃ¡vel | onde |
|---|---|
| `GOOGLE_API_KEY` | a de sempre |
| `GEMINI_API_KEY` | idem (deduplicada se for o mesmo valor) |
| `GEMINI_API_KEY_INEMACCBOT_TIME` | `projects/1056030032122` |
| `GEMINI_API_KEY_INEMACCBOT_PROMPTS` | `projects/1000152753819` |

Todas saem de `~/projetos/wifi/.env` (ou do `.env` local).

**O que faz trocar e o que faz esperar** â€” a distinÃ§Ã£o Ã© o ponto:

- **429 (cota) e 403 (bloqueada)** â†’ passa para a chave seguinte **na hora**. Ela
  estÃ¡ em outro projeto, com cota prÃ³pria; esperar nÃ£o resolveria nada.
- **500 / 502 / 503** â†’ Ã© o Gemini congestionado. A chave nÃ£o tem culpa, e trocar
  nÃ£o ajuda: espera (20s, 40s, 60s, 90s, 120s) e insiste na mesma.

O upload do arquivo grande tambÃ©m entra nessa conta: um 429 nele troca de chave
em vez de virar erro final â€” Ã© justamente o caso em que o vÃ­deo jÃ¡ foi baixado e
comprimido, e perder tudo ali seria caro.

Quando usa uma chave que nÃ£o Ã© a primeira, ele diz qual, no stderr. Se todas
falharem por cota, o erro lista **quais** falharam, em vez de um "deu erro".


## Sites que exigem sessÃ£o (Facebook, Instagram, TikTok, YouTube)

O `yt-dlp` cru Ã© recusado pelo Facebook com `Cannot parse data` â€” nÃ£o Ã© versÃ£o
velha nem falta de `--impersonate` (os dois foram testados em 2026-08-24 e
falharam igual). Ã‰ **sessÃ£o**: com os cookies do Firefox o mesmo link lista os
formatos e baixa em 7 segundos.

O script acha o perfil sozinho (o primeiro com `cookies.sqlite`, procurando no
`snap` primeiro) e sÃ³ acrescenta as flags nesses domÃ­nios. Para forÃ§ar outro
perfil:

    export ANALISEVIDEO_FIREFOX_PROFILE=~/snap/firefox/common/.mozilla/firefox/xxxx.default

**Sem Firefox na mÃ¡quina** (VPS), o download segue sem cookies e avisa no stderr
â€” para os sites pÃºblicos continua funcionando.


## Ordem dos motores (2026-08-26: o Gemini direto vai na frente)

    GOOGLE_API_KEY â†’ GEMINI_API_KEY â†’ â€¦_TIME â†’ â€¦_PROMPTS  â†’  z-ai/glm-5.3-flash

Em 2026-08-25 a reserva tinha ido para a frente porque o `stealth/ox-alpha` era
**grÃ¡tis** e as trÃªs chaves do Gemini estouravam a cota diÃ¡ria com ~9 anÃ¡lises.
Esse modelo **sumiu do OpenRouter** â€” era o risco anotado desde o comeÃ§o. A
reserva de hoje Ã© paga, entÃ£o ela volta para trÃ¡s: gastar crÃ©dito com cota do
Gemini de sobra nÃ£o faz sentido. Ela entra quando a cota do dia acaba.

Para inverter (a reserva na frente), **sem tocar em cÃ³digo**:

    export ANALISEVIDEO_MOTOR=reserva

## Reserva: quando o motor da frente recusa

O passo caro do `analisa` nÃ£o Ã© a chamada de API â€” Ã© o **download e a
compressÃ£o** que vÃªm antes. Um clipe de 41 MB do Facebook leva minutos para
chegar atÃ© ali. Em 2026-08-24, trÃªs chaves distintas do Gemini responderam 429
no mesmo minuto e esse trabalho todo foi jogado fora.

Por isso existe a reserva. A ordem Ã©:

    GOOGLE_API_KEY â†’ GEMINI_API_KEY â†’ â€¦_TIME â†’ â€¦_PROMPTS â†’ z-ai/glm-5.3-flash

O `z-ai/glm-5.3-flash` (OpenRouter, `OPENROUTER_API_KEY` no `wifi/.env`) aceita
vÃ­deo e devolve o MESMO JSON â€” o prompt nÃ£o muda em nada. Entrou em 2026-09-08 no
lugar do `google/gemini-3.7-flash`: custa ~10x menos (US$ 0,075/M de entrada
contra 0,75) e, sendo outro provedor, nÃ£o divide a cota nem a congestÃ£o com o
Gemini da frente.

**VÃ­deo no OpenRouter exige US$ 1,00 disponÃ­veis NA CHAVE** â€” nÃ£o na conta. Uma
chave com teto (`limit` em openrouter.ai/settings/keys) abaixo disso responde
`402 â€¦ requires at least $1.00 in balance for video` para qualquer vÃ­deo, mesmo
um clipe de 20 s, enquanto texto passa. Foi assim que a reserva ficou muda de
2026-08 atÃ© 2026-09-08. O `analisa` agora imprime o corpo do erro.

Ele fica ATRÃS de propÃ³sito: **Ã© pago**. Sempre que ele responde, o `analisa`
avisa no stderr (`ATENCAO: ... reserva paga`) â€” motor pago nÃ£o roda calado.
Para trocar o modelo da reserva, `OPENROUTER_VIDEO_MODEL`.

`_modelo` no JSON registra QUEM analisou: sem isso o banco diria Gemini para uma
anÃ¡lise que o Gemini nÃ£o fez.

Sem `OPENROUTER_API_KEY`, tudo funciona como antes â€” a reserva Ã© opcional.


## Reprocessar sem baixar de novo

O slug vem do TÃTULO, e o Facebook devolve "Facebook" para todo link â€” entÃ£o
cada tentativa criava `facebook-2`, `-3`, `-4`â€¦ e baixava tudo outra vez. Em
2026-08-24 o MESMO clipe foi baixado **quatro vezes** (141 MB de rede e disco)
por um erro de cota que nÃ£o tinha nada a ver com o download.

Agora a **URL Ã© a identidade**: se jÃ¡ houver uma pasta com aquela url e o
arquivo ainda no disco, ela Ã© reusada â€” download e compressÃ£o sÃ£o pulados. E a
versÃ£o comprimida (`analise-src.mp4`), se existir, tambÃ©m Ã© reaproveitada.

Duas consequÃªncias prÃ¡ticas:

- **Repetir `analisa <url>` depois de um erro Ã© barato** â€” vai direto para a
  anÃ¡lise. Ã‰ o caminho para reprocessar o que falhou por cota.
- **`--keep-src` passa a valer a pena** em vÃ­deo que vocÃª pode querer reanalisar
  (com `--prompt` diferente, por exemplo). Sem ele, o arquivo Ã© apagado no fim e
  a prÃ³xima anÃ¡lise baixa de novo.

Passar um `slug` explÃ­cito desliga o reuso: ali vocÃª estÃ¡ dizendo qual pasta
quer, e adivinhar outra seria pior.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>