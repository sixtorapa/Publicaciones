# Procedimiento — agente semanal de "Publicaciones"

Este repo alimenta una rutina en la nube que se ejecuta una vez por semana
(lunes, ~9:00 Europe/Madrid) y propone ideas de post para LinkedIn a partir
del perfil de Sixto Ramírez Parras (`perfil.md`), evitando repetir ángulos ya
usados (`historial.md`).

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
   - Haz commit + push de ese cambio al repo.
   - No edites `perfil.md` automáticamente — ese fichero lo amplía Sixto a
     mano cuando quiere añadir contexto nuevo.

3. **Generar las ideas nuevas.**
   - Lee `perfil.md` (perfil + trayectoria + proyectos) y el `historial.md`
     ya actualizado.
   - Redacta **2-3 propuestas de post completas** (texto listo para copiar a
     LinkedIn, no solo un titular), variando el ángulo: una técnica/decisión
     de ingeniería concreta, una de aprendizaje/reflexión, una de resultado
     con cifras reales — sin repetir ángulos que ya aparezcan en el
     historial.
   - Tono: profesional pero directo, sin venderse de forma artificial;
     anécdotas concretas con cifras/decisiones reales, no generalidades.

4. **Crear el borrador en Gmail.**
   - Un único borrador (`create_draft`) a `sixtorapa@gmail.com`, asunto
     "Ideas de post — semana del [fecha]", cuerpo con las opciones numeradas.
   - Al final del cuerpo: *"Borra las que no uses, edita la elegida si
     quieres, y dale a Enviar."*
   - Etiqueta el mensaje/hilo con `Publicaciones` (`label_message` /
     `label_thread`) para que se pueda encontrar fácil la próxima ronda.

5. **Avisar.**
   - Un borrador solo no notifica nada — manda un `PushNotification` tipo
     "Ideas de post de la semana listas en Borradores de Gmail".

6. Cierra dejando comiteado y empujado `historial.md` con la entrada de esta
   ronda (estado "pendiente" hasta la próxima revisión).
