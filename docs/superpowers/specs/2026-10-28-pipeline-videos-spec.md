---
proyecto: Videos-Diario-NomosTin
tema: Spec del pipeline de videos «¿Qué pasaría si…?» (fase 1)
fecha: 2026-10-28
autor: arquitecto (Claude)
estado: borrador. Todo lo marcado «propuesto» lo decide Mauricio; lo marcado «piloto» lo confirma la tarea 0.
---

# Pipeline de videos «¿Qué pasaría si…?» · spec de la fase 1

Fuentes: [pipeline técnico](../../investigacion/2026-10-28-pipeline-tecnico.md), [estrategia viral](../../investigacion/2026-10-28-estrategia-viral-y-redes.md), [requerimientos y costos](../../investigacion/2026-10-28-requerimientos-y-costos.md). Framework del canal: [docs/framework/README.md](../../framework/README.md). Diagrama: [docs/framework/workflow.html](../../framework/workflow.html).

## 1. Problema

Producir cada día entre 3 y 10 videos verticales de 30 a 60 s, todos verificados y etiquetados como IA, y publicarlos en YouTube Shorts, TikTok e Instagram Reels a su hora. El costo debe tener un tope duro y un humano debe aprobar todo el primer mes. Sin servidor propio y con lo mínimo que funcione.

## 2. Alcance de la fase 1

**Entra:**
1. Un job diario en GitHub Actions que produce el lote del día siguiente: elige temas, escribe el guion, lo verifica, genera imágenes y un clip, sintetiza la voz y monta el video en **Remotion** (composiciones React/TSX parametrizadas por el JSON del guion), con subtítulos karaoke, rótulo, barra de progreso y música.
2. La revisión humana se hace en un issue de GitHub con una casilla por video y red. La etiqueta `aprobado` dispara la programación en Upload-Post.
3. Un tope de gasto diario, idempotencia ante reintentos y reejecuciones, y avisos en el issue.
4. La cadencia escalonada por red (rampa 3 → 6 → 10 en YouTube), con horarios en un archivo.
5. Un registro en CSV dentro del repo (historial de videos y métricas) y los binarios en Cloudflare R2.

**Fuera (fase 2 o nunca):**
- Publicar sin revisión humana, la reserva de 5 videos aprobados (RF-09) y la revisión por muestra. Se agregan cuando se apague la revisión total.
- Panel web, base de datos, colas, n8n y microservicios.
- Telegram. Los avisos van al issue de GitHub, con su notificación en la app móvil. **Propuesto:** si Mauricio prefiere Telegram, se agregan unas 15 líneas y un bot.
- Lectura automática de métricas por API (YouTube Analytics, TikTok, Meta). La tarea 0 confirma si Upload-Post las expone; si no, se exportan a mano una vez por semana.
- Responder comentarios de forma automática, el canal en inglés, una voz clonada y las apps propias de las tres redes.

## 3. Decisiones de arquitectura

| Tema | Decisión | Por qué |
|---|---|---|
| Orquestación | GitHub Actions con cron, en repo privado, `ubuntu-latest` (2 vCPU) y `timeout-minutes: 150` | No necesita servidor ni la RAM de la PC, corre Node + Chromium headless y guarda los secretos en GitHub Secrets |
| Motor de edición | **Remotion** (decisión de Mauricio, 2026-10-28): un proyecto en `video/` con varias plantillas visuales; `npx remotion render` por video. FFmpeg (`ffprobe`) solo valida la salida | Subtítulos por palabra, rótulos, gancho y transiciones en React, con plantillas que varían de verdad (§11). Licencia: §11.4 |
| Lenguaje | Python 3.12 con `uv` para el pipeline (deps: `anthropic`, `requests`, `boto3`; dev: `pytest`, `ruff`). TypeScript + React solo en `video/` (Node 22, deps: `remotion`, `@remotion/cli`, `@remotion/media`, `@remotion/captions`, `react`, `react-dom`, `zod`) | Python orquesta, Remotion dibuja. La frontera entre los dos es un único archivo `props.json` (§5.10) |
| Estado | Binarios y `estado.json` por video en R2; textos chicos en CSV dentro del repo (`datos/`) | Sin base de datos. R2 da 10 GB gratis y no cobra salida. Los CSV se versionan con git |
| Ejecución | **Por etapas sobre todo el lote**, en un solo proceso y sin hilos: todos los guiones → todas las verificaciones → se envían todas las imágenes a Higgsfield y se espera → se envían todos los clips y se espera → todas las voces → todos los renders (el proyecto Remotion se empaqueta una sola vez) | Las esperas de Higgsfield se pagan en minutos del runner: enviar en tanda las reduce de la suma de todas a la más larga. `ponytail:` techo de unos 10 videos por día en 150 min (§11.3) |
| Revisión | Un issue de GitHub por lote. Las casillas marcadas se publican y la etiqueta `aprobado` dispara el workflow `publicar` | Funciona desde el celular (GitHub Mobile), sin panel ni servidor |
| Publicación | Upload-Post Basic, una llamada por video y por red, con fecha programada | Ya tiene la auditoría de TikTok y de Meta. Lo confirma el piloto |
| Avisos | Comentario en el issue del lote; un issue nuevo con la etiqueta `alerta` para lo grave; correo de GitHub si el workflow falla | Ya existe y no hay que construir nada |

