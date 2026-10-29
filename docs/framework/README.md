---
proyecto: Videos-Diario-NomosTin
tema: Framework del canal «¿Qué pasaría si…?»
fecha: 2026-10-28
autor: arquitecto (Claude)
estado: propuesto. Nada de esto lo aprobó Mauricio todavía.
---

# Framework del canal «¿Qué pasaría si…?»

Es el manual de operación del canal: qué se publica, cómo se escribe, qué se revisa, cuánto se publica y cómo se aprende cada semana. El sistema que lo ejecuta está en la [spec](../superpowers/specs/2026-10-28-pipeline-videos-spec.md) y el flujo de punta a punta, en el [diagrama](workflow.html). Las cifras y fuentes vienen de las tres investigaciones de `docs/investigacion/` (2026-10-28).

> **Regla de lectura:** «propuesto» = valor por defecto hasta que Mauricio decida. Todo lo ajustable vive en `config.toml`; cambiar el canal no exige tocar código.

## 1. Pilares de contenido

### 1.1 Subnichos y mezcla inicial (propuesto)

| Subnicho (`config.mezcla`) | % del lote | Riesgo | Regla propia |
|---|---|---|---|
| `espacio` | 15 % | Bajo | Se puede ser visual y fantasioso; se etiqueta igual |
| `ciencia` (física y Tierra) | 15 % | Bajo | Fuentes: NASA, ESA, revistas de divulgación |
| `cuerpo` (cuerpo humano) | 25 % | Alto | Narrador en off; **nunca** un «doctor» generado. Sin consejos ni dosis. Cierra con «consulta a un médico» cuando aplica |
| `peru_latam` | 20 % | Medio-alto | Es el diferencial. Para desastres en ciudades reales: datos de IGP, INDECI y SENAMHI, tono informativo, cierre con prevención y sin imágenes hechas para asustar |
| `historia` (historia alternativa) | 15 % | Medio | Se dice en voz que es especulación («nadie lo sabe, pero…») |
| `animales` | 10 % | Bajo | Sin animales sufriendo |

Desde la semana 3, la mezcla se ajusta con el ciclo semanal (§6).

### 1.2 Fuera del canal (propuesto)

Política real, finanzas, tragedias recientes con víctimas, caras de personas reales, logos, personajes con derechos (Pikachu, Marvel) y clips de terceros.

### 1.3 Formatos (variedad contra el «contenido inauténtico» de YouTube)

Ningún formato pasa de 4 videos por día (`formatos.max_por_dia`).

| Formato | Estructura de la escalada |
|---|---|
| `escalada` | 3 o 4 pasos, cada uno peor o más raro que el anterior |
| `linea_de_tiempo` | «En 1 segundo… en 1 día… en 1 año… en 1 millón de años» |
| `mito_vs_dato` | Lo que la gente cree, contra lo que dice la ciencia, paso a paso |
| `tu_vs` | En segunda persona: «tú, en ese mundo», qué te pasa a ti |

### 1.4 Plantillas visuales (Remotion)

El video se arma en **Remotion**: composiciones React parametrizadas por el JSON del guion (spec §5.10 y §11). Habrá 3 plantillas visuales en la fase 1 (propuesto): `Cinetica` (cortes rápidos y subtítulo grande con rebote), `Documental` (zoom lento, tipografía serif y barra de progreso fina) y `Pizarra` (fondo de papel, flechas y anotaciones). Cada una cambia la tipografía, la paleta, el estilo de los subtítulos karaoke, el gancho y las transiciones. Las tres comparten el rótulo «Escenario hipotético · Hecho con IA», la barra de progreso y el texto del gancho de 0 a 2 s.

- Rotan en el lote y ninguna pasa de 4 videos por día. Formato narrativo (§1.3) × plantilla visual (§1.4) = 12 combinaciones, la defensa principal contra la política de YouTube sobre el contenido de plantilla.
- Las diseña `disenador-ui` con el plugin de Remotion para Claude Code (`/remotion:remotion-create`) y deja un brief por plantilla en `docs/framework/plantillas.md`; Codex las programa.
- Regla para el render: nada de desenfoques, sombras grandes ni WebGL, porque el runner no tiene GPU (spec §11.1).
- Una plantilla nueva entra cuando el ciclo semanal (§6) apaga otra o cuando las tres llevan 4 semanas.

## 2. Plantilla de guion (45 s ± 10)

La versión completa está en la estrategia, §2.2; Claude la recibe en `prompts/guionista.md`. Cada fila es un `rol` del JSON del guion (spec §5.4).

