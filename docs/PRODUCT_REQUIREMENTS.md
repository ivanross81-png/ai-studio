# Requisitos de producto

## Visión
Aplicación web para desarrollar historias audiovisuales serializadas —incluidas novelas con lenguaje audiovisual latinoamericano— desde una idea hasta un montaje exportable. La IA asiste; el usuario conserva el control editorial.

## Flujo principal
1. Crear proyecto y definir formato, idioma, género, tono y duración.
2. Generar/editar premisa, sinopsis, biblia narrativa, personajes, relaciones, localizaciones y arcos.
3. Crear episodios, escenas, diálogos y lista de planos.
4. Definir referencias visuales/voz y restricciones de continuidad.
5. Generar clips por API en jobs asíncronos.
6. Revisar, aceptar, descartar o regenerar un plano sin invalidar otros.
7. Montar clips, audio, música y subtítulos en timeline.
8. Renderizar revisión y exportar máster; comparar versiones.

## MVP incluido
- CRUD de proyectos y biblia narrativa versionada.
- Personajes, localizaciones, episodios, escenas, planos y relaciones explícitas.
- Estados de borrador/revisión/aprobado.
- Contrato de proveedor audiovisual y proveedor fake para tests.
- Jobs persistentes, reintentos seguros, errores normalizados y costos si el proveedor los expone.
- Biblioteca de assets y trazabilidad de prompts/modelo/parámetros.
- Timeline web básico: ordenar/recortar clips, pistas de video/audio, subtítulos simples y exportación FFmpeg.
- Docker Compose sin necesidad de API keys reales.

## Fuera del MVP
Colaboración en tiempo real, entrenamiento de modelos, GPU local, NLE profesional completo, publicación directa en redes y aprobación autónoma de contenido costoso.

## Requisitos no funcionales
Seguridad, autorización, límites de costo, trabajos recuperables, logs estructurados, almacenamiento S3-compatible, accesibilidad y operaciones largas fuera de requests HTTP.

## Aceptación
- Proyecto e historia persisten entre reinicios.
- Cada plano pertenece a una escena; escenas referencian personajes/localizaciones.
- Jobs sobreviven reinicios y la idempotencia evita duplicados.
- Claves de proveedor nunca llegan al navegador.
- Un clip aceptado se puede montar y exportar junto con audio.
- Regenerar una escena no modifica versiones aprobadas no relacionadas.
- Tests cubren éxito, fallo de proveedor, timeout y retry.

## Riesgos
Licencias/API/precios cambiantes, continuidad audiovisual imperfecta, costos de regeneración y complejidad de codecs/render.