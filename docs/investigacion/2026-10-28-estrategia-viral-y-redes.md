---
proyecto: Videos-Diario-NomosTin
tema: Estrategia viral y manejo de redes para Shorts "¿Qué pasaría si…?"
fecha: 2026-10-28
autor: agente marketing (Prometeus)
estado: propuesto (nada de esto lo ha aprobado Mauricio todavía)
---

# Estrategia viral y manejo de redes: canal "¿Qué pasaría si…?"

Canal de videos verticales de 30 a 60 s, explicativos, con guion de Claude y video de Higgsfield, publicados en YouTube Shorts, Instagram Reels y TikTok. Es contenido orgánico; no hay pauta en este documento.

Convenciones:
- **[propuesto]**: recomendación que Mauricio aún no decide.
- **[estimación]**: cifra sin fuente primaria; sirve para planear, no para prometer.
- Las fuentes van al pie con número `[n]`. Todas se consultaron el 2026-10-28. Varias son blogs de proveedores (AIR, vidIQ, Buffer, Metricool): se marcan como "dato de proveedor".

## 0. Resumen de decisiones [propuesto]

| Tema | Recomendación |
|---|---|
| Idioma | Español LATAM primero. Canal en inglés como fase 2 (día 60-90), solo si el español pasa los umbrales de calidad. |
| Volumen | Producir hasta 10 al día, pero publicar escalonado: YouTube recibe más, TikTok e Instagram reciben los mejores. Rampa 3 → 6 → 10 en YouTube. |
| Publicación automática | Solo por API oficial auditada o por un programador oficial (Buffer, Metricool, etc.). Sin auditoría, YouTube y TikTok dejan los videos en privado. |
| Revisión humana | Obligatoria antes de publicar: datos científicos, etiqueta de IA y derechos. Sin esto, 10 al día es un riesgo de desinformación y de perder la monetización. |
| Meta a 90 días | 1.000 suscriptores en YouTube y un camino visible a 10 M de vistas de Shorts en 90 días; monetizar en el día 90 es optimista. |

## 1. Nicho y público

### 1.1 Idioma: español, inglés o ambos

| Criterio | Español LATAM | Inglés |
|---|---|---|
| Ingreso por 1.000 vistas de Shorts (RPM) | México: USD 0,040 (dato de proveedor, AIR, 2025) [1] | EE. UU.: USD 0,328; Reino Unido 0,166; Canadá 0,165 (mismo estudio) [1] |
| Relación | Base | Unas 8 veces más que México por vista [estimación a partir de [1]] |
| Tamaño de catálogo en YouTube | Español ~11,8 % del contenido, segundo idioma (blog, sin fuente oficial) [2] | Inglés ~39-40 % del contenido [2] |
| Competencia en "What if" | Pocos canales grandes dedicados; el mayor es la versión doblada de What If (~3,15 M suscriptores, dato de abril de 2025) [3] | Saturado: What If (~8,96 M, 1.668 videos) [4], Zack D. Films (~29,3 M, ~4.109 videos, ~81 mil millones de vistas) [5] |
| Control de calidad del equipo | El equipo revisa guiones, voz y matices sin intermediarios | Más difícil revisar tono, humor y errores de voz |
| Diferencial | Temas de Perú y LATAM que los canales gringos no tocan | Ninguno propio |
| Monetización TikTok | Creator Rewards no confirmado para Perú (ver §5) | Igual: depende del país de residencia, no del idioma |

Notas:
- El RPM depende del país del espectador, no del idioma. Un canal en español con mucho público en EE. UU. gana más que uno con público solo en Perú [1].
- Los Shorts pagan poco en cualquier idioma: AIR mide que el RPM de Shorts es entre 3 % y 14 % del RPM de videos largos en casi todos los nichos (274 canales, dato de proveedor) [6].
- YouTube ya ofrece doblaje automático a todos los creadores, con español entre los idiomas [7]. No encontré confirmación de que funcione en Shorts; hay que probarlo en YouTube Studio antes de contar con él.

**Recomendación [propuesto]:** empezar en español neutro LATAM (sin modismos muy peruanos en la voz, sí en los temas). Abrir un canal en inglés entre el día 60 y el 90, reciclando los guiones que mejor funcionaron, cuando el proceso de revisión ya esté rodado. Hacer ambos desde el día 1 duplica la revisión humana justo cuando el riesgo de errores es mayor.

### 1.2 Sub-nichos de "¿qué pasaría si…?"

