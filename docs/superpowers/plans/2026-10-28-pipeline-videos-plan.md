# Pipeline de videos «¿Qué pasaría si…?» · plan de implementación (fase 1)

> **Para agentes:** SUB-SKILL REQUERIDA: usa superpowers:subagent-driven-development (recomendado) o superpowers:executing-plans para implementar este plan tarea por tarea. Los pasos usan casillas (`- [ ]`) para el seguimiento.
> **Protocolo del equipo:** Codex programa y valida **sin commit**; el `revisor` aprueba; Claude hace el commit y el PR. Una tarea por Codex.

**Meta:** un job diario en GitHub Actions que produce el lote de videos de mañana, lo pone a revisión en un issue y, con la etiqueta `aprobado`, lo programa en Upload-Post para las tres redes.

**Arquitectura:** Python sin servidor que avanza por etapas sobre todo el lote y deja el montaje a composiciones de Remotion parametrizadas por `props.json` (spec §5.10). Cada etapa deja su artefacto en R2 y un `estado.json` por video que hace idempotente cualquier reejecución. Los textos chicos (temas, historial y métricas) van en CSV dentro del repo. La revisión humana es un issue con casillas.

**Stack:** Python 3.12 + `uv` (`anthropic`, `requests`, `boto3`; `pytest`, `ruff`) orquesta; **Remotion** (Node 22, React/TSX, en `video/`) monta y renderiza; `ffprobe` valida; GitHub Actions; Cloudflare R2; Higgsfield API; ElevenLabs; Upload-Post.

**Spec:** [`docs/superpowers/specs/2026-10-28-pipeline-videos-spec.md`](../specs/2026-10-28-pipeline-videos-spec.md). Framework: [`docs/framework/README.md`](../../framework/README.md). Codex lee la spec completa antes de cada tarea: los contratos JSON y CSV están en la spec §5 y no se repiten aquí.

## Restricciones globales

- Python 3.12, gestionado con `uv`. Dependencias de ejecución: **solo** `anthropic`, `requests` y `boto3`. Desarrollo: `pytest` y `ruff`. Toda dependencia nueva necesita el visto bueno del arquitecto.
- Node 22 solo dentro de `video/`. Dependencias de `video/`: **solo** las de la spec §3 (`remotion`, `@remotion/cli`, `@remotion/media`, `@remotion/captions`, `react`, `react-dom`, `zod`), más `typescript` y `tsx` como dev, todas con versión exacta y `package-lock.json`. Todas las piezas `remotion`/`@remotion/*` van en la misma versión.
- Sin `filter: blur()`, sombras grandes, WebGL ni Canvas animado en las composiciones (spec §11.1).
- Configuración solo en `config.toml` (se lee con `tomllib`, de la stdlib). Secretos solo por `os.environ` (spec §7). Nunca se registran cabeceras, claves ni cuerpos de petición.
- Todo proveedor externo se **inyecta** (parámetro o argumento con valor por defecto), para que los tests usen falsos: ningún test llama a la red ni gasta dinero.
- Hora: todas las horas de publicación en `America/Lima` con `zoneinfo`; los ISO llevan desplazamiento (`-05:00`).
- Modelo de Claude: `claude-sonnet-5-5` (desde `config.produccion.modelo_guion`), SDK oficial `anthropic` y salida estructurada con `output_config={"format": {...}}`. Sin `temperature`. Lleva `betas=["server-side-fallback-2026-07-01"]` y `fallbacks="default"`, y se revisa `stop_reason == "refusal"` antes de leer `content`.
- Los textos que ve una persona (issue, avisos y rótulo) van en español.
- Validación de cada tarea: `uv run ruff check . && uv run pytest -q` en verde y, si toca `video/`, además `cd video && npx tsc --noEmit && npx tsx --test src/*.test.ts`.
- Ponytail: sin clases si basta una función; sin capas de abstracción para un único proveedor. Cada atajo con techo conocido lleva un comentario `# ponytail:` que nombre el techo.

## Foco de revisión

1. **Reejecutar `generar` después de una caída a mitad del lote:** no debe hacer ninguna llamada paga ya hecha, y debe retomar un `request_id` de Higgsfield pendiente en vez de enviar otro (test en T12).
2. **Tope de gasto con respaldos y reescrituras:** reintentos, reescritura del guion y paso de clip a imagen no pueden dejar el gasto del lote por encima del tope (test en T12).
3. **Etiqueta `aprobado` puesta dos veces o `publicar` reejecutado:** cero publicaciones duplicadas por (video, red) (test en T13).
4. **Hora de Lima frente al UTC del runner:** un horario «07:00» se programa como `T07:00:00-05:00` del día del lote. Si ya pasó, se corre a `ahora + 15 min × k`, nunca a una hora pasada (test en T13).
5. **Issue editado a mano:** `[X]` en mayúscula, `*` en vez de `-`, espacios al final y líneas borradas o agregadas no rompen la lectura; una línea ajena se ignora (test en T13).

---

## Orden y dependencias

```
T0 piloto (Mauricio + Claude) ─► T1 esqueleto ─► T2 almacén y gasto ─┬─► T3 temas
                                                                     ├─► T4 guion ─► T5 verificar
                                                                     ├─► T6 higgsfield (tandas)
                                                                     └─► T7 voz y captions ──────────────┐
T8 diseño de plantillas (disenador-ui, en paralelo desde el día 1) ─► T9 Remotion + Cinetica ─┬─► T10 Documental y Pizarra
                                                                                               └─► T11 render desde Python ◄┘
T3..T7 + T9..T11 ─► T12 generar + issue + workflow ─► T13 publicar + workflow ─► T14 primera corrida real (Mauricio + Claude)
T13 + 2 semanas de datos ─► T15 informe semanal (opcional)
```

- En serie: T1 → T2 → (T3 a T7) → T12 → T13 → T14. **T3, T4, T6 y T7 pueden ir en paralelo** (cada una toca su módulo y su test); T5 espera a T4.
- La rama de Remotion: T8 (diseño, sin código) puede arrancar ya. T9 necesita T8 y T1 (por `ci.yml` y `.gitignore`). T10 y T11 pueden ir en paralelo después de T9; T11 también necesita T7.
- T1 define **todas** las claves de `config.toml` de la spec §6, para que ninguna tarea posterior toque ese archivo. Si una tarea necesita una clave nueva, para y avisa.
- `pyproject.toml` solo lo toca T1; `video/package.json` solo lo toca T9.

