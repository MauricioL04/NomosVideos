---
proyecto: Videos-Diario-NomosTin (canal «¿Qué pasaría si…?»)
fecha: 2026-10-28
autor: investigador (Claude)
estado: investigación para que decida Mauricio. Reemplaza los supuestos de pago de la spec y de los otros dos informes.
---

# Stack de costo casi cero · Claude Max + plan web de Higgsfield

**Restricción nueva:** Mauricio ya paga Claude Max y un plan web de Higgsfield. El canal es personal (Remotion es gratis para individuos, [remotion.dev/license](https://www.remotion.dev/license)). El gasto extra tiene un tope de **USD 20 al mes y es solo para emergencias**.

Convención: **[V]** verificado hoy en una fuente oficial; **[S]** fuente secundaria; **[I]** inferencia mía, que se confirma en el piloto.

---

## 1. Claude Max en lugar de la Claude API

| Pregunta | Respuesta | Fuente |
|---|---|---|
| ¿`claude -p` o la Action pueden usar Max? | **Sí.** `claude setup-token` genera un token OAuth que dura **un año** y es «para pipelines de CI y scripts donde no hay login por navegador». Requiere Pro, Max, Team o Enterprise. **Solo hace llamadas al modelo:** no trae los conectores de claude.ai, aunque los MCP locales sí funcionan. `--bare` no lee el token | [V] https://code.claude.com/docs/en/authentication |
| ¿Y `anthropics/claude-code-action`? | Acepta `claude_code_oauth_token` («Pro and Max users can generate this by running `claude setup-token`») y trae ejemplos de automatizaciones programadas | [V] https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md |
| ¿Lo permiten los términos? | **Sí, para uso propio.** Los Consumer Terms (8/10/2025) prohíben el acceso automatizado «salvo con API key **o donde lo permitamos explícitamente**», y la doc de Claude Code lo permite de forma explícita para CI. El límite está en otra parte: OAuth es para el «uso ordinario» del comprador, y quien construye un **producto o servicio para terceros** debe usar API key. Además, los límites anunciados de Pro y Max «asumen un uso ordinario e individual». Un canal personal, con el token de Mauricio y sin usuarios externos, entra en el uso permitido [I]. Anthropic se reserva el derecho de hacer cumplir estas reglas sin aviso | [V] https://www.anthropic.com/legal/consumer-terms · https://code.claude.com/docs/en/legal-and-compliance |
| Límites | Max 5x o 20x sobre Pro, con una ventana de 5 h más un tope semanal **compartido con el chat y con Claude Code** | [V] https://support.claude.com/en/articles/11014257-about-claude-s-max-plan-usage · [S] https://claudelog.com/claude-code-limits/ |
| Rutinas en la nube (`/schedule`) | Disponibles en Pro y Max; corren en la infraestructura de Anthropic con la laptop apagada; frecuencia mínima de **1 vez por hora**; **100 ejecuciones por hora** por cuenta; gastan la suscripción igual que una sesión interactiva; pueden usar **los conectores MCP de claude.ai**; red «Trusted» por defecto, editable, y secretos de red. Están en **research preview** | [V] https://code.claude.com/docs/en/routines |

**Cómo encaja:** 10 guiones diarios con una pasada de verificación pesan poco frente a una sesión normal de programación [I]. El riesgo real es que el canal compita con el trabajo diario de Mauricio por la misma cuota semanal.

## 2. Higgsfield con el plan web, sin la API de pago

| Pregunta | Respuesta | Fuente |
|---|---|---|
| ¿Los créditos del plan sirven en la API `open.higgsfield.ai`? | **No.** La API es un «producto para desarrolladores independiente», con saldo propio en USD de pago por uso y claves propias | [V] https://docs.higgsfield.ai/docs/help/faq.md · https://higgsfield.ai/creator-hub/help-center/mcp-cli/how-do-i-access-higgsfield-via-cli |
| ¿Y en el MCP y el CLI? | **Sí.** El CLI (`@higgsfield/cli`, `higgsfield auth login`, sin API key) y el MCP (`mcp.higgsfield.ai/mcp`) usan «la misma cuenta y los mismos créditos del plan, con las mismas tarifas» | [V] mismo centro de ayuda · npm `@higgsfield/cli` 1.2.2 (16/10/2026) |
| ¿Los ilimitados sirven por MCP o CLI? | **No.** «Los modelos ilimitados y las generaciones gratis de los planes solo están disponibles a través de higgsfield.ai y no en MCP/CLI». Además, el ToS §10.6 permite **frenar o poner en cola lenta el uso automatizado** de los ilimitados | [V] https://higgsfield.ai/pricing · https://higgsfield.ai/terms-of-use-agreement |
| ¿Se puede automatizar? | **Sí.** El ToS (actualizado el 23/10/2026) da licencia de API, MCP y CLI para «fines personales o comerciales internos» (§1.2). El §11.12 trata expresamente a los «agentes y plataformas de automatización (incluso a través de MCP)» y hace responsable al usuario de lo que generen. Prohíbe abrir **varias cuentas para esquivar límites** (§1.5) | [V] https://higgsfield.ai/terms-of-use-agreement |
| ¿Corre sin interfaz? | El CLI inicia sesión por OAuth en el navegador; no hay un token documentado para CI. **El camino limpio es el conector MCP dentro de una rutina de Claude Code**, porque los conectores llegan por los servidores de Anthropic. En GitHub Actions no sirve: el token de `setup-token` no trae conectores | [V] https://github.com/higgsfield-ai/skills/blob/main/INSTALL_FOR_AGENTS.md · routines · authentication; la rutina con el MCP de Higgsfield es [I] y se prueba en el piloto |

**Planes** [V] https://higgsfield.ai/pricing: Starter, USD 15 al mes, **200 créditos**. Plus, USD 39 al mes con pago anual o 49 mensual, **1000 créditos**. Ultra, USD 99 con pago anual o 129 mensual, **3000 créditos** (también hay opciones de 6000 y 9000). Free, sin MCP.

**Créditos por pieza** (tabla de la misma página):
- Clips de 5 s: Kling 3.0 en 720p, unos 7; Kling 2.5 Turbo en 720p, unos 4; Wan 3.0, 5 en 480p y 8,75 en 720p; Seedance 2.0 Fast en 720p, unos 12; Seedance 2.0 en 720p, unos 22; Seedance 1.5 en 720p, unos 3.
- Imágenes: **Soul 2.0, 0,12** créditos por imagen; Nano Banana 2, 2.

**Videos por plan, en formato híbrido** (un 30 % más por regeneraciones; voz y música fuera de Higgsfield):

| Mezcla por video | Créditos | Starter (al día) | Plus (al día) | Ultra (al día) |
|---|---|---|---|---|
| Solo imágenes: 8 Soul 2.0 animadas en Remotion | 1,25 | 5,3 | 26 | 80 |
| 1 clip de 5 s con Kling 2.5 Turbo + 6 Soul | 6,1 | 1,1 | **5,4** | 16 |
| 1 clip de 5 s con Wan 3.0 480p + 6 Soul | 7,4 | 0,9 | 4,5 | 13 |
| **1 clip de 5 s con Kling 3.0 720p + 6 Soul** | 10,0 | 0,7 | **3,3** | **10** |
| 2 clips con Kling 3.0 + 5 Soul | 19,0 | 0,4 | 1,8 | 5,3 |

Que Soul 2.0 genere en 9:16 por MCP está [I], igual que el costo en créditos por la vía MCP. Se mide con `higgsfield account status` antes y después de un lote de prueba.

## 3. Voz gratis en español, con tiempos por palabra

| Opción | Gratis | ¿Uso comercial? | Tiempos por palabra | Fuente |
|---|---|---|---|---|
| **Azure Speech F0** | **500 000 caracteres neurales al mes**. Al pasarse **no cobra**: da error 429 hasta el mes siguiente | Sí [I: los términos de Azure no lo restringen en F0] | **Sí, con el evento `WordBoundary`** (offset por palabra) | [V] https://azure.microsoft.com/en-us/pricing/details/speech/ · https://learn.microsoft.com/azure/ai-services/speech-service/how-to-speech-synthesis · https://learn.microsoft.com/answers/a/12256221 |
| Google Cloud TTS | Chirp 3 HD y Neural2: 1 M de caracteres al mes; WaveNet y Standard: 4 M | Sí | No por palabra en Chirp 3 HD [I] → Whisper | [V] https://cloud.google.com/text-to-speech/pricing |
| ElevenLabs Free | 10 000 créditos al mes (unos 15 videos) | **No**: la licencia comercial empieza en Starter (USD 6; 1 el primer mes hasta el 1/11) | Sí (`with-timestamps`) | [V] https://elevenlabs.io/pricing |
| Edge TTS (`edge-tts`) | Sin límite publicado | No es una API oficial: usa el servicio de lectura de Edge | Sí (`--write-subtitles`) | [V] https://github.com/rany2/edge-tts |
| Kokoro-82M local | Gratis, licencia Apache 2.0, corre en CPU | Sí | No → Whisper | [V] https://huggingface.co/hexgrad/Kokoro-82M (3 voces en español) |
| Voz de Higgsfield | Sí existe (`higgsfield voices list`) y gasta créditos | Sí (ToS §4.4) | No → faster-whisper | [V] https://github.com/higgsfield-ai/skills (faceless-video) |

**Cuenta:** 10 videos al día × 650 caracteres × 1,2 ≈ **234 000 caracteres al mes**, menos de la mitad del límite gratis de Azure. Respaldo: Kokoro con `faster-whisper` en el runner.

## 4. Publicar en 3 redes (con 3 o 4 cuentas de TikTok)

| Vía | Costo | Qué permite | Fuente |
|---|---|---|---|
| Upload-Post Free | 0 | 10 subidas al mes, 2 perfiles, **TikTok no** (devuelve 403) | [V] https://docs.upload-post.com/resources/pricing-and-limits |
| **Upload-Post Basic** | **USD 24 mensual, o 16 al mes con pago anual (192 al año)** | Subidas ilimitadas y 5 perfiles, cada uno con una cuenta por red: **caben las 4 de TikTok**. Topes por cuenta: TikTok 15, YouTube 30 e Instagram 50 al día | [V] mismo |
| YouTube Data API | 0 | `videos.insert`: 100 llamadas al día por proyecto. La doc actual dice que los proyectos sin verificar **no** quedan forzados a privado. Si la app de OAuth queda en «Testing», el token vence a los 7 días: hay que pasarla a «In production» sin verificar | [V] https://developers.google.com/youtube/v3/docs/videos/insert · https://developers.google.com/identity/protocols/oauth2#expiration |
| Instagram API | 0 | Con «Standard Access» alcanza para **cuentas propias**, sin App Review. El token largo dura unos 60 días y se renueva | [S] https://www.blotato.com/blog/instagram-posting-api · [V] https://developers.facebook.com/docs/instagram-platform/overview |
| TikTok, app propia | 0 | Sin auditoría, **todo queda privado**. La auditoría rechaza de forma explícita «una herramienta para subir contenido a las cuentas que tú o tu equipo manejan». **No hay vía pública propia** | [V] https://developers.tiktok.com/doc/content-posting-api-reference-direct-post · https://developers.tiktok.com/doc/content-sharing-guidelines |
| TikTok, borrador en el buzón (`video.upload`) | 0 | El video llega como borrador y Mauricio lo publica desde la app. Exige que aprueben el scope; no menciona auditoría. No está claro si lo aprueban para uso propio [I] | [V] https://developers.tiktok.com/doc/content-posting-api-get-started-upload-content |
| TikTok Studio (manual) | 0 | Programar a mano desde la web, unos 1–2 min por video y cuenta [I] | — |

## 5. Música y almacenamiento

- **Música:** generar una vez una biblioteca de 10 a 20 pistas instrumentales con Higgsfield (`sonilo_music`, con créditos). El usuario es dueño de lo generado y puede usarlo comercialmente ([V] ToS §4.4 y https://higgsfield.ai/creator-hub/help-center/account/who-owns-my-generations-and-can-i-use-them-commercially).
  - Pixabay permite uso comercial, pero tiene reclamos de Content ID de terceros ([S] https://hellothematic.com/thematic-vs-pixabay/).
  - La licencia de la Audio Library de YouTube fuera de YouTube depende de cada pista ([S] https://lickd.co/youtube-audio-library/).
  - En los dos casos, se guarda la fuente en `licencias.csv`.
- **Cloudflare R2:** 10 GB-mes, 1 M de operaciones de clase A y 10 M de clase B gratis, y sin costo de salida ([V] https://developers.cloudflare.com/r2/pricing/). Con unos 25 MB por MP4 final, 300 videos ocupan unos 7,5 GB: hay que borrar los intermedios a los 7 días y los finales a los 60 días, después de publicar [I].
- **GitHub Actions:** gratis en repos **públicos**. En un repo privado con GitHub Free hay USD 12 de cómputo (unas **2000 min de Linux**) y **500 MB de artefactos** ([V] https://docs.github.com/en/billing/concepts/product-billing/github-actions). El job de 150 min diarios de la spec (unas 4500 min al mes) **no entra**. Hay que renderizar menos videos al día, medir los minutos reales o hacer público el repo, sin secretos en el código. Los artefactos no sirven de almacén: van a R2.

---

## Tabla final

| Componente | Opción elegida | USD/mes | Riesgo |
|---|---|---|---|
| Guion y verificación | **Rutina de Claude Code** (Max) en la nube, 1 al día; escribe el JSON del lote en el repo | 0 (incluido en Max) | Medio: research preview; comparte la cuota semanal con el trabajo de Mauricio |
| Imágenes y clips | **Conector MCP de Higgsfield dentro de la misma rutina**, con los créditos del plan | 0 (incluido en el plan) | Medio: falta probar la rutina con el MCP; el plan define el techo de videos |
| Voz | **Azure Speech F0**, voz neural es-MX/es-PE, con `WordBoundary` | 0 | Bajo: tope duro sin cobro; respaldo con Kokoro + faster-whisper |
| Subtítulos | Tiempos de Azure; faster-whisper solo como respaldo | 0 | Bajo |
| Montaje | Remotion en GitHub Actions | 0 | Medio: los minutos de un repo privado (2000) no alcanzan para 10 videos al día |
| Música | Biblioteca generada una sola vez con Higgsfield | 0 (unos pocos créditos) | Bajo |
| Almacenamiento | R2 gratis, con borrado automático | 0 | Bajo: vigilar los 10 GB |
| Publicación YouTube e Instagram | APIs oficiales | 0 | Bajo a medio: tokens que vencen (IG, unos 60 días) |
| Publicación TikTok (3–4 cuentas) | **Mes 1: TikTok Studio a mano** (va con la revisión humana); emergencia: **Upload-Post Basic** | 0 → 24 (o 16 con pago anual) | Alto si se automatiza: la API propia queda en privado para siempre |
| **Total** | | **USD 0 de base; 16–24 si se activa Upload-Post** | |

**Videos al día que permite el plan de Higgsfield** (1 clip Kling 3.0 + 6 imágenes; entre paréntesis, con Kling 2.5 Turbo):
- Starter: **0,7** (1,1). Con este plan conviene la mezcla de solo imágenes: 5 al día.
- Plus: **3,3** (5,4).
- Ultra: **10** (16).

## Recomendación

1. **Base de USD 0:**
   - Una rutina de Claude Code hace el guion y la verificación, y genera las imágenes y el clip por el MCP de Higgsfield con los créditos del plan. Deja el JSON y las URLs en el repo.
   - Un workflow de GitHub Actions baja los medios, hace la voz con Azure F0 y renderiza con Remotion. YouTube e Instagram se publican por sus APIs oficiales.
2. **TikTok, el mes 1, a mano** desde TikTok Studio. Coincide con la revisión humana que ya pide la spec. Los USD 20 de emergencia se reservan para **Upload-Post Basic** (24 mensual). El pago anual (16 al mes, USD 192 por adelantado) supera el tope mensual, así que **lo decide Mauricio**.
3. **Cadencia según el plan que ya tiene.** Con Plus, 3 videos al día con clip (o 5 con Kling 2.5 Turbo). Con Ultra, 10. No abrir una segunda cuenta de Higgsfield: el ToS §1.5 lo prohíbe.
4. **Piloto (S), antes de tocar la spec:**
   - Una rutina con el conector de Higgsfield genera 1 video. Se miden los créditos reales (con `account status`) y se confirma que sale en 9:16.
   - Azure F0 con `WordBoundary` en es-PE.
   - Una subida a YouTube y otra a Instagram por API.
   - Contar los minutos de Actions de un render.

**Para la spec:** donde dice Claude API, Higgsfield API, ElevenLabs y Upload-Post, poner rutina con Max, MCP con el plan, Azure F0 y APIs oficiales más TikTok manual. El tope de gasto (RF-13) pasa de USD a **créditos por día**.