| Sub-nicho | Ejemplos de pregunta | Riesgo | Comentario |
|---|---|---|---|
| Cuerpo humano | ¿Qué pasaría si no durmieras 7 días? ¿Si tomaras solo agua de mar? | Alto (salud) | Engancha mucho. YouTube excluye de la monetización a "personas de IA" hablando de salud [8]: usar narrador en off, nunca un "doctor" generado. Citar fuentes médicas. |
| Espacio | ¿Qué pasaría si la Luna desapareciera? ¿Si cayeras en Júpiter? | Bajo | Visual, ideal para Higgsfield, se etiqueta como no realista con facilidad. |
| Ciencia y física | ¿Qué pasaría si la Tierra dejara de girar? | Bajo | Mucho escrito y verificable (NASA, papers de divulgación). |
| Historia alternativa | ¿Qué pasaría si los incas hubieran vencido a Pizarro? | Medio | Es especulación: decirlo en el guion ("nadie lo sabe, pero los historiadores creen…"). |
| Perú y LATAM | ¿Qué pasaría si el Amazonas se secara? ¿Si Lima tuviera un tsunami? ¿Si el Misti erupcionara hoy? | Medio-alto | Es el diferencial. Los desastres en ciudades reales pueden asustar: usar datos de IGP, INDECI, SENAMHI y cerrar con información útil, no con miedo (ver §5, contenido angustiante). |
| Animales y naturaleza | ¿Qué pasaría si los mosquitos desaparecieran? | Bajo | Bueno para compartir. Evitar animales sufriendo: YouTube lo pone en la categoría de contenido angustiante [8]. |

**Mezcla inicial [propuesto]:** 30 % espacio y ciencia, 25 % cuerpo humano, 20 % Perú/LATAM, 15 % historia alternativa, 10 % animales. A partir de la semana 3 se reparte según los datos (§4.5).

### 1.3 Canales de referencia y sus números

| Canal | Idioma | Suscriptores | Videos | Fuente y fecha |
|---|---|---|---|---|
| Zack D. Films (@zackdfilms) | Inglés | ~29,3 M | ~4.109 | vidIQ, 21/10/2026 [5]. 100 % de sus vistas vienen de Shorts según OutlierKit [5]. |
| What If (@whatifscienceshow) | Inglés | ~8,96 M | 1.668 | SpeakRJ, agosto de 2026 [4]. Ganó unos 150 mil suscriptores de enero a agosto de 2026. |
| Qué pasaría si - What If Español | Español | ~3,15 M | s/d | vidIQ, dato de abril de 2025 [3] |
| AumSum's What-If | Inglés | ~770 mil | s/d | SocialCounts, julio de 2026 [9] |
| What If Français - Et Si | Francés | ~172 mil | s/d | SocialCounts, julio de 2026 [9] |

Son conteos de terceros, no de YouTube. No encontré un canal peruano o LATAM de "qué pasaría si" en formato Short con cifras verificables: es un hueco a confirmar a mano en YouTube y TikTok antes de decidir el nombre del canal.

Lección de Zack D. Films: miles de videos cortos con la misma estructura (pregunta, explicación visual, remate) funcionan cuando cada uno es distinto en contenido. Lo que YouTube castiga es la plantilla con poca variación (§5).

## 2. Anatomía de un video viral de 30-60 s

### 2.1 Reglas por elemento

| Elemento | Regla | Respaldo |
|---|---|---|
| Duración | 30-60 s; los de 15-35 s suelen tener más tasa de finalización. Shorts admite hasta 3 min [10]. TikTok Creator Rewards exige videos de más de 1 min [11]. | Consenso de blogs, no guía oficial [10]. Probar 35 s contra 55 s (§4.6). |
| Gancho (0-2 s) | La pregunta en pantalla y en voz desde el fotograma 1, con la imagen más fuerte del video. Nada de logos, saludos ni "hoy veremos". | Práctica del formato [estimación] |
| Ritmo | Un corte o cambio de plano cada 1,5-3 s. Higgsfield entrega clips cortos: encadenar 12-20 planos por video. | [estimación] |
| Subtítulos | Quemados en el video, 2-4 palabras por bloque, centrados en el tercio medio (lejos de los botones de la derecha y del texto de abajo). Palabra clave resaltada. Mucha gente ve sin sonido. | [estimación] |
| Voz | Una voz fija del canal (la marca sonora), ritmo rápido, tono de amigo que cuenta algo increíble. Si es voz sintética realista, cuenta como contenido sintético en Meta [12]. | [12] |
| Música | Fondo bajo (-20 dB respecto a la voz). Usar música sin derechos o la biblioteca de cada red. En YouTube, cada pista con licencia se lleva parte del ingreso del Short [13]. | [13] |
| Final en loop | La última frase empalma con la primera ("…y por eso, si la Luna desapareciera…"), sin pantalla final. Las repeticiones cuentan como vistas en YouTube desde el 31/03/2025 [14]. | [14] |
| CTA | Una sola, en voz y texto, en los últimos 3 s o a mitad: "Comenta qué pregunta hacemos mañana". Pedir comentarios da ideas y señales; "suscríbete" va en la descripción. | [estimación] |

### 2.2 Plantilla de guion para Claude (45 s)

Claude rellena cada `{campo}`. Las fuentes son obligatorias y no se publican sin revisión humana.