## 4. Estructura del repo

```
Videos-Diario-NomosTin/
├─ pyproject.toml              # uv; deps: anthropic, requests, boto3; dev: pytest, ruff
├─ config.toml                 # todo lo ajustable (ver §6); sin secretos
├─ prompts/
│  ├─ guionista.md             # sistema del guionista (plantilla de 45 s del framework)
│  └─ verificador.md           # sistema del revisor de veracidad
├─ esquemas/guion.schema.json  # esquema JSON del guion (se pasa a output_config.format)
├─ video/                      # proyecto Remotion (TSX); lo único en Node del repo
│  ├─ package.json             # remotion, @remotion/cli, @remotion/media, @remotion/captions, react, react-dom, zod
│  ├─ remotion.config.ts
│  ├─ src/
│  │  ├─ Root.tsx              # registra una <Composition> por plantilla; duración por calculateMetadata
│  │  ├─ props.ts              # esquema zod de PropsVideo (§5.10), fuente de verdad del contrato en TS
│  │  ├─ comunes/              # Subtitulos, Rotulo, BarraProgreso, Gancho, Escena (imagen con zoom / clip), Musica
│  │  └─ plantillas/           # Cinetica.tsx, Documental.tsx, Pizarra.tsx (nombres propuestos)
│  └─ public/
│     ├─ fuentes/              # tipografías OFL + OFL.txt
│     ├─ musica/               # pistas ≤ 10, ≤ 50 MB, + licencias.csv (archivo, fuente, licencia, url)
│     └─ trabajo/              # (gitignored) medios del video que se está renderizando
├─ datos/
│  ├─ temas.csv                # banco de temas
│  ├─ historial.csv            # una fila por video producido
│  └─ metricas.csv             # una fila por video, red y semana
├─ nomos/
│  ├─ config.py      # carga y valida config.toml y los secretos del entorno
│  ├─ reintentos.py  # reintentar(fn, esperas)
│  ├─ almacen.py     # R2: existe, leer_json, escribir_json, subir, bajar, url_firmada
│  ├─ gasto.py       # Presupuesto: tope diario
│  ├─ temas.py       # elegir temas del banco (mezcla, formatos, duplicados)
│  ├─ guion.py       # Claude guionista + validar_guion
│  ├─ verificar.py   # URLs + Claude verificador
│  ├─ higgsfield.py  # imagen 9:16 y animación, con request_id persistido
│  ├─ voz.py         # ElevenLabs con marcas de tiempo → captions (formato @remotion/captions)
│  ├─ render.py      # arma props.json, copia medios a video/public/trabajo, npx remotion render, ffprobe
│  ├─ revision.py    # texto del issue y lectura de las casillas; avisos
│  ├─ publicar.py    # Upload-Post
│  └─ cli.py         # python -m nomos.cli {generar,publicar,semana}
├─ tests/ (+ tests/fixtures/ con las respuestas reales del piloto, sin secretos)
├─ docs/piloto/2026-11-resultados.md
└─ .github/workflows/{ci.yml,generar.yml,publicar.yml}
```

## 5. Flujo y contratos entre etapas

IDs: `fecha` = día de **publicación** (AAAA-MM-DD, hora de Lima); `video_id` = `AAAA-MM-DD-vNN` (NN de 01 a 99). Prefijo en R2: `lotes/{fecha}/{video_id}/`.

### 5.1 `generar` (cron diario, propuesto 13:00 Lima = 18:00 UTC, para la fecha de mañana)

1. Calcula `n = cadencia.youtube[semana] + produccion.extra_por_lote`, donde `semana = (fecha - canal.fecha_inicio).days // 7`, recortada al último índice de la lista.
2. `temas.elegir(...)` toma `n` temas `pendiente` de `datos/temas.csv`.
3. Recorre las etapas **en orden y cada una sobre todos los videos vivos del lote**, saltando cada artefacto que ya existe en R2:
   `guion.json` → `verificacion.json` → `img-{k}.png` (todas enviadas en tanda) → `clip-{k}.mp4` (en tanda) → `voz.mp3` + `voz.json` → `props.json` → `final.mp4`.
   Un video entra al lote solo si su costo estimado completo cabe en el tope (§8); ese monto queda **reservado** hasta que se conoce el real. Si un tema se rechaza en guion o verificación, se toma el siguiente tema pendiente; como máximo, `n + 3` temas por lote.
