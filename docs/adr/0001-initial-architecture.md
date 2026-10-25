# ADR 0001: Arquitectura inicial

- Estado: Propuesto
- Fecha: 2026-10-25

## Contexto
Se requiere una plataforma web para narrativa, generación audiovisual remota, almacenamiento, edición de timeline y exportación, sin GPU local.

## Decisión propuesta
- Monolito modular FastAPI + Next.js.
- PostgreSQL para dominio/versiones; Redis y workers para tareas largas.
- Almacenamiento S3-compatible para binarios.
- FFmpeg aislado para render.
- Contratos de proveedor desacoplados y fake provider para CI.
- No usar Electron como base de la aplicación web.

## Consecuencias
Permite probar sin API real y cambiar de proveedor, pero añade complejidad operativa y depende de límites reales de API. La continuidad visual/sonora no puede garantizarse perfectamente.

## Alternativas
- Fork completo de LTX Desktop: no elegido por posible acoplamiento a escritorio.
- Todo en Next.js: no elegido para jobs largos/render.
- Microservicios desde día uno: complejidad prematura.
- GPU local/ComfyUI: fuera del objetivo inicial.

## Pendientes
Elegir biblioteca timeline, worker framework, proveedor inicial, auth/tenancy y licencia del producto.