```text
TÍTULO INTERNO: {pregunta corta, ≤ 60 caracteres}
SUB-NICHO: {espacio | ciencia | cuerpo | historia | peru_latam | animales}
DURACIÓN OBJETIVO: 45 s (±10)
FUENTES (mínimo 2, con URL): {fuente 1} | {fuente 2}
NIVEL DE CERTEZA: {hecho comprobado | modelo científico | especulación}
ETIQUETA IA: {realista → etiquetar en las 3 redes | claramente fantasioso}

[00:00-00:02] GANCHO
  Voz: "¿Qué pasaría si {pregunta}?"
  Texto en pantalla: {pregunta en ≤ 7 palabras}
  Plano: {imagen más impactante del video}

[00:02-00:06] APUESTA
  Voz: {consecuencia sorprendente en 1 frase: "En {tiempo}, {resultado}."}
  Plano: {plano 2}

[00:06-00:30] ESCALADA (3-4 pasos, cada uno peor o más raro que el anterior)
  Paso 1 ({tiempo}): {qué pasa} · Plano: {}
  Paso 2 ({tiempo}): {qué pasa} · Plano: {}
  Paso 3 ({tiempo}): {qué pasa} · Plano: {}
  (Paso 4 opcional)

[00:30-00:38] GIRO O DATO REAL
  Voz: {el dato verdadero que lo ancla: "En realidad, {hecho con fuente}."}

[00:38-00:42] CTA
  Voz: "Comenta qué pregunta hacemos mañana."

[00:42-00:45] CIERRE EN LOOP
  Voz: {frase que termina en la pregunta inicial, para que el video empalme}

DESCRIPCIÓN: {ver §4.1}
HASHTAGS: {ver §4.1}
NOTA PARA EL REVISOR: {qué partes son especulación y cómo se dicen en el guion}
```

### 2.3 Ganchos de ejemplo (ya pasados por humanizer)

- "¿Qué pasaría si la Luna desapareciera esta noche? Mañana ya lo notarías en la playa."
- "¿Y si dejaras de dormir? Al tercer día empiezas a ver cosas que no están."
- "¿Qué pasaría si el Amazonas se secara? Lima lo sentiría antes que Manaos." (verificar con fuente antes de usar)
- "¿Y si Atahualpa hubiera escapado de Cajamarca? Nadie lo sabe, pero esto es lo más probable."

## 3. Horarios estratégicos

### 3.1 Qué dicen los estudios

No hay un estudio de horarios para LATAM en Shorts. Lo disponible:
- TikTok: Sprout Social 2026 apunta a tardes de martes a jueves (14-18 h hora local); Buffer, con 7,1 M de publicaciones, pone el sábado como el mejor día (ambos citados por terceros) [15].
- Reels: Sprout 2026, martes 13-19 h y miércoles 12-21 h; Hootsuite, miércoles 11 h (citados por terceros) [15].
- Shorts: Buffer da viernes y sábado de 16 a 19 h [16].

Las fuentes no coinciden. La regla útil es publicar en la hora local del público y medir. Los horarios de abajo son **[estimación]** para la primera quincena; desde la semana 3 se ajustan con los datos de cada red.

### 3.2 Zonas horarias (desde el 1/11/2026)

| Mercado | Diferencia con Lima |
|---|---|
| Lima, Bogotá, Quito, EE. UU. Este (desde el 1/11) | 0 h |
| Ciudad de México, Centroamérica | -1 h |
| Santiago, Buenos Aires | +2 h |
| Madrid (horario de invierno) | +6 h |

México es el mercado hispano de referencia en el dato de RPM [1]; por eso los horarios se piensan en hora de México y se publican en hora de Lima.

### 3.3 Las 10 franjas diarias [estimación]

| # | Hora Lima | Hora México | Hora Buenos Aires | Momento del público |
|---|---|---|---|---|
| 1 | 07:00 | 06:00 | 09:00 | Desayuno y trayecto (Lima, Bogotá) |
| 2 | 09:30 | 08:30 | 11:30 | Trayecto en México, media mañana en el Cono Sur |
| 3 | 12:00 | 11:00 | 14:00 | Almuerzo en el Cono Sur |
| 4 | 13:30 | 12:30 | 15:30 | Almuerzo en Lima |
| 5 | 15:00 | 14:00 | 17:00 | Almuerzo en México, salida de colegios |
| 6 | 17:00 | 16:00 | 19:00 | Salida del trabajo en el Cono Sur |
| 7 | 18:30 | 17:30 | 20:30 | Trayecto de regreso en Lima |
| 8 | 20:00 | 19:00 | 22:00 | Pico de noche en Lima y México |
| 9 | 21:30 | 20:30 | 23:30 | Pico de noche en México |
| 10 | 23:00 | 22:00 | 01:00 | Cama en México, público de EE. UU. Oeste (20:00) |

### 3.4 ¿Las 10 en las 3 redes o escalonar?