4. Escribe `lotes/{fecha}/lote.json`, agrega las filas a `historial.csv`, marca los temas como `usado` y crea el issue de revisión (una sola vez: el número queda en `lote.json`).
5. El workflow hace commit de `datos/` con `GITHUB_TOKEN` (`contents: write`).

### 5.2 `publicar` (al poner la etiqueta `aprobado` en un issue `lote`)

1. Lee las casillas del cuerpo del issue (§5.8).
2. Por cada red, toma los videos marcados en el orden del issue y les asigna los horarios de `horarios.<red>` de esa fecha. Si un horario ya pasó, el video se programa en `ahora + 15 min × k`. Nunca se pasa de `topes_red.<red>` por día, contando lo que ya está programado.
3. Llama a Upload-Post una vez por (video, red), salvo que `estado.publicaciones[red]` ya exista. Guarda el resultado en `estado.json` y en `historial.csv` y comenta un resumen en el issue.

### 5.3 `datos/temas.csv`

`id,pregunta,subnicho,formato,origen,estado,creado`
- `id`: `t0001`, `t0002`, …; `subnicho` ∈ `config.mezcla` (claves); `formato` ∈ `config.formatos.lista`.
- `origen` ∈ `investigador|comentario|replica|mauricio`; `estado` ∈ `pendiente|usado|descartado`.

### 5.4 `guion.json` (lo devuelve Claude con `output_config.format` según `esquemas/guion.schema.json`; se valida además con `validar_guion`)

```json
{
  "pregunta": "¿Qué pasaría si la Luna desapareciera?",
  "subnicho": "espacio",
  "formato": "escalada",
  "certeza": "modelo_cientifico",
  "fuentes": [{"url": "https://science.nasa.gov/...", "respalda": "mareas"}, {"url": "https://...", "respalda": "eje terrestre"}],
  "escenas": [
    {"rol": "gancho", "narracion": "¿Qué pasaría si la Luna desapareciera esta noche?", "texto_pantalla": "¿Y si la Luna desaparece?", "prompt_visual": "...", "tipo": "clip"},
    {"rol": "apuesta", "narracion": "...", "texto_pantalla": "", "prompt_visual": "...", "tipo": "imagen"}
  ],
  "nota_revisor": "La parte del eje es un modelo, no un hecho.",
  "publicacion": {
    "youtube": {"titulo": "¿Qué pasaría si la Luna desapareciera?", "descripcion": "...", "hashtags": ["#shorts", "#ciencia", "#quepasariasi"]},
    "tiktok": {"texto": "...", "hashtags": ["#quepasariasi", "#ciencia", "#curiosidades"]},
    "instagram": {"texto": "...", "hashtags": ["#ciencia", "#espacio", "#quepasariasi"], "comentario_fuentes": "Fuentes: ..."}
  }
}
```

Reglas de `validar_guion(g, cfg) -> list[str]` (lista vacía = válido):
- `pregunta` empieza con «¿Qué pasaría si» o «¿Y si» y tiene ≤ 60 caracteres.
- `subnicho` está en `cfg.mezcla`; `formato` en `cfg.formatos.lista`; `certeza` ∈ `hecho|modelo_cientifico|especulacion`.
- Hay ≥ 2 `fuentes`, todas con `https://` y con dominio distinto entre sí.
- Tiene de 6 a 10 escenas; la primera es `gancho` y la última, `cierre`; hay exactamente un `cta`; los roles ∈ `gancho|apuesta|escalada|giro|cta|cierre`.
- La narración total tiene de **75 a 140 palabras** (de 30 a 60 s a unas 2,4 palabras por segundo).
- Las escenas con `tipo == "clip"` son ≤ `cfg.produccion.clips_por_video`.
- `youtube.titulo` tiene ≤ 60 caracteres y de 2 a 3 hashtags; TikTok e Instagram, de 3 a 5.
- Ningún `prompt_visual` contiene palabras de `cfg.prohibidas` (personas reales, marcas y personajes con derechos, sin distinguir mayúsculas).

Si el JSON no valida: se pide una vez más, con los errores como mensaje. Si vuelve a fallar, el tema se descarta (`estado=descartado`, con el motivo en el issue).

### 5.5 `verificacion.json`