| Tiempo | Rol | Qué lleva |
|---|---|---|
| 0–2 s | `gancho` | La pregunta en voz y en pantalla (≤ 7 palabras) con la imagen más fuerte. Es la única escena con clip generado. Sin logos ni saludos |
| 2–6 s | `apuesta` | Una consecuencia sorprendente en una frase: «En {tiempo}, {resultado}» |
| 6–30 s | `escalada` (×3–4) | Según el formato (§1.3). Un plano por paso |
| 30–38 s | `giro` | El dato real con fuente: «En realidad, …» |
| 38–42 s | `cta` | Una sola: «Comenta qué pregunta hacemos mañana» |
| 42–45 s | `cierre` | Una frase que termina en la pregunta inicial, para que el video empalme en loop |

En total son de 6 a 10 escenas y de 75 a 140 palabras. La voz es fija, rápida y con tono de amigo; el español es neutro LATAM, sin modismos en la voz pero con temas peruanos.

## 3. Textos por red (propuesto)

| | YouTube Shorts | TikTok | Instagram Reels |
|---|---|---|---|
| Título o primera línea | La pregunta, ≤ 60 caracteres | La pregunta | La pregunta |
| Descripción | 1-2 frases + fuentes + «Contenido hecho con IA» | 1 frase + «¿Tú qué crees?» | 1-2 frases + una pregunta; las fuentes van en el primer comentario |
| Hashtags | 2-3 (`#shorts #ciencia #quepasariasi`) | 3-5 | 3-5, del tema |
| Etiqueta de IA | `containsSyntheticMedia` | `is_aigc` | `is_ai_generated` |

**Todos** los videos van etiquetados como IA en las tres redes, aunque sean fantasiosos, y llevan el rótulo «Escenario hipotético · Hecho con IA» en pantalla.

## 4. Reglas de calidad y veracidad

### 4.1 Lo que revisa la máquina (bloquea)

1. El guion tiene que cumplir el esquema y las reglas (spec §5.4); si no, se descarta.
2. Al menos 2 fuentes `https` de dominios distintos que respondan 200.
3. El verificador (Claude con `web_fetch`) lee las fuentes y aprueba, pide corregir una vez o rechaza.
4. La duración final queda entre 30 y 60 s y el audio no se corta.
5. Los prompts no llevan términos de la lista `prohibidas`.

### 4.2 Lo que revisa la persona, en el issue del lote (fase 1: el 100 %)

Para cada video, en unos 60 a 90 s:

- [ ] Los números y afirmaciones coinciden con las fuentes. Lo especulativo se dice como especulación.
- [ ] La imagen no miente: escalas, mapas, banderas y lugares reales.
- [ ] No hay caras reales, logos ni texto deforme generado por el modelo.
- [ ] Si es de salud, no da consejos; si es un desastre, tiene tono informativo.
- [ ] Se lee el rótulo de IA y los subtítulos no tapan nada.
- [ ] El video no es casi igual a otro ya publicado.

Si falla algo, se desmarcan sus casillas: el video no se publica y el motivo se anota en un comentario. Los rechazos sirven para afinar los prompts.

### 4.3 Protocolo ante un error ya publicado

Se fija un comentario con la corrección en las tres redes. Si el error es grave (salud, desastre o un dato falso central), se borra el video. Se registra en `historial.csv` como `etapa=retirado` y se corrige el prompt. Meta: **0 errores graves publicados**.

## 5. Cadencia por red

### 5.1 Rampa (propuesta; `config.cadencia`)

| Red | Semanas 1-2 | Semanas 3-4 | Semana 5 en adelante | Qué se publica |
|---|---|---|---|---|
| YouTube Shorts | 3/día | 6/día | hasta 10/día | Todo lo aprobado |
| TikTok | 2/día | 3/día | 3-4/día | Los mejores del lote; en la fase 1 los elige el revisor |
| Instagram Reels | 1/día | 2/día | 2-3/día | Lo mejor; probar ganchos con Trial Reels |

Se produce 1 video extra por lote, de reserva para el revisor.

### 5.2 Puertas para subir de escalón

Se miran cada quincena y hay que cumplir las tres:

1. Cero errores científicos detectados después de publicar.
2. «Vistos frente a deslizados» no baja más de 10 % respecto a la quincena anterior.
3. La tasa de rechazo del revisor es menor al 20 % y se revisó todo con el checklist.

Si falla una, se queda en el escalón y se corrige. **Revisión al día 60:** si las vistas y los «vistos frente a deslizados» quedan por debajo del rango bajo de las metas (estrategia, §6), se para la rampa antes de seguir pagando.

### 5.3 Horarios (estimación inicial, hora de Lima; `config.horarios`)

YouTube: 07:00, 09:30, 12:00, 13:30, 15:00, 17:00, 18:30, 20:00, 21:30 y 23:00 (se usan los primeros n). TikTok: 13:30, 18:30, 20:00 y 21:30. Instagram: 12:00, 20:00 y 21:30. Las dos franjas peores se mueven cada quincena.

## 6. Ciclo semanal (cada lunes)