---

### Tarea 0: Piloto de una semana (manual, sin código de producto)

**Quiénes:** Mauricio (cuentas, pagos y OAuth) + Claude (`investigador`/`devops`), que guía, mide y anota. Presupuesto: **USD 20 en Higgsfield** y la prueba gratis de Upload-Post.

**Archivos:**
- Crear: `docs/piloto/2026-11-resultados.md` (tabla de resultados y decisiones)
- Crear: `tests/fixtures/higgsfield_submit.json`, `higgsfield_status_completed.json`, `higgsfield_status_failed.json`, `elevenlabs_timestamps.json` (recortado a unas 40 palabras), `uploadpost_respuesta.json` y `uploadpost_error.json`. Son respuestas reales **sin claves ni URLs firmadas**.

- [ ] **Paso 1: Cuentas.** Mauricio crea el correo de la marca y las cuentas de YouTube, Instagram profesional y TikTok con 2FA (requerimientos, A1–A2), y empieza el calentamiento manual de 3 a 5 días en paralelo.
- [ ] **Paso 2: Higgsfield.** Cuenta en open.higgsfield.ai, recarga de USD 20 y clave. Anotar las rutas exactas de los endpoints (Soul 2 imagen, Kling 3.0 std imagen→video, Wan 3.0 imagen→video) y si aceptan `aspect_ratio`/9:16.
- [ ] **Paso 3: 10 clips.** Los mismos 5 prompts de gancho (de los ejemplos del framework), 5 s, imagen 9:16 de Soul → video: 5 con Kling y 5 con Wan. Por clip, anotar: costo estimado (endpoint de estimación) y cobrado, tiempo hasta `completed`, si sale 9:16 sin bandas, y si es aceptable (manos, física y texto) a juicio de Mauricio.
- [ ] **Paso 4: Voz.** ElevenLabs: una narración de unos 110 palabras con `/with-timestamps` y dos voces candidatas. Guardar la respuesta recortada como fixture y anotar el costo y la voz elegida.
- [ ] **Paso 5: Upload-Post.** Prueba gratis (o Basic, si TikTok no entra en la gratis). Se conectan las tres cuentas a un perfil y se publica **1 video de prueba por red** (un clip del paso 3) con las banderas de IA. Anotar los nombres exactos de los campos (red, título, descripción, `scheduled_date`, `is_aigc`, `is_ai_generated` y `containsSyntheticMedia`). Confirmar si acepta una **URL** de video o solo el archivo, si acepta `Idempotency-Key` y si da métricas por publicación por API.
- [ ] **Paso 6: Verificar en las apps** que el video de TikTok es **público** y que la etiqueta de IA aparece en las tres redes. Captura de pantalla en el documento.
- [ ] **Paso 7: Claude.** Una llamada de prueba con `web_fetch_20260209` junto con `output_config.format`, para confirmar que se pueden combinar (spec §5.5).
- [ ] **Paso 7b: Remotion en el runner.** En un repo privado de prueba, un workflow con `workflow_dispatch` (como el de la doc de SSR de Remotion) que cree la plantilla oficial de captions estilo TikTok (`npx create-video@latest`), la deje en **45 s a 1080×1920** con un PNG con zoom, un clip de 5 s del paso 3 y la voz del paso 4, y corra `npx remotion benchmark` y `npx remotion render`. Anotar el `--concurrency` óptimo, los **minutos de render**, el total del job y el pico de RAM. Si pasa de 5 min, se decide la alternativa de la spec §11.3.
- [ ] **Paso 7c: Licencia de Remotion.** Mauricio decide de quién es el canal (personal o Prometeus) y, con eso, si se compra «Remotion for Automators» (USD 100 al mes como mínimo; spec §11.4).
- [ ] **Paso 8: Decidir y anotar** en `docs/piloto/2026-11-resultados.md`: `modelo_video`, `modelo_imagen`, `voces`, `modelo_voz`, `[precios]` reales, `render.concurrencia`, la licencia de Remotion, el modo de envío a Upload-Post (URL o archivo) y la fuente de métricas (API o exportación).

**Criterio para seguir (go/no-go):**
- TikTok publica **público** por Upload-Post. Si no: plan B, que lo decide Mauricio (TikTok manual desde el MP4 aprobado en la fase 1, o Blotato).
- Hay un modelo con ≥ 6 de 10 clips aceptables, ≤ USD 0,50 por clip aceptado de 5 s y una mediana ≤ 10 min hasta `completed`. Si no, se prueba Veo 3.1 Lite (Gemini API) antes de construir.
- Las banderas de IA quedan visibles en las tres redes.
- El render de 45 s en `ubuntu-latest` tarda ≤ 5 min, o hay una alternativa de la spec §11.3 elegida con su costo.

**Validación:** el documento tiene la tabla de 10 clips completa, las 3 capturas, la medición de render y los valores decididos del paso 8; los fixtures existen y `grep -riE "key|secret|token|X-Amz-Signature" tests/fixtures` no devuelve nada.

---

### Tarea 1: Esqueleto, configuración y CI

**Archivos:**
- Crear: `pyproject.toml`, `.gitignore` (`.env`, `__pycache__/`, `.venv/`, `salida/`, `video/node_modules/`, `video/out/`, `video/build/`, `video/public/trabajo/`), `config.toml` (todas las claves de la spec §6, con los valores del piloto), `nomos/__init__.py`, `nomos/config.py`, `nomos/reintentos.py` y `.github/workflows/ci.yml`
- Test: `tests/conftest.py` (fixture `cfg` = `cargar_config()` copiado con `copy.deepcopy`, con `fecha_inicio = 2026-11-16` y los `[precios]` de la spec §6 fijos para los tests), `tests/test_config.py` y `tests/test_reintentos.py`

**Interfaces:**
- Produce: `cargar_config(ruta: Path = Path("config.toml")) -> dict` (valida y devuelve el TOML como dict); `semana_de(cfg: dict, fecha: date) -> int`; `cupo(cfg: dict, red: str, fecha: date) -> int`; `secreto(nombre: str) -> str` (lanza `RuntimeError(f"Falta el secreto {nombre}")`); `reintentar(fn: Callable[[], T], esperas: tuple[int, ...] = (60, 300, 900), dormir=time.sleep) -> T`; `class ErrorPermanente(Exception)`.

- [ ] **Paso 1: Tests que fallan**

