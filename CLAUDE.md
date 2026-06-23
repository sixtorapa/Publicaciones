# Procedimiento — agente semanal de "Publicaciones"

Este repo alimenta una rutina en la nube que se ejecuta una vez por semana
(lunes, ~9:00 Europe/Madrid) y propone ideas de post para LinkedIn a partir
de investigación real sobre IA/AI Engineering (papers, lanzamientos,
tendencias, debates) conectada con la perspectiva propia de Sixto Ramírez
Parras (`perfil.md`), evitando repetir ángulos ya usados (`historial.md`).

**Enfoque del contenido — híbrido, no autobiográfico:** cada post parte de
algo real que está pasando en el mercado/la investigación de IA (no de un
proyecto propio), y lo conecta con la experiencia o el punto de vista de
Sixto. No son posts sobre "lo que hice yo" — son posts sobre "esto está
pasando en el sector, y esto es lo que pienso/lo que he visto que confirma o
contradice esto en mi propia experiencia". El perfil (`perfil.md`) se usa
como fuente de opinión/contraste, no como tema principal.

No hay forma de leer LinkedIn por API (bloquea scraping) ni de enviar correos
directamente por Gmail MCP (solo `create_draft`, no hay "send"). El diseño
usa esa limitación como mecanismo de feedback: el propio acto de "Enviar" el
borrador, hecho por Sixto, es la señal de qué propuesta eligió.

## Ciclo de cada ejecución

1. **Revisar la ronda anterior antes de proponer nada nuevo.**
   - Busca en Gmail (`search_threads`) el hilo más reciente con asunto
     "Ideas de post — semana del ..." o etiqueta `Publicaciones`.
   - Si no existe todavía la etiqueta `Publicaciones`, créala (`create_label`)
     — solo la primera vez.
   - Determina el estado con `get_thread`:
     - Aparece en **Enviados** → se considera "publicado". Registra el texto
       final tal cual quedó (puede que Sixto lo haya editado).
     - Sigue **solo en Borradores**, sin tocar → "no usado" (ese ángulo no
       convenció).
     - **No se encuentra** (lo borró) → "descartado".

2. **Actualizar `historial.md`.**
   - Añade una entrada con fecha, resultado de la ronda anterior y texto
     final si hubo publicación.
   - Intenta hacer commit + push de ese cambio al repo, pero esto es
     "mejor esfuerzo": el entorno en la nube puede no tener permiso de
     escritura sobre el repo (solo lectura, por ser público). Si el push
     falla, no es un error fatal — sigue con el resto del procedimiento
     igualmente. La fuente de verdad real del historial es Gmail (los
     hilos etiquetados `Publicaciones`), no este fichero.
   - No edites `perfil.md` automáticamente — ese fichero lo amplía Sixto a
     mano cuando quiere añadir contexto nuevo.

3. **Investigar (WebSearch/WebFetch).**
   - Busca 2-3 cosas reales y recientes (últimas ~2 semanas; si no hay nada
     reciente bueno, amplía el rango antes que inventar) en el espacio de
     AI Engineering / LLMs / RAG / agentes: papers relevantes, lanzamientos
     de producto, benchmarks, debates del sector, informes de mercado.
   - Para cada hallazgo, guarda: fuente (nombre + URL real), de qué trata en
     2-3 frases, y si la fuente tiene una imagen/gráfico/diagrama público
     que ilustre el punto (URL directa de esa imagen, no inventada).
   - No repitas fuentes/temas ya usados en rondas anteriores según
     `historial.md`.

4. **Generar las ideas nuevas.**
   - Lee `perfil.md` (perfil + trayectoria + proyectos) y el `historial.md`
     ya actualizado.
   - Redacta **2-3 propuestas de post completas** (texto listo para copiar a
     LinkedIn, no solo un titular). Cada una parte de UNO de los hallazgos
     del paso 3 (cítalo: qué es y de dónde sale) y lo conecta con la
     experiencia/opinión propia de Sixto desde `perfil.md` — de acuerdo o en
     contraste, pero con argumento concreto, no una mención de pasada.
   - No conviertas el post en autobiografía: el hallazgo de mercado es el
     protagonista, la experiencia propia es el contraste/aval.
   - Varía el ángulo entre las 2-3 propuestas (un paper/benchmark, un
     lanzamiento de producto, una tendencia/debate) sin repetir ángulos que
     ya aparezcan en el historial.
   - Tono: profesional pero directo, sin venderse de forma artificial; cifras
     y argumentos concretos, no generalidades.
   - Para cada propuesta, si encontraste una imagen/gráfico público real en
     el paso 3 que la ilustre, añade una línea: "Imagen sugerida: [URL
     directa] — descárgala y súbela tú a LinkedIn." Si no hay ninguna imagen
     real aplicable, no inventes una ni la menciones.

5. **Crear el borrador en Gmail.**
   - Un único borrador (`create_draft`) a `sixtorapa@gmail.com`, asunto
     "Ideas de post — semana del [fecha]", cuerpo con las opciones numeradas,
     cada una con su fuente citada y su imagen sugerida si aplica.
   - Al final del cuerpo: *"Borra las que no uses, edita la elegida si
     quieres, y dale a Enviar."*
   - Etiqueta el mensaje/hilo con `Publicaciones` (`label_message` /
     `label_thread`) para que se pueda encontrar fácil la próxima ronda.

6. **Avisar.**
   - Un borrador solo no notifica nada — manda un `PushNotification` tipo
     "Ideas de post de la semana listas en Borradores de Gmail".

7. Cierra habiendo creado el borrador y enviado el aviso — eso es lo
   imprescindible. El commit del historial es un extra, no un requisito
   para considerar la ronda completada.
