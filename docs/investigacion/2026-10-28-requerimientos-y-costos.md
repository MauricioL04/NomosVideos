---
proyecto: Videos-Diario-NomosTin (canal «¿Qué pasaría si…?»)
fecha: 2026-10-28
autor: analista-negocio (Claude)
estado: borrador para que decida Mauricio
---

# Requerimientos y costos · canal automatizado «¿Qué pasaría si…?»

**Pedido.** Un canal de videos cortos donde Claude escribe el guion y los prompts, Higgsfield genera el video, el montaje (voz, subtítulos y música) sale automático y se publican 10 videos al día en YouTube Shorts, Instagram Reels y TikTok, en horarios elegidos.

**Convenciones.** Todo lo que no decidió Mauricio va marcado **propuesto**. Precios en USD, revisados el 2026-10-28 y **sin impuestos**. En Perú, los servicios digitales de proveedores del exterior pagan IGV del 18 %; conviene confirmarlo con la contadora (**propuesto**). Prioridad: S (horas), M (un día), L (varios días).

---

## Hallazgos que cambian el plan

1. **Higgsfield tiene dos productos distintos.** La web (planes Starter, Plus y Ultra, con créditos) y la **API de pago por uso**, en `open.higgsfield.ai`, que cobra en USD por segundo de video. La propia página de planes aclara que los modelos «ilimitados» y las generaciones gratis **no aplican por MCP/CLI**, solo en higgsfield.ai. Para automatizar se usa la API.
2. **El costo lo decide casi entero el video generado: entre el 82 % y el 87 % del total.** Lo demás (Claude, voz, publicación y almacenamiento) suma menos de USD 90 al mes en el escenario recomendado.
3. **Publicar por API propia trae trámites largos.** Sin auditoría, TikTok deja los videos en **privado** y lo mismo pasa con YouTube en proyectos no verificados. Un agregador como Upload-Post (USD 16–24/mes) ya tiene esas aprobaciones y permite publicar desde la primera semana. **Recomendación (propuesto):** usar el agregador el primer mes y tramitar en paralelo las apps propias solo si después compensa.
4. **Hay riesgo de monetización en YouTube.** Desde julio de 2026, YouTube no monetiza el contenido «inauténtico»: videos de IA repetitivos o hechos con plantilla y sin un arco narrativo propio. Diez videos diarios con el mismo molde son justo ese perfil. Hace falta variedad real de formato y una revisión humana al inicio.
5. **TikTok tiene un tope aproximado de 15 publicaciones diarias por cuenta**, compartido entre todas las apps. Diez al día entran, pero quedan cerca del límite.

---

## A. Lo que Mauricio debe tener o hacer

Ordenado por dependencia. Los tiempos son de trámite, no de trabajo continuo.

### A1. Base (día 1, unas 3 horas en total)

| # | Qué | Detalle | Tiempo estimado |
|---|---|---|---|
| 1 | **Nombre del canal y handle** | Elegir el nombre y comprobar que el mismo @handle esté libre en YouTube, Instagram y TikTok, y también el dominio si se va a usar uno | 30 min a 1 día (es una decisión) |
| 2 | **Correo de la marca** | Una cuenta Gmail nueva solo para el canal; desde ella se crean todas las demás. No usar el correo personal | 15 min |
| 3 | **Gestor de contraseñas y 2FA** | Activar 2FA con app autenticadora (no por SMS) en el correo y en cada cuenta que siga. Guardar los códigos de respaldo fuera de la PC | 30 min |
| 4 | **Tarjeta internacional** | Débito o crédito con compras en USD por internet. **Propuesto:** una tarjeta o cuenta aparte, con un límite igual al costo máximo mensual, como freno físico al gasto | 0 min si ya la tiene; 1 a 7 días si hay que pedirla al banco |
| 5 | **Teléfono de la marca** | Para verificar las cuentas. Puede servir el chip de WhatsApp Business que ya compró para Leads, si no lo usa para otra cosa | 0 a 1 día |

### A2. Cuentas en las redes (día 1 a 2)