```python
def test_config_real_valida():
    cfg = cargar_config()
    assert abs(sum(cfg["mezcla"].values()) - 1.0) < 1e-9

def test_cupo_por_semana_y_tope_de_lista(cfg):           # fecha_inicio = 2026-11-16
    assert cupo(cfg, "youtube", date(2026, 11, 16)) == 3
    assert cupo(cfg, "youtube", date(2026, 11, 30)) == 6  # semana 2
    assert cupo(cfg, "youtube", date(2027, 6, 1)) == 10   # repite el último
    assert cupo(cfg, "youtube", date(2026, 11, 10)) == 0  # antes del inicio

def test_config_invalida(tmp_path):  # mezcla suma 0.9 / falta [tope] / red desconocida en cadencia
    ...  # cada caso: pytest.raises(ValueError, match=<clave>)

def test_reintentar_429_y_5xx_y_no_4xx():
    # 2 fallos con requests.HTTPError(status 503) y luego OK → devuelve OK, 3 llamadas
    # un 400 → se relanza al primer intento, como ErrorPermanente
    # 429 con Retry-After: 120 y esperas (0, 0, 0) → dormir recibe 120
```

- [ ] **Paso 2:** `uv run pytest tests/test_config.py tests/test_reintentos.py -q` → FAIL (no existe el módulo).
- [ ] **Paso 3: Implementar.** `cargar_config` revisa: la mezcla suma 1 (±1e-9); las listas de `cadencia` no están vacías y son ≥ 0; las redes ∈ {youtube, tiktok, instagram} en `cadencia`, `horarios` y `topes_red`; cada `horarios.<red>` tiene ≥ max(`cadencia.<red>`) entradas `HH:MM`; `tope.usd_dia > 0`. `cupo` = 0 antes de `fecha_inicio`. `reintentar` reintenta con `requests.ConnectionError`, `Timeout` y `HTTPError` 429/5xx; con otro 4xx lanza `ErrorPermanente`.
- [ ] **Paso 4:** `ci.yml`: en `push` y `pull_request`, `ubuntu-latest`, `astral-sh/setup-uv`, `sudo apt-get install -y ffmpeg`, `uv sync` y `uv run ruff check . && uv run pytest -q` (job `python`; el job `video` lo agrega T9).
- [ ] **Paso 5:** `uv run ruff check . && uv run pytest -q` → todo en verde. Sin commit.

---

### Tarea 2: Almacén R2 y presupuesto

**Archivos:**
- Crear: `nomos/almacen.py` y `nomos/gasto.py`
- Test: `tests/test_almacen.py` y `tests/test_gasto.py`; modificar `tests/conftest.py` (agregar `AlmacenFalso` en memoria y el fixture `almacen_falso`)

**Interfaces:**
- Produce: `class Almacen(cliente=None, bucket: str | None = None)` con `existe(clave) -> bool`, `leer_json(clave) -> dict | None`, `escribir_json(clave, datos: dict)`, `subir(clave, ruta: Path)`, `bajar(clave, ruta: Path)`, `url_firmada(clave, segundos=604800) -> str` y `listar(prefijo) -> list[str]`. Sin cliente, crea `boto3.client("s3", endpoint_url=f"https://{R2_ACCOUNT_ID}.r2.cloudflarestorage.com", ...)` con los secretos.
- Produce: `AlmacenFalso` en `tests/conftest.py`, con la misma interfaz sobre un dict (lo usan todas las tareas siguientes).
- Produce: `estimar_video(cfg, n_imagenes: int, n_clips: int, caracteres: int) -> float` (fórmula de la spec §8); `class Presupuesto(tope: float, gastado: float = 0.0, aviso_pct: int = 80)` con `cabe(usd) -> bool`, `cargar(usd)`, `restante -> float` y `en_aviso -> bool`; `gastado_del_lote(almacen, fecha: str) -> float` (suma `costos` de `lotes/{fecha}/*/estado.json`).

- [ ] **Paso 1: Tests que fallan**

```python
def test_presupuesto():
    p = Presupuesto(tope=1.0, gastado=0.7, aviso_pct=80)
    assert p.cabe(0.3) and not p.cabe(0.31) and not p.en_aviso
    p.cargar(0.15); assert p.en_aviso and round(p.restante, 2) == 0.15

def test_estimar_video(cfg):  # precios de la spec §6: 1 clip de 5 s, 8 imágenes, 650 caracteres
    assert estimar_video(cfg, 8, 1, 650) == pytest.approx(5*0.063 + 8*0.015 + 0.65*0.08 + 2*0.05)

def test_gastado_del_lote_suma_estados(almacen_falso):
    # dos estado.json con costos {claude: .03, higgsfield: .2} y {elevenlabs: .05} → 0.28
    # un estado.json de otra fecha no cuenta

def test_almacen_con_cliente_falso():
    # Almacen(cliente=<stub con put_object/get_object/head_object>, bucket="b"):
    # escribir_json + leer_json devuelve lo mismo; existe False ante un 404 (ClientError), True después
```

- [ ] **Paso 2:** ejecutar → FAIL.
- [ ] **Paso 3:** implementar. `leer_json` devuelve `None` si la clave no existe (y nunca traga otros errores).
- [ ] **Paso 4:** `uv run ruff check . && uv run pytest -q` → verde.

---

### Tarea 3: Banco de temas

**Archivos:** Crear `nomos/temas.py` y `datos/temas.csv` (solo el encabezado + 3 filas de ejemplo de los ganchos del framework) · Test: `tests/test_temas.py`

**Interfaces:**
- Consume: `cargar_config` (T1).
- Produce: `leer_temas(ruta) -> list[dict]`; `escribir_temas(ruta, temas)`; `normalizar(texto) -> str` (minúsculas, sin tildes vía `unicodedata`, sin signos ni «qué pasaría si»/«y si»); `es_duplicado(pregunta, previas: list[str], umbral=0.85) -> bool` (`difflib.SequenceMatcher` sobre el texto normalizado; `# ponytail: similitud léxica, embeddings si se cuelan paráfrasis`); `elegir(temas, previas, cfg, n) -> list[dict]`.

- [ ] **Paso 1: Tests que fallan**

