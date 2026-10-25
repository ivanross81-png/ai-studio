# Novela AI Studio

Plataforma web de producción audiovisual narrativa con IA: biblia, personajes, escenas, generación de clips por API, composición de timeline y exportación FFmpeg.

Estado: documentación fundacional; la aplicación aún no está implementada.

## Stack propuesto
- Web: Next.js + TypeScript
- API: Python + FastAPI
- Datos: PostgreSQL
- Jobs: Redis + workers
- Media: almacenamiento S3-compatible
- Montaje: FFmpeg en worker aislado
- Desarrollo: Docker Compose

## Documentación
- `AGENTS.md`: instrucciones para OpenCode
- `docs/PRODUCT_REQUIREMENTS.md`: requisitos y alcance MVP
- `docs/ARCHITECTURE.md`: arquitectura y modelo de dominio
- `docs/ROADMAP.md`: fases de implementación
- `docs/LTX_DESKTOP_ASSESSMENT.md`: evaluación de LTX Desktop
- `docs/adr/0001-initial-architecture.md`: decisión arquitectónica

No se requiere GPU local. LTX es un proveedor candidato sujeto a validar API, capacidades, costos y licencias. No integrar código de LTX Desktop sin revisar acoplamiento con Electron y términos de licencia.