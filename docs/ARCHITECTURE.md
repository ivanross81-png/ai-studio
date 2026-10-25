# Arquitectura inicial

## Enfoque
Monolito modular primero; frontend independiente; PostgreSQL como fuente de verdad de metadatos; object storage para binarios; jobs largos asíncronos.

```text
Browser -> Next.js -> FastAPI -> PostgreSQL
                           |---- Redis queue -> workers -> Video/LLM APIs
                           |                         |---- FFmpeg render
                           |---- S3-compatible object storage
```

## Módulos API
- `projects`: proyectos y permisos.
- `story`: biblia, personajes, relaciones, episodios y versiones.
- `storyboard`: escenas, planos, prompts y continuidad.
- `generations`: jobs, estados, retries, costos.
- `providers`: contratos y adaptadores.
- `assets`: metadatos y URLs firmadas.
- `editing`: secuencias, pistas, clips y timeline.
- `rendering`: export jobs y presets.
- `audit`: operaciones de impacto/costo.

## Entidades conceptuales
`Project`, `StoryBible`, `Character`, `Location`, `Episode`, `Scene`, `Shot`, `GenerationJob`, `Asset`, `EditSequence`, `TimelineTrack`, `TimelineClip`, `RenderJob`. Diseñar cardinalidades, restricciones e índices antes de migraciones.

## Flujo de generación
API valida request y presupuesto, persiste `queued` con idempotency key, worker marca `running`, adaptador envía solicitud, guarda provider job ID y consulta estado según capacidades documentadas. Resultado validado va a object storage y job termina `succeeded`/`failed`.

Estados sugeridos: `queued`, `running`, `succeeded`, `failed`, `cancel_requested`, `cancelled`, `expired`. Cancelar internamente no implica que el proveedor detenga una ejecución facturable.

## Contrato de proveedor
Definir capacidades explícitas y métodos equivalentes a `submit_generation`, `get_status`, `fetch_result` y cancelación solo si está soportada. No asumir que todos los proveedores aceptan duración, audio, referencias, semillas o resoluciones iguales. LTX API será candidato tras revisar docs oficiales, límites, precios, retención y términos vigentes. Mantener proveedor fake.

## Render
La timeline almacena clips/pistas ordenados. El worker traduce una secuencia validada a argumentos FFmpeg como lista, no strings de shell. Ejecutar sin privilegios, con límites de CPU/memoria/tiempo, directorios temporales aislados y limpieza. MVP: trim, orden, audio, subtítulos simples y MP4.

## Seguridad
Secretos solo en backend; URLs firmadas breves; límites de duración/tamaño/tipo; autorización por recurso; protección SSRF; FFmpeg aislado; auditoría de acciones costosas; retención/eliminación por definir.

## Entorno local
Docker Compose: web, API, worker, PostgreSQL, Redis y MinIO. El proveedor fake debe permitir un recorrido completo sin credenciales reales.

## Pruebas
Unitarias de dominio/estados/timeline; contratos de proveedor; integración de DB/cola/storage; E2E crear proyecto -> clip fake -> timeline -> export.