```python
def test_duplicados():
    assert es_duplicado("¿Qué pasaría si la Luna desapareciera?", ["¿Y si la luna desapareciera?"])
    assert not es_duplicado("¿Qué pasaría si la Luna desapareciera?", ["¿Qué pasaría si el Sol se apagara?"])

def test_elegir_respeta_mezcla_y_formatos(cfg):
    # banco: 20 pendientes de cada subnicho, formatos repartidos; n=10
    # → cuotas por resto mayor de mezcla×10; empates por orden en config.toml
    #   → {espacio: 2, ciencia: 2, cuerpo: 2, peru_latam: 2, historia: 1, animales: 1}
    # → ningún formato > cfg["formatos"]["max_por_dia"]
    # → dentro de cada subnicho, primero los más antiguos (columna creado)

def test_elegir_rellena_si_falta_un_subnicho(cfg):   # 0 de animales → completa n con otros subnichos
def test_elegir_salta_duplicados_y_usados(cfg):      # un pendiente duplicado de una previa no sale; los 'usado' nunca salen
def test_elegir_menos_si_no_alcanza(cfg):            # 3 pendientes, n=5 → devuelve 3, sin error
```

- [ ] **Paso 2:** FAIL. **Paso 3:** implementar (sin cambiar estados: eso lo hace T10). **Paso 4:** ruff + pytest en verde.

---

### Tarea 4: Guionista

**Archivos:** Crear `nomos/guion.py`, `prompts/guionista.md` y `esquemas/guion.schema.json` · Test: `tests/test_guion.py`, `tests/fixtures/guion_valido.json`

**Interfaces:**
- Consume: `cargar_config`, `reintentar` (T1).
- Produce: `validar_guion(g: dict, cfg: dict) -> list[str]` (reglas exactas de la spec §5.4; un mensaje en español por regla rota); `escribir_guion(tema: dict, cfg: dict, cliente=None, errores_previos: list[str] | None = None, problemas: list[dict] | None = None) -> tuple[dict, float]` (guion, costo en USD calculado con `usage` × precios de Sonnet 5.5: 2 USD de entrada y 10 USD de salida por MTok).

- [ ] **Paso 1: Tests que fallan**

```python
def test_guion_valido(cfg): assert validar_guion(cargar("guion_valido.json"), cfg) == []

@pytest.mark.parametrize("cambio, error", [
    (lambda g: g["fuentes"].pop(), "fuentes"),
    (lambda g: g["fuentes"][1].update(url=g["fuentes"][0]["url"]), "dominio"),
    (lambda g: g["escenas"].reverse(), "gancho"),
    (lambda g: g["escenas"][1].update(narracion="palabra " * 150), "palabras"),
    (lambda g: g["escenas"][2].update(tipo="clip"), "clip"),          # con clips_por_video = 1
    (lambda g: g.update(pregunta="La Luna desaparece"), "pregunta"),
    (lambda g: g["escenas"][3].update(prompt_visual="Pikachu en Lima"), "prohibid"),
])
def test_guion_invalido(cfg, cambio, error): ...  # assert any(error in e for e in validar_guion(g, cfg))

def test_escribir_guion_envia_esquema_y_errores(cfg):
    # cliente falso que guarda los kwargs de messages.create y devuelve el texto de guion_valido.json
    # → kwargs["model"] == "claude-sonnet-5-5"; output_config.format.type == "json_schema"
    # → con errores_previos=["x"], el último mensaje de usuario contiene "x"
    # → stop_reason "refusal" → lanza ErrorPermanente
```

- [ ] **Paso 2:** FAIL.
- [ ] **Paso 3:** `prompts/guionista.md` resume el framework §1–§3 (roles, formatos, tono, fuentes, prohibiciones y salud). El tema, el formato y los errores previos van en el mensaje de usuario, no en el sistema, para no romper el caché. El esquema refleja la spec §5.4 con `additionalProperties: false` y `required` en todos los niveles.
- [ ] **Paso 4:** ruff + pytest en verde.

---

### Tarea 5: Verificador

**Archivos:** Crear `nomos/verificar.py` y `prompts/verificador.md` · Test: `tests/test_verificar.py`

**Interfaces:**
- Consume: `guion` (T4), `reintentar` (T1).
- Produce: `fuentes_vivas(urls: list[str], get=requests.get) -> list[str]` (devuelve las que **no** responden 200); `verificar(guion: dict, cfg: dict, cliente=None, get=requests.get) -> tuple[dict, float]` (el `verificacion.json` de la spec §5.5 y su costo).

- [ ] **Paso 1: Tests que fallan**

```python
def test_fuente_caida_rechaza_sin_llamar_a_claude(cfg):
    # get falso: 1 url → 404; cliente falso que falla si lo llaman
    v, costo = verificar(guion, cfg, cliente=explota, get=falso)
    assert v["veredicto"] == "rechazar" and costo == 0

def test_verificador_usa_web_fetch_y_parsea(cfg):
    # cliente falso devuelve {"veredicto":"corregir","problemas":[{"escena":4,...}],"fuentes_ok":[...]}
    # → tools contiene {"type": "web_fetch_20260209", "name": "web_fetch", "max_uses": 4}
    # → v["veredicto"] == "corregir"

def test_veredicto_desconocido_es_rechazar(cfg):  # "quizas" → "rechazar"
```

- [ ] **Paso 2:** FAIL. **Paso 3:** implementar según lo que el piloto (T0, paso 7) confirmó sobre combinar `web_fetch` con `output_config.format`. **Paso 4:** en verde.

---

### Tarea 6: Cliente de Higgsfield

**Archivos:** Crear `nomos/higgsfield.py` · Test: `tests/test_higgsfield.py` (usa los fixtures de T0)

**Interfaces:**
- Consume: `reintentar`, `ErrorPermanente`, `secreto` (T1) y `Almacen` (T2).
- Produce:
  - `enviar(ruta_modelo: str, cuerpo: dict, clave_idem: str, post=requests.post) -> str`: devuelve el `request_id`. Cabeceras `Authorization: Key {ID}:{SECRET}` e `Idempotency-Key`.
  - `esperar(request_id: str, get=requests.get, dormir=time.sleep, max_min=20) -> dict`: devuelve `{"estado": "completed|failed|nsfw", "url": str | None, "costo": float | None}`.
  - `generar(almacen, est: dict, clave_est: str, paso: str, ruta_modelo, cuerpo, destino: str, ...) -> float`: si `destino` existe en R2 → 0. Si `est["higgsfield_pendientes"][paso]` existe, espera **ese** id. Si no, envía, guarda el id en `est` y escribe `clave_est` en R2 **antes** de esperar. Al quedar `completed`, baja el archivo, lo sube a `destino`, borra el pendiente y devuelve el costo (el informado o, si falta, la estimación). Con `failed`/`nsfw` lanza `FalloGeneracion(estado)`.
  - `Trabajo` (dataclass: `clave_est`, `paso`, `ruta_modelo`, `cuerpo`, `destino`, `estimado`) y `generar_tanda(almacen, trabajos: list[Trabajo], ...) -> dict[str, float | FalloGeneracion]` (clave = `destino`). Primero envía **todos** los que no tienen destino ni pendiente y guarda cada id; después consulta en ronda hasta que todos terminen. `generar` = `generar_tanda` con un solo trabajo.