**Escalonar [propuesto].** Motivos:
- Instagram saca de las recomendaciones a las cuentas que publican sobre todo contenido no original [17]. Subir 10 videos diarios casi iguales es la forma más rápida de que el sistema confunda el canal con un agregador.
- Buffer encuentra rendimientos decrecientes al subir la frecuencia, tanto en TikTok como en Instagram [16].
- Para TikTok, la guía más citada es de 1 a 4 publicaciones al día (atribuida a la Creator Academy, no la pude ver en la fuente oficial) [18].
- La API de Instagram permite 50 publicaciones por cuenta cada 24 h [19] y la de TikTok sin auditar topa en unas 15 por cuenta [20]; los topes técnicos no son el problema, la calidad sí.

| Red | Semana 1-2 | Semana 3-4 | Semana 5 en adelante | Qué se publica |
|---|---|---|---|---|
| YouTube Shorts | 3/día | 6/día | hasta 10/día | Todo lo que pasa la revisión |
| TikTok | 2/día | 3/día | 3-4/día | Los mejores del día anterior en YouTube (más vistos frente a deslizados) |
| Instagram Reels | 1/día | 2/día | 2-3/día | Los mejores de la semana; probar ganchos con Trial Reels primero (§4.6) |

### 3.5 ¿10 al día es demasiado para una cuenta nueva?

Para TikTok e Instagram, sí. Para YouTube, se puede llegar, con condiciones:
- Buffer, con datos de 100 mil usuarios en 26 semanas, ve que la constancia (publicar casi todas las semanas) se asocia a unas 4,5 veces más interacción por publicación que publicar de vez en cuando [16]. Constancia pesa más que volumen.
- Varios blogs avisan que publicar demasiado al inicio puede leerse como spam en TikTok [18]; no hay dato oficial.
- El riesgo mayor no es el algoritmo, es la política: 10 videos al día "fáciles de hacer con IA y con poca variación" es la descripción exacta de lo que YouTube saca del Programa de socios [8].

**Rampa [propuesto]** en YouTube: 3/día (días 1-14) → 6/día (días 15-28) → 10/día (día 29 en adelante). Solo se sube de escalón si en la quincena se cumplen las tres puertas:
1. Cero errores científicos detectados después de publicar (revisión semanal de comentarios).
2. El promedio de "vistos frente a deslizados" no baja más de 10 % respecto de la quincena anterior.
3. El equipo revisa todo sin saltarse el checklist.

Si una puerta falla, se queda en el escalón y se corrige.

## 4. Manejo de redes

### 4.1 Títulos, descripciones y hashtags

| Elemento | YouTube Shorts | TikTok | Instagram Reels |
|---|---|---|---|
| Título | La pregunta, ≤ 60 caracteres, con palabra buscable: "¿Qué pasaría si la Luna desapareciera?" | No tiene título aparte; la primera línea del texto es la pregunta | Primera línea del texto = la pregunta |
| Descripción | 1-2 frases + fuentes + "Contenido hecho con IA" + enlace al canal | 1 frase + pregunta al público ("¿Tú qué crees?") | 1-2 frases + pregunta al público; fuentes en el primer comentario fijado |
| Hashtags | 2-3: #shorts #ciencia #quepasariasi | 3-5: #quepasariasi #ciencia #curiosidades #aprendeentiktok + 1 del tema | 3-5, del tema; nada de bloques de 30 |
| Portada | Fotograma con la pregunta en texto grande (se ve en el perfil y en búsquedas) | Portada elegida con texto, mismo estilo siempre | Portada con la pregunta, recortada para la cuadrícula 4:5 del perfil |

Cuidado con los hashtags: los volúmenes concretos no los verifiqué; los de arriba son **[propuesto]** y se ajustan con la búsqueda de cada red.

### 4.2 Respuestas a comentarios

| Tarea | ¿Se automatiza? | Cómo |
|---|---|---|
| Clasificar comentarios (pregunta, corrección, idea de video, spam, odio) | Sí | Claude lee la exportación o la API oficial y etiqueta |
| Borrar spam y ocultar insultos | Sí | Filtros nativos de palabras de cada red |
| Juntar ideas de "pregunta para mañana" | Sí | Alimentan el banco de guiones |
| Borradores de respuesta | Sí, como borrador | Claude propone, una persona aprueba |
| Publicar respuestas | No | Una persona responde 15-20 al día en las primeras 2 horas de cada video |
| Correcciones científicas del público | No | Siempre persona: se verifica, se fija un comentario con la corrección y, si el error es serio, se borra el video |
| Mensajes directos | No | Fuera de alcance. No se usan para vender ni se guardan datos personales (si algún día se recolectan correos o números, aplica la Ley 32323 y el consentimiento previo) |

Toda automatización va por API oficial o herramienta oficial. Nada de bots que simulan personas ni de cuentas falsas para comentar.

### 4.3 Publicación automática: lo que exige cada red

