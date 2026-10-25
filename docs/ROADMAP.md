# Roadmap

## Fase 0 — Descubrimiento
- [ ] Auditar licencias comerciales de modelos, pesos, API y código.
- [ ] Inspeccionar LTX Desktop: timeline, proyecto, FFmpeg, Electron y filesystem.
- [ ] Probar API de video aislada: calidad, costo, latencia, límites y derechos.
- [ ] Evaluar biblioteca de timeline web y registrar ADRs.

## Fase 1 — Scaffold
- [ ] Monorepo `apps/web`, `apps/api`, `apps/worker`, `packages/contracts`, `infra/`.
- [ ] Docker Compose, `.env.example`, lint/typecheck/tests y CI.
- [ ] Health/readiness, logs estructurados y errores uniformes.
- [ ] Proveedor fake determinista; app arranca sin API keys.

## Fase 2 — Narrativa
- [ ] CRUD proyectos, biblia, personajes, localizaciones, episodios, escenas y planos.
- [ ] Versionado y estados de aprobación; reglas de continuidad.

## Fase 3 — Generación
- [ ] Contrato de proveedor, jobs persistentes, idempotencia, cola y worker.
- [ ] Adaptador real tras validar documentación y licencia.
- [ ] Timeouts, retry/backoff, límites y registro de costos.

## Fase 4 — Assets y continuidad
- [ ] Galería por escena/plano; aceptar/rechazar/regenerar.
- [ ] Referencias visuales, historial de prompts y versiones.
- [ ] Mostrar capacidades reales de proveedor.

## Fase 5 — Timeline/exportación
- [ ] Prototipo de timeline web y selección de librería.
- [ ] Pistas, trim, orden, audio, secuencias versionadas.
- [ ] Worker FFmpeg aislado, export MP4 y tests E2E.

## Fase 6 — Beta privada
- [ ] Autenticación/autorización, aislamiento de datos, cuotas y observabilidad.
- [ ] Retención, eliminación, backups y recuperación probados.

## No hacer todavía
Microservicios prematuros, entrenamiento, NLE profesional completo o generación costosa sin presupuesto/confirmación.

## Definición de MVP
Crear historia, estructurar escenas/planos, generar clips por proveedor remoto, revisar/regenerar un plano, montarlo con audio y exportar MP4 conservando versiones y trazabilidad; ejecutar con proveedor fake sin GPU ni credenciales.