- [ ] **Paso 1: Tests que fallan**

```python
def test_reanuda_request_pendiente(almacen_falso):
    est = {"higgsfield_pendientes": {"clip-1": "req_viejo"}, "costos": {}}
    # post falso que falla si lo llaman; get falso devuelve status completed (fixture) para req_viejo
    # → no hubo POST, destino existe en R2, "clip-1" ya no está en pendientes

def test_guarda_request_id_antes_de_esperar(almacen_falso):
    # get falso lanza KeyboardInterrupt en la primera consulta
    # → después, almacen.leer_json(clave_est)["higgsfield_pendientes"]["img-1"] == id del fixture submit

def test_salta_si_destino_existe(almacen_falso): ...   # costo 0, sin POST ni GET
def test_nsfw_lanza(almacen_falso): ...                 # FalloGeneracion("nsfw")
def test_idempotency_key(almacen_falso): ...           # cabecera == clave_idem recibida
def test_tanda_envia_todo_antes_de_esperar(almacen_falso): ...  # 3 trabajos → 3 POST antes del primer GET; uno nsfw no frena a los otros
```

- [ ] **Paso 2:** FAIL. **Paso 3:** implementar con las rutas y los campos del documento del piloto (escribirlos tal cual; si una ruta no está en el documento, parar y avisar). **Paso 4:** en verde.

---

### Tarea 7: Voz y captions

**Archivos:** Crear `nomos/voz.py` · Test: `tests/test_voz.py` (fixture `elevenlabs_timestamps.json`)

**Interfaces:**
- Consume: `reintentar`, `secreto` (T1).
- Produce: `captions_desde_alineacion(al: dict) -> list[dict]`: devuelve objetos `Caption` de `@remotion/captions` (`text`, `startMs`, `endMs`, `timestampMs` y `confidence: None`), uno por palabra; desde el segundo, `text` lleva el espacio inicial y los ms son enteros. También `sintetizar(texto: str, cfg: dict, post=requests.post) -> tuple[bytes, dict, float]`: el mp3, el `voz.json` de la spec §5.6 y el costo = `len(texto)/1000 × precios.voz_por_1000_caracteres`.

- [ ] **Paso 1: Tests que fallan**

```python
def test_captions_desde_fixture():
    c = captions_desde_alineacion(cargar("elevenlabs_timestamps.json")["alignment"])
    assert c[0]["text"].startswith("¿") and c[0]["startMs"] == 0
    assert all(x["text"].startswith(" ") for x in c[1:])
    assert all(a["endMs"] <= b["startMs"] for a, b in zip(c, c[1:]))
    assert "".join(x["text"] for x in c) == texto_original_normalizado
    assert all(isinstance(x["startMs"], int) and x["timestampMs"] == (x["startMs"] + x["endMs"]) // 2 for x in c)

def test_espacios_dobles_y_salto_de_linea(): ...  # no generan captions vacíos
def test_sintetizar_usa_with_timestamps(cfg): ... # URL termina en /with-timestamps; decodifica audio_base64; duracion = último endMs/1000
```

- [ ] **Paso 2:** FAIL. **Paso 3:** implementar. **Paso 4:** ruff + pytest en verde.

---

### Tarea 8: Diseño de las plantillas visuales (`disenador-ui`, sin código de producto)

**Quién:** el agente `disenador-ui` en Claude Code, con el plugin de Remotion (`/remotion:remotion-create` y las Remotion Agent Skills) y la skill `ui-ux-pro-max`. Mauricio aprueba. Puede correr en paralelo con T1–T7.

**Archivos:** Crear `docs/framework/plantillas.md`.

- [ ] **Paso 1:** brief de cada una de las 3 plantillas (`Cinetica`, `Documental` y `Pizarra`; los nombres son propuestos): tipografía (OFL, con el archivo exacto), paleta (claro sobre video, con contraste ≥ 4,5:1 del subtítulo sobre un fondo cualquiera gracias a un contorno o una caja), estilo de los subtítulos (tamaño, palabra activa, máximo de 2 líneas, `combineTokensWithinMilliseconds`), entrada y salida de escenas, el gancho de 0 a 2 s, el rótulo de IA y la barra de progreso.
- [ ] **Paso 2:** con el plugin, un boceto de cada plantilla en un proyecto Remotion de prueba, **fuera del repo** (un directorio temporal). Capturas de 3 fotogramas (0,5 s, 15 s y 40 s) en el brief.
- [ ] **Paso 3:** comprobar las reglas de la spec §11.1: sin blur, sin sombras grandes y sin WebGL; texto lejos de los bordes (los 250 px de abajo y los 120 px de la derecha quedan libres por los botones de las apps).

**Aceptación:** `plantillas.md` con 3 briefs, 9 capturas y el visto bueno de Mauricio. Codex programa T9 y T10 a partir de este documento.

---

### Tarea 9: Proyecto Remotion, componentes comunes y plantilla `Cinetica`

**Archivos:**
- Crear: `video/package.json` (dependencias de la spec §3 con versión exacta, Node 22), `video/package-lock.json`, `video/tsconfig.json`, `video/remotion.config.ts`, `video/src/index.ts`, `video/src/Root.tsx`, `video/src/props.ts`, `video/src/comunes/{Escena,Subtitulos,Gancho,Rotulo,BarraProgreso,Musica}.tsx`, `video/src/plantillas/Cinetica.tsx`, `video/public/fuentes/` (las TTF del brief + `OFL.txt`), `video/public/musica/licencias.csv` (encabezado `archivo,fuente,licencia,url`), `video/public/ejemplo/` (2 PNG, 1 clip de 2 s y 1 mp3 de 32 s, sintéticos y de pocos KB) y `video/props-ejemplo.json`
- Modificar: `.github/workflows/ci.yml` (job `video`); las entradas de `video/` en `.gitignore` ya las dejó T1
- Test: `video/src/props.test.ts` (con `node --test` y `tsx`, sin framework extra) y el render de prueba en CI