```json
{"veredicto": "aprobado", "problemas": [{"escena": 4, "afirmacion": "...", "motivo": "..."}], "fuentes_ok": ["https://..."]}
```
- Primero, `requests.get` a cada fuente (timeout de 15 s y redirecciones permitidas): si alguna no responde 200, el veredicto es `rechazar` sin llamar a Claude.
- Después, Claude (`cfg.produccion.modelo_verificador`) con la herramienta `web_fetch_20260209` (`max_uses` = 4) lee las fuentes y devuelve el JSON. **Piloto:** confirmar que `web_fetch` se puede combinar con `output_config.format`; si no, se pide el JSON en texto y se valida con `json.loads` y una comprobación de las claves.
- `corregir` → una sola reescritura del guion con los `problemas` como mensaje y una nueva verificación. `rechazar`, o `corregir` dos veces → el tema se descarta.

### 5.6 `voz.json`

`{"duracion": 44.2, "captions": [{"text": "¿Qué", "startMs": 0, "endMs": 210, "timestampMs": 105, "confidence": null}, {"text": " pasaría", "startMs": 230, ...}]}`

- `captions` usa el tipo `Caption` de `@remotion/captions` tal cual: una entrada por palabra; **desde la segunda, `text` empieza con un espacio**, porque así separa las páginas `createTikTokStyleCaptions` (doc de Remotion). `timestampMs` = punto medio de la palabra.
- Se arma con la alineación por carácter de `POST /v1/text-to-speech/{voice_id}/with-timestamps` (`characters`, `character_start_times_seconds` y `character_end_times_seconds`), partiendo por espacios. La narración que se envía es `" ".join(e.narracion for e in escenas)`.

### 5.7 `estado.json` (uno por video; es la fuente de verdad para la idempotencia y el costo)

```json
{
  "video_id": "2026-11-16-v01", "tema_id": "t0007",
  "etapa": "listo",
  "costos": {"claude": 0.031, "higgsfield": 0.21, "elevenlabs": 0.05},
  "higgsfield_pendientes": {"clip-1": "req_abc"},
  "intentos_guion": 1,
  "motivo_fallo": null,
  "publicaciones": {"youtube": {"programado": "2026-11-16T07:00:00-05:00", "id": "...", "error": null}}
}
```
`etapa` ∈ `nuevo|guion|verificado|imagenes|clips|voz|renderizado|listo|fallido|descartado`. Mientras el costo real no se conoce, `costos.reserva` guarda lo estimado (§8). Se escribe en R2 después de **cada** llamada paga.

### 5.8 Issue de revisión (`revision.py`)

Título: `Lote {fecha} · {n} videos · USD {gasto:.2f}`. Etiqueta: `lote`. Por video:

```
### v01 · ¿Qué pasaría si la Luna desapareciera?
espacio · escalada · modelo científico · USD 0.42
[video](url firmada 7 días) · [guion](url firmada)
Fuentes: <url1> · <url2>
Verificador: aprobado. Nota: La parte del eje es un modelo, no un hecho.
- [x] v01 youtube
- [x] v01 tiktok
- [ ] v01 instagram
```
- Por defecto se marcan: YouTube en todos los videos, salvo en los `extra_por_lote` del final; TikTok en los `cadencia.tiktok[semana]` primeros e Instagram en los `cadencia.instagram[semana]` primeros.
- Al final, el checklist humano del framework (§4 de `docs/framework/README.md`), como recordatorio sin casillas que se lean.
- Regex de lectura: `^\s*[-*] \[([ xX])\] (v\d{2}) (youtube|tiktok|instagram)\s*$` (multilínea). Las líneas que no coinciden se ignoran.

### 5.9 `historial.csv` y `metricas.csv`

- `historial.csv`: `fecha,video_id,tema_id,pregunta,subnicho,formato,certeza,etapa,costo_usd,youtube,tiktok,instagram`. Las tres últimas columnas guardan la hora programada o `error:<código>`. Se actualiza la fila de cada `video_id`, sin duplicarla. Si un video se borra por un error, se pone `etapa=retirado` a mano (framework §4.3).
- `metricas.csv`: `semana,video_id,red,vistas,pct_visto,vistos_vs_deslizados,compartidos,comentarios,seguidores`. La llena una exportación manual o la API de Upload-Post (decisión del piloto).

### 5.10 `props.json` (frontera Python → Remotion)

Lo escribe `render.py`; lo valida el esquema zod de `video/src/props.ts` (si no valida, Remotion falla antes de renderizar). Las rutas son relativas a `video/public/` (se cargan con `staticFile`).

