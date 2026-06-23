# Publicación y distribución de contenido

Bounded context responsable del ciclo de vida técnico del video: subida,
procesamiento, almacenamiento, reproducción y transmisión en vivo.

No gestiona metadata visible (título, descripción, tags) ni decide si un
video es público — eso corresponde al contexto de Catálogo editorial y derechos.

## Decisiones de diseño

Decisión 1: Estados del vídeo al subirlo.
El Requisito RF-P1 nos habla de la subida y RF-P2 nos habla del procesamiento del video por separado, pero creemos que en la práctica son fases del mismo proceso.
Es por esto que decidimos unificar los valores en "status" con los valores "Uploading", "pending", "processing", "ready", "failed" y "uploadCancelled".
Una opción alternativa es tener 2 campos por separado, pero eso haría que se crucen 2 opciones que no pueden ir juntas, como el estado de procesando cuando el proceso de subida del video esté cancelado.
Al tener sólo el campo "status" con dichos valores se elimina ese problema.

Decisión 2: No hay entidades separadas para subida y procesamiento.
El modelo presenta video, sesión de reproducción y transmisión en vivo, no creamos más entidades aparte para subida ni procesamiento ya que ambas son procesos que le suceden a un video, no son objetos con identidad propia.

Decisión 3: Las calidades tendrán su propio estado.
Cómo sabemos el video se genera en 360p, 720p, 1080p, cada una tendrá su propio estado independiente, esto nos permite que una calidad pueda fallar sin que las demás se bloqueen, un vídeo puede estar disponible en 720p mientras reintentamos el 1080p, en vez de que el video quede como fallido.

Decisión 4: 2 puertas para subida completa y subida por partes.
RF P1 pide soportar ambas, por esto diseñamos 2 endpoints diferentes: "PUT/videoassets/{id}/content" para el archivo de video completo y "PUT/videoassets/{id}/parts/{n}" para subida por partes, al separar esto evitamos mezclar 2 flujos de datos distintos en una misma puerta, esto es más fácil de entender y de implementar que juntarlos en un endpoint.

Decisión 5: El modo de subida no se debe declarar por el actor.
No vamos a pedir que quién sube el video deba especificar que está subiendo el video completo o por partes, el modo se determina por el endpoint que use después, si lo hacemos declarar antes eso nos haría agregar una validación de la subida para que el que suba el video respete la forma en que eligió subirlo, lo cuál sería como agregar algo innecesario según nuestro diseño.

Decisión 6: Un sólo PATCH cubre progreso y finalización de una sesión.
RF P3 menciona inicio, progreso y finalización como 3 momentos de una sesión de reproducción de un video, el inicio es un POST porque crea un momento nuevo, en el caso del progreso y la finalización comparten el mismo PATCH, para actualizar una sesión existente, crea un tercer endpoint sólo para finalizar hubiera duplicado la lógica sin necesidad en verdad.

Decisión 7: Para las operaciones de cancelar, reintentar, iniciar y pausar usamos "PUT/recurso/{id}/acción" en vez de forzarlas dentro de un HTTP genérico. El REST puro dice que todo debería ser crear/leer/actualizar/borrar, pero en la práctica muchas acciones de negocio no encajan en ese molde."Cancelar una subida" no es lo mismo que "borrar el video". Modelarlo comoacción explícita comunica mejor la intención.