**Interfaces:**
- Consume: el contrato `props.json` de la spec §5.10 y el brief de T8.
- Produce: `PropsVideo` (zod) en `props.ts`; una `<Composition id="Cinetica">` de 1080×1920 a 30 fps con `calculateMetadata` → `durationInFrames = Math.ceil(duracion_s * 30)`; y los componentes comunes, que reciben solo lo que usan y el estilo por props, para que T10 los reutilice.

- [ ] **Paso 1: Test que falla** (`props.test.ts`): `PropsVideo.parse(props-ejemplo.json)` pasa; sin `rotulo` lanza; con `escenas` vacías lanza; con `plantilla` desconocida lanza.
- [ ] **Paso 2:** `cd video && npx tsx --test src/props.test.ts` → FAIL.
- [ ] **Paso 3: Implementar** según la spec §11.1 y el brief: `Escena` con imagen y `interpolate` de escala o posición según `movimiento`, o clip con `<Video>` de `@remotion/media` que congela el último fotograma si es corto; `Subtitulos` con `createTikTokStyleCaptions` y `white-space: pre`; `Musica` con `<Audio volume={volumen_musica}>`. Las rutas pasan por `staticFile`.
- [ ] **Paso 4: Verificar**

```
cd video && npm ci && npx tsc --noEmit && npx tsx --test src/props.test.ts
npx remotion render src/index.ts Cinetica out/prueba.mp4 --props=props-ejemplo.json
ffprobe -v error -show_entries stream=width,height,codec_name -show_entries format=duration out/prueba.mp4
```
Esperado: tipos en verde y test en verde. `ffprobe` debe dar `1080`, `1920`, `h264` y `aac`, y una duración de 32,3 ± 0,1 s. Además, anotar en el reporte de la tarea el tiempo de render medido en local.
- [ ] **Paso 5:** job `video` en `ci.yml`: `actions/setup-node` (22), `npm ci`, `npx tsc --noEmit`, el test y `npx remotion render ... --frames=0-89` (3 s, para que el CI sea rápido).
- [ ] **Paso 6:** `disenador-ui` compara el render con el brief (las capturas de los mismos 3 fotogramas) antes del `revisor`.

---

### Tarea 10: Plantillas `Documental` y `Pizarra`

**Archivos:** Crear `video/src/plantillas/Documental.tsx` y `video/src/plantillas/Pizarra.tsx`; modificar `video/src/Root.tsx` (registrarlas) y `video/src/props.ts` (enum de `plantilla`) · Test: el mismo de T9, con dos casos más

**Interfaces:**
- Consume: los componentes comunes de T9 (sin duplicar la lógica; solo cambia el estilo y la composición).
- Produce: `<Composition id="Documental">` y `<Composition id="Pizarra">`, con el mismo `PropsVideo`.

- [ ] **Paso 1:** en `props.test.ts`, `plantilla: "Documental"` y `"Pizarra"` pasan la validación → FAIL antes de tocar el enum.
- [ ] **Paso 2:** implementar las dos plantillas según el brief de T8.
- [ ] **Paso 3:** renderizar `props-ejemplo.json` con las 3 plantillas (`--frames=0-89`) y comprobar con `ffprobe` como en T9.
- [ ] **Paso 4:** `disenador-ui` compara con el brief; `npx tsc --noEmit` y los tests en verde.

T10 puede ir en paralelo con T11 (no comparten archivos).

---

### Tarea 11: Render desde Python

**Archivos:** Crear `nomos/render.py` · Test: `tests/test_render.py`

**Interfaces:**
- Consume: `Almacen` (T2), el `voz.json` de T7 y el contrato de la spec §5.10.
- Produce:
  - `tiempos_escenas(escenas: list[dict], captions: list[dict], duracion_s: float) -> list[tuple[float, float]]`: la regla de la spec §5.10. Para cada escena se cuentan sus palabras con `len(narracion.split())`.
  - `elegir_plantilla(cfg, indice: int, fecha: date, ya_usadas: dict[str, int]) -> str` y `elegir_musica(dir: Path, semilla: str) -> str | None` (determinista).
  - `armar_props(guion, voz_json, medios: dict[int, tuple[str, str]], cfg, video_id, plantilla, musica) -> dict`: `medios` va de `índice de escena` a `(tipo, ruta relativa a video/public)`.
  - `empaquetar(dir_video: Path, correr=subprocess.run) -> Path` (`npx remotion bundle`, una vez por corrida).
  - `renderizar(bundle: Path, props: dict, salida: Path, cfg, frames: str | None = None, correr=subprocess.run) -> float`: escribe `props.json`, ejecuta `npx remotion render <bundle> <plantilla> <salida> --props=<props.json> --concurrency=<cfg>`, reintenta 1 vez si sale ≠ 0, valida con `ffprobe` y devuelve la duración. Si falla, lanza `FalloRender(mensaje con el stderr recortado a 2000)`.
  - `preparar_medios(almacen, video_id, est, dir_publico: Path) -> dict` y `limpiar(dir_publico, video_id)`.

- [ ] **Paso 1: Tests que fallan**

```python
def test_tiempos_escenas_continuos():
    t = tiempos_escenas(escenas, captions, duracion_s=44.5)
    assert t[0][0] == 0 and t[-1][1] == 44.5
    assert all(a[1] == b[0] for a, b in zip(t, t[1:]))

def test_props_cumplen_contrato(cfg):
    p = armar_props(guion, voz, medios, cfg, "2026-11-16-v01", "Cinetica", "musica/a.mp3")
    assert p["rotulo"] == cfg["etiquetas"]["rotulo"] and p["duracion_s"] == pytest.approx(voz["duracion"] + 0.3)
    assert [e["tipo"] for e in p["escenas"]][0] == "clip" and p["escenas"][1]["movimiento"] != p["escenas"][2]["movimiento"]
    assert set(p) == {"plantilla","duracion_s","pregunta","texto_gancho","rotulo","escenas","voz","musica","volumen_musica","captions","semilla"}

def test_plantillas_rotan_y_respetan_maximo(cfg): ...  # 11 videos → ninguna plantilla > max_por_dia_plantilla; mismo día y mismo índice → misma plantilla
def test_renderizar_reintenta_una_vez(cfg, tmp_path): ...  # correr falso: 1.º ≠ 0, 2.º ok → 2 llamadas; dos fallos → FalloRender
def test_renderizar_rechaza_salida_invalida(cfg, tmp_path): ...  # ffprobe falso con 1920x1080 o 25 s → FalloRender
```

