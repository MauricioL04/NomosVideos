# Pipeline técnico · canal «¿Qué pasaría si…?» (10 videos/día × 3 redes)

Fecha: 2026-10-28 · Autor: investigador (Claude) · Estado: investigación, sin código.

Convención: **[V]** = verificado en fuente primaria (doc oficial o página del proveedor); **[S]** = fuente secundaria (blog o comparador); **[I]** = inferencia mía, a confirmar en el piloto.

---

## 0. Recomendación en una línea

**Python en GitHub Actions (cron diario) → Claude API (Sonnet 5.5) escribe guion y prompts → Higgsfield API (imagen 9:16 con Soul + imagen→video con Kling 3.0 o Wan 3.0) → ElevenLabs TTS con timestamps → subtítulos ASS + FFmpeg → Upload-Post (API) publica en YouTube, Instagram y TikTok con la etiqueta de IA puesta.** Costo estimado: **~250 a 650 USD/mes** según el modelo de video. Empezar con 3 a 5 videos/día y subir a 10 cuando la retención lo justifique (ver riesgos en §5).

---

## 1. Higgsfield

### 1.1 ¿Hay API pública? Sí.

| Tema | Dato | Fuente |
|---|---|---|
| Portal | `cloud.higgsfield.ai` redirige (301) a **open.higgsfield.ai**; consola en console.higgsfield.ai | [V] https://open.higgsfield.ai/ · https://docs.higgsfield.ai/ |
| Base URL | `https://api.higgsfield.ai` | [V] https://docs.higgsfield.ai/ |
| Autenticación | Cabecera `Authorization: Key <API_KEY_ID>:<API_KEY_SECRET>` | [V] https://docs.higgsfield.ai/ |
| Patrón | `POST /<modelo>` asíncrono → `request_id`, `status_url`, `cancel_url`; estados queued, in_progress, completed, failed, nsfw, canceled | [V] https://docs.higgsfield.ai/docs/openapi.json |
| Endpoints de ejemplo (OpenAPI) | `/kling-video/v2.5-turbo/pro/text-to-video`, `/kling-video/v2.5-turbo/pro/image-to-video`, `/minimax/hailuo-2.3/standard/text-to-video`, `/higgsfield-ai/soul/standard` (imagen) | [V] https://docs.higgsfield.ai/docs/openapi.json (la doc aclara que el OpenAPI es suplementario; el catálogo vigente está en la consola) |
| SDK | Python `pip install higgsfield-client`; TypeScript `npm install @higgsfield/client` (solo servidor) | [V] https://docs.higgsfield.ai/docs/how-to/sdk.md |
| Webhooks | Sí, con firma; recomiendan polling como respaldo | [V] https://docs.higgsfield.ai/docs/how-to/webhooks.md |
| Límites de tasa | No hay cifras públicas: la concurrencia y la cola se ven en la consola; cola llena → 429 + Retry-After, sin cobro; reintentar con el mismo Idempotency-Key | [V] https://docs.higgsfield.ai/docs/concepts/rate-limits.md |
| Cobro | Saldo en USD (no créditos); solo se cobra lo exitoso; failed/nsfw no se cobran; el saldo vence al año; hay endpoint de **estimación** | [V] https://docs.higgsfield.ai/docs/concepts/billing-and-retention.md |
| Retención | Las salidas se borran **a los 7 días**: hay que descargarlas | [V] mismo enlace |
| Tiempo de generación | **No publicado** | medir en el piloto [I] |

### 1.2 Modelos y precios en la API (open.higgsfield.ai, 28/10/2026)

Precios por segundo «desde», muchos con descuento promocional (entre paréntesis, el precio sin descuento). [V] https://open.higgsfield.ai/

| Modelo | USD/s | Notas |
|---|---|---|
| Wan 3.0 (Alibaba) | 0.020 (0.050) | el más barato |
| Wan 3.0 Prime | 0.027 (0.068) | |
| Kling 3.0 std | 0.042 (0.084) | audio nativo; clips de 3 a 15 s ([V] página del modelo) |
| Wan 2.7 | 0.050 (0.100) | |
| MiniMax H3 | 0.065 (0.130) | |
| Grok Imagine Video 1.5 | 0.080 | |
| LTX 2.5 Fast / Pro | 0.090 / 0.120 | |
| Seedance 2.0 | 0.099 (0.141) | 4 a 15 s, 480p a 4K; se cobra por tokens (seg × ancho × alto × 24 / 1024; 0.014 USD por 1000 tokens). **A 720p vertical sale ~0.30 USD/s de lista** (mi cálculo [I]) |
| Kling 4.0 | 0.110 (0.157) | |
| Seedance 2.5 | 0.144 (0.206) | |
| Cinema Studio 4.0 (Higgsfield) | 0.206 | |
| Imagen: Soul 2 / Z-Image Turbo / Soul Standard | 0.003 / 0.015 / 0.094 por imagen | Soul acepta aspect_ratio 9:16 ([V] OpenAPI) |