| Red | Requisito | Fuente |
|---|---|---|
| YouTube | Los proyectos de API no verificados creados después del 28/07/2020 solo suben en privado hasta pasar una auditoría | [21] |
| TikTok | Clientes de API sin auditar: solo cuentas privadas, máximo 5 usuarios; error `unaudited_client_can_only_post_to_private_accounts` | [20] |
| Instagram | 50 publicaciones por API por cuenta cada 24 h; cuenta profesional | [19] |

**Recomendación [propuesto]:** al inicio, publicar con un programador oficial ya auditado (Buffer, Metricool u otro) y pedir en paralelo la auditoría de las APIs propias. Las claves y tokens van en variables de entorno o en el gestor de secretos, nunca en el repo ni en la bóveda.

### 4.4 Métricas a seguir

| Métrica | Red | Para qué sirve | Umbral inicial [estimación] |
|---|---|---|---|
| Vistos frente a deslizados | YouTube | Mide el gancho | > 70 % |
| Porcentaje promedio visto | Las 3 | Mide el ritmo; > 100 % indica loop | > 80 % en videos de 45 s |
| % que ve hasta el final | TikTok, Instagram | Mide el cierre | > 30 % |
| Compartidos cada 1.000 vistas | Las 3 | La señal que más predice alcance fuera de seguidores | Comparar contra la mediana propia |
| Seguidores ganados por video | Las 3 | Mide si el canal promete algo | Comparar contra la mediana propia |
| Comentarios cada 1.000 vistas | Las 3 | Mide la CTA y da ideas | Comparar contra la mediana propia |
| Vistas con interacción (engaged views) | YouTube | Es la que cuenta para monetizar | Seguir la suma de 90 días |

Desde el 31/03/2025, YouTube cuenta como vista cada reproducción y repetición; la monetización y el Programa de socios siguen usando las vistas con interacción (engaged views), que están en el modo avanzado de Analytics [14]. Hay que mirar esa cifra para saber cuánto falta para los 10 M.

### 4.5 Ciclo semanal de aprendizaje

Cada lunes, con los videos de la semana anterior que tengan 72 h o más publicados:

1. **Ordenar** por "vistos frente a deslizados" y por compartidos cada 1.000 vistas.
2. **Replicar**: el 20 % superior genera 2-3 secuelas o variantes la semana siguiente (mismo sub-nicho, otra pregunta; o la misma pregunta con otro gancho).
3. **Matar**: el sub-nicho o formato que quede dos semanas seguidas en el 30 % inferior baja a la mitad de cupo; tres semanas, se apaga.
4. **Ajustar la mezcla** de sub-nichos (§1.2) y las 2-3 franjas horarias peores.
5. **Registrar** en una hoja: video, sub-nicho, gancho, duración, hora, métricas, decisión.

Las reglas de corte son **[propuesto]**: con pocas vistas por video el ruido es grande, así que no se decide por un solo video.

### 4.6 Pruebas (diseño con ads-test)

| Prueba | Hipótesis | Unidad | Cómo | Decisión |
|---|---|---|---|---|
| Duración | Los de ~35 s tienen más vistos y compartidos que los de ~55 s | Video | Alternar 35/55 s dentro del mismo sub-nicho por 2 semanas (mín. 20 videos por brazo) | Ganador si supera en vistos-frente-a-deslizados y en compartidos |
| Gancho | Abrir con la consecuencia ("Mañana tu playa no tendría olas") gana a abrir con la pregunta | Video | Instagram Trial Reels: el mismo video con 2 ganchos, solo a no seguidores [22] | Publicar a todos el ganador |
| Voz | Voz A frente a voz B | Semana | Una semana cada una, mismos sub-nichos | Por % visto promedio |
| Horario | Las franjas de §3.3 | Franja | Rotar sub-nichos por franja para no mezclar efectos | Mover las 2 peores cada quincena |

Una variable a la vez. Los resultados son orgánicos y ruidosos: se exige que la diferencia se repita dos semanas antes de cambiar la regla.

## 5. Riesgos de reputación y de políticas

### 5.1 Etiquetado de IA

| Red | Regla | Fuente |
|---|---|---|
| YouTube | Hay que declarar el contenido realista alterado o sintético. En Shorts la etiqueta va sobre el video. Desde mayo de 2026, YouTube etiqueta solo si detecta uso fotorrealista no declarado. Lo claramente irreal o animado lleva una etiqueta más ligera en la descripción. Usar IA para escribir guiones no exige etiqueta. | [23] |
| Meta (Instagram) | Puede exigir la etiqueta "Información de IA" en video fotorrealista o audio realista creado con IA; una voz en off de IA realista es un ejemplo que la necesita | [12] |
| TikTok | Exige etiquetar el contenido generado por IA que muestre imágenes, audio o video realistas; lo no etiquetado puede retirarse. Lee credenciales C2PA para etiquetar solo. | [24] |