- [ ] **Paso 2:** FAIL. **Paso 3:** implementar.
- [ ] **Paso 4: Prueba real** (se salta si no hay `npx` o `ffprobe`): `test_render_real_con_ejemplo` usa los medios de `video/public/ejemplo/`, renderiza `--frames=0-89` y comprueba 1080×1920.
- [ ] **Paso 5:** `uv run ruff check . && uv run pytest -q` en verde.

---

### Tarea 12: Orquestación de `generar`, issue de revisión y workflow

**Archivos:** Crear `nomos/revision.py` (solo el texto del issue y los avisos), `nomos/cli.py` y `.github/workflows/generar.yml` · Test: `tests/test_generar.py` y `tests/test_revision_texto.py`

**Interfaces:**
- Consume: todo T1–T11 (salvo T8, que es diseño).
- Produce:
  - `generar_lote(fecha: date, cfg: dict, almacen, prov: Proveedores, gh: GitHub, raiz: Path = Path(".")) -> dict` (devuelve `lote.json`). `Proveedores` es un `dataclass` con las funciones inyectables: `guion=escribir_guion`, `verificar=verificar`, `higgsfield=generar`, `voz=sintetizar`, `empaquetar=empaquetar` y `renderizar=renderizar`. `higgsfield` es `generar_tanda` (T6).
  - `texto_issue(fecha, videos: list[dict], cfg) -> str` (spec §5.8, con las casillas por defecto).
  - `class GitHub` con `crear_issue(titulo, cuerpo, etiquetas) -> int`, `editar_issue(n, cuerpo)`, `comentar(n, texto)` y `buscar_issue_abierto(titulo) -> int | None`. Usa la API REST con `GITHUB_TOKEN` y `GITHUB_REPOSITORY`; en los tests, un falso.
  - `avisar(gh, motivo: str, fecha: str, texto: str)`: abre un issue `alerta` titulado `Alerta {fecha} · {motivo}`, sin duplicados.
  - CLI: `python -m nomos.cli generar [--fecha AAAA-MM-DD] [--max N]` (por defecto, mañana en Lima; `--max` limita el lote para la primera corrida).

- [ ] **Paso 1: Tests que fallan** (todo con `AlmacenFalso`, proveedores falsos que cuentan llamadas y `GitHub` falso)

```python
def test_lote_completo_crea_issue_e_historial(cfg, tmp_path):
    lote = generar_lote(date(2026,11,16), cfg, alm, prov, gh, raiz=tmp_path)
    assert len(lote["videos"]) == 3 + cfg["produccion"]["extra_por_lote"]
    assert gh.issues[0]["etiquetas"] == ["lote"] and "- [x] v01 youtube" in gh.issues[0]["cuerpo"]
    # historial.csv: 4 filas etapa=listo; temas.csv: esos 4 temas en 'usado'

def test_reejecucion_no_paga_nada(cfg, tmp_path):
    generar_lote(...); prov.reiniciar_contadores()
    generar_lote(...)
    assert prov.llamadas_pagas == 0 and len(gh.issues) == 1       # foco 1

def test_caida_a_mitad_retoma(cfg, tmp_path):
    # prov.voz lanza en el video 2 → la excepción se propaga; segunda corrida: v01 sin llamadas, v02 sigue desde voz

def test_tope_nunca_se_pasa(cfg, tmp_path):                         # foco 2
    cfg["tope"]["usd_dia"] = 1.0  # precios de prueba: un video ≈ 0.6
    lote = generar_lote(...)
    assert gastado_del_lote(alm, "2026-11-16") <= 1.0
    assert any("tope" in a["titulo"] for a in gh.alertas)
    # los temas sin usar siguen 'pendiente'

def test_clip_sin_presupuesto_o_fallido_pasa_a_imagen(cfg, tmp_path): ...
def test_tema_rechazado_toma_el_siguiente_hasta_n_mas_3(cfg, tmp_path): ...  # el verificador rechaza siempre → 7 temas probados, 0 videos, alerta
def test_guion_invalido_dos_veces_descarta(cfg, tmp_path): ...
def test_banco_bajo_avisa(cfg, tmp_path): ...                         # < 14 pendientes → alerta
def test_tandas_higgsfield(cfg, tmp_path): ...                         # todas las imágenes del lote se envían antes de la primera espera; los clips, después de las imágenes
def test_bundle_una_vez(cfg, tmp_path): ...                            # 4 videos → empaquetar 1 vez, renderizar 4
def test_reserva_libera_al_fallar(cfg, tmp_path): ...                  # un video fallido en el render deja libre su reserva no gastada para el siguiente tema
```

- [ ] **Paso 2:** FAIL.
- [ ] **Paso 3:** implementar `generar_lote` **por etapas sobre todo el lote** (spec §5.1 y §3), escribiendo `estado.json` después de cada llamada paga y aplicando las reglas de §8–§10. La clave de cada artefacto es `lotes/{fecha}/{video_id}/{nombre}`.
- [ ] **Paso 4:** `generar.yml`: `schedule: cron "0 18 * * *"` + `workflow_dispatch` (inputs `fecha` y `max`), `concurrency: {group: nomos, cancel-in-progress: false}`, `timeout-minutes: 150`, `permissions: {contents: write, issues: write}`, secretos de la spec §7 como `env`, `apt-get install ffmpeg` (por `ffprobe`), `actions/setup-node` 22 con caché de npm, `cd video && npm ci && npx remotion browser ensure` y, al final, `git add datos/ && git commit -m "datos: lote $FECHA" && git push` solo si hay cambios (`git diff --cached --quiet || ...`), **también si el paso anterior falla** (`if: always()`).
- [ ] **Paso 5:** `uv run ruff check . && uv run pytest -q` → verde; `uv run python -m nomos.cli generar --help` sale con código 0.

---

### Tarea 13: Publicar en Upload-Post y workflow

**Archivos:** Crear `nomos/publicar.py`, modificar `nomos/revision.py` (lectura de casillas) y `nomos/cli.py` (subcomando `publicar`), y crear `.github/workflows/publicar.yml` · Test: `tests/test_publicar.py`

