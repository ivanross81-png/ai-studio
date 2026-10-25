# Instrucciones para OpenCode

Actuá como ingeniero principal de una plataforma web de producción audiovisual. La documentación del repositorio es la fuente de verdad.

## Antes de cambiar código
1. Leer README, requisitos, arquitectura, roadmap y ADRs relevantes.
2. Inspeccionar el estado real del repo; no asumir módulos existentes.
3. Identificar criterios de aceptación y dependencias; proponer un plan breve si el cambio cruza módulos.

## Reglas obligatorias
- No inventar endpoints, parámetros, capacidades, precios ni licencias de proveedores.
- Verificar documentación oficial vigente antes de integrar LTX y encapsularlo detrás de un contrato interno.
- No exponer claves en frontend, logs, fixtures o commits.
- No requerir GPU local; inferencia audiovisual mediante APIs remotas.
- Tratar LTX Desktop como referencia hasta revisar licencia y acoplamiento con Electron/filesystem.
- Persistir trabajos asíncronos; diseñar idempotencia, timeouts, límites y reintentos acotados.
- Separar dominio narrativo, proveedor, cola, almacenamiento y renderizado.
- Validar autorización en cada acceso a proyectos, jobs y assets.
- Versionar guiones y resultados aprobados; no sobrescribir silenciosamente.
- Agregar tests pertinentes y ejecutar lint, typecheck y tests disponibles.
- No añadir dependencias sin justificar mantenimiento y licencia.
- Al finalizar, informar archivos cambiados, pruebas ejecutadas y limitaciones.

## Convenciones
- TypeScript estricto; Python con type hints y Pydantic.
- Nombres de código en inglés; documentación de producto en español.
- IDs opacos UUID/ULID; fechas UTC en persistencia.
- No acoplar frontend a payloads nativos de proveedores.

## Flujo
Criterios de aceptación -> contratos/estados -> implementación mínima -> tests -> checks -> documentación/ADR -> resumen.