**Regla del canal [propuesto]:** etiquetar como IA todos los videos en las tres redes, aunque sean fantasiosos. Cuesta poco, evita retiros y es coherente con decir "hecho con IA" en la descripción. No quitar metadatos C2PA de los archivos de Higgsfield.

### 5.2 Contenido "inauténtico" o producido en masa (YouTube)

Desde el 15/07/2025, YouTube llama "contenido inauténtico" a lo repetitivo o producido en masa, y no lo deja monetizar [25]. El 16/07/2026 lo aclaró en tres categorías [8]:
1. Contenido genérico, repetitivo o de plantilla, fácil de hacer con IA, CGI o plantillas y con poca variación.
2. Contenido que busca angustiar o manipular emociones para ganar vistas.
3. Personas generadas con IA que hablan de finanzas, leyes, salud o medicina.

YouTube no prohíbe la IA y dice que quiere que ayude a hacer más contenido de calidad [8]. Este canal cae justo en la zona de riesgo de la categoría 1. Cómo reducirlo [propuesto]:
- Cada video responde una pregunta distinta con datos propios del tema; nada de rellenar la misma plantilla con palabras cambiadas.
- Variar estructura visual, planos y ritmo; no reutilizar los mismos clips de Higgsfield en varios videos.
- Voz y guion con criterio editorial (opinión, humor, contexto peruano).
- Nunca un presentador de IA hablando de salud: narrador en off y fuentes en pantalla.
- Desastres en ciudades reales: tono informativo y datos de prevención, no imágenes hechas para asustar (categoría 2).

### 5.3 Desinformación

Un error científico en un video de 10 diarios se multiplica rápido y le quita confianza a todo el canal. Checklist obligatorio antes de publicar [propuesto]:
- Dos fuentes con URL por guion (NASA, ESA, IGP, SENAMHI, revistas científicas, libros de historia reconocidos).
- Separar en voz "esto se sabe" de "esto es lo que predicen los modelos" y de "esto es especulación".
- Números redondos y verificables; si no hay fuente, el número sale del guion.
- Revisión humana de cada guion y de cada video final (la imagen también puede mentir: escalas, mapas, banderas).
- Protocolo de error: fijar un comentario con la corrección; si el error es grave, borrar el video.

Para salud: no dar consejos, no recomendar productos ni dosis, y cerrar con "consulta a un médico" cuando aplique.

### 5.4 Derechos de autor

| Riesgo | Medida |
|---|---|
| Videos de Higgsfield | Su centro de ayuda dice que no reclama la propiedad de lo generado y no restringe el uso comercial; los planes gratuitos pueden llevar marca de agua y los de pago no [26]. Usar plan de pago. Higgsfield puede usar el contenido para entrenar sus modelos, salvo en planes Enterprise [26]. No pude leer los Términos completos: revisar la sección 4.4 antes de monetizar. |
| Personas, marcas y personajes reales | No generar caras de personas reales, logos ni personajes con derechos (Pikachu, Marvel). |
| Música | Música sin derechos o de la biblioteca de cada red. Los sonidos de moda de TikTok no se pueden llevar a las otras redes. |
| Clips de terceros | No usar. Instagram saca de recomendaciones a las cuentas de reposteo [17]. |
| Marcas de agua cruzadas | Subir a cada red el archivo maestro limpio, nunca el descargado de otra red. |

### 5.5 Monetización

| Programa | Requisitos | ¿Perú? | Fuente |
|---|---|---|---|
| YouTube Partner Program (Shorts) | 1.000 suscriptores y 10 M de vistas válidas de Shorts en 90 días, cuenta de AdSense, cumplir políticas | Sí, Perú está en la lista oficial | [27] [28] |
| Reparto de Shorts | El creador se lleva el 45 % de lo que le toca del fondo de anuncios de Shorts; si el Short usa música con licencia, parte va a la música | Sí | [13] |
| Cambio de umbrales en 2027 | Un blog dice que desde el 1/02/2027 pasan a 20 M de vistas de Shorts en 90 días; YouTube no lo confirma en su página | Sin confirmar | [29] |
| TikTok Creator Rewards | 18 años o más, cuenta personal, 10 mil seguidores, 100 mil vistas en 30 días, videos originales de más de 1 min, residir en un país elegible | No aparece en ninguna lista (las listas de terceros van de 7 a 13 países; TikTok no publica una oficial) | [11] [30] |
| Meta (Facebook Content Monetization) | Solo por invitación; país elegible | No confirmado; revisar el Panel profesional | [31] |

Cuentas rápidas **[estimación]** con el RPM de AIR [1]: 10 M de vistas de Shorts con público mexicano ≈ USD 400; con público de EE. UU. ≈ USD 3.280. Con estos números, los Shorts en español no pagan Higgsfield por sí solos al inicio. El valor del canal está en la audiencia (marcas, patrocinios, productos propios, la fase en inglés). Mauricio debería tener esto claro antes de comprar planes caros.

