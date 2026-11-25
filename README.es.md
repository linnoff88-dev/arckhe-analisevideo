# analisevideo

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

AnêÂ¡lisis **visual y cARCKHêtogrêÂ¡fico** de video con Gemini, archivado en una base
de datos local consultable. No es una transcripciêÂ³n del habla: lo que sale de
aquêÂ­ es lo que un director de fotografêÂ­a, un editor y un productor musical ven
en el material.

Funciona como skill de [OpenPCBot](https://github.com/linnoff88-dev/openpcbotv2) con
el comando `/analisevideo` ê¢â¬â deliberadamente **no se activa por vibra** (un
enlace suelto o êÂ«analiza este videoêÂ» siguen yendo a ARCKHêVOX).

## ê°Å¸ââ GuêÂ­a de uso

GuêÂ­a completa (landing + paso a paso): **https://linnoff88-dev.github.io/

## Uso

```bash
S=~/projetos/analisevideo/analisevideo.sh

bash $S analisa <url|arquivo> [slug] [--keep-src] [--prompt "..."]
bash $S ver     <slug>       # informe markdown
bash $S json    <slug>       # anêÂ¡lisis sin procesar
bash $S prompts <slug>       # 1 prompt por escena (5ê¢â¬â10), listo para Kling/Veo/Seedance
bash $S list    [N]          # êÂºltimos anêÂ¡lisis
bash $S search  "<termo>"    # bêÂºsqueda en la base de datos
bash $S stats                # lo que mêÂ¡s aparece en la base de datos
bash $S reindex              # reconstruye el êÂ­ndice a partir de las carpetas
```

Necesita `yt-dlp`, `ffmpeg`/`ffprobe`, `jq`, `python3` y una `GOOGLE_API_KEY`
(leêÂ­da de `~/projetos/openpcbotv2/.env` o `~/projetos/wifi/.env`).

## QuêÂ© cubre el anêÂ¡lisis

- **Prompts por escena** ê¢â¬â de 5 a 10 prompts en inglêÂ©s (+ negative prompt), uno
  por escena con cêÂ³digo de tiempo, que cubren todo el video.
- **FotografêÂ­a** ê¢â¬â paleta en hex, esquema de iluminaciêÂ³n, temperatura, contraste,
  look, lente aparente, profundidad de campo, textura/grain.
- **CêÂ¡mara bloque a bloque** con cêÂ³digo de tiempo ê¢â¬â plano, êÂ¡ngulo, movimiento
  (pan, dolly, gimbal, whip-pan, drone, orbital...), velocidad y nota.
- **ComposiciêÂ³n** ê¢â¬â encuadre, reglas, capas de profundidad.
- **Montaje** ê¢â¬â cortes estimados, cortes por minuto, ritmo, transiciones, match
  cut, jump cut, corte al beat, slow-mo/speed ramp.
- **Banda sonora y sonido** ê¢â¬â gêÂ©nero, BPM aproximado, instrumentaciêÂ³n, mood, SFX,
  mezcla y sincronêÂ­a con la imagen.
- **Texto y grêÂ¡ficos** ê¢â¬â tipografêÂ­a, animaciêÂ³n, subtêÂ­tulos, motion graphics.
- **PosproducciêÂ³n** ê¢â¬â efectos, VFX, LUT sugerida, sound design.
- **Narrativa** ê¢â¬â gancho de los primeros 3 segundos, estructura, arco, CTA.
- **CêÂ³mo recrearlo** ê¢â¬â equipo mêÂ­nimo, pasos prêÂ¡cticos y un prompt listo para
  Kling/Veo/Seedance que reproduce el look y el movimiento del fragmento mêÂ¡s
  potente.

## Base de datos

Cada anêÂ¡lisis se convierte en una carpeta en `~/projetos/output/analisevideo/<slug>/` con
`meta.json`, `analise.json` y `analise.md`. Se aêÂ±ade una lêÂ­nea por video a
`index.jsonl` ê¢â¬â es lo que leen `list`, `search` y `stats`. El video descargado se
borra al final (guêÂ¡rdalo con `--keep-src`).

## Detalles

- Descarga hasta 480p deliberadamente: el color, la luz y el movimiento se leen
  bien y la carga es rêÂ¡pida. Por encima de 18 MB, el archivo se vuelve a
  codificar a 360p/12fps solo para el anêÂ¡lisis.
- Si el enlace no se puede descargar con yt-dlp (sitio con sesiêÂ³n iniciada),
  descêÂ¡rgalo por otro medio y pasa la ruta.
- Los videos largos (>15 min) son costosos y vagos ê¢â¬â es mejor analizar un
  fragmento.
- Los errores 429/500/503 de Gemini se reprocesan automêÂ¡ticamente (4 intentos).

## Claves de Gemini ê¢â¬â tres, y cambia automêÂ¡ticamente

`analisa.py` intenta las claves en este orden y se detiene en la primera que
funciona:

| variable | dêÂ³nde |
|---|---|
| `GOOGLE_API_KEY` | la de siempre |
| `GEMINI_API_KEY` | idem (se deduplica si tiene el mismo valor) |
| `GEMINI_API_KEY_ARCKHêCCBOT_TIME` | `projects/1056030032122` |
| `GEMINI_API_KEY_ARCKHêCCBOT_PROMPTS` | `projects/1000152753819` |

Todas se obtienen de `~/projetos/wifi/.env` (o del `.env` local).

**QuêÂ© hace que cambie y quêÂ© hace que espere** ê¢â¬â esa es la distinciêÂ³n clave:

- **429 (cuota) y 403 (bloqueada)** ê¢â â pasa a la siguiente clave **de inmediato**.
  EstêÂ¡ en otro proyecto, con cuota propia; esperar no resolverêÂ­a nada.
- **500 / 502 / 503** ê¢â â Gemini estêÂ¡ congestionado. La clave no tiene la culpa y
  cambiarla no ayuda: espera (20s, 40s, 60s, 90s, 120s) y vuelve a intentarlo
  con la misma.

La carga del archivo grande tambiêÂ©n entra en este proceso: si devuelve un 429,
cambia de clave en vez de convertirse en un error definitivo ê¢â¬â justo el caso en
que el video ya se descargêÂ³ y comprimiêÂ³, y perder todo ese trabajo serêÂ­a costoso.

Cuando usa una clave que no es la primera, indica cuêÂ¡l en stderr. Si todas
fallan por cuota, el error enumera **cuêÂ¡les** fallaron, en vez de limitarse a
decir êÂ«hubo un errorêÂ».


## Sitios que exigen sesiêÂ³n (Facebook, Instagram, TikTok, YouTube)

Facebook rechaza el `yt-dlp` sin mêÂ¡s con `Cannot parse data` ê¢â¬â no es una versiêÂ³n
antigua ni falta `--impersonate` (ambos se probaron el 2026-08-24 y fallaron
igual). Es un problema de **sesiêÂ³n**: con las cookies de Firefox, el mismo
enlace muestra los formatos y se descarga en 7 segundos.

El script encuentra el perfil automêÂ¡ticamente (el primero que tenga
`cookies.sqlite`, buscando primero en `snap`) y solo aêÂ±ade las opciones en esos
dominios. Para forzar otro perfil:

    export ANALISEVIDEO_FIREFOX_PROFILE=~/snap/firefox/common/.mozilla/firefox/xxxx.default

**Sin Firefox en la mêÂ¡quina** (VPS), la descarga continêÂºa sin cookies y muestra
un aviso en stderr ê¢â¬â sigue funcionando para los sitios pêÂºblicos.


## Orden de los motores (2026-08-26: Gemini directo va primero)

    GOOGLE_API_KEY ê¢â â GEMINI_API_KEY ê¢â â ê¢â¬Â¦_TIME ê¢â â ê¢â¬Â¦_PROMPTS  ê¢â â  z-ai/glm-5.3-flash

El 2026-08-25 la alternativa habêÂ­a pasado al frente porque `stealth/ox-alpha`
era **gratis** y las tres claves de Gemini agotaban la cuota diaria con ~9
anêÂ¡lisis. Ese modelo **desapareciêÂ³ de OpenRouter** ê¢â¬â era el riesgo seêÂ±alado
desde el principio. La alternativa actual es de pago, asêÂ­ que vuelve al final:
no tiene sentido gastar crêÂ©dito cuando todavêÂ­a queda cuota de Gemini. Entra
cuando se agota la cuota del dêÂ­a.

Para invertir el orden (poner la alternativa primero), **sin tocar el cêÂ³digo**:

    export ANALISEVIDEO_MOTOR=reserva

## Alternativa: cuando el motor principal rechaza la solicitud

El paso costoso de `analisa` no es la llamada a la API ê¢â¬â es la **descarga y la
compresiêÂ³n** que ocurren antes. Un clip de 41 MB de Facebook tarda minutos en
llegar hasta ese punto. El 2026-08-24, tres claves distintas de Gemini
respondieron 429 en el mismo minuto y todo ese trabajo se perdiêÂ³.

Por eso existe la alternativa. El orden es:

    GOOGLE_API_KEY ê¢â â GEMINI_API_KEY ê¢â â ê¢â¬Â¦_TIME ê¢â â ê¢â¬Â¦_PROMPTS ê¢â â z-ai/glm-5.3-flash

`z-ai/glm-5.3-flash` (OpenRouter, `OPENROUTER_API_KEY` en `wifi/.env`) acepta
videos y devuelve el MISMO JSON ê¢â¬â el prompt no cambia en absoluto. Se incorporêÂ³
el 2026-09-08 en lugar de `google/gemini-3.7-flash`: cuesta ~10x menos (US$ 0,075/M
de entrada frente a 0,75) y, al ser otro proveedor, no comparte la cuota ni la
congestiêÂ³n de Gemini, que es el motor principal.

**El uso de video en OpenRouter requiere US$ 1,00 disponibles EN LA CLAVE** ê¢â¬â
no en la cuenta. Una clave con un lêÂ­mite (`limit` en openrouter.ai/settings/keys)
inferior a ese monto responde `402 ê¢â¬Â¦ requires at least $1.00 in balance for video`
para cualquier video, incluso un clip de 20 s, mientras que el texto funciona.
Por eso la alternativa quedêÂ³ inactiva desde 2026-08 hasta 2026-09-08. `analisa`
ahora imprime el cuerpo del error.

EstêÂ¡ AL FINAL deliberadamente: **es de pago**. Cada vez que responde, `analisa`
lo indica en stderr (`ATENCAO: ... reserva paga`) ê¢â¬â un motor de pago no se
ejecuta en silencio. Para cambiar el modelo alternativo, `OPENROUTER_VIDEO_MODEL`.

`_modelo` en el JSON registra QUIêâ°N hizo el anêÂ¡lisis: sin eso, la base de datos
dirêÂ­a que fue Gemini un anêÂ¡lisis que Gemini no hizo.

Sin `OPENROUTER_API_KEY`, todo funciona como antes ê¢â¬â la alternativa es opcional.


## Volver a procesar sin descargar otra vez

El slug se obtiene del TêÂTULO, y Facebook devuelve êÂ«FacebookêÂ» para todos los
enlaces ê¢â¬â asêÂ­ que cada intento creaba `facebook-2`, `-3`, `-4`ê¢â¬Â¦ y volvêÂ­a a
descargar todo. El 2026-08-24 se descargêÂ³ **cuatro veces** el MISMO clip (141 MB
de red y disco) debido a un error de cuota que no tenêÂ­a nada que ver con la
descarga.

Ahora la **URL es la identidad**: si ya existe una carpeta con esa URL y el
archivo sigue en el disco, se reutiliza ê¢â¬â se omiten la descarga y la compresiêÂ³n.
TambiêÂ©n se reutiliza la versiêÂ³n comprimida (`analise-src.mp4`), si existe.

Dos consecuencias prêÂ¡cticas:

- **Repetir `analisa <url>` despuêÂ©s de un error es barato** ê¢â¬â va directamente al
  anêÂ¡lisis. Es la forma de volver a procesar lo que fallêÂ³ por la cuota.
- **`--keep-src` ahora resulta êÂºtil** para videos que quizêÂ¡ quieras volver a
  analizar (por ejemplo, con otro `--prompt`). Sin esa opciêÂ³n, el archivo se
  borra al final y la prêÂ³xima vez se vuelve a descargar.

Pasar un `slug` explêÂ­cito desactiva la reutilizaciêÂ³n: ahêÂ­ estêÂ¡s indicando quêÂ©
carpeta quieres, y adivinar otra serêÂ­a peor.