```json
{
  "plantilla": "Cinetica",
  "duracion_s": 44.5,
  "pregunta": "¿Qué pasaría si la Luna desapareciera?",
  "texto_gancho": "¿Y si la Luna desaparece?",
  "rotulo": "Escenario hipotético · Hecho con IA",
  "escenas": [
    {"rol": "gancho", "inicio_s": 0.0, "fin_s": 2.1, "tipo": "clip", "medio": "trabajo/2026-11-16-v01/clip-1.mp4", "movimiento": "acercar"},
    {"rol": "apuesta", "inicio_s": 2.1, "fin_s": 6.0, "tipo": "imagen", "medio": "trabajo/2026-11-16-v01/img-2.png", "movimiento": "alejar"}
  ],
  "voz": "trabajo/2026-11-16-v01/voz.mp3",
  "musica": "musica/pista-03.mp3",
  "volumen_musica": 0.12,
  "captions": [{"text": "¿Qué", "startMs": 0, "endMs": 210, "timestampMs": 105, "confidence": null}],
  "semilla": "2026-11-16-v01"
}
```
- `plantilla` ∈ `config.render.plantillas` (el id de la `<Composition>`). Se elige de forma rotativa: `plantillas[(orden_del_dia + índice) % len]`, sin pasar de `render.max_por_dia_plantilla`.
- `escenas[].inicio_s/fin_s`: desde el `startMs` de la primera palabra de la escena hasta el de la primera palabra de la siguiente; la primera arranca en 0, la última termina en `duracion_s`, y no quedan huecos.
- `duracion_s` = duración de la voz + 0,3 s. La `<Composition>` fija `durationInFrames = ceil(duracion_s × 30)` con `calculateMetadata`.
- `movimiento` ∈ `acercar|alejar|paneo_izq|paneo_der` y se alterna por escena; en un clip se ignora. `musica` puede ser `null`.

## 6. Configuración (`config.toml`)

Valores propuestos. `config.py` los valida al arrancar y falla con un mensaje claro si falta una clave o un valor está fuera de rango.

```toml
[canal]
nombre = "PENDIENTE"            # decide Mauricio
idioma = "es-419"
zona = "America/Lima"
fecha_inicio = "2026-11-16"     # propuesto: tras el piloto y el calentamiento de cuentas

[produccion]
extra_por_lote = 1
clips_por_video = 1             # solo el gancho es clip; el resto son imágenes con zoom (escenario «mínimo»)
segundos_clip = 5
modelo_video = "PILOTO"         # wan-3.0 o kling-3.0-std: lo decide la tarea 0 (ruta exacta del endpoint)
modelo_imagen = "PILOTO"        # soul-2 (ruta exacta del endpoint en la tarea 0)
modelo_guion = "claude-sonnet-5-5"
modelo_verificador = "claude-sonnet-5-5"
voces = ["PILOTO"]              # voice_id de ElevenLabs; una voz fija del canal
modelo_voz = "PILOTO"           # p. ej., eleven_flash o multilingual (precio en [precios])

[tope]
usd_dia = 8.0                   # propuesto (mes 1 ≈ 190 USD con IGV)
aviso_pct = 80

[precios]                       # USD de lista, solo para estimar antes de cada llamada; se actualizan con el piloto
video_por_segundo = 0.063
imagen = 0.015
voz_por_1000_caracteres = 0.08
claude_por_guion = 0.05         # techo por llamada (guion o verificación)

[cadencia]                      # videos por día; el índice es la semana desde fecha_inicio; se repite el último
youtube   = [3, 3, 6, 6, 10]
tiktok    = [2, 2, 3, 3, 4]
instagram = [1, 1, 2, 2, 3]

[horarios]                      # hora de Lima (HH:MM); se usan los primeros n de cada red
youtube   = ["07:00","09:30","12:00","13:30","15:00","17:00","18:30","20:00","21:30","23:00"]
tiktok    = ["13:30","18:30","20:00","21:30"]
instagram = ["12:00","20:00","21:30"]

[topes_red]                     # publicaciones por día y cuenta (Upload-Post)
youtube = 30
tiktok = 15
instagram = 50

[mezcla]                        # proporción del lote por subnicho (suma 1.0)
espacio = 0.15                  # el doc de estrategia da 30 % a «espacio y ciencia»: aquí se reparte
ciencia = 0.15
cuerpo = 0.25
peru_latam = 0.20
historia = 0.15
animales = 0.10

[formatos]
lista = ["escalada", "linea_de_tiempo", "mito_vs_dato", "tu_vs"]
max_por_dia = 4

[render]
plantillas = ["Cinetica", "Documental", "Pizarra"]   # propuesto; las diseña disenador-ui con el plugin de Remotion
max_por_dia_plantilla = 4
concurrencia = "PILOTO"         # valor de --concurrency; lo da `npx remotion benchmark` en el runner (tarea 0)
volumen_musica = 0.12           # ≈ −18 dB bajo la voz

[etiquetas]
rotulo = "Escenario hipotético · Hecho con IA"   # se ve en todo el video, arriba

prohibidas = ["pikachu", "marvel", "disney", "coca-cola"]   # se amplía con el uso; Claude también tiene la regla en el prompt
```

## 7. Secretos

