# Catálogo editorial y derechos

Bounded context responsable de qué contenido existe públicamente, cómo se
presenta y bajo qué reglas puede mostrarse (por edad, territorio o moderación).

No gestiona el archivo técnico del video ni decide cómo se reproduce — eso
corresponde al contexto de Publicación y distribución de contenido.

## Decisiones de diseño

Decisión 1: Las restricciones se guardan como historial, no como un campo que se sobreescribe: RF-C6 pide tres cosas: aplicar restricciones, mantener el historial y consultarel estado actual. En vez de guardar "el estado actual" y "el historial" como dosestructuras separadas, modelamos cada restricción como un registro permanente confecha de aplicación y fecha de levantamiento. Los registros sin fecha de levantamiento son los activos ahora mismo. Los que sí la tienen son el historial. Una sola estructura resuelve las tres viñetas sin duplicar datos.

Decisión 2: El contenido no tiene une stado "restringido": El campo Status de contenido editorial tiene 3 valores: borrador, publicado y despublicado, las restricciones no agregan un cuarto valor porque un contenido puede estar publicado para la mayoría del mundo y restringido para un grupo especifico.

Decisión 3: El orden de una playlist es implícito, sin campo numérico para las posiciones: RF-C4 pide poder reordenar ítems sin especificar cómo se guarda el orden internamente. Decidimos que el orden lo da la posición de cada ítem dentro del arreglo, sin un campo numérico separado. Si cada ítem tuviera su propio númerode posición, reordenar obligaría a renumerar todos los ítems siguientes cada vez que se mueve uno. Guardando directamente la lista en el orden correcto, reordenar se reduce a reemplazar la lista completa por una nueva.

Decisión 4: Vincular el video y definir la metadata es una operación: RF-C2 presenta en dos viñetas separadas "vincular un asset técnico" y "definir título, descripción, tags, categoría, miniatura". Decidimos unirlas en un solo endpoint POST /content porque ambas cosas ocurren siempre juntas en la práctica. Nadie vincula un video técnico sin intención de describirlo editorialmente al mismo tiempo. Separar las hubiera creado un estado intermedio sin utilidad real.

Decisión 5: Catálogo valida que el video esté listo antes de crear un borrador: Al crear contenido editorial, el videoAssetId debe corresponder a un video que Publicación ya notificó como técnicamente listo. Esta validación se hace contra un registro interno que Catálogo construye a partir de los eventos que recibe, no consultando a Publicación en tiempo real. Así, si Publicación estuviera caída
en ese momento, Catálogo puede seguir funcionando sin bloquearse.

Decisión 6: Actualizaciones con PATCH, no PUT (aunque usted profe nos recomendara usar sólo PUT, para efectos de la tarea usamos PATCH en esta API): Tanto el canal como el contenido se actualizan con PATCH. Mantuvimos la misma convención que ya aplicamos en Publicación, buscando consistencia entre las dos APIs del equipo. PUT obligaría a reenviar el objeto completo en cada cambio, con el riesgo de pisar accidentalmente campos que no se querían tocar.

Decisión 7: Despublicar y ocultar son la misma acción: RF-C3 menciona "despublicar u ocultar contenido" en un solo requisito, usando la "o" como si fueran intercambiables. Decidimos tratarlos como la misma operación porque el resultado para el espectador es idéntico: el contenido deja de ser visible. Crear dos endpoints para una distinción que el enunciado no aclara hubiera sumado complejidad sin un motivo concreto.

Decisión 8: DELETE sí se usa para quitar items de una playlist: En Publicación evitamos DELETE para cancelar una subida porque cancelar no borra nada, solo cambia un estado. Acá es distinto: quitar un ítem de una playlist borra de verdad esa relación de pertenencia. El contenido en sí no desaparece, pero su membresía en esa colección sí. No hay ningún requisito de conservar historial de qué estuvo en una playlist, así que un borrado directo es la elección correcta.

Decisión 9: Levantar una reestricción no la borra, la marca con una fecha: Para retirar una restricción usamos POST /restrictions/{id}/lift en vez de DELETE. Esta decisión es el contraste directo de la anterior: ahí borrábamos porque no había historial que conservar; acá no podemos borrar porque RF-C6 pide explícitamente mantener el historial de restricciones. Levantar una restricción solo completa su fecha de levantamiento en el registro existente, sin eliminarlo.

Decisión 10: Reordenar una playlist reemplaza el orden completo: PUT /playlists/{id}/items recibe la lista completa de IDs en el nuevo orden, en vez de mover los ítems de a uno. Pensamos en cómo funciona arrastar y soltar en una interfaz real: cuando el usuario termina de reordenar, la pantalla ya conoce el orden final completo. Mandar ese resultado de una sola vez es más simple y evita conflictos si dos personas reordenan al mismo tiempo.

Decisión 11: Resolver una reclamación es un único endpoint con tres resultados: RF-C5 menciona tres desenlaces posibles: aceptar, disputar, retirar. Los modelamos como un solo endpoint POST/claims/{id}/resolve con un campo "resolution" que indica el resultado. Los tres desenlaces son la misma acción de negocio — resolver la reclamación — con un valor distinto al final. Crear tres endpoints separados hubiera triplicado las puertas para representar una sola decisión que ocurre en un único momento.

Decisión 12: La miniatura es una URL, sin mecanismo para subir la imagen: RF-C2 pide poder definir una miniatura sin especificar cómo se carga. Decidimos tratarla como un campo de texto con una URL, asumiendo que la imagen ya está alojada en algún lugar externo. Construir un sistema de subida de imágenes dentro de Catálogo hubiera duplicado una responsabilidad de manejo de archivos que en espíritu le corresponde al contexto de Publicación.

Decisión 13: Endpoints de consulta se agregan por necesidad: Varias puertas no surgen de ninguna viñeta explícita del enunciado, pero son necesarias para que el resto funcione: GET /content/{id} para poder mostrar el formulario de edición, GET /channels/{id}/playlists para poder navegar a una playlist desde el canal, GET /content/{id}/claims para poder revisar una reclamación antes de resolverla. El enunciado describe el comportamiento de negocio, no cada pieza de soporte necesaria para que ese comportamiento funcione.

Decisión 14: Aceptar una reclamación no aplica una restricción de manera automática: RF-C5 dice "el sistema debe poder restringir un contenido como consecuencia de una reclamación". Aplicamos el mismo criterio de lectura que usamos en Publicación con convert-to-vod: "debe poder" describe una capacidad disponible, no algo que pasa solo. Aceptar una reclamación y aplicar una restricción son dos pasos separados y deliberados. Quien modera elige explícitamente qué restricción aplicar y de qué tipo, sin que el sistema lo decida por su cuenta.

Decisión 15: País y edad los aporta quién consulta, no los resuelve catálogo: GET /content/{id}/visibility recibe "country" y "age" como parámetros. Catálogo no intenta buscar el perfil del espectador por su cuenta porque la identidad del usuario es una plataforma compartida fuera del alcance de este bounded context (sección 2.2 del enunciado). Catálogo solo decide si su contenido es visible dadas esas condiciones — no quién es el espectador.

Decisión 16: Nuevo rol "rights entity" para quien presenta reclamaciones (mi segunda fav): RF-C5 no especifica quién puede registrar una reclamación. Introdujimos un tercer rol además de "creator" y "viewer". Lo llamamos "rights_entity" y no simplemente "entidad" porque ese término ya se usa en todo el proyecto para referirse a las fichas del modelo de datos. Agregar ambigüedad en la presentación oral hubiera sido un error evitable con solo elegir un nombre más específico.