- **Veo y Sora no aparecen en el catálogo de la API** (sí en la app de consumo y en el MCP, según blogs [S] https://creatify.ai/blog/what-is-higgsfield-mcp-what-it-does-what-it-costs-and-how-to-use-it). Si se quiere Veo, se usa directo de Google (§1.5).
- **9:16:** en el OpenAPI, el endpoint de texto→video de Kling 2.5 no expone aspect_ratio [V]. Lo seguro es **imagen 9:16 (Soul) → imagen→video**, que hereda la proporción [I]. Confirmar en la consola, modelo por modelo.
- Planes de la app de consumo (no API): Starter 19 USD (270 créditos), Plus 59 USD, Ultra 129 USD; los créditos no se acumulan [S] https://www.krea.ai/blog/higgsfield-pricing-explained-2026-unlimited-credits-and-real-monthly-costs. **Para automatizar se usa la API con saldo en USD, no el plan.**

### 1.3 MCP, n8n, Make y Zapier

- **MCP oficial**: https://mcp.higgsfield.ai/mcp, con OAuth; gasta los créditos del plan de consumo (desde el plan de 19 USD); lanzado en abril de 2026 [S] https://jonathansblog.co.uk/higgsfield-claude-mcp · https://creatify.ai/blog/what-is-higgsfield-mcp-what-it-does-what-it-costs-and-how-to-use-it. Sirve para trabajo interactivo en Claude, **no para un pipeline desatendido** [I].
- **n8n, Make y Zapier**: no encontré nodo oficial. Se integra con un nodo HTTP genérico contra la API [I]. Hay un conector de terceros (MindCloud) [S] https://mindcloud.co/docs/universal/rest/higgsfield-ai/latest.

### 1.4 Términos: uso comercial

- El usuario es dueño de las salidas y **puede monetizar en YouTube**; no hace falta licencia aparte (ToS §4.4) [V] https://higgsfield.ai/creator-hub/help-center/account/who-owns-my-generations-and-can-i-use-them-commercially
- Higgsfield puede entrenar con el contenido, salvo en Enterprise [V] mismo enlace. No leí el ToS completo: revisar §5 (restricciones) antes de lanzar.

### 1.5 Alternativas (USD por segundo de video)

| Proveedor | Modelo | USD/s | Fuente |
|---|---|---|---|
| Higgsfield | Wan 3.0 / Kling 3.0 std | 0.020 / 0.042 (promo) | [V] https://open.higgsfield.ai/ |
| Google Gemini API | Veo 3.1 Lite 720p / 1080p | 0.05 / 0.08 (con audio, preview, sin capa gratis) | [S] https://benchlm.ai/media-pricing/veo |
| Google Gemini API | Veo 3.1 Fast 720p / 1080p | 0.10 / 0.12 | [S] mismo |
| Google Gemini API | Veo 3.1 estándar | 0.40 | [S] mismo |
| Vertex AI | Veo 3.1 Lite, solo video | 0.03 (720p) / 0.05 (1080p) | [S] mismo |
| Runway API | Gen-4 Turbo (solo imagen→video) | 0.05 (5 créditos/s a 0.01) | [S] https://www.activepieces.com/blog/runway-api-pricing-per-second-for-every-video-model |
| Kling API oficial | Kling 3.0 1080p con audio | 0.168 | [S] https://creatify.ai/blog/higgsfield-vs-kling-ai-quality-cost-per-second-and-which-to-use-for-ads-in-2026 |
| fal.ai | Kling O3 std i2v / Wan 2.2 720p / Seedance 2.0 | 0.084 / 0.08 / 0.30 | [S] https://www.atlascloud.ai/blog/guides/cheapest-api-provider-seedance-kling-wan |

Conclusión: **la API de Higgsfield hoy es igual o más barata** que las alternativas para Kling y Wan, pero sus precios son **promocionales** y pueden cambiar. Plan B: Veo 3.1 Lite en la Gemini API, o fal.ai.

---

## 2. Resto del pipeline

### 2.1 Costo por video de 45 s (supuesto: ~650 caracteres de guion)

| Pieza | Opción mínima | USD/video | Fuente |
|---|---|---|---|
| Guion + prompts | Claude Sonnet 5.5 (2 USD in / 10 USD out por MTok) | ~0.02–0.05 | [V] https://platform.claude.com/docs/en/about-claude/pricing |
| Voz en off en español | ElevenLabs v4: 0.08 USD/1K caracteres; Flash/Turbo: 0.04 | 0.05 / 0.03 | [V] https://elevenlabs.io/pricing/api |
| Alternativa de voz | OpenAI gpt-4o-mini-tts ≈ 0.015 USD/min (estimado de OpenAI) | ~0.01 | [S] https://community.openai.com/t/understanding-gpt-4o-mini-tts-pricing-input-characters-cost/1151816 |
| Música | Biblioteca fija de ~30 pistas, generada una vez con ElevenLabs Music (0.15 USD/min, uso comercial desde Starter) o bajada de la Audio Library de YouTube | ~0 | [V] https://elevenlabs.io/pricing/api · [S] https://www.foximusic.com/blog/?p=546 |
| Video | ver §2.2 | 0.6 – 6 | §1 |

Pixabay permite uso comercial, pero sus pistas pueden recibir reclamos de Content ID de terceros [S] https://hellothematic.com/thematic-vs-pixabay/. Por eso conviene una biblioteca propia o generada.

### 2.2 Video: segundos generados y costo mensual (10 videos/día, 30 días)

| Estrategia | s generados/video | Modelo | USD/video | USD/mes |
|---|---|---|---|---|
| A. Todo generado (6 clips × 8 s) | 48 | Wan 3.0 (0.020) | 0.96 | ~290 |
| A | 48 | Kling 3.0 std (0.042) | 2.02 | ~605 |
| A | 48 | Veo 3.1 Lite 720p (0.05) | 2.40 | ~720 |
| A | 48 | Seedance 2.0 720p (~0.30 de lista) | ~14 | ~4300 |
| **B. Híbrido (recomendado)**: 3 clips × 5 s + 4 imágenes 9:16 con zoom lento (Ken Burns) | 15 + 4 imágenes | Kling 3.0 + Soul 2 | ~0.65 | **~200** |

Hay que sumar un margen de **×1.5** por tomas descartadas: lo que se rechaza por calidad sí se cobra [I]. Rango realista: **~300 a 900 USD/mes solo en video**.

### 2.3 Subtítulos tipo karaoke: lo mínimo que funciona

1. El endpoint de ElevenLabs `/v1/text-to-speech/{voice_id}/with-timestamps` devuelve el audio y la **alineación por carácter** [V] https://elevenlabs.io/blog/new-text-to-speech-endpoints-with-timestamps. Se agrupan los caracteres en palabras; no hace falta Whisper.
2. Se genera un archivo .ass con resaltado por palabra (etiqueta de karaoke o un evento por palabra) y se quema con el filtro `ass` de FFmpeg (libass) [V] https://ffmpeg.org/ffmpeg-filters.html#ass. La fuente tipográfica va en el repo.
3. Con el mismo FFmpeg se unen los clips (concat), se aplica el zoom lento a las imágenes, se mezclan voz y música con ducking (sidechaincompress) y se exporta en 1080×1920, H.264/AAC [I, práctica estándar].

| Opción | Costo | Veredicto |
|---|---|---|
| **FFmpeg + ASS** | 0 USD | **Recomendado** |
| Remotion | gratis para individuos y empresas pequeñas | Más vistoso, pero requiere Node y Chromium para renderizar [V] https://www.remotion.dev/license |
| Submagic API | packs de 250 min ≈ 57 USD/mes | Innecesario a esta escala [S] https://www.hooked.so/compare/submagic-pricing |
| Creatomate / Shotstack | no verificado | Solo si no se quiere tocar FFmpeg |
| FFmpeg en la nube de Upload-Post | incluido (Basic: 300 min/mes) | Útil como respaldo [V] https://www.upload-post.com/llms-full.txt |

**Ganchos:** se resuelven en el guion y el prompt (pregunta en pantalla en el segundo 0, primer plano de impacto, subtítulo grande con la palabra clave). Es un tema de guion, no de herramienta [I].

---

## 3. Publicación cruzada (30 publicaciones/día)

### 3.1 APIs oficiales

| Plataforma | Dato clave | Fuente |
|---|---|---|
| **YouTube Data API v3** | Desde el 1/6/2026, videos.insert tiene **cubeta de cuota propia: 100 llamadas/día** por proyecto. La doc actual dice «Videos uploaded from unverified API projects are not restricted to private viewing mode», lo que contradice la regla de 2020 (privado sin auditoría): hay que probarlo. El campo status.containsSyntheticMedia declara la IA. Si la app de OAuth queda en modo «Testing», el refresh token vence a los 7 días | [V] https://developers.google.com/youtube/v3/docs/videos/insert · https://developers.google.com/youtube/v3/revision_history · https://developers.google.com/youtube/v3/docs/videos · https://developers.google.com/identity/protocols/oauth2#expiration |
| **Instagram Graph API** | Reels con media_type=REELS y video_url público; requiere **cuenta profesional** (Business o Creator); permiso instagram_business_content_publish; tope de **100 publicaciones por API en 24 h** (la misma página dice 50 en la sección de carruseles); parámetro **is_ai_generated=true** | [V] https://developers.facebook.com/docs/instagram-platform/content-publishing/ |
| **TikTok Content Posting API** | Scope video.publish; **sin auditoría, todo queda privado** y solo se puede publicar en cuentas privadas; errores spam_risk_too_many_posts (tope diario por usuario) y reached_active_user_cap; parámetro **is_aigc** | [V] https://developers.tiktok.com/doc/content-posting-api-reference-direct-post |

Montar las tres apps propias implica **auditoría de TikTok, App Review de Meta y verificación de OAuth de Google**: semanas de trámite para un solo canal [I].

### 3.2 Servicios intermedios

| Servicio | Precio/mes | Programación | API | Notas | Fuente |
|---|---|---|---|---|---|
| **Upload-Post** | Free (10 subidas, sin TikTok) · **Basic 24 USD (16 anual): 5 perfiles, subidas ilimitadas** · Professional 50 USD | Sí (scheduled_date, colas) | REST, SDK Python/JS, **MCP**, n8n/Make | Expone is_aigc (TikTok), is_ai_generated (IG) y containsSyntheticMedia (YT). Topes por cuenta en 24 h: **TikTok 15, YouTube 30, Instagram 50** | [V] https://www.upload-post.com/llms-full.txt · https://docs.upload-post.com/guides/limit-of-uploads |
| Blotato | Starter 29 USD (20 cuentas) | Sí | API y MCP en todo plan pago | También genera video con IA | [V] https://www.blotato.com/pricing |
| Ayrshare | Premium 149 USD | Sí | Sí | Sobra para un canal | [V] https://www.ayrshare.com/pricing/ |
| Metricool, Buffer, Later, Repurpose.io | no verificado | Sí | Limitada o sin API pública para publicar video | No los recomiendo para un flujo desatendido | [I] |

**Recomendación:** **Upload-Post Basic (24 USD/mes)**. Un perfil equivale a una cuenta por red, y 10 videos/día caben en todos los topes (el más justo es TikTok, con 15). Al ser una app ya auditada por TikTok, evita el trámite propio [I: inferido de que publica en público para sus clientes; confirmarlo en la prueba gratis].

---

## 4. Orquestación

| Opción | Costo | ¿FFmpeg? | Veredicto |
|---|---|---|---|
| **GitHub Actions con cron** (repo privado) | Gratis dentro de los minutos del plan; ~15–25 min/día ≈ 450–750 min/mes [I] | Sí (ubuntu-latest) | **Recomendado**: corre en la nube, sin RAM local, con los secretos en GitHub Secrets. Ojo: el cron puede retrasarse y en repos públicos se desactiva tras 60 días sin actividad. En 2026 los runners alojados bajaron hasta un 39 % [V] https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/ |
| n8n Cloud Starter | ~20–24 EUR/mes, 2500 ejecuciones | No en la nube [I] → necesita un render externo | [S] https://www.cloudzero.com/blog/n8n-pricing/ |
| n8n autoalojado o VPS + cron | VPS ~5 USD/mes [I] | Sí | Válido, pero hay que mantener un servidor |
| Make / Zapier | por operación | No | Necesita Creatomate o Shotstack: más caro |
| Rutinas de Claude Code | Pro 5 y Max 15 ejecuciones/día; intervalo mínimo de 1 h; research preview | Depende del sandbox | Útil para que Claude **revise** el lote, no para renderizar [S] https://www.implicator.ai/anthropic-ships-claude-code-routines-cloud-automations-that-run-without-your-mac/ |
| PC de Mauricio | 0 | Sí | Descartado: poca RAM y Windows |

**Flujo propuesto (un job diario):** generar 10 guiones con la Claude API → validar el JSON y descartar temas repetidos contra un historial en el repo → Higgsfield (imágenes y clips en paralelo, respetando la concurrencia) → TTS → FFmpeg → subir el MP4 a un almacenamiento con URL pública temporal → Upload-Post con scheduled_date repartido en el día y las banderas de IA → guardar el log en el repo. **Al inicio, recomendado:** programar a +12 h para que un humano revise el lote antes de que salga.

---

## 5. Políticas y riesgos

| Tema | Regla | Fuente |
|---|---|---|
| **YouTube: contenido inauténtico** (15/7/2025) | No se monetiza el contenido «mass-produced, generic, repetitive». Ejemplos literales: «AI-generated content made with generic or unoriginal templates giving the impression of mass production», «templated storylines… with minimal or no narrative», «videos that stitch together unrelated or inconsistent AI clips» y «realistic visuals tricking viewers into believing a… natural disaster has occurred». La IA está permitida al servicio de una narrativa original | [V] https://support.google.com/youtube/answer/1311392 |
| YouTube: etiqueta de IA | Hay que declarar el contenido realista alterado o sintético; en Shorts la etiqueta va sobre el video (2026). Por API: containsSyntheticMedia | [S] https://ppc.land/youtube-shifts-generative-ai-labels-to-spots-viewers-will-actually-see/ · [V] https://developers.google.com/youtube/v3/docs/videos |
| Meta | Etiqueta obligatoria para video fotorrealista o audio realista generado o alterado; puede haber sanciones. Por API: is_ai_generated | [V] https://help.instagram.com/761121959519495 · https://developers.facebook.com/docs/instagram-platform/content-publishing/ |
| TikTok | Etiqueta AIGC obligatoria en contenido realista; por API: is_aigc. Creator Rewards: según fuentes secundarias, exige videos **de más de 1 min**, excluiría el contenido «totalmente IA» y no está en todos los países | [V] https://developers.tiktok.com/doc/content-posting-api-reference-direct-post · [S] https://adaptlypost.com/en/blog/tiktok-creator-fund-eligibility-requirements |
| Topes diarios | YouTube: 100 inserciones/día por proyecto; Instagram: 100 (o 50) en 24 h; TikTok: tope por usuario (Upload-Post aplica 15) | §3 |

**Riesgos principales, en orden:**

1. **Desmonetización en YouTube por «inauthentic content».** Un formato fijo de 10 videos/día con plantilla es justo lo que describe la política. Para mitigarlo: un tema y un dato verificable por video, voz y estructura que varíen, una serie con personaje o narrador propio y revisión humana del lote. Conviene valorar 3–5 videos/día de calidad en lugar de 10.
2. **Desinformación.** Los «¿qué pasaría si…?» con desastres realistas (un tsunami en Lima, por ejemplo) chocan con la política de imágenes engañosas. Hace falta un rótulo «hipotético» visible y la etiqueta de IA siempre puesta.
3. **TikTok sin monetización directa** para videos de menos de 1 min (no verificado en fuente oficial). Ahí el valor sería el alcance.
4. **Precios volátiles en Higgsfield** (descuentos promocionales del 30 al 60 %) y **retención de 7 días.** Hay que descargar en el momento y tener un plan B (Veo Lite o fal.ai).
5. **Una cuenta nueva que publica 10 videos/día desde el primer día** puede parecer spam [I]. Conviene subir de a poco: 3 → 5 → 10.
6. El tiempo de generación y el soporte de 9:16 por modelo **no están confirmados**: medirlos en el piloto.

---

## 6. Próximos pasos (piloto de 1 semana)

1. Crear la cuenta en open.higgsfield.ai, cargar 20 USD y medir 10 clips de Kling 3.0 y Wan 3.0 en imagen→video 9:16: tiempo, aceptación y costo real con el endpoint de estimación.
2. Abrir la prueba gratis de Upload-Post y publicar 1 video de prueba por red con las banderas de IA; confirmar que en TikTok sale público.
3. Recién entonces, spec y plan para Codex (repo nuevo y GitHub Actions).