En GitHub Secrets (nunca en el repo, en la bóveda ni en los registros): `ANTHROPIC_API_KEY`, `HIGGSFIELD_KEY_ID`, `HIGGSFIELD_KEY_SECRET`, `ELEVENLABS_API_KEY`, `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`, `UPLOAD_POST_API_KEY` y `UPLOAD_POST_USER` (y `REMOTION_LICENSE_KEY` solo si se compra la licencia de §11.4). `GITHUB_TOKEN` lo da Actions. `config.py` los lee de `os.environ` y falla si falta uno que la orden actual necesita. En local se exportan a mano; `.env` va en `.gitignore`. Ningún módulo registra cabeceras ni cuerpos de petición.

Frenos de gasto fuera del código (los configura Mauricio): saldo prepago en Higgsfield, límite mensual en la consola de Anthropic, un plan con tope en ElevenLabs y una tarjeta con límite propio.

## 8. Tope de gasto (`gasto.py`)

- `Presupuesto(tope_usd, gastado_usd)`: `gastado` = suma de `costos` (incluida `costos.reserva`) de todos los `estado.json` del lote, recalculada al arrancar, así que una reejecución no lo reinicia. Al aceptar un video se reserva su estimado completo; cada costo real descuenta su parte de la reserva, y al terminar o fallar el video se libera lo que queda.
- `cabe(estimado) -> bool`, `cargar(proveedor, usd)` y `aviso() -> bool` (si `gastado ≥ tope × aviso_pct / 100`).
- **Un video solo empieza si su costo estimado completo cabe** en lo que queda: `clips × segundos × video_por_segundo + imágenes × imagen + caracteres/1000 × voz + 2 × claude_por_guion`. Así, el tope no se pasa salvo por un error de estimación de una llamada.
- Si al clip le falta presupuesto, se reemplaza por una imagen con zoom. Si al video no le alcanza, el lote termina; los temas no usados siguen `pendiente`, se abre un issue `alerta` («tope alcanzado: X de N videos») y el issue del lote sale con lo que hay.
- El costo real se carga con el monto que devuelve cada proveedor si lo informa (Higgsfield lo da por petición; piloto); si no, con la estimación.

## 9. Idempotencia y recuperación

| Riesgo | Regla |
|---|---|
| El job se cae a mitad del lote | Se vuelve a correr `generar --fecha F`: cada etapa con artefacto en R2 se salta y no se paga dos veces |
| Higgsfield ya cobró, pero el job murió antes de descargar | El `request_id` se guarda en `estado.json` **antes** de consultar el estado; al volver a correr se consulta ese `request_id` en vez de enviar otro. Cabecera `Idempotency-Key: {video_id}:{paso}:{intento}` |
| Las salidas de Higgsfield vencen a los 7 días | Se descargan y se suben a R2 apenas quedan `completed` |
| La etiqueta `aprobado` se pone dos veces o se reejecuta `publicar` | Se salta cada (video, red) con `publicaciones[red]` existente; cabecera `Idempotency-Key: {video_id}:{red}` en Upload-Post si la acepta (piloto) |
| Dos workflows a la vez | `concurrency: group: nomos, cancel-in-progress: false` en `generar` y en `publicar` |
| El issue se crea dos veces | El número del issue se guarda en `lote.json`; si existe, se edita en vez de crear otro |

## 10. Reintentos y respaldos

`reintentos.reintentar(fn, esperas=(60, 300, 900))`: reintenta ante errores de red, 429 (respetando `Retry-After` si es mayor) y 5xx. Los 4xx restantes no se reintentan. En los tests las esperas son `(0, 0, 0)`.

| Etapa | Falla | Respuesta |
|---|---|---|
| Guion | JSON inválido o reglas | 1 reintento con los errores → tema descartado |
| Verificación | `corregir` / `rechazar` | 1 reescritura / tema descartado |
| Imagen | `failed` | 1 reintento → video `fallido` |
| Imagen o clip | `nsfw` | 1 reintento con el prompt suavizado por el guionista → si es clip, imagen; si es imagen, `fallido` |
| Clip | Agota los reintentos o se pasa del tope | Se usa la imagen de la escena con zoom |
| Voz | Agota los reintentos | Video `fallido` |
| Render | `npx remotion render` ≠ 0 | 1 reintento (Chromium a veces falla al arrancar) → video `fallido`, con el stderr recortado a 2000 caracteres en `motivo_fallo` |
| Render | `ffprobe`: no es 1080×1920, no es H.264/AAC o la duración está fuera de 30–60 s | Video `fallido` |
| Upload-Post | Agota los reintentos o devuelve un error | `publicaciones[red].error` + issue `alerta` |

Un video `fallido` no detiene el lote: se toma el siguiente tema, con el límite de §5.1.