**Interfaces:**
- Consume: `Almacen`, `cargar_config`, `cupo` y `reintentar` (T1–T2); `GitHub` y `avisar` (T12).
- Produce:
  - `leer_casillas(cuerpo: str) -> list[tuple[str, str]]`: los pares (`vNN`, red) marcados, en orden de aparición, con la regex de la spec §5.8.
  - `asignar_horarios(marcados, fecha: date, cfg, ahora: datetime, ya_programados: dict[str, int]) -> list[tuple[str, str, datetime]]`.
  - `programar(video_id, red, cuando: datetime, guion: dict, url_o_ruta, cfg, post=requests.post) -> dict`: el cuerpo de Upload-Post con los campos del documento del piloto, las banderas de IA y `Idempotency-Key: {video_id}:{red}` si el piloto lo confirmó.
  - `publicar_lote(numero_issue: int, cfg, almacen, gh, ahora=None, post=requests.post, raiz=Path(".")) -> dict`.
  - CLI: `python -m nomos.cli publicar --issue N`.

- [ ] **Paso 1: Tests que fallan**

```python
def test_leer_casillas_tolerante():                                     # foco 5
    cuerpo = "- [x] v01 youtube\n* [X] v02 tiktok  \n- [ ] v03 youtube\ntexto libre\n- [x] v99 facebook"
    assert leer_casillas(cuerpo) == [("v01","youtube"), ("v02","tiktok")]

def test_horarios_en_lima_y_corrimiento(cfg):                          # foco 4
    ahora = datetime(2026,11,16,8,0,tzinfo=LIMA)
    r = asignar_horarios([("v01","youtube"),("v02","youtube"),("v03","youtube")], date(2026,11,16), cfg, ahora, {})
    assert r[0][2] == datetime(2026,11,16,8,15,tzinfo=LIMA)          # las 07:00 ya pasaron → ahora + 15
    assert r[1][2].isoformat() == "2026-11-16T09:30:00-05:00"
    assert all(x[2] > ahora for x in r)

def test_respeta_topes_red(cfg): ...      # tiktok con 14 ya programados y 3 marcados → programa 1 + aviso
def test_doble_aprobado_no_duplica(cfg):                               # foco 3
    publicar_lote(...); post.llamadas.clear(); publicar_lote(...)
    assert post.llamadas == []
def test_banderas_ia_siempre(cfg): ...    # los 3 cuerpos llevan sus banderas de IA en verdadero
def test_401_avisa_y_sigue(cfg): ...      # tiktok 401 → error en estado/historial + alerta; youtube igual se programa
```

- [ ] **Paso 2:** FAIL.
- [ ] **Paso 3:** implementar según la spec §5.2. El video se envía como URL firmada o como archivo, según el piloto.
- [ ] **Paso 4:** `publicar.yml`: `on: issues: types: [labeled]`, `if: github.event.label.name == 'aprobado' && contains(github.event.issue.labels.*.name, 'lote')`, la misma `concurrency` que `generar`, `permissions: {contents: write, issues: write}` y el commit de `datos/` como en T12. También tiene `workflow_dispatch` con el input `issue`.
- [ ] **Paso 5:** ruff + pytest en verde.

---

### Tarea 14: Primera corrida real (Mauricio + Claude, sin código nuevo)

**Precondiciones:** T0 dio «go» y T1–T13 están aprobadas e integradas; el repo privado está en GitHub con los secretos cargados (`devops`), con ≥ 30 temas en `datos/temas.csv` (`investigador` + Mauricio), con ≥ 3 pistas con licencia en `video/public/musica/`, con la decisión de licencia de Remotion tomada (spec §11.4), y con la cuenta de Upload-Post conectada.

- [ ] **Paso 1:** `gh workflow run generar.yml -f fecha=<mañana> -f max=1`. Esperado: un issue `Lote …` con 1 video; en `estado.json`, `etapa=listo`.
- [ ] **Paso 2:** Mauricio revisa con el checklist del framework §4.2 y pone `aprobado`. Esperado: el workflow `publicar` en verde y el comentario con las 3 horas programadas.
- [ ] **Paso 3:** a la hora programada, comprobar en las 3 apps que el video es público, que lleva la etiqueta de IA y que tiene el rótulo y los subtítulos legibles.
- [ ] **Paso 4:** reejecutar `generar` con la misma fecha → 0 llamadas pagas en el log y gasto sin cambios.
- [ ] **Paso 4b:** anotar los minutos del run por etapa (log) y compararlos con la tabla de la spec §11.3; si el render pasa de 5 min por video, aplicar la alternativa 1 o 2 de §11.3 antes de subir la cadencia.
- [ ] **Paso 5:** contrastar el costo de `estado.json` con los paneles de Higgsfield, ElevenLabs y Anthropic (error < 5 %); si no cuadra, se corrige `[precios]`.
- [ ] **Paso 6:** encender el cron con 3 videos por día desde `fecha_inicio`.

**Aceptación:** los 6 pasos con evidencia (enlaces a las tres publicaciones y al run) en `docs/piloto/2026-11-resultados.md`, sección «Primera corrida».

---

### Tarea 15 (opcional, desde la semana 3): Informe semanal

**Archivos:** Crear `nomos/semana.py` y el subcomando `semana` en `cli.py` · Test: `tests/test_semana.py`

**Interfaces:**
- Produce: `informe(metricas: list[dict], historial: list[dict], semana: str, previas: list[str]) -> str` (markdown para `docs/semanas/{semana}.md`). Tiene que incluir: el ranking por `vistos_vs_deslizados` y por compartidos cada 1000 vistas; el 20 % superior marcado «replicar»; y los subnichos y formatos que llevan **2 semanas seguidas** en el 30 % inferior marcados «mitad de cupo», o «apagar» si son 3 (según los informes `previas`).

- [ ] **Paso 1: Tests que fallan:** 10 videos sintéticos → los 2 mejores salen como «replicar»; un subnicho en el 30 % inferior en esta semana y en la anterior → «mitad de cupo»; con < 5 videos en la semana, el informe dice «muestra insuficiente» y no recomienda cortes.
- [ ] **Paso 2:** FAIL. **Paso 3:** implementar. **Paso 4:** en verde.

Hasta que exista, el agente `marketing` aplica a mano las reglas del framework §6.