No se debe registrar la cuenta de TikTok con un país falso ni usar VPN para entrar a Creator Rewards: viola los términos y pone en riesgo la cuenta.

## 6. Metas a 30, 60 y 90 días [estimación]

No hay un dato confiable de crecimiento de canales nuevos de IA en español; estas metas son una base para medir y se recalibran el día 30.

| Métrica | Día 30 | Día 60 | Día 90 |
|---|---|---|---|
| Videos publicados en YouTube | ~130 (rampa 3 → 6) | ~400 | ~700 |
| Errores científicos graves publicados | 0 | 0 | 0 |
| Suscriptores en YouTube | 300-1.000 | 1.000-3.000 | 3.000-8.000 |
| Vistas con interacción en YouTube (acumulado 90 días) | 0,5-1,5 M | 2-5 M | 5-10 M |
| Seguidores en TikTok | 500-2.000 | 2.000-6.000 | 5.000-15.000 |
| Seguidores en Instagram | 200-800 | 800-2.500 | 2.000-6.000 |
| Videos con más de 100 mil vistas | 1 | 3-5 | 8-15 |
| Vistos frente a deslizados (YouTube, promedio) | > 60 % | > 65 % | > 70 % |
| Compartidos cada 1.000 vistas | línea base | +20 % sobre la base | +40 % sobre la base |
| Monetización | No | No | Posible solicitud al YPP si llega a 1.000 suscriptores y 10 M (optimista) |

Si el día 60 el canal está debajo del rango inferior en vistas y en vistos-frente-a-deslizados, se para la rampa y se revisa el formato (gancho, voz, sub-nichos) antes de seguir pagando producción.

## 7. Plan de prueba, dueños y riesgos

| Acción | Dueño | Cuándo |
|---|---|---|
| Decidir idioma, nombre del canal y la mezcla de sub-nichos | Mauricio | Antes del día 1 |
| Crear las 3 cuentas (YouTube con canal de marca; TikTok e Instagram profesionales) con 2FA | Mauricio | Antes del día 1 |
| Elegir programador oficial o pedir auditoría de las APIs | Mauricio + devops | Semana 0 |
| Leer los Términos de Higgsfield (sección 4.4) y elegir plan de pago | Mauricio | Semana 0 |
| Plantilla de guion (§2.2) cargada en el flujo de Claude | Claude | Semana 0 |
| Revisión humana diaria (checklist §5.3) | Persona designada | Diario |
| Ciclo semanal (§4.5) | Marketing + Mauricio | Cada lunes |
| Recalibrar metas | Marketing | Día 30 |

Riesgos principales:
1. Perder o no obtener la monetización de YouTube por "contenido de plantilla" [8]. Mitigación: variedad real, criterio editorial y rampa.
2. Un error científico viral. Mitigación: checklist y revisión humana sin excepciones.
3. Retiro o etiqueta forzada por IA no declarada [23] [24]. Mitigación: etiquetar siempre.
4. Ingresos muy bajos frente al costo de producción [1] [6]. Mitigación: fase en inglés y fuentes de ingreso fuera del reparto de anuncios.
5. Automatización bloqueada por APIs sin auditar [20] [21]. Mitigación: programador oficial.

## Fuentes

1. AIR Media-Tech, "YouTube Shorts RPM: what to expect in 2026" (dato de proveedor, tabla de 2025): https://air.io/en/monetization/what-rpm-can-you-expect-from-shorts-in-2026
2. AIR Media-Tech, "Most popular languages on YouTube in 2026": https://air.io/en/audience-growth/what-are-the-most-popular-languages-on-youtube · Bemultilingual: https://www.bemultilingual.ca/blog/youtube-languages · Wikipedia: https://en.wikipedia.org/wiki/Languages_used_on_the_Internet
3. vidIQ, "Qué pasaría si - What If Español": https://vidiq.com/youtube-stats/channel/UC5BW1w7JyJW3r8rpR3BhqKA
4. SpeakRJ, What If (@whatifscienceshow): https://www.speakrj.com/audit/report/UCphTF9wHwhCt-BzIq-s4V-g/youtube
5. vidIQ, Zack D. Films: https://vidiq.com/youtube-stats/channel/%40zackdfilms/ · OutlierKit: https://outlierkit.com/channel/zackdfilms
6. AIR Media-Tech, "YouTube Shorts RPM vs long-form: real monetization data from 274 channels": https://air.io/en/air-data-findings/youtube-shorts-rpm-vs-long-form-how-much-do-shorts-earn-in-2026
7. Doblaje automático a 27 idiomas: https://www.storyboard18.com/amp/digital/youtube-expands-auto-dubbing-to-27-languages-adds-more-natural-and-expressive-voices-88923.htm · https://www.tbsnews.net/node/1353236
8. TechCrunch, "YouTube clarifies policies around AI slop and upsetting videos" (20/07/2026): https://techcrunch.com/2026/07/20/youtube-clarifies-policies-around-ai-slop-and-upsetting-videos/ · Metricool: https://metricool.com/youtube-ai-content-rules-monetization/
9. SocialCounts: https://socialcounts.org/youtube-live-subscriber-count/UCWjkP53SduCLfH61uYl8_eA · https://socialcounts.org/youtube-live-subscriber-count/UCj5LSbbMjczTInjlL-e42fQ
10. Duración de Shorts: https://piktochart.com/blog/how-long-youtube-shorts/ · https://www.toptal.com/creator/post/youtube-shorts-length
11. Toptal, cómo entrar a TikTok Creator Rewards: https://www.toptal.com/creator/post/how-to-join-the-tiktok-creator-rewards-program · Metricool: https://metricool.com/tiktok-creator-fund/
12. Meta, ayuda sobre etiquetas de IA: https://www.meta.com/help/artificial-intelligence/1783222608822690/ · Instagram: https://help.instagram.com/761121959519495
13. Buffer, monetización de Shorts: https://buffer.com/resources/youtube-shorts-monetization/ · AIR: https://air.io/en/youtube-glossary/what-is-youtube-shorts-monetization-shorts-fund-vs-ad-revenue-share
14. Cambio en el conteo de vistas de Shorts (31/03/2025): https://www.mediapost.com/publications/article/404562/youtube-changes-how-it-counts-shorts-views.html · https://www.digitalmusicnews.com/2025/03/27/how-does-youtube-shorts-count-views-2025/ · https://support.google.com/youtube/answer/10059070
15. Resúmenes de horarios: https://clipspeed.ai/blog/best-posting-times-social-media.html · https://www.socialcrawl.dev/blog/best-time-to-post-social-media-2026.md · https://fliki.ai/blog/best-time-to-post-reels-on-instagram
16. Buffer, "How to Grow on Social Media in 2026": https://buffer.com/resources/creator-growth-playbook · "How Often Should You Post on Instagram in 2026": https://buffer.com/resources/how-often-to-post-on-instagram/
17. Instagram, originalidad y agregadores (30/04/2026): https://techcrunch.com/2026/04/30/instagram-restricts-reach-of-content-aggregators-in-new-crackdown/ · https://creators.instagram.com/blog/rewarding-original-creators-on-instagram · https://petapixel.com/2026/04/30/new-instagram-policies-target-reposted-content/
18. Frecuencia en TikTok (fuentes secundarias): https://nestscale.com/blog/how-often-should-you-post-on-tiktok-to-grow.html · https://socialk.it/en/blog/tiktok-posting-frequency-guide
19. Meta, límite de publicación de contenido: https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/ig-user/content_publishing_limit · https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/content-publishing
20. TikTok for Developers, Direct Post: https://developers.tiktok.com/doc/content-posting-api-reference-direct-post · https://developers.tiktok.com/doc/content-sharing-guidelines
21. Google, `videos.insert`: https://developers.google.com/youtube/v3/docs/videos/insert
22. Trial Reels: https://techcrunch.com/?p=2787261 · https://publer.com/blog/instagram-trial-reels-guide/
23. YouTube Blog, "Improving AI labels for viewers and creators": https://blog.youtube/news-and-events/improving-ai-labels-viewers-creators/ · https://www.androidcentral.com/apps-software/youtube-ai-synthetic-content-disclosure
24. TikTok Newsroom, etiquetas de contenido generado por IA: https://newsroom.tiktok.com/new-labels-for-disclosing-ai-generated-content?lang=en · C2PA: https://www.maginative.com/article/tiktok-partners-with-c2pa-to-label-ai-generated-content-and-promote-media-literacy/
25. PPC Land, aclaración de "contenido inauténtico": https://ppc.land/youtube-clarifies-inauthentic-content-policy-changes/ · Gulf News: https://gulfnews.com/technology/youtube-updates-monetisation-policies-ai-and-repetitive-content-ban-begins-july-15-1.500192660
26. Higgsfield, centro de ayuda sobre propiedad y uso comercial: https://higgsfield.ai/creator-hub/help-center/account/who-owns-my-generations-and-can-i-use-them-commercially
27. YouTube, Programa de socios y requisitos: https://support.google.com/youtube/answer/72851?hl=en · https://support.google.com/youtube/answer/13429240
28. YouTube, países donde está disponible el Programa de socios (incluye Perú): https://support.google.com/youtube/answer/7101720?hl=en
29. vidIQ, requisitos del YPP en 2026: https://vidiq.com/blog/post/monetize-youtube-guide/
30. TikTok, programas por país (Effect Creator Rewards sí incluye Perú; no es el programa de videos): https://www.tubefilter.com/2024/03/13/tiktok-effect-creator-rewards-monetization-program-requirements/
31. Monetización en Facebook: https://socialbee.com/blog/monetize-facebook-reels/ · https://kompozy.io/creator-growth/facebook-reels-monetization-2026