## 11. Montaje en Remotion (`video/` + `nomos/render.py`)

### 11.1 Reparto entre Python y Remotion

| Python (`render.py`) | Remotion (`video/src`) |
|---|---|
| Baja de R2 los medios del video a `video/public/trabajo/{video_id}/` y los borra al terminar | `Root.tsx`: una `<Composition>` por plantilla, 1080×1920, 30 fps, con duración por `calculateMetadata` |
| Calcula los tiempos de escena y elige la plantilla y la música (spec §5.10) | `comunes/Escena`: imagen con zoom o paneo por `interpolate` (Ken Burns) o clip con `<Video>` de `@remotion/media` (el que la doc de rendimiento recomienda; no `OffthreadVideo`). Si el clip es más corto que la escena, se congela el último fotograma |
| Escribe `props.json` | `comunes/Subtitulos`: `createTikTokStyleCaptions({captions, combineTokensWithinMilliseconds: 1200})`, palabra activa resaltada, tercio medio de la pantalla y `white-space: pre` |
| `npx remotion bundle` una vez por corrida, y `npx remotion render <bundle> <plantilla> final.mp4 --props=props.json --concurrency=<cfg>` por video (piloto: confirmar que `render` acepta el directorio del bundle; si no, el entry point) | `comunes/Gancho` (texto grande de 0 a 2 s), `comunes/Rotulo` (siempre visible arriba), `comunes/BarraProgreso` y `comunes/Musica` (`<Audio>` con `volumen_musica`) |
| `ffprobe`: 1080×1920, H.264, AAC y duración entre 30 y 60 s | Tipografía OFL local desde `public/fuentes` |

- Salida: H.264 + AAC (lo que Remotion produce por defecto en `.mp4`). No se normaliza el volumen: YouTube, TikTok e Instagram normalizan solos. `ponytail:` sin `loudnorm` ni ducking; se agregan con un paso de FFmpeg si la revisión lo pide.
- Reglas de diseño para el render en CPU (doc de rendimiento de Remotion): **sin** `filter: blur()`, `box-shadow` grandes, WebGL ni Canvas 2D animado, porque en un runner sin GPU son muy lentos. Si hace falta un fondo difuminado, se genera como imagen.
- Plantillas: 3 en la fase 1 (propuesto). Cada una cambia la tipografía, la paleta, la posición y el estilo de los subtítulos, la entrada de las escenas, el gancho y la barra de progreso. Así se ataca la política de YouTube contra el contenido de plantilla, junto con los 4 formatos narrativos. Las diseña `disenador-ui` con el plugin de Remotion para Claude Code (`/remotion:remotion-create`) y deja un brief por plantilla en `docs/framework/plantillas.md`; Codex las programa.

### 11.2 Validación local del proyecto Remotion

`npx tsc --noEmit` y un render de 3 s con `video/props-ejemplo.json` (`--frames=0-89`) en CI. Studio (`npx remotion studio`) solo se usa para diseñar; no corre en el pipeline.

### 11.3 Tiempo de render y minutos de GitHub Actions

**No hay una medición publicada para nuestro caso; la mide la tarea 0** con un render de 45 s a 1080×1920 en `ubuntu-latest`. Hasta entonces, valen estas estimaciones:

| Partida por corrida | Estimación [I] | Base |
|---|---|---|
| Preparar el runner (uv, `npm ci`, Chrome Headless Shell, FFmpeg) | 3-4 min | Una vez por corrida |
| `npx remotion bundle` | ~1 min | Una vez por corrida |
| Claude (guion + verificación) | ~1,5 min por video | Llamadas en serie |
| Higgsfield en tanda (imágenes + clips) | 5-12 min por lote | Dos esperas (la más larga de cada tanda), no una por video |
| Voz | < 0,5 min por video | |
| **Render de 45 s (1350 fotogramas) en 2 vCPU** | **1,5-5 min por video** | Entre 5 y 15 fotogramas por segundo con imágenes, un clip y texto, sin efectos de GPU |

| Escenario | Videos por corrida (con 1 extra) | Minutos por día | Minutos al mes (30 días) | Costo de Actions |
|---|---|---|---|---|
| Semanas 1-2 | 4 | 20-45 | 600-1350 | 0 (GitHub Free trae USD 12 ≈ 2000 min de Linux 2-core) |
| Semanas 3-4 | 7 | 30-70 | 900-2100 | 0 a ~USD 1 |
| 10 por día | 11 | 40-100 | 1200-3000 | 0 a ~USD 6 (excedente a USD 0,006/min); con GitHub Pro (USD 18 incluidos ≈ 3000 min) entra entero |

