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
