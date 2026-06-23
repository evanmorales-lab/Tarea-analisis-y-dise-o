# Publicación y distribución de contenido

Bounded context responsable del ciclo de vida técnico del video: subida,
procesamiento, almacenamiento, reproducción y transmisión en vivo.

No gestiona metadata visible (título, descripción, tags) ni decide si un
video es público — eso corresponde al contexto de Catálogo editorial y derechos.

## Decisiones de diseño

Decisión 1: Estados del vídeo al subirlo.
El Requisito RF-P1 nos habla de la subida y RF-P2 nos habla del procesamiento del video por separado, pero creemos que en la práctica son fases del mismo proceso.
Es por esto que decidimos unificar los valores en "status" con los valores "Uploading", "pending", "processing", "ready", "failed" y "uploadCancelled".
Una opción alternativa es tener 2 campos por separado, pero eso haría que se crucen 2 opciones que no pueden ir juntas, como el estado de procesadno cuando el proceso de subida del video esté cancelado.
Al tener sólo el campo "status" con dichos valores se elimina ese problema.

Decisión 2: No hay entidades separadas para subida y procesamiento.
El modelo presenta video, sesión de reproducción y transmisión en vivo, no creamos más entidades aparte para subida ni procesamiento ya que ambas son procesos que le suceden a un video, no son objetos con identidad propia.

Decisión 3: Las calidades tendrán su propio estado.
Cómo sabemos el video se genera en 360p, 720p, 1080p, cada una tendrá su propio estado independiente, esto nos permite que una calidad pueda fallar sin que las demás se bloqueen, un vídeo puede estar disponible en 720p mientras reintentamos el 1080p, en vez de que el video quede como fallido.

Decisión 4: 2 puertas para subida completa y subida por partes.
RF P1 pide soportar ambas, por esto diseñamos 2 endpoints diferentes: "Put/videoassets/{id}/content" para el archivo de video completo y "PUT/videoassets/{id}/parts/{n}" para subida por partes, al separar esto evitamos mezclar 2 flujos de datos distintos en una misma puerta, esto es más fácil de entender y de implementar que juntarlos en un endpoint.
