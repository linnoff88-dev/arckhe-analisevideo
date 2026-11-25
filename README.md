# analisevideo

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

AnêÂ¡lise **visual e cARCKHêtogrêÂ¡fica** de vêÂ­deo com o Gemini, arquivada num banco
local pesquisêÂ¡vel. NêÂ£o êÂ© transcriêÂ§êÂ£o de fala: o que sai daqui êÂ© o que um diretor
de fotografia, um montador e um produtor musical enxergam no material.

Roda como skill do [OpenPCBot](https://github.com/linnoff88-dev/openpcbotv2) sob o
comando `/analisevideo` ê¢â¬â de propêÂ³sito **nêÂ£o dispara por vibe** (link solto ou
"analisa esse vêÂ­deo" continuam indo pro ARCKHêVOX).

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://linnoff88-dev.github.io/

## Uso

```bash
S=~/projetos/analisevideo/analisevideo.sh

bash $S analisa <url|arquivo> [slug] [--keep-src] [--prompt "..."]
bash $S ver     <slug>       # relatêÂ³rio markdown
bash $S json    <slug>       # anêÂ¡lise crua
bash $S prompts <slug>       # 1 prompt por cena (5ê¢â¬â10), pronto pra Kling/Veo/Seedance
bash $S list    [N]          # êÂºltimas anêÂ¡lises
bash $S search  "<termo>"    # busca no banco
bash $S stats                # o que mais aparece no banco
bash $S reindex              # reconstrêÂ³i o êÂ­ndice a partir das pastas
```

Precisa de `yt-dlp`, `ffmpeg`/`ffprobe`, `jq`, `python3` e uma `GOOGLE_API_KEY`
(lida de `~/projetos/openpcbotv2/.env` ou `~/projetos/wifi/.env`).

## O que a anêÂ¡lise cobre

- **Prompts por cena** ê¢â¬â 5 a 10 prompts em inglêÂªs (+ negative prompt), um por
  cena com timecode, cobrindo o vêÂ­deo inteiro.
- **Fotografia** ê¢â¬â paleta em hex, esquema de luz, temperatura, contraste, look,
  lente aparente, profundidade de campo, textura/grain.
- **CêÂ¢mera bloco a bloco** com timecode ê¢â¬â plano, êÂ¢ngulo, movimento (pan, dolly,
  gimbal, whip-pan, drone, orbital...), velocidade e nota.
- **ComposiêÂ§êÂ£o** ê¢â¬â enquadramento, regras, camadas de profundidade.
- **Montagem** ê¢â¬â cortes estimados, cortes por minuto, ritmo, transiêÂ§êÂµes, match
  cut, jump cut, corte no beat, slow-mo/speed ramp.
- **Trilha e som** ê¢â¬â gêÂªnero, BPM aproximado, instrumentaêÂ§êÂ£o, mood, SFX, mixagem
  e sincronia com a imagem.
- **Texto e grafismo** ê¢â¬â tipografia, animaêÂ§êÂ£o, legendas, motion graphics.
- **PêÂ³s** ê¢â¬â efeitos, VFX, LUT sugerida, sound design.
- **Narrativa** ê¢â¬â gancho dos 3 primeiros segundos, estrutura, arco, CTA.
- **Como refazer** ê¢â¬â equipamento mêÂ­nimo, passos prêÂ¡ticos e um prompt pronto para
  Kling/Veo/Seedance reproduzindo o look e o movimento do trecho mais forte.

## Banco

Cada anêÂ¡lise vira uma pasta em `~/projetos/output/analisevideo/<slug>/` com
`meta.json`, `analise.json` e `analise.md`. Uma linha por vêÂ­deo êÂ© acrescentada a
`index.jsonl` ê¢â¬â êÂ© o que `list`, `search` e `stats` leem. O vêÂ­deo baixado êÂ©
apagado no fim (guarde com `--keep-src`).

## Detalhes

- Download em atêÂ© 480p de propêÂ³sito: cor, luz e movimento se leem bem e o upload
  fica rêÂ¡pido. Acima de 18 MB o arquivo êÂ© reencodado para 360p/12fps sêÂ³ para a
  anêÂ¡lise.
- Link que o yt-dlp nêÂ£o pega (site logado): baixe por fora e passe o caminho.
- VêÂ­deo longo (>15 min) fica caro e vago ê¢â¬â prefira analisar um trecho.
- 429/500/503 do Gemini sêÂ£o reprocessados automaticamente (4 tentativas).

## Chaves do Gemini ê¢â¬â trêÂªs, e ele troca sozinho

O `analisa.py` tenta as chaves nesta ordem, parando na primeira que funcionar:

| variêÂ¡vel | onde |
|---|---|
| `GOOGLE_API_KEY` | a de sempre |
| `GEMINI_API_KEY` | idem (deduplicada se for o mesmo valor) |
| `GEMINI_API_KEY_ARCKHêCCBOT_TIME` | `projects/1056030032122` |
| `GEMINI_API_KEY_ARCKHêCCBOT_PROMPTS` | `projects/1000152753819` |

Todas saem de `~/projetos/wifi/.env` (ou do `.env` local).

**O que faz trocar e o que faz esperar** ê¢â¬â a distinêÂ§êÂ£o êÂ© o ponto:

- **429 (cota) e 403 (bloqueada)** ê¢â â passa para a chave seguinte **na hora**. Ela
  estêÂ¡ em outro projeto, com cota prêÂ³pria; esperar nêÂ£o resolveria nada.
- **500 / 502 / 503** ê¢â â êÂ© o Gemini congestionado. A chave nêÂ£o tem culpa, e trocar
  nêÂ£o ajuda: espera (20s, 40s, 60s, 90s, 120s) e insiste na mesma.

O upload do arquivo grande tambêÂ©m entra nessa conta: um 429 nele troca de chave
em vez de virar erro final ê¢â¬â êÂ© justamente o caso em que o vêÂ­deo jêÂ¡ foi baixado e
comprimido, e perder tudo ali seria caro.

Quando usa uma chave que nêÂ£o êÂ© a primeira, ele diz qual, no stderr. Se todas
falharem por cota, o erro lista **quais** falharam, em vez de um "deu erro".


## Sites que exigem sessêÂ£o (Facebook, Instagram, TikTok, YouTube)

O `yt-dlp` cru êÂ© recusado pelo Facebook com `Cannot parse data` ê¢â¬â nêÂ£o êÂ© versêÂ£o
velha nem falta de `--impersonate` (os dois foram testados em 2026-08-24 e
falharam igual). êâ° **sessêÂ£o**: com os cookies do Firefox o mesmo link lista os
formatos e baixa em 7 segundos.

O script acha o perfil sozinho (o primeiro com `cookies.sqlite`, procurando no
`snap` primeiro) e sêÂ³ acrescenta as flags nesses domêÂ­nios. Para forêÂ§ar outro
perfil:

    export ANALISEVIDEO_FIREFOX_PROFILE=~/snap/firefox/common/.mozilla/firefox/xxxx.default

**Sem Firefox na mêÂ¡quina** (VPS), o download segue sem cookies e avisa no stderr
ê¢â¬â para os sites pêÂºblicos continua funcionando.


## Ordem dos motores (2026-08-26: o Gemini direto vai na frente)

    GOOGLE_API_KEY ê¢â â GEMINI_API_KEY ê¢â â ê¢â¬Â¦_TIME ê¢â â ê¢â¬Â¦_PROMPTS  ê¢â â  z-ai/glm-5.3-flash

Em 2026-08-25 a reserva tinha ido para a frente porque o `stealth/ox-alpha` era
**grêÂ¡tis** e as trêÂªs chaves do Gemini estouravam a cota diêÂ¡ria com ~9 anêÂ¡lises.
Esse modelo **sumiu do OpenRouter** ê¢â¬â era o risco anotado desde o comeêÂ§o. A
reserva de hoje êÂ© paga, entêÂ£o ela volta para trêÂ¡s: gastar crêÂ©dito com cota do
Gemini de sobra nêÂ£o faz sentido. Ela entra quando a cota do dia acaba.

Para inverter (a reserva na frente), **sem tocar em cêÂ³digo**:

    export ANALISEVIDEO_MOTOR=reserva

## Reserva: quando o motor da frente recusa

O passo caro do `analisa` nêÂ£o êÂ© a chamada de API ê¢â¬â êÂ© o **download e a
compressêÂ£o** que vêÂªm antes. Um clipe de 41 MB do Facebook leva minutos para
chegar atêÂ© ali. Em 2026-08-24, trêÂªs chaves distintas do Gemini responderam 429
no mesmo minuto e esse trabalho todo foi jogado fora.

Por isso existe a reserva. A ordem êÂ©:

    GOOGLE_API_KEY ê¢â â GEMINI_API_KEY ê¢â â ê¢â¬Â¦_TIME ê¢â â ê¢â¬Â¦_PROMPTS ê¢â â z-ai/glm-5.3-flash

O `z-ai/glm-5.3-flash` (OpenRouter, `OPENROUTER_API_KEY` no `wifi/.env`) aceita
vêÂ­deo e devolve o MESMO JSON ê¢â¬â o prompt nêÂ£o muda em nada. Entrou em 2026-09-08 no
lugar do `google/gemini-3.7-flash`: custa ~10x menos (US$ 0,075/M de entrada
contra 0,75) e, sendo outro provedor, nêÂ£o divide a cota nem a congestêÂ£o com o
Gemini da frente.

**VêÂ­deo no OpenRouter exige US$ 1,00 disponêÂ­veis NA CHAVE** ê¢â¬â nêÂ£o na conta. Uma
chave com teto (`limit` em openrouter.ai/settings/keys) abaixo disso responde
`402 ê¢â¬Â¦ requires at least $1.00 in balance for video` para qualquer vêÂ­deo, mesmo
um clipe de 20 s, enquanto texto passa. Foi assim que a reserva ficou muda de
2026-08 atêÂ© 2026-09-08. O `analisa` agora imprime o corpo do erro.

Ele fica ATRêÂS de propêÂ³sito: **êÂ© pago**. Sempre que ele responde, o `analisa`
avisa no stderr (`ATENCAO: ... reserva paga`) ê¢â¬â motor pago nêÂ£o roda calado.
Para trocar o modelo da reserva, `OPENROUTER_VIDEO_MODEL`.

`_modelo` no JSON registra QUEM analisou: sem isso o banco diria Gemini para uma
anêÂ¡lise que o Gemini nêÂ£o fez.

Sem `OPENROUTER_API_KEY`, tudo funciona como antes ê¢â¬â a reserva êÂ© opcional.


## Reprocessar sem baixar de novo

O slug vem do TêÂTULO, e o Facebook devolve "Facebook" para todo link ê¢â¬â entêÂ£o
cada tentativa criava `facebook-2`, `-3`, `-4`ê¢â¬Â¦ e baixava tudo outra vez. Em
2026-08-24 o MESMO clipe foi baixado **quatro vezes** (141 MB de rede e disco)
por um erro de cota que nêÂ£o tinha nada a ver com o download.

Agora a **URL êÂ© a identidade**: se jêÂ¡ houver uma pasta com aquela url e o
arquivo ainda no disco, ela êÂ© reusada ê¢â¬â download e compressêÂ£o sêÂ£o pulados. E a
versêÂ£o comprimida (`analise-src.mp4`), se existir, tambêÂ©m êÂ© reaproveitada.

Duas consequêÂªncias prêÂ¡ticas:

- **Repetir `analisa <url>` depois de um erro êÂ© barato** ê¢â¬â vai direto para a
  anêÂ¡lise. êâ° o caminho para reprocessar o que falhou por cota.
- **`--keep-src` passa a valer a pena** em vêÂ­deo que vocêÂª pode querer reanalisar
  (com `--prompt` diferente, por exemplo). Sem ele, o arquivo êÂ© apagado no fim e
  a prêÂ³xima anêÂ¡lise baixa de novo.

Passar um `slug` explêÂ­cito desliga o reuso: ali vocêÂª estêÂ¡ dizendo qual pasta
quer, e adivinhar outra seria pior.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>