# Práctica 3.5 – Página web "Noticias climáticas del día"

Reporte técnico de la práctica **3.5-P1 Página web "Noticias climáticas del día"**, de la materia Desarrollo Sustentable (ACD-0908).

| Campo | Detalle |
|---|---|
| Alumno | Cesar Jesus Gamez Mendoza (trabajo individual) |
| Carrera | Ingeniería en Sistemas Computacionales, Tecnológico Nacional de México, Campus Mazatlán |
| Unidad y tema | 3. Escenario socio-cultural, 3.5 Difusión de la conciencia ambiental mediante automatización de la información |
| Herramientas | Make (Webhooks, RSS, Text aggregator, Make AI Agent, Webhook Response) y GitHub Pages |
| Página publicada | https://cesargm-max.github.io/practica-3-5-noticias-climaticas/ |
| Fecha de las pruebas | 30 de octubre de 2026 |

## Cómo funciona

El usuario elige un tema en la página y presiona "Ver noticias". La página manda el tema a Make, Make busca las cinco noticias más recientes en Google Noticias, las junta en un solo texto, una IA las resume y la respuesta regresa a la página, que la muestra con los enlaces a las fuentes.

Flujo: **Página web → Webhook → RSS → Text aggregator → Make AI Agent → Webhook response → Página web**

### Escenario `Noticias_Climaticas_Cesar`

| N.º | Módulo | Configuración | Función |
|---|---|---|---|
| 1 | Webhooks, Custom webhook | Webhook `Noticias_Climaticas_Cesar` | Recibe el campo `tema` que envía la página con `fetch` y URLSearchParams. |
| 2 | RSS, Retrieve RSS feed items | URL `https://news.google.com/rss/search?q={{encodeURL(1.tema)}}&hl=es-419&gl=MX&ceid=MX:es-419`, máximo 5 resultados | Lee las noticias del tema en Google Noticias México. `encodeURL` permite temas con espacios y acentos. |
| 3 | Tools, Text aggregator | Fuente: RSS. Texto: `Título: {{2.title}}`, `Fecha: {{2.dateCreated}}`, `Enlace: {{2.url}}` | Junta las cinco noticias en un solo texto. Sin este módulo la IA se ejecutaría cinco veces (una por noticia) y gastaría cinco veces más créditos. |
| 4 | Make AI Agent, Run an agent | Conexión Make's AI Provider (sin API key). Entrada: `Tema: {{1.tema}}` y `Noticias: {{3.text}}` | Resume las noticias con un prompt que le prohíbe inventar datos y le exige un formato fijo: resumen del día, noticias con el motivo por el que importan y una acción concreta para un estudiante. |
| 5 | Webhooks, Webhook response | Estado 200, cuerpo = respuesta del agente, cabeceras `Content-Type: text/plain; charset=utf-8` y `Access-Control-Allow-Origin: *` | Devuelve el texto a la página. La cabecera de origen permite que la página en GitHub Pages lea la respuesta. |

El escenario se activó con **Immediately as data arrives**, por lo que responde en cuanto llega una consulta (entre 16 y 35 segundos en las pruebas).

### La página (`index.html`)

Es un solo archivo con HTML, CSS y JavaScript, sin librerías externas. Tiene un título con una explicación, una lista desplegable "Tema" con 6 opciones, el botón "Ver noticias", el mensaje "Buscando y resumiendo noticias..." mientras espera, un recuadro que respeta los saltos de línea, enlaces clicables en las líneas que empiezan con http, la fecha del día y el mensaje de error "No se pudieron obtener las noticias, intenta de nuevo." Se publicó con GitHub Pages desde la rama `main`.

### Temas elegidos

| Bloque | Tema |
|---|---|
| A. Cambio climático | olas de calor México |
| B. Impacto humano | deforestación México |
| C. Consumo | plásticos de un solo uso |
| D. Energía y soluciones | energías renovables México |
| E. Local | sequía Sinaloa |
| E. Local | manglares Sinaloa |

Los 6 temas devolvieron 5 noticias con su enlace, sin errores.

---

## Comparación de 3 temas con la fuente original

| Tema | Noticia | Lo que dijo la IA | Lo que dice la fuente | ¿Es fiel? |
|---|---|---|---|---|
| Manglares Sinaloa | "Planta de amoniaco 'dañaría 22,500 hectáreas de manglares' en Sinaloa", La Jornada (17 de junio de 2026) | Señala el tamaño de la superficie de manglar que podría verse afectada por un proyecto industrial. | La Red Mexicana de Afectados por la Minería asegura que, si la planta de Topolobampo opera, afectará unas 22 mil 500 hectáreas de manglares, y pide revocar la autorización de impacto ambiental. | Sí, con matiz. La cifra y el proyecto son correctos, pero la IA no dice que es una afirmación de una organización y no un dato oficial. En otra consulta agregó "servicios ecosistémicos", frase que la nota no menciona. |
| Sequía Sinaloa | "Sinaloa reduce sequía, pero 26% del territorio sigue anormalmente seco", Noroeste (26 de julio de 2026) | Indica mejoría parcial, pero más de una cuarta parte del estado aún enfrenta condiciones secas que requieren atención. | El Monitor Nacional de Sequía de la Conagua reporta afectaciones en 10 municipios, 26 % del territorio anormalmente seco, embalses con niveles bajos y una reducción frente a julio de 2024. | Sí en lo que pude leer (título y entrada); el resto de la nota pide registro. "Que requieren atención" es interpretación de la IA. |
| Deforestación México | "Aprueban 85.3 mdd del Fondo Verde para el Clima a México por frenar la deforestación", La Jornada (28 de octubre de 2026) | Confirma la aprobación del financiamiento internacional para medidas de reducción de deforestación en México. | El Fondo Verde para el Clima aprobó 85.3 millones de dólares como pago por resultados: México redujo 12.9 millones de toneladas de CO2 equivalente entre 2018 y 2019, y el dinero se usará durante seis años con la FAO en conservación y ganadería sostenible. | Sí, con matiz. El monto y la aprobación son correctos, pero el pago es por una reducción ya lograda y la IA lo presenta como si fuera solo para medidas nuevas. |

### Créditos consumidos por consulta

En el historial de ejecuciones de Make, cada consulta usó **5 operaciones** (webhook, RSS, Text aggregator, Make AI Agent y Webhook response) y consumió **entre 8.8 y 10.9 créditos**, con un promedio de **9.7 créditos por consulta**. En total se hicieron 11 consultas (6 temas, repeticiones y la prueba del webhook), con unos 107 créditos.

---

## Preguntas de reflexión

**1. ¿Qué tema tuvo más noticias recientes y cuál menos? ¿Qué dice eso sobre la atención que los medios dan a cada problema?**

Con el límite de 5 resultados todos los temas devolvieron 5 noticias, así que el número no sirve para compararlos; lo que sí se nota es qué tan recientes eran. La más reciente fue la de deforestación en México (Fondo Verde, 28 de octubre de 2026), repetida por varios medios (El Norte, MURAL y El Vigía publicaron el mismo titular), mientras que en manglares y sequía de Sinaloa aparecieron notas de junio y julio de 2026, de hace tres o cuatro meses. En plásticos de un solo uso las primeras noticias (de El Tiempo y La Nación) no parecían ser de México, sino de otros países de Latinoamérica. Esto sugiere que los medios cubren sobre todo lo que tiene un evento noticioso (un anuncio, un financiamiento o una cumbre), y mucho menos los problemas lentos y cotidianos, como la sequía local o el plástico, que casi nunca son "noticia del día" aunque afecten más a la gente. Para saber con precisión cuál tema tiene más cobertura habría que subir el límite de resultados y contar cuántas noticias tienen fecha de la última semana.

**2. ¿Los resúmenes de la IA fueron fieles a las noticias originales? ¿Qué riesgos tiene difundir información resumida por IA sin verificarla?**

En las tres noticias comparadas el resumen coincidió con la fuente en lo esencial (cifras, lugares y proyectos), pero en dos casos le faltó un matiz importante: no dijo que la cifra de 22,500 hectáreas es una afirmación de una organización y no un dato oficial, y presentó el pago del Fondo Verde como si fuera para acciones nuevas cuando es por resultados ya logrados. En otra consulta también agregó una frase ("servicios ecosistémicos") que la nota no decía. El riesgo es que estos pequeños cambios se repitan en miles de lecturas y terminen convirtiendo una opinión o un dato parcial en un hecho, además de que la IA solo resume los titulares y fragmentos que recibe del RSS, no el artículo completo. Por eso la página incluye el enlace de cada noticia y un aviso para verificar en la fuente, y el prompt le pide usar únicamente la información recibida.

**3. ¿Qué tan presentes estuvieron los temas locales (Sinaloa, Mazatlán) frente a los nacionales o globales?**

Los dos temas locales estuvieron bien presentes: sequía y manglares de Sinaloa devolvieron 5 noticias cada uno, de medios como Noroeste, El Sol de México, Revista Espejo y ADN Informativo, además de notas de La Jornada e Infobae sobre Topolobampo. Sin embargo, casi todas hablaban de Culiacán, Ahome o Topolobampo, y solo apareció una mención a Mazatlán en una de las consultas ("propuestas de intervención local en Mazatlán"). Es decir, el estado de Sinaloa tiene cobertura, pero lo que le pasa a nuestro puerto sale poco. Los temas nacionales tuvieron notas de medios grandes (Milenio, El Economista, La Jornada, Gaceta UNAM) y los globales se mezclaron con noticias de otros países, como pasó con los plásticos.

**4. ¿A quién en su comunidad le sería útil esta página y cómo la harían llegar a esas personas?**

Le sería útil a estudiantes y maestros de las carreras del Tecnológico que necesitan fuentes recientes para sus tareas, a pescadores, productores y personas de la zona costera de Mazatlán interesadas en la sequía o en los manglares, y a cualquier persona que no tiene tiempo de leer varias noticias. La haría llegar con el enlace de GitHub Pages en los grupos de WhatsApp de la escuela, con un código QR en carteles en el campus, y pidiendo a la coordinación de la carrera que lo comparta en sus redes. Para las personas que no usan computadora, la mejor opción sería mandar el resumen por Telegram o WhatsApp.

**5. ¿Qué otra herramienta de automatización podría usar un ingeniero en sistemas para difundir la conciencia ambiental?**

Un ingeniero en sistemas podría programar el mismo escenario de Make para que cada mañana mande por correo o Telegram un boletín con el resumen de los temas elegidos, sin que nadie tenga que entrar a la página. También podría conectar una API de calidad del aire o de clima para enviar alertas cuando haya contaminación o calor extremo, usar un formulario con una hoja de cálculo para recibir reportes ciudadanos (como el buzón de la práctica 3.1) o armar un chatbot que oriente sobre separación de residuos (como el clasificador de la práctica 3.3). Herramientas como n8n o Zapier hacen lo mismo que Make, y con una hoja de cálculo se pueden llevar estadísticas de qué temas consulta más la gente.
