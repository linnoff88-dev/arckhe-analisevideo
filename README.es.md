# analisevideo

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

AnÃ¡lisis **visual y cARCKHÉtogrÃ¡fico** de video con Gemini, archivado en una base
de datos local consultable. No es una transcripciÃ³n del habla: lo que sale de
aquÃ­ es lo que un director de fotografÃ­a, un editor y un productor musical ven
en el material.

Funciona como skill de [OpenPCBot](https://github.com/linnoff88-dev/openpcbotv2) con
el comando `/analisevideo` â€” deliberadamente **no se activa por vibra** (un
enlace suelto o Â«analiza este videoÂ» siguen yendo a ARCKHÉVOX).

## ðŸ“– GuÃ­a de uso

GuÃ­a completa (landing + paso a paso): **https://linnoff88-dev.github.io/

## Uso

```bash
S=~/projetos/analisevideo/analisevideo.sh

bash $S analisa <url|arquivo> [slug] [--keep-src] [--prompt "..."]
bash $S ver     <slug>       # informe markdown
bash $S json    <slug>       # anÃ¡lisis sin procesar
bash $S prompts <slug>       # 1 prompt por escena (5â€“10), listo para Kling/Veo/Seedance
bash $S list    [N]          # Ãºltimos anÃ¡lisis
bash $S search  "<termo>"    # bÃºsqueda en la base de datos
bash $S stats                # lo que mÃ¡s aparece en la base de datos
bash $S reindex              # reconstruye el Ã­ndice a partir de las carpetas
```

Necesita `yt-dlp`, `ffmpeg`/`ffprobe`, `jq`, `python3` y una `GOOGLE_API_KEY`
(leÃ­da de `~/projetos/openpcbotv2/.env` o `~/projetos/wifi/.env`).

## QuÃ© cubre el anÃ¡lisis

- **Prompts por escena** â€” de 5 a 10 prompts en inglÃ©s (+ negative prompt), uno
  por escena con cÃ³digo de tiempo, que cubren todo el video.
- **FotografÃ­a** â€” paleta en hex, esquema de iluminaciÃ³n, temperatura, contraste,
  look, lente aparente, profundidad de campo, textura/grain.
- **CÃ¡mara bloque a bloque** con cÃ³digo de tiempo â€” plano, Ã¡ngulo, movimiento
  (pan, dolly, gimbal, whip-pan, drone, orbital...), velocidad y nota.
- **ComposiciÃ³n** â€” encuadre, reglas, capas de profundidad.
- **Montaje** â€” cortes estimados, cortes por minuto, ritmo, transiciones, match
  cut, jump cut, corte al beat, slow-mo/speed ramp.
- **Banda sonora y sonido** â€” gÃ©nero, BPM aproximado, instrumentaciÃ³n, mood, SFX,
  mezcla y sincronÃ­a con la imagen.
- **Texto y grÃ¡ficos** â€” tipografÃ­a, animaciÃ³n, subtÃ­tulos, motion graphics.
- **PosproducciÃ³n** â€” efectos, VFX, LUT sugerida, sound design.
- **Narrativa** â€” gancho de los primeros 3 segundos, estructura, arco, CTA.
- **CÃ³mo recrearlo** â€” equipo mÃ­nimo, pasos prÃ¡cticos y un prompt listo para
  Kling/Veo/Seedance que reproduce el look y el movimiento del fragmento mÃ¡s
  potente.

## Base de datos

Cada anÃ¡lisis se convierte en una carpeta en `~/projetos/output/analisevideo/<slug>/` con
`meta.json`, `analise.json` y `analise.md`. Se aÃ±ade una lÃ­nea por video a
`index.jsonl` â€” es lo que leen `list`, `search` y `stats`. El video descargado se
borra al final (guÃ¡rdalo con `--keep-src`).

## Detalles

- Descarga hasta 480p deliberadamente: el color, la luz y el movimiento se leen
  bien y la carga es rÃ¡pida. Por encima de 18 MB, el archivo se vuelve a
  codificar a 360p/12fps solo para el anÃ¡lisis.
- Si el enlace no se puede descargar con yt-dlp (sitio con sesiÃ³n iniciada),
  descÃ¡rgalo por otro medio y pasa la ruta.
- Los videos largos (>15 min) son costosos y vagos â€” es mejor analizar un
  fragmento.
- Los errores 429/500/503 de Gemini se reprocesan automÃ¡ticamente (4 intentos).

## Claves de Gemini â€” tres, y cambia automÃ¡ticamente

`analisa.py` intenta las claves en este orden y se detiene en la primera que
funciona:

| variable | dÃ³nde |
|---|---|
| `GOOGLE_API_KEY` | la de siempre |
| `GEMINI_API_KEY` | idem (se deduplica si tiene el mismo valor) |
| `GEMINI_API_KEY_ARCKHÉCCBOT_TIME` | `projects/1056030032122` |
| `GEMINI_API_KEY_ARCKHÉCCBOT_PROMPTS` | `projects/1000152753819` |

Todas se obtienen de `~/projetos/wifi/.env` (o del `.env` local).

**QuÃ© hace que cambie y quÃ© hace que espere** â€” esa es la distinciÃ³n clave:

- **429 (cuota) y 403 (bloqueada)** â†’ pasa a la siguiente clave **de inmediato**.
  EstÃ¡ en otro proyecto, con cuota propia; esperar no resolverÃ­a nada.
- **500 / 502 / 503** â†’ Gemini estÃ¡ congestionado. La clave no tiene la culpa y
  cambiarla no ayuda: espera (20s, 40s, 60s, 90s, 120s) y vuelve a intentarlo
  con la misma.

La carga del archivo grande tambiÃ©n entra en este proceso: si devuelve un 429,
cambia de clave en vez de convertirse en un error definitivo â€” justo el caso en
que el video ya se descargÃ³ y comprimiÃ³, y perder todo ese trabajo serÃ­a costoso.

Cuando usa una clave que no es la primera, indica cuÃ¡l en stderr. Si todas
fallan por cuota, el error enumera **cuÃ¡les** fallaron, en vez de limitarse a
decir Â«hubo un errorÂ».


## Sitios que exigen sesiÃ³n (Facebook, Instagram, TikTok, YouTube)

Facebook rechaza el `yt-dlp` sin mÃ¡s con `Cannot parse data` â€” no es una versiÃ³n
antigua ni falta `--impersonate` (ambos se probaron el 2026-08-24 y fallaron
igual). Es un problema de **sesiÃ³n**: con las cookies de Firefox, el mismo
enlace muestra los formatos y se descarga en 7 segundos.

El script encuentra el perfil automÃ¡ticamente (el primero que tenga
`cookies.sqlite`, buscando primero en `snap`) y solo aÃ±ade las opciones en esos
dominios. Para forzar otro perfil:

    export ANALISEVIDEO_FIREFOX_PROFILE=~/snap/firefox/common/.mozilla/firefox/xxxx.default

**Sin Firefox en la mÃ¡quina** (VPS), la descarga continÃºa sin cookies y muestra
un aviso en stderr â€” sigue funcionando para los sitios pÃºblicos.


## Orden de los motores (2026-08-26: Gemini directo va primero)

    GOOGLE_API_KEY â†’ GEMINI_API_KEY â†’ â€¦_TIME â†’ â€¦_PROMPTS  â†’  z-ai/glm-5.3-flash

El 2026-08-25 la alternativa habÃ­a pasado al frente porque `stealth/ox-alpha`
era **gratis** y las tres claves de Gemini agotaban la cuota diaria con ~9
anÃ¡lisis. Ese modelo **desapareciÃ³ de OpenRouter** â€” era el riesgo seÃ±alado
desde el principio. La alternativa actual es de pago, asÃ­ que vuelve al final:
no tiene sentido gastar crÃ©dito cuando todavÃ­a queda cuota de Gemini. Entra
cuando se agota la cuota del dÃ­a.

Para invertir el orden (poner la alternativa primero), **sin tocar el cÃ³digo**:

    export ANALISEVIDEO_MOTOR=reserva

## Alternativa: cuando el motor principal rechaza la solicitud

El paso costoso de `analisa` no es la llamada a la API â€” es la **descarga y la
compresiÃ³n** que ocurren antes. Un clip de 41 MB de Facebook tarda minutos en
llegar hasta ese punto. El 2026-08-24, tres claves distintas de Gemini
respondieron 429 en el mismo minuto y todo ese trabajo se perdiÃ³.

Por eso existe la alternativa. El orden es:

    GOOGLE_API_KEY â†’ GEMINI_API_KEY â†’ â€¦_TIME â†’ â€¦_PROMPTS â†’ z-ai/glm-5.3-flash

`z-ai/glm-5.3-flash` (OpenRouter, `OPENROUTER_API_KEY` en `wifi/.env`) acepta
videos y devuelve el MISMO JSON â€” el prompt no cambia en absoluto. Se incorporÃ³
el 2026-09-08 en lugar de `google/gemini-3.7-flash`: cuesta ~10x menos (US$ 0,075/M
de entrada frente a 0,75) y, al ser otro proveedor, no comparte la cuota ni la
congestiÃ³n de Gemini, que es el motor principal.

**El uso de video en OpenRouter requiere US$ 1,00 disponibles EN LA CLAVE** â€”
no en la cuenta. Una clave con un lÃ­mite (`limit` en openrouter.ai/settings/keys)
inferior a ese monto responde `402 â€¦ requires at least $1.00 in balance for video`
para cualquier video, incluso un clip de 20 s, mientras que el texto funciona.
Por eso la alternativa quedÃ³ inactiva desde 2026-08 hasta 2026-09-08. `analisa`
ahora imprime el cuerpo del error.

EstÃ¡ AL FINAL deliberadamente: **es de pago**. Cada vez que responde, `analisa`
lo indica en stderr (`ATENCAO: ... reserva paga`) â€” un motor de pago no se
ejecuta en silencio. Para cambiar el modelo alternativo, `OPENROUTER_VIDEO_MODEL`.

`_modelo` en el JSON registra QUIÃ‰N hizo el anÃ¡lisis: sin eso, la base de datos
dirÃ­a que fue Gemini un anÃ¡lisis que Gemini no hizo.

Sin `OPENROUTER_API_KEY`, todo funciona como antes â€” la alternativa es opcional.


## Volver a procesar sin descargar otra vez

El slug se obtiene del TÃTULO, y Facebook devuelve Â«FacebookÂ» para todos los
enlaces â€” asÃ­ que cada intento creaba `facebook-2`, `-3`, `-4`â€¦ y volvÃ­a a
descargar todo. El 2026-08-24 se descargÃ³ **cuatro veces** el MISMO clip (141 MB
de red y disco) debido a un error de cuota que no tenÃ­a nada que ver con la
descarga.

Ahora la **URL es la identidad**: si ya existe una carpeta con esa URL y el
archivo sigue en el disco, se reutiliza â€” se omiten la descarga y la compresiÃ³n.
TambiÃ©n se reutiliza la versiÃ³n comprimida (`analise-src.mp4`), si existe.

Dos consecuencias prÃ¡cticas:

- **Repetir `analisa <url>` despuÃ©s de un error es barato** â€” va directamente al
  anÃ¡lisis. Es la forma de volver a procesar lo que fallÃ³ por la cuota.
- **`--keep-src` ahora resulta Ãºtil** para videos que quizÃ¡ quieras volver a
  analizar (por ejemplo, con otro `--prompt`). Sin esa opciÃ³n, el archivo se
  borra al final y la prÃ³xima vez se vuelve a descargar.

Pasar un `slug` explÃ­cito desactiva la reutilizaciÃ³n: ahÃ­ estÃ¡s indicando quÃ©
carpeta quieres, y adivinar otra serÃ­a peor.