| # | Qué | Detalle | Tiempo estimado |
|---|---|---|---|
| 6 | **YouTube** | Crear el canal con el correo de la marca. Verificar el teléfono en youtube.com/verify; no es obligatorio para Shorts, pero habilita funciones y baja el riesgo de bloqueo | 20 min, más hasta 24 h de verificación |
| 7 | **Instagram profesional** | Crear la cuenta y pasarla a **Creador o Empresa**. Con la «API de Instagram con inicio de sesión de Instagram», Meta ya no exige una página de Facebook para publicar; sí la pide si se usa la vía de inicio de sesión con Facebook ([Meta, content publishing](https://developers.facebook.com/documentation/instagram-platform/content-publishing)). **Propuesto:** crear igual la página de Facebook y vincularla en Meta Business Suite, porque cuesta 15 minutos y abre Reels en Facebook y los anuncios después | 45 min |
| 8 | **TikTok** | Crear la cuenta con el correo de la marca. Una cuenta personal o de creador sirve para la API | 15 min |
| 9 | **Calentamiento de cuentas** (propuesto) | Usar las tres cuentas a mano durante 3 a 5 días (perfil completo, foto, bio y 1 o 2 publicaciones manuales) antes de automatizar. Las cuentas nuevas que publican en masa desde el primer día corren más riesgo de ser marcadas | 3 a 5 días, en paralelo con lo demás |

### A3. Servicios del sistema (día 2, unas 2 horas)

| # | Qué | Detalle | Tiempo estimado |
|---|---|---|---|
| 10 | **Higgsfield API** | Cuenta en `open.higgsfield.ai`, clave de API y recarga inicial. **Propuesto:** USD 50 para la prueba | 30 min |
| 11 | **Anthropic Console** | Cuenta de API, tarjeta y **límite de gasto mensual** configurado en la consola | 20 min |
| 12 | **ElevenLabs** | Cuenta de API de pago por uso, o un plan si va a clonar su voz | 15 min |
| 13 | **Cloudflare R2** | Cuenta y un bucket para guardar los videos | 30 min |
| 14 | **Agregador de publicación** (ruta recomendada) | Upload-Post: conectar las tres cuentas al perfil por OAuth | 30 min |
| 15 | **Servidor** | Un VPS pequeño, o la PC de Mauricio encendida a horario fijo (ver la parte C) | 30 min |

Las claves de API **no se guardan en la bóveda ni en el repo**: van en el `.env` del servidor.

### A4. Ruta propia, sin agregador (opcional, en paralelo, de 2 a 8 semanas)

Solo hace falta si Mauricio quiere ahorrarse el agregador (USD 16–24/mes) o tener control total. Ninguno de estos pasos bloquea el arranque si se usa el agregador.

| # | Plataforma | Pasos | Tiempo estimado |
|---|---|---|---|
| 16 | **Google Cloud (YouTube)** | Crear el proyecto, activar YouTube Data API v3, configurar la pantalla de consentimiento OAuth (con política de privacidad y dominio) y pedir la verificación del permiso `youtube.upload`. Los videos subidos desde proyectos no verificados quedan **privados** hasta pasar la auditoría de cumplimiento. La cuota diaria por defecto es de 10 000 unidades; el costo de cada subida (antes 1600 unidades) habría bajado mucho desde diciembre de 2025, pero las fuentes no coinciden, así que hay que confirmarlo en la consola | Proyecto: 1 h. Verificación y auditoría: de 1 a 6 semanas (estimado de terceros) |
| 17 | **Meta for Developers** | Crear la app, agregar el producto Instagram y el permiso `instagram_content_publish`. Si solo publica en sus **propias** cuentas, con rol en la app, puede trabajar en modo de desarrollo sin App Review (**verificar** al crearla). La verificación del negocio solo se pide para el acceso avanzado. Tope: 50 publicaciones por API cada 24 h según la referencia `content_publishing_limit` (otra página de Meta dice 100) | De 1 a 2 h; de días a semanas si pide App Review |
| 18 | **TikTok for Developers** | Crear la app, agregar el Content Posting API (Direct Post), conseguir el permiso `video.publish` y una URL de sitio, política de privacidad y términos. Después, **la auditoría**: sin ella, todo se publica en privado (`SELF_ONLY`) y la API rechaza las cuentas públicas | App y permiso: de días a 2 semanas. Auditoría: de 2 a 6 semanas, según terceros; TikTok no publica un plazo |
| 19 | **Dominio y página legal** | Las tres apps piden una política de privacidad y términos en una URL propia | De 2 a 4 h, más el costo del dominio |

---

## B. Requisitos del sistema

### B1. Funcionales

| ID | Requisito | Criterio de aceptación | Prioridad |
|---|---|---|---|
| RF-01 | **Generación diaria** | A una hora fija (por ejemplo, 00:30 de Lima) el sistema produce los 10 videos del día siguiente: tema, guion, prompts, clips o imágenes, voz, subtítulos, música y montaje final en 1080×1920 | M |
| RF-02 | **Guion y prompts con Claude** | Cada video tiene una pregunta «¿Qué pasaría si…?», un gancho en los primeros 2 s, entre 3 y 6 escenas con su prompt visual, la narración (unas 110 palabras para 45 s) y el título, la descripción y los hashtags de cada red. La salida es JSON validado contra un esquema; si no valida, se pide de nuevo | M |
| RF-03 | **No repetir temas** | Antes de aceptar un tema, se compara con todos los temas publicados (texto normalizado y similitud semántica por embeddings o por un juicio de Claude). Si la similitud supera el umbral, se descarta. En una prueba con 300 temas no debe haber dos iguales ni paráfrasis evidentes | M |
| RF-04 | **Variedad de formato** | Cada video lleva una plantilla de entre al menos 4 (ciencia, historia alternativa, cuerpo humano, espacio, etc.), con voz, ritmo y estilo visual distintos. Ninguna plantilla pasa de 4 de los 10 videos del día. Mitiga el riesgo de «contenido inauténtico» de YouTube | S |
| RF-05 | **Montaje automático** | Con FFmpeg: clips e imágenes animadas (zoom y paneo), voz, subtítulos palabra por palabra quemados en el video y música a −18 dB por debajo de la voz. La duración final está entre 30 y 60 s y el audio no se recorta | M |
| RF-06 | **Revisión humana opcional** | Hay un modo «aprobar antes de publicar»: Mauricio recibe los 10 videos del día (por Telegram o en una página simple) y aprueba, rechaza o pide regenerar cada uno. Si no responde antes de la hora límite, se aplica la regla que haya configurado: publicar o no publicar. Se activa y desactiva con una variable, sin tocar código | M |
| RF-07 | **Publicación en horarios** | Cada video sale en las 3 redes según una tabla de horarios de Lima, editable en un archivo. Sugerencia inicial (**propuesto**, por ajustar con datos): 07:00, 09:00, 11:00, 12:30, 14:00, 16:00, 18:00, 19:30, 21:00 y 22:30. Desfase de 0 a 10 min entre redes | S |
| RF-08 | **Reintentos** | Si falla un paso (Higgsfield, voz o subida), se reintenta hasta 3 veces con espera creciente (1, 5 y 15 min). Si sigue fallando, se cambia a un modelo de respaldo más barato o se reemplaza la escena por una imagen animada. Si aun así no sale, el video pasa a «fallido» y entra uno de la **reserva** (RF-09) | M |
| RF-09 | **Reserva de videos** | Siempre hay al menos 5 videos aprobados y sin publicar en reserva, para cubrir caídas sin dejar huecos en el calendario | S |
| RF-10 | **Registro de publicaciones** | Una tabla (SQLite o Postgres) guarda por video: id, tema, plantilla, costo real por proveedor, estado, y por red la fecha y hora, el id o URL de la publicación y el error si lo hubo. Se puede consultar «¿qué se publicó ayer y dónde?» en una sola consulta | S |
| RF-11 | **Etiqueta de IA** | Cada publicación marca el contenido como generado por IA donde la red lo permite: en TikTok, el campo de contenido generado por IA; en YouTube, la divulgación de contenido alterado o sintético cuando el video parece real; en Meta, la etiqueta de IA | S |
| RF-12 | **Guardar los videos** | El MP4 final, el JSON del guion y los prompts se guardan en R2 con la ruta `aaaa/mm/dd/<id>`. Los clips intermedios se borran a los 30 días (**propuesto**) y los finales se conservan | S |
| RF-13 | **Costo máximo diario** | Hay un tope diario configurable (por ejemplo, USD 20). Antes de cada llamada paga, el sistema suma el gasto del día y, si se pasaría del tope, no la hace: baja a la alternativa barata (imágenes animadas) o se detiene y avisa. Ningún día puede superar el tope en más de una llamada | S |
| RF-14 | **Alertas** | Llega un mensaje (**propuesto:** Telegram) cuando falla un video tras los reintentos, cuando la reserva baja de 5, cuando el gasto del día pasa del 80 % del tope, cuando una red rechaza una publicación o vence un token, y cada día un resumen con lo publicado, el costo y los errores | S |
| RF-15 | **Métricas básicas** (después del mes 1) | Una vez al día se leen las vistas y la retención de cada publicación y se guardan junto al video, para ajustar los horarios y las plantillas | M |

### B2. No funcionales

| ID | Requisito | Criterio de aceptación | Prioridad |
|---|---|---|---|
| RNF-01 | **Fiabilidad** | Al menos el 95 % de las publicaciones programadas salen a su hora, con un margen de 15 min, medido por semana | M |
| RNF-02 | **Idempotencia** | Reintentar una subida nunca crea un duplicado: se usa una clave de idempotencia por video y red (Upload-Post acepta el encabezado `Idempotency-Key`) | S |
| RNF-03 | **Secretos** | Las claves van en `.env` o en un gestor de secretos, nunca en git, en la bóveda ni en los registros | S |
| RNF-04 | **Costo trazable** | Cada video registra su costo real por proveedor; el resumen diario cuadra con la facturación con un error menor al 5 % | S |
| RNF-05 | **Respeto de límites** | El sistema no pasa de 15 publicaciones diarias en TikTok, 50 en Instagram ni el tope de YouTube, aunque haya reintentos | S |
| RNF-06 | **Derechos** | Solo se usa música con licencia comercial válida en las 3 redes, o música generada con derechos; se guarda la fuente de cada pista. Sin caras de personas reales ni marcas registradas en los prompts | S |
| RNF-07 | **Recuperación** | Si el servidor se reinicia, el proceso retoma desde el último paso guardado sin regenerar lo ya pagado | M |
| RNF-08 | **Simplicidad** (ponytail) | Un script de Python con un programador de tareas (cron o un temporizador de systemd) y FFmpeg. Sin n8n ni colas mientras un solo proceso alcance | S |

### Revisión humana en el primer mes: sí

**Recomendación:** activar RF-06 **todo el primer mes** y publicar menos al principio. Semana 1: 3 videos al día; semana 2: 5; semanas 3 y 4: 10, si la tasa de rechazo baja del 20 %.

Motivos:
- Las cuentas son nuevas y diez videos de IA al día desde el día 1 se parecen a una granja de contenido; es lo que YouTube excluye de la monetización.
- Los modelos de video fallan seguido (manos, texto y física rara). Un error visible en un video de «ciencia» le quita credibilidad al canal.
- Revisar 10 videos de 45 s toma unos 10 a 15 minutos al día, y lo rechazado sirve para afinar los prompts.
- Después, se pasa a revisar solo una muestra de 2 videos al día y el resto se publica solo.

---

## C. Costo mensual estimado (300 videos de 30 a 60 s)

### Supuestos

- Duración media de 45 s; narración de unos 650 caracteres por video (unos 195 000 al mes). Se suma un 20 % por regeneraciones de voz.
- Para el video se suma un **30 % por regeneraciones**: no todo clip sale usable. Fuentes de terceros reportan varios intentos por clip bueno ([Creatify](https://creatify.ai/blog/higgsfield-pricing-(2026)-plans-and-what-you-ll-actually-pay)).
- Precios de la **API de Higgsfield** a precio **de lista**. La página también muestra un «precio con descuento máximo» de 30 a 60 % menos, pero no explica las condiciones; si se confirma, el costo de video baja en esa proporción.

### Precios de referencia verificados el 2026-10-28

| Servicio | Precio | Fuente |
|---|---|---|
| Higgsfield API · Kling 3.0 Pro | USD 0,084/s de lista (0,042 con descuento máximo) | [open.higgsfield.ai/pricing](https://open.higgsfield.ai/pricing) |
| Higgsfield API · Kling 3.0 Standard (imagen a video) | USD 0,063/s de lista | ídem |
| Higgsfield API · Wan 3.0 | USD 0,050/s de lista (0,020 con descuento hasta el 31/10) | ídem |
| Higgsfield API · Seedance 2.0 / 2.5 | USD 0,141/s y 0,206/s de lista | ídem |
| Higgsfield API · imágenes (Z-Image Turbo / Soul 2) | USD 0,015 y desde 0,003 por imagen | [open.higgsfield.ai/explore](https://open.higgsfield.ai/explore) |
| Higgsfield web · Starter / Plus / Ultra (pago anual) | USD 15 / 39 / 99 al mes; 200 / 1000 / 3000 créditos. Un Kling 3.0 de 8 s en 720p cuesta unos 14 créditos. Los «ilimitados» no aplican por MCP/CLI | [higgsfield.ai/pricing](https://higgsfield.ai/pricing) |
| Claude Haiku 4.5 / Sonnet 4.6 / Opus 4.8 | USD 1/5, 3/15 y 5/25 por millón de tokens de entrada/salida; Batch con −50 % | [devtk.ai](https://devtk.ai/en/blog/claude-api-pricing-guide-2026/), [morphllm](https://www.morphllm.com/anthropic-claude-api-pricing) (confirmar en la consola de Anthropic) |
| ElevenLabs TTS | USD 0,04 por 1000 caracteres (Flash/Turbo) y 0,08 (v3 y Multilingual) | [elevenlabs.io/pricing/api](https://elevenlabs.io/pricing/api) |
| Upload-Post | Basic USD 24/mes (16 con pago anual): 5 perfiles, subidas ilimitadas. Topes por cuenta: Instagram 50, TikTok 15 y YouTube 30 al día | [docs.upload-post.com](https://docs.upload-post.com/resources/pricing-and-limits) |
| Ayrshare | Premium USD 149/mes (1 perfil) | [Blotato](https://www.blotato.com/blog/ayrshare-pricing), [ayrshare.com](https://www.ayrshare.com/pricing.md) |
| n8n Cloud | Starter unos €24/mes (2500 ejecuciones); la edición comunitaria autoalojada es gratis | [cloudzero](https://www.cloudzero.com/blog/n8n-pricing/) |
| VPS Hetzner | CX23 unos €5,99/mes (agotado en octubre de 2026); CPX22 unos €19,99/mes | [findstack](https://findstack.com/resources/hetzner-price-increase-2026), [northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases) |
| Cloudflare R2 | USD 0,015 por GB al mes; 10 GB gratis; sin costo de salida | [Semaphore](https://semaphore.io/blog/cloudflare-r2%22) (confirmar en Cloudflare) |

### Tres escenarios (USD al mes, sin IGV)

| Partida | Mínimo | Recomendado | Premium |
|---|---|---|---|
| **Visual (Higgsfield API)** | 6 imágenes + 1 clip de gancho de 5 s con Wan 3.0 → **≈ 133** | 15 s de Kling 3.0 Pro + 6 imágenes animadas → **≈ 527** | 45 s generados con Kling 3.0 Pro → **≈ 1474** (con Seedance 2.5: ≈ 3615) |
| Guion y prompts (Claude API) | Haiku 4.5 → ≈ 3 | Sonnet 4.6, con una pasada de autocrítica → ≈ 21 | Opus 4.8 → ≈ 35 |
| Voz (TTS) | ElevenLabs Flash → ≈ 10 (con Piper local: 0) | ElevenLabs v3 → ≈ 19 | ElevenLabs v3 o voz clonada → ≈ 19, más el plan si la clona |
| Subtítulos | Whisper local o las marcas de tiempo del TTS → 0 | 0 | 0 |
| Música | Biblioteca libre de derechos → 0 | 0 | Biblioteca comercial (precio no verificado) |
| Publicación | Upload-Post Basic anual → 16 | Upload-Post Basic mensual → 24 | Ayrshare Premium → 149 |
| Orquestador y servidor | PC de Mauricio → 0 | VPS de 4 GB → ≈ 22 | VPS → ≈ 22 |
| Almacenamiento (R2) | 0 (entra en los 10 GB gratis el primer mes) | ≈ 1 | ≈ 2 |
| **Total** | **≈ 162** | **≈ 614** | **≈ 1701** |
| Total con IGV del 18 % (propuesto) | ≈ 191 | ≈ 725 | ≈ 2007 |
| **Peso del video en el total** | **82 %** | **86 %** | **87 %** |
| Costo por video | ≈ 0,54 | ≈ 2,05 | ≈ 5,67 |

Cálculo del visual: (segundos de clip × precio por segundo + imágenes × precio por imagen) × 1,3 × 300. Ejemplo para el recomendado: (15 × 0,084 + 6 × 0,015) × 1,3 × 300 = 526,5.

**¿Conviene el plan web en vez de la API?** El plan Ultra (USD 99–129 al mes, 3000 créditos) da unos 1700 s de Kling 720p al mes, unos 6 s por video. No alcanza para el escenario recomendado y sus ilimitados no sirven para automatizar. La API es la opción para operar a escala.

### Qué pesa más y cómo bajarlo

Pesan más los **segundos de video generado**. Por orden de impacto:

1. **Más imágenes animadas y menos clips.** Una imagen cuesta de USD 0,003 a 0,015 y se anima gratis con FFmpeg (zoom, paneo o parallax). Usar video generado solo en el gancho (de 3 a 5 s) y en una escena clave baja el costo del visual entre 70 y 80 %. Es la diferencia entre el escenario mínimo y el recomendado.
2. **Un modelo más barato para el relleno.** Wan 3.0 (0,050/s) cuesta la mitad que Kling 3.0 Pro (0,084/s) y una cuarta parte que Seedance 2.5 (0,206/s). Se puede usar Kling o Seedance solo en el gancho.
3. **Imagen a video en lugar de texto a video.** Primero se genera una imagen barata, se valida y recién entonces se anima. Eso baja la tasa de regeneración, que es el 30 % del costo.
4. **720p en vez de 1080p**, donde el modelo cobra distinto. En un teléfono casi no se nota.
5. **Reutilizar.** Una biblioteca propia de clips genéricos (espacio, océano, ciudad, explosión) que se recombinan.
6. **Confirmar el «descuento máximo» de Higgsfield.** Si sale de un compromiso de gasto o de un volumen, puede bajar el visual entre 30 y 50 %.
7. **Publicar menos.** Con 5 videos al día el costo baja a la mitad. Con cuentas nuevas, la calidad suele rendir más que el volumen (**propuesto**, por validar con las métricas del mes 1).
8. **Claude no es donde se ahorra:** pesa menos del 4 %. Aun así, Batch API (−50 %) y caché del prompt del sistema.

---

## D. Preguntas abiertas para Mauricio

| # | Pregunta | Recomendación |
|---|---|---|
| 1 | **¿En qué idioma?** | **Español neutro.** Cuesta lo mismo y hay menos competencia que en inglés, aunque en inglés se paga más por vista. Se puede abrir un segundo canal en inglés reutilizando los guiones traducidos en el mes 3 |
| 2 | **¿Cómo se llama el canal?** ¿Va bajo la marca PROMETEO? | Un nombre propio, corto y con el handle libre en las 3 redes. **No** asociarlo a PROMETEO hasta ver cómo responde el público. «NomosTin» es el nombre de la carpeta; ¿también del canal? |
| 3 | **¿Cuál es el presupuesto mensual máximo?** | Empezar con el **mínimo, unos USD 190 con IGV, en el mes 1**, con un tope diario de USD 8. Subir al recomendado solo si las métricas lo justifican |
| 4 | **¿Muestra su cara o su voz?** | **Voz sí, cara no.** Clonar su voz en ElevenLabs da autenticidad, que es lo que YouTube pide para monetizar, sin exponerse. La cara no hace falta en este formato |
| 5 | **¿Revisa los videos antes de publicar?** | **Sí, todo el mes 1** (10 a 15 min diarios); después, solo una muestra |
| 6 | **¿10 al día desde el inicio?** | **No.** Subir de 3 a 5 y luego a 10 en cuatro semanas |
| 7 | **¿Para qué es el canal: ingresos por vistas, audiencia para vender algo o probar el flujo?** | Definirlo cambia las métricas. **Propuesto:** objetivo del mes 1 = flujo estable y 3 formatos ganadores medidos por retención |
| 8 | **¿Agregador o APIs propias?** | **Agregador** (Upload-Post) en el mes 1. Las apps propias se tramitan en paralelo solo si va a escalar a varios canales |
| 9 | **¿Dónde corre el sistema: su PC o un VPS?** | **VPS** desde el principio si el presupuesto lo permite: la PC apagada o con poca RAM deja huecos en el calendario |
| 10 | **¿Qué temas quedan fuera?** | Excluir salud, finanzas y política real, y tragedias: son las categorías que YouTube marca como sensibles o manipuladoras |
| 11 | **¿Qué música?** | Biblioteca libre con licencia comercial que sirva en las 3 redes, y guardar la licencia de cada pista. No usar música en tendencia de TikTok por API |
| 12 | **¿Por qué canal recibe las alertas y la revisión?** | Telegram, con un bot propio: es gratis y ya está en sus pendientes de VELU |
| 13 | **¿Quién programa el sistema y cuándo?** Compite con Leads, Roblox y la universidad | Lo programa Codex según el protocolo del equipo. Tamaño estimado: **L** (de 4 a 6 días con revisión). Mauricio decide dónde entra en [[Prioridades]] |
| 14 | **¿Factura como persona o como empresa?** | Si el canal genera ingresos, consultar con la contadora el RUC y el IGV de los servicios digitales |

---

## Fuentes

- Higgsfield, planes web: https://higgsfield.ai/pricing (leído el 2026-10-28)
- Higgsfield, API y precios por segundo: https://open.higgsfield.ai/pricing y https://open.higgsfield.ai/explore (leídos el 2026-10-28)
- Creatify, costos reales en Higgsfield: https://creatify.ai/blog/higgsfield-pricing-(2026)-plans-and-what-you-ll-actually-pay
- ElevenLabs, precios de la API: https://elevenlabs.io/pricing/api (leído el 2026-10-28)
- Upload-Post, precios y límites: https://docs.upload-post.com/resources/pricing-and-limits (leído el 2026-10-28)
- Ayrshare: https://www.ayrshare.com/pricing.md · https://www.blotato.com/blog/ayrshare-pricing
- Claude API: https://devtk.ai/en/blog/claude-api-pricing-guide-2026/ · https://www.morphllm.com/anthropic-claude-api-pricing
- n8n: https://www.cloudzero.com/blog/n8n-pricing/
- Hetzner: https://findstack.com/resources/hetzner-price-increase-2026 · https://northflank.com/blog/hetzner-cloud-server-price-increases
- Cloudflare R2: https://semaphore.io/blog/cloudflare-r2%22
- TikTok, Direct Post y auditoría: https://developers.tiktok.com/doc/content-posting-api-reference-direct-post
- TikTok, pautas de publicación y tope diario: https://developers.tiktok.com/doc/content-sharing-guidelines
- TikTok, plazos de aprobación (terceros): https://adaptlypost.com/blog/why-does-my-app-need-approval-post-tiktok
- Instagram, publicación de contenido: https://developers.facebook.com/documentation/instagram-platform/content-publishing · tope: https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/ig-user/content_publishing_limit
- YouTube, cuota y auditoría: https://developers.google.com/youtube/v3/determine_quota_cost · https://www.blotato.com/blog/youtube-api-pricing · https://www.activepieces.com/blog/youtube-api-quota-exceeded-how-to-fix-and-avoid-it
- YouTube, contenido inauténtico: https://techcrunch.com/2026/07/20/youtube-clarifies-policies-around-ai-slop-and-upsetting-videos/ · https://www.androidheadlines.com/2026/07/youtube-monetization-rules-ai-slop-inauthentic-content.html

**Por verificar antes de comprar:** las condiciones del «descuento máximo» de Higgsfield; el costo actual de cuota de `videos.insert` en YouTube; si la app de Meta en modo de desarrollo basta para publicar en cuentas propias; el precio de la música comercial y del dominio; y el IGV de los servicios digitales con la contadora.