Entran los videos con 72 h o más publicados.

1. **Medir.** Las métricas de la semana van a `datos/metricas.csv`, por exportación o por API (lo decide el piloto). Columnas: vistas, % visto, vistos frente a deslizados, compartidos, comentarios y seguidores.
2. **Ordenar** por «vistos frente a deslizados» y por compartidos cada 1000 vistas.
3. **Replicar.** El 20 % superior genera 2 o 3 temas nuevos en el banco (`origen=replica`): mismo subnicho y otra pregunta, o la misma pregunta con otro gancho.
4. **Cortar.** Un subnicho o formato que pasa 2 semanas seguidas en el 30 % inferior baja a la mitad de su cuota; con 3 semanas, se apaga.
5. **Ajustar** `config.mezcla`, los horarios y la cadencia (puertas, §5.2).
6. **Llenar el banco** hasta tener al menos 3 semanas de temas: ideas de los comentarios, réplicas y la investigación nueva.
7. **Registrar** las decisiones en `docs/semanas/AAAA-Wnn.md` (una página: qué se replicó, qué se cortó y por qué).

Se cambia una variable a la vez (duración, gancho, voz u horario). Una diferencia tiene que repetirse dos semanas antes de cambiar la regla. Con pocas vistas por video, no se decide por un solo video.

## 7. Quién hace qué

| Rol | Quién | Qué hace | Cuándo |
|---|---|---|---|
| Dueño | **Mauricio** | Decide el nombre, el idioma, el presupuesto y la mezcla; aprueba el lote en el issue (fase 1); responde los comentarios de las primeras 2 h; ejecuta el ciclo del lunes con marketing | Diario (10-15 min) y los lunes (30 min) |
| Investigador de temas | Agente `investigador` (Claude) | Propone 20 a 30 temas por semana con 2 fuentes candidatas cada uno, revisa que no estén repetidos en `historial.csv` y los agrega a `temas.csv` como `pendiente` | Semanal, antes del lunes |
| Guionista | La API de Claude dentro del pipeline (`prompts/guionista.md`). El **arquitecto** mantiene el prompt y lo ajusta con los rechazos | Escribe el guion, los prompts visuales y los textos por red en JSON | En cada lote, solo |
| Revisor de veracidad | La API de Claude (`prompts/verificador.md`) + Mauricio | La máquina lee las fuentes y bloquea; la persona aplica el checklist de §4.2 | En cada lote |
| Marketing y comunidad | Agente `marketing` (Claude) | Arma el informe semanal, propone qué replicar y qué cortar, redacta respuestas a comentarios (Mauricio las publica) y diseña las pruebas A/B | Lunes |
| Devops | Agente `devops` (Claude) | Secretos, workflows, R2, Upload-Post, tokens vencidos y alertas `alerta` | Al arrancar y ante alertas |
| Diseño de plantillas | Agente `disenador-ui` (Claude Code + plugin de Remotion) | Brief visual de cada plantilla (tipografía, paleta, movimiento, subtítulos y gancho) y prueba en Remotion Studio; aprueba el resultado de Codex | Al arrancar y cuando se cambia una plantilla |
| Construcción | Arquitecto → Codex → `revisor` | El plan en `docs/superpowers/plans/`; Codex programa el pipeline y las composiciones de Remotion, el revisor aprueba y Claude integra | Por tarea |

## 8. Decisiones pendientes de Mauricio

| # | Decisión | Valor por defecto (propuesto) |
|---|---|---|
| 1 | Nombre y handle del canal (¿«NomosTin»?) | Un nombre propio, sin la marca PROMETEO al inicio |
| 2 | Idioma | Español neutro LATAM; inglés recién entre el día 60 y el 90 |
| 3 | Presupuesto | Tope de USD 8 por día (≈ USD 190 al mes con IGV) en el mes 1 |
| 4 | ¿Voz propia clonada? | Voz sintética fija en el mes 1; clonar después |
| 5 | Revisión | El 100 % en el mes 1; después, una muestra de 2 por día |
| 6 | Mezcla y exclusiones | §1.1 y §1.2 |
| 7 | Avisos | Issue de GitHub (Telegram, si lo prefiere) |
| 8 | Objetivo del mes 1 | Un flujo estable y 3 formatos ganadores por retención; monetizar no es meta del mes 1 |
| 9 | Música | Biblioteca libre con licencia comercial para las 3 redes, con la licencia de cada pista guardada |
| 10 | **¿De quién es el canal?** Define la licencia de Remotion | Personal de Mauricio: Remotion gratis. De Prometeus (4 socios): «Remotion for Automators», mínimo USD 100 al mes (spec §11.4). **Propuesto:** personal al inicio y pasarlo a Prometeus si genera ingresos |
| 11 | Plantillas visuales | Las 3 de §1.4 |