Fuentes: [facturación de GitHub Actions](https://docs.github.com/en/billing/concepts/product-billing/github-actions), consultada el 2026-10-28 (sin minutos fijos: USD 12 en Free y USD 18 en Pro; Linux 2-core a USD 0,006/min; los runners estándar son gratis en repos públicos). Los minutos son una conversión propia.

**Conclusión:** a 10 por día cabe en el plan gratuito, o cuesta unos pocos dólares de excedente, **siempre que el render por video quede en ≤ 5 min y la corrida en ≤ 150 min**. Si el piloto mide más, se aplica en este orden (propuesto):

1. **Pagar el excedente.** Es lo más simple. Ni con 5000 min al mes se pasa de unos USD 18.
2. **Repartir el render en una matriz de jobs.** Un job `render` con `strategy.matrix` de 2 a 4 runners, cada uno con su parte del lote desde R2. Usa los mismos minutos y menos tiempo de reloj, y se resuelve con ~20 líneas de YAML.
3. **Remotion Lambda.** Según el ejemplo de costos de Remotion, un video de 1 min (1440 fotogramas) cuesta unos USD 0,017-0,021 de cómputo y tarda ~19 s; 300 videos al mes ≈ USD 6, más S3 y la cuenta de AWS. Conviene si hace falta velocidad o más de 10 por día.
4. Runner más grande (4 u 8 núcleos): se cobra aparte y no usa los minutos incluidos. Solo si las opciones 2 y 3 no sirven.

Descartado: hacer público el repo para tener minutos gratis, porque expondría guiones, temas e historial.

### 11.4 Licencia de Remotion: decide Mauricio

Verificada en remotion.pro/license el 2026-10-28:

| Caso | Licencia | Costo |
|---|---|---|
| El canal es **personal de Mauricio** (individuo) | Gratis, con uso comercial | USD 0 |
| El canal es de **Prometeus** (4 socios: empresa de 4 o más personas) | «Remotion for Automators» | USD 0,01 por render, con un **mínimo de USD 100 al mes**: con 300 videos al mes se paga el mínimo |

- Costo mensual total con la licencia: escenario mínimo ≈ USD 190 con IGV → **≈ USD 290** si el canal es de Prometeus (requerimientos §C). La licencia pesa más que todo lo que no es video.
- Si se compra, la clave va en el secreto `REMOTION_LICENSE_KEY` y se pasa al render como pida la licencia. Remotion manda telemetría solo cuando se usa `licenseKey`.
- **Propuesto:** arrancar el canal como personal de Mauricio y pasarlo a Prometeus cuando genere ingresos, y en ese momento comprar la licencia. Que Mauricio lo confirme con los socios: es una decisión de negocio y de titularidad del canal.

## 12. Registro y avisos

- `logging` a stdout con el formato `%(asctime)s %(levelname)s %(message)s`; cada línea de un video empieza con su `video_id`. El registro estructurado es `estado.json`.
- Al terminar `generar`, se comenta en el issue del lote: los videos listos, los fallidos con su motivo, el gasto contra el tope y el banco de temas restante.
- Se abre un issue `alerta` (uno por motivo y fecha, sin duplicados) cuando: se alcanza el tope; el gasto pasa del `aviso_pct`; quedan menos de 14 temas `pendiente`; Upload-Post responde 401/403 (token vencido o red desconectada); o no se logra ningún video listo en el lote.
- Si el workflow falla por una excepción no controlada, GitHub manda el correo.

## 13. Etiquetado y políticas (no negociable)

- Upload-Post siempre recibe las banderas de IA en las tres redes: `is_aigc` (TikTok), `is_ai_generated` (Instagram) y `containsSyntheticMedia` (YouTube). Los nombres exactos del formulario se confirman en el piloto.
- El rótulo «Escenario hipotético · Hecho con IA» se ve en todo el video. No se quitan los metadatos C2PA de Higgsfield a propósito; el render de Remotion no los conserva y es aceptable porque la etiqueta va por la API.
- En los prompts no se generan caras de personas reales, logos ni personajes con derechos (regla en `prompts/guionista.md`, más la lista `prohibidas`). Los temas de salud no dan consejos ni dosis.

## 14. Criterios de aceptación de la fase 1

1. `uv run ruff check . && uv run pytest` y `cd video && npx tsc --noEmit && npx remotion render` de prueba (props de ejemplo) en verde en CI.
2. Una reejecución de `generar` sobre un lote completo no hace ninguna llamada paga (test con proveedores falsos).
3. Con el tope en 1 USD y precios de prueba, el gasto del lote nunca pasa del tope (test).
4. Poner `aprobado` dos veces no programa dos veces (test).
5. Primera corrida real (tarea 12): un lote de 1 a 3 videos que pasa la revisión, sale a su hora en las tres redes con la etiqueta de IA visible, con un costo dentro del 5 % de lo facturado.
