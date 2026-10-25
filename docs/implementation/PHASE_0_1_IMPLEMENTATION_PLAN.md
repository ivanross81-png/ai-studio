# Plan de implementación Fase 0/1 — AI Studio

- **Estado:** Pendiente de aprobación explícita
- **Fecha:** 2026-10-25
- **Rama de documentación:** `docs/phase-0-1-plan`
- **Commit base:** `9cfbfd2` (`main`)
- **Alcance de este documento:** planificación. No se ha implementado código de aplicación.
- **Fuente de verdad:** `AGENTS.md`, `README.md` y `docs/`.

> Este documento debe aprobarse antes de iniciar la Etapa B (implementación).
> La aprobación de este plan no autoriza por sí sola operaciones destructivas,
> despliegues, publicación de secretos ni consumo de APIs de pago.

---

## 1. Resumen ejecutivo

### Objetivo

Convertir el repositorio, actualmente documental, en un monorepo ejecutable con un
flujo mínimo de extremo a extremo:

1. Crear un proyecto desde la interfaz web.
2. Crear un trabajo de generación audiovisual simulada asociado al proyecto.
3. Persistir el trabajo y encolarlo en Redis.
4. Procesarlo en un worker independiente.
5. Consultar estado y resultado desde la API y la interfaz.
6. Descargar un artefacto claramente identificado como simulado desde MinIO.

### Resultado esperado

- Frontend Next.js funcional y responsive.
- API FastAPI versionada en `/api/v1`, con OpenAPI y health/readiness.
- PostgreSQL con migraciones Alembic versionadas.
- Redis + ARQ + worker persistente.
- MinIO S3-compatible para binarios, con metadatos en PostgreSQL.
- Proveedor `fake` determinista, sin claves, GPU, red ni costes.
- Docker Compose reproducible con health checks y volúmenes.
- Pruebas automatizadas nativas y pruebas de integración opcionales.
- Documentación de instalación, ejecución, pruebas y limitaciones.

### Restricción principal detectada

El entorno local de trabajo **no tiene Docker, Docker Compose, WSL, PostgreSQL ni
Redis instalados**. Es posible implementar y ejecutar las pruebas nativas con
SQLite, fakeredis y almacenamiento en memoria/moto, pero la validación real de
Docker Compose, PostgreSQL, Redis y MinIO queda bloqueada hasta instalar Docker
Desktop o disponer de otro entorno equivalente.

---

## 2. Estado actual del repositorio

### 2.1 Repositorio remoto

- URL: `https://github.com/ivanross81-png/ai-studio`
- Rama por defecto: `main`
- Último commit analizado: `9cfbfd2`
- Historial: 9 commits, todos de documentación.
- Contenido remoto: `.env.example`, `.gitignore`, `AGENTS.md`, `README.md` y `docs/`.

### 2.2 Documentos leídos

- `AGENTS.md`
- `README.md`
- `.env.example`
- `.gitignore`
- `docs/PRODUCT_REQUIREMENTS.md`
- `docs/ARCHITECTURE.md`
- `docs/ROADMAP.md`
- `docs/LTX_DESKTOP_ASSESSMENT.md`
- `docs/adr/0001-initial-architecture.md`

### 2.3 Componentes existentes

| Componente | Estado |
|---|---|
| Frontend | No existe |
| API | No existe |
| Base de datos | No existe |
| Migraciones | No existen |
| Cola/worker | No existen |
| Almacenamiento S3 | No existe |
| Proveedor fake | No existe |
| Docker Compose | No existe |
| Pruebas | No existen |
| CI | No existe |

El `README.md` lo confirma: “la aplicación aún no está implementada”.

### 2.4 Entorno local comprobado

| Herramienta | Versión / estado |
|---|---|
| Sistema operativo | Windows, PowerShell 5.1 |
| Git | 2.54.0 |
| Node.js | 24.15.0 |
| npm | 11.12.1 |
| Python | 3.12.10 |
| uv | 0.12.9 |
| GitHub CLI | Autenticado como `ivanross81-png` |
| Docker / Compose | No instalado |
| WSL | No instalado |
| PostgreSQL local | No disponible |
| Redis local | No disponible |
| FFmpeg | Instalado; no se usará en este hito |

### 2.5 Documentación que condiciona el trabajo

- `docs/PRODUCT_REQUIREMENTS.md`: proveedor fake, jobs persistentes, reintentos,
  almacenamiento S3-compatible y Docker Compose sin API keys reales.
- `docs/ARCHITECTURE.md`: monolito modular, PostgreSQL como fuente de verdad,
  workers para tareas largas, contrato de proveedor desacoplado, MinIO local.
- `docs/ROADMAP.md`: Fase 1 (scaffold) y porciones de Fase 2/3 necesarias para el
  flujo mínimo.
- `docs/adr/0001-initial-architecture.md`: monolito modular, sin Electron, sin
  microservicios prematuros, sin GPU local.
- `docs/LTX_DESKTOP_ASSESSMENT.md`: LTX Desktop es solo referencia futura; no se
  integra código ni dependencias en este hito.
- `AGENTS.md`: TypeScript estricto, Python tipado, UUID/ULID, UTC, separación de
  dominio/proveedor/cola/storage, autorización por recurso, sin secretos.

### 2.6 Brechas y contradicciones detectadas

1. **Alcance del roadmap vs. misión.** La Fase 1 del roadmap es solo scaffold,
   mientras que la misión exige un corte vertical de proyectos y jobs (partes de
   Fases 2 y 3). Se resuelve documentándolo como **corte vertical deliberado**,
   limitado a `Project` y `GenerationJob`.
2. **Autorización.** `AGENTS.md` exige validar autorización en cada acceso, pero
   la autenticación está fuera del alcance. Se implementará una aplicación
   **solo para desarrollo local**, sin auth, con puertos ligados a localhost y una
   limitación de seguridad explícita. No se simulará autenticación.
3. **`packages/contracts`.** El roadmap lo menciona, pero aún no existe una
   necesidad de código compartido en runtime. Se creará únicamente como artefacto
   de contratos OpenAPI + tipos TypeScript generados, sin lógica.
4. **CI.** El roadmap Fase 1 incluye CI; la misión no la exige. Se propone un
   workflow mínimo, no verificable localmente sin push/ejecución remota.
5. **`.env.example` incompleto.** Faltan variables de web, CORS, puertos,
   reintentos, timeouts y worker. Se actualizará durante la implementación.
6. **ADR 0001 en estado “Propuesto”.** No se reescribirá. Se añadirán ADR 0002 y
   0003 para las decisiones nuevas.
7. **MinIO.** La imagen oficial no incluye `curl`. El health check correcto es
   `mc ready local`, con imagen fijada por tag.
8. **FFmpeg.** Está instalado en la máquina, pero no forma parte de este hito. El
   artefacto fake será JSON, no video.

### 2.7 Decisiones existentes que se respetan

- Monolito modular, no microservicios.
- Next.js + TypeScript para web.
- FastAPI + Pydantic para API.
- PostgreSQL como fuente de verdad.
- Redis + workers para tareas largas.
- S3-compatible para binarios.
- Proveedor fake para desarrollo y pruebas.
- Sin Electron, sin GPU local, sin LTX real.
- Código en inglés, documentación de producto en español.
- IDs opacos, fechas UTC.

---

## 3. Alcance

### 3.1 Incluido

- Monorepo `apps/web`, `apps/api`, `apps/worker`, `packages/contracts`, `infra/`.
- Configuración centralizada y validada.
- API `/api/v1` con health, readiness, proyectos y jobs.
- PostgreSQL, SQLAlchemy 2.x, Alembic y migración inicial.
- Redis, ARQ y worker persistente.
- Almacenamiento S3-compatible con MinIO y bucket privado.
- Proveedor fake con contrato tipado y resultado determinista.
- Artefacto JSON simulado, con aviso explícito.
- Frontend mínimo: listado, creación, detalle, lanzamiento, polling y resultado.
- Docker Compose con API, worker, web, PostgreSQL, Redis, MinIO y migraciones.
- Health checks, volúmenes, `.env.example` y documentación operativa.
- Pruebas unitarias, de API, worker, frontend e integración nativa.
- ADR 0002 y 0003.
- CI mínimo (sujeto a aprobación).

### 3.2 Excluido

- Integración real con LTX o cualquier proveedor audiovisual.
- Generación real de imágenes, audio o video.
- FFmpeg, timeline, edición, render o exportación.
- Entrenamiento o ejecución local de modelos.
- Autenticación, autorización empresarial, multitenencia y cuotas.
- Facturación, créditos o control de presupuesto.
- WebSockets, colaboración en tiempo real o notificaciones.
- Despliegue en producción, Kubernetes, Terraform o cloud.
- Story bible, personajes, escenas, planos, assets o continuidad narrativa.
- Reprogramación, cancelación o borrado de jobs.
- URLs prefirmadas o acceso público directo a MinIO.

### 3.3 Desviaciones conscientes

- **Un job activo por proyecto.** Se aplica como protección básica contra
  duplicaciones. No es una regla definitiva del producto; se documenta como
  decisión de este hito.
- **Un worker.** La recuperación de jobs `running` al arrancar asume una única
  instancia. Se documenta la limitación.
- **Sin auth.** Todos los endpoints son accesibles en localhost. No se presenta
  la aplicación como lista para producción.

---

## 4. Arquitectura propuesta

### 4.1 Vista general

```text
Navegador
  |
  | HTTP (fetch, CORS limitado)
  v
Next.js 16 (apps/web)
  |
  | /api/v1
  v
FastAPI 0.162 (apps/api)
  |-----------------------> PostgreSQL 17
  |                         projects, generation_jobs
  |
  |-----------------------> Redis 7.4
  |                         cola ARQ
  |
  |-----------------------> MinIO / S3
  |                         artefacto JSON privado
  |
  `-----------------------> Worker (apps/worker)
                              |
                              |-- ARQ
                              |-- FakeVideoProvider
                              `-- MinIO / S3
```

### 4.2 Límites y responsabilidades

| Capa | Responsabilidad |
|---|---|
| `api/routes` | Validación HTTP, códigos de estado, serialización |
| `services` | Casos de uso, transacciones, idempotencia |
| `repositories` | Acceso a datos y consultas parametrizadas |
| `models` | Modelos SQLAlchemy y restricciones |
| `schemas` | Contratos Pydantic de entrada/salida |
| `providers` | Contrato y adaptadores de generación |
| `storage` | Contrato y adaptador S3 |
| `queue` | Contrato y adaptador ARQ |
| `jobs` | Máquina de estados, processor y recuperación |
| `worker` | Runtime ARQ y señales de proceso |
| `web` | Presentación y consumo de API; sin lógica de negocio |

### 4.3 Flujo de una petición

1. Next.js envía `POST /api/v1/projects/{id}/jobs`.
2. FastAPI valida con Pydantic y consulta el proyecto.
3. El servicio crea el job en PostgreSQL dentro de una transacción.
4. Tras el commit, encola el job en Redis con ARQ.
5. FastAPI responde `202 Accepted` con la representación del job.
6. El worker reclama el job, invoca al proveedor fake y sube el artefacto.
7. El frontend consulta `GET /api/v1/jobs/{id}` hasta un estado terminal.

### 4.4 Persistencia síncrona

Se usará SQLAlchemy 2.x en modo síncrono con psycopg 3. Motivos:

- Menor complejidad para un scaffold.
- Compatibilidad directa con Alembic y con SQLite en pruebas.
- Las operaciones de base de datos del worker son cortas; el trabajo lento es la
  llamada al proveedor y la subida a S3.

Si en fases futuras la carga lo justifica, se podrá migrar a SQLAlchemy async sin
cambiar los contratos HTTP.

---

## 5. Estructura objetivo del monorepo

```text
.
├── apps/
│   ├── api/
│   │   ├── alembic/
│   │   │   ├── env.py
│   │   │   └── versions/
│   │   ├── src/ai_studio/
│   │   │   ├── api/v1/
│   │   │   │   ├── routes/
│   │   │   │   │   ├── health.py
│   │   │   │   │   ├── projects.py
│   │   │   │   │   └── jobs.py
│   │   │   │   └── router.py
│   │   │   ├── core/
│   │   │   │   ├── config.py
│   │   │   │   ├── logging.py
│   │   │   │   └── errors.py
│   │   │   ├── db/
│   │   │   │   ├── base.py
│   │   │   │   ├── session.py
│   │   │   │   └── models/
│   │   │   ├── schemas/
│   │   │   ├── repositories/
│   │   │   ├── services/
│   │   │   ├── providers/
│   │   │   │   ├── base.py
│   │   │   │   ├── fake.py
│   │   │   │   └── factory.py
│   │   │   ├── storage/
│   │   │   │   ├── base.py
│   │   │   │   └── s3.py
│   │   │   ├── queue/
│   │   │   │   ├── base.py
│   │   │   │   └── arq_queue.py
│   │   │   ├── jobs/
│   │   │   │   ├── processor.py
│   │   │   │   ├── tasks.py
│   │   │   │   └── recovery.py
│   │   │   └── main.py
│   │   ├── tests/
│   │   ├── Dockerfile
│   │   └── pyproject.toml
│   └── worker/
│       ├── src/ai_studio_worker/
│       │   ├── main.py
│       │   └── settings.py
│       ├── tests/
│       └── pyproject.toml
├── apps/web/
│   ├── src/app/
│   │   ├── page.tsx
│   │   ├── projects/new/page.tsx
│   │   └── projects/[id]/page.tsx
│   ├── src/components/
│   ├── src/lib/
│   ├── tests/
│   ├── Dockerfile
│   ├── next.config.ts
│   ├── tsconfig.json
│   └── package.json
├── packages/
│   └── contracts/
│       ├── openapi.json
│       ├── src/index.ts
│       └── package.json
├── infra/
│   └── minio/init-bucket.sh
├── docs/
│   ├── API.md
│   ├── DEVELOPMENT.md
│   ├── implementation/PHASE_0_1_IMPLEMENTATION_PLAN.md
│   └── adr/
│       ├── 0002-async-jobs-arq.md
│       └── 0003-fake-provider-and-artifacts.md
├── docker-compose.yml
├── .dockerignore
├── .editorconfig
├── .env.example
├── package.json
├── package-lock.json
├── pyproject.toml
└── uv.lock
```

### 5.1 Justificación de carpetas

- `apps/api`: contiene el dominio y la infraestructura Python compartida con el
  worker. Evita duplicar modelos, servicios y contratos.
- `apps/worker`: solo compone el runtime ARQ. Depende del paquete de la API.
- `apps/web`: frontend independiente, consumidor de la API.
- `packages/contracts`: artefacto generado desde OpenAPI para tipar el cliente.
  No contiene lógica de negocio.
- `infra`: scripts de inicialización de servicios locales.
- `docs/implementation`: plan y seguimiento de tareas.
- `docs/adr`: decisiones arquitectónicas.

### 5.2 Workspaces

- Python: workspace `uv` en la raíz con miembros `apps/api` y `apps/worker`.
- Node: workspaces npm en la raíz con `apps/web` y `packages/contracts`.
- Lockfiles `uv.lock` y `package-lock.json` se versionan.

---

## 6. Decisiones técnicas

| Área | Decisión propuesta | Alternativas evaluadas | Motivo |
|---|---|---|---|
| Cola | ARQ 0.28 sobre Redis 7.4 | RQ, Celery | Asyncio nativo, API pequeña, reintentos, `arq --check` |
| ORM | SQLAlchemy 2.0 + psycopg 3 (sync) | SQLAlchemy async, SQLModel | Simplicidad, Alembic directo, tests SQLite |
| Migraciones | Alembic 1.20 | `create_all` al arrancar | Entorno nuevo reproducible; no depende del arranque |
| Validación | Pydantic 2.14 + pydantic-settings | Dataclasses, env manual | Validación al arrancar y contratos tipados |
| API | FastAPI 0.162 | Flask, Django REST | OpenAPI, tipado, ecosistema Pydantic |
| Web | Next.js 16 App Router + React 19 + TS 5.9 | Vite SPA, Remix | Stack propuesto en README; routing y build probados |
| Contratos | OpenAPI → tipos TS generados | Tipos manuales | Evita deriva entre backend y frontend |
| Provider | Protocolo `submit/get_status/fetch_result` | Un único `generate()` | Alineado con `docs/ARCHITECTURE.md`; permite polling real |
| Artefacto fake | JSON `simulated: true` en S3 | MP4 con FFmpeg | No simular video real ni añadir FFmpeg |
| Descarga | Proxy por API | URLs prefirmadas | Evita exponer MinIO y el desajuste de endpoints interno/público |
| Polling | `setInterval` 2 s en cliente | WebSockets, SSE | Simplicidad suficiente para el hito |
| Tests | SQLite + fakeredis + moto por defecto | Solo Docker | Permite validar sin Docker; integración opcional |
| Auth | Fuera de alcance; solo localhost | Auth fake | No confundir seguridad simulada con real |
| Imágenes | Tags fijados | `latest` | Reproducibilidad |

### 6.1 Versiones candidatas

Se fijarán con lockfiles:

- Next.js `16.4.3`
- React `19.3.1`
- TypeScript `5.9.3`
- FastAPI `0.162.0`
- Pydantic `2.14.1`
- pydantic-settings `2.16.1`
- SQLAlchemy `2.0.55`
- Alembic `1.20.1`
- ARQ `0.28.0`
- boto3 `1.43.x`
- pytest `9.1.1`
- pytest-asyncio `1.4.1`
- httpx `0.28.1`
- fakeredis `2.40.0`
- moto `5.2.4`
- ruff `0.17.2`
- mypy `2.4.0`
- Python `3.12`
- Node.js `24`

Las versiones definitivas se verificarán al generar los lockfiles y las imágenes;
cualquier cambio se reflejará en este documento.

### 6.2 ADR previstos

- **ADR 0002:** procesamiento asíncrono con ARQ, estados de job, idempotencia,
  reintentos y recuperación con un único worker.
- **ADR 0003:** contrato de proveedor fake y manejo de artefactos en S3 con
  descarga a través de la API, sin FFmpeg.
- **ADR 0001:** se conserva sin cambios. Su estado “Propuesto” no se altera.

---

## 7. Modelo de datos inicial

### 7.1 `projects`

| Campo | Tipo | Restricciones |
|---|---|---|
| `id` | UUID | PK, generado en aplicación |
| `name` | varchar(200) | NOT NULL, no vacío |
| `description` | text | NULL |
| `status` | varchar(20) | NOT NULL, default `draft` |
| `created_at` | timestamptz | NOT NULL, UTC |
| `updated_at` | timestamptz | NOT NULL, UTC, actualizado en cada cambio |

Estados de proyecto:

- `draft`: valor inicial.
- `active`: reservado para Fase 2.
- `archived`: reservado para Fase 2.

No se implementan transiciones de estado de proyecto en este hito.

Índices:

- `ix_projects_created_at` sobre `created_at` para el listado.
- `ix_projects_name` opcional para búsquedas futuras; no se añade si no se usa.

### 7.2 `generation_jobs`

| Campo | Tipo | Restricciones |
|---|---|---|
| `id` | UUID | PK, generado en aplicación |
| `project_id` | UUID | FK `projects.id`, NOT NULL, `ON DELETE CASCADE` |
| `type` | varchar(40) | NOT NULL, valor `video_generation` |
| `status` | varchar(20) | NOT NULL, default `queued` |
| `provider` | varchar(40) | NOT NULL, valor `fake` |
| `idempotency_key` | varchar(100) | NULL |
| `input` | JSON | NOT NULL; prompt, duración y parámetros |
| `result` | JSON | NULL; metadatos del artefacto |
| `error` | JSON | NULL; `code`, `message`, `retryable` |
| `attempts` | integer | NOT NULL, default 0 |
| `max_attempts` | integer | NOT NULL, default 3 |
| `created_at` | timestamptz | NOT NULL, UTC |
| `updated_at` | timestamptz | NOT NULL, UTC |
| `started_at` | timestamptz | NULL |
| `finished_at` | timestamptz | NULL |

Estados de job:

- `queued`: persistido, pendiente de worker.
- `running`: reclamado por un worker.
- `succeeded`: artefacto almacenado y resultado persistido.
- `failed`: error definitivo tras agotar reintentos o error no reintentable.

No se implementan `cancelled`, `expired`, `cancel_requested` en este hito.

Índices y restricciones:

- `ix_generation_jobs_project_created` sobre `(project_id, created_at DESC)`.
- `ix_generation_jobs_status` sobre `status`.
- Único parcial `(project_id, idempotency_key)` cuando `idempotency_key` no es nulo.
- Único parcial `(project_id)` cuando `status IN ('queued', 'running')`.
- `CHECK` de estados permitidos.
- `CHECK` de `attempts >= 0` y `max_attempts > 0`.

### 7.3 Relaciones

- Un proyecto tiene muchos jobs.
- Un job pertenece a un proyecto.
- Borrar un proyecto elimina sus jobs y metadatos asociados. El borrado de
  proyectos no forma parte de este hito; la regla queda preparada.

### 7.4 Fechas y zona horaria

- Todas las fechas se persisten en UTC con `timestamptz`.
- La API serializa ISO 8601 con offset UTC (`Z`).
- El frontend muestra fechas en la zona local del navegador.

### 7.5 Resultado del job

```json
{
  "simulated": true,
  "provider": "fake",
  "provider_job_id": "fake-...",
  "seed": "...",
  "duration_seconds": 5,
  "artifact": {
    "object_key": "projects/<uuid>/jobs/<uuid>/simulated-result.json",
    "content_type": "application/json",
    "size_bytes": 512,
    "sha256": "...",
    "download_url": "/api/v1/jobs/<uuid>/artifact"
  },
  "notice": "Artefacto simulado. No se generó video real."
}
```

### 7.6 Error del job

```json
{
  "code": "provider_unavailable",
  "message": "El proveedor no pudo completar la solicitud.",
  "retryable": true
}
```

No se persisten trazas, credenciales ni respuestas completas de proveedores.

---

## 8. Contratos de API

Base: `/api/v1`. Documentación interactiva en `/api/v1/docs`.
OpenAPI JSON en `/api/v1/openapi.json`.

### 8.1 Endpoints

| Método | Ruta | Éxito | Errores relevantes |
|---|---|---|---|
| GET | `/health` | 200 `{status:"ok"}` | — |
| GET | `/ready` | 200 con checks | 503 `not_ready` |
| GET | `/projects` | 200 lista | 400 |
| POST | `/projects` | 201 proyecto | 400 |
| GET | `/projects/{project_id}` | 200 proyecto | 404 |
| GET | `/projects/{project_id}/jobs` | 200 lista de jobs | 404 |
| POST | `/projects/{project_id}/jobs` | 202 job | 400, 404, 409 |
| GET | `/jobs/{job_id}` | 200 job | 404 |
| GET | `/jobs/{job_id}/artifact` | 200 archivo | 404 |

### 8.2 `GET /health`

Liveness. No consulta dependencias.

```json
{
  "status": "ok",
  "service": "ai-studio-api",
  "version": "0.1.0"
}
```

### 8.3 `GET /ready`

Readiness. Consulta PostgreSQL, Redis y MinIO.

```json
{
  "status": "ready",
  "checks": {
    "database": "ok",
    "redis": "ok",
    "storage": "ok"
  }
}
```

Si falla una dependencia: HTTP 503 con `status: "not_ready"` y el check en
`error`. No se exponen URLs, credenciales ni detalles internos.

### 8.4 Proyectos

`POST /projects`

```json
{
  "name": "Serie piloto",
  "description": "Prueba de flujo simulado"
}
```

Validación:

- `name`: 1–200 caracteres, sin espacios únicamente.
- `description`: máximo 2000 caracteres, opcional.

Respuesta 201: proyecto con `id`, `name`, `description`, `status`,
`created_at`, `updated_at`.

`GET /projects`: devuelve `{ "items": [...], "total": N, "limit": 50, "offset": 0 }`.
Orden por `created_at DESC`. Límite máximo 100.

`GET /projects/{project_id}`: 200 o 404 `project_not_found`.

`GET /projects/{project_id}/jobs`: devuelve jobs recientes del proyecto,
ordenados por `created_at DESC`, con límite básico.

### 8.5 Crear job

`POST /projects/{project_id}/jobs`

```json
{
  "prompt": "Plano nocturno de una ciudad lluviosa",
  "duration_seconds": 5,
  "idempotency_key": "9c0f4f7e-..."
}
```

Validación:

- `prompt`: 1–2000 caracteres.
- `duration_seconds`: entero entre 1 y 60; default 5.
- `idempotency_key`: 8–100 caracteres, opcional.

Comportamiento:

- 202 con `Location: /api/v1/jobs/{job_id}` y job en `queued`.
- 404 si el proyecto no existe.
- 409 `active_job_exists` si el proyecto ya tiene un job `queued/running`.
- 409 `idempotency_conflict` si la clave ya existe con otro payload.
- Replay con misma clave y mismo payload: 200 con `Idempotent-Replayed: true`
  y el job existente.

### 8.6 Consultar job

`GET /jobs/{job_id}`: 200 con todos los campos del job, incluidos `result` y
`error` cuando correspondan. 404 `job_not_found`.

### 8.7 Descargar artefacto

`GET /jobs/{job_id}/artifact`

- 200 con `Content-Type: application/json` y
  `Content-Disposition: attachment`.
- 404 si el job no existe, no terminó con éxito o no tiene artefacto.
- El archivo se lee de S3 y se transmite a través de la API. No se exponen
  credenciales ni URLs internas.

### 8.8 Formato de error

```json
{
  "error": {
    "code": "active_job_exists",
    "message": "El proyecto ya tiene un trabajo activo.",
    "details": {
      "existing_job_id": "..."
    }
  }
}
```

Códigos previstos:

| HTTP | Código | Caso |
|---|---|---|
| 400 | `validation_error` | Payload inválido |
| 404 | `project_not_found` | Proyecto inexistente |
| 404 | `job_not_found` | Job inexistente |
| 404 | `artifact_not_found` | Artefacto inexistente |
| 409 | `active_job_exists` | Job activo en el proyecto |
| 409 | `idempotency_conflict` | Clave reutilizada con otro payload |
| 500 | `internal_error` | Error no controlado |
| 503 | `not_ready` | Dependencia crítica no disponible |

### 8.9 Cabeceras

- `X-Request-ID`: se genera si no viene; se devuelve siempre.
- `Location`: al crear un job.
- `Idempotent-Replayed`: en replay idempotente.
- CORS limitado a orígenes configurados.

---

## 9. Flujo asíncrono

### 9.1 Ciclo de vida

```text
POST /projects/{id}/jobs
        |
        v
   [PostgreSQL: queued]
        |
        | commit
        v
   [Redis / ARQ: enqueue, _job_id = job.id]
        |
        v
   Worker reclama con UPDATE condicional
        |
        v
   [PostgreSQL: running, attempts += 1, started_at]
        |
        v
   Provider fake: submit -> get_status -> fetch_result
        |
        v
   Subida a S3 + metadatos
        |
        +--> éxito --> [succeeded, result, finished_at]
        |
        `--> fallo reintentable y attempts < max --> [queued, error, Retry]
        |
        `--> fallo definitivo --> [failed, error, finished_at]
```

### 9.2 Idempotencia y duplicados

- La API exige opcionalmente `idempotency_key`.
- La restricción única evita dos jobs con la misma clave en un proyecto.
- Un replay con la misma clave y payload devuelve el job existente.
- Un job activo por proyecto evita dobles clics accidentales.
- ARQ usa `_job_id = job.id` para deduplicar encolados.
- El worker reclama el job con una actualización condicional
  `status = 'queued'`; si no afecta filas, termina sin reprocesar.
- La base de datos es la fuente de verdad; Redis no lo es.

### 9.3 Reintentos

- Máximo de intentos configurable (`JOB_MAX_ATTEMPTS`, default 3).
- Backoff exponencial acotado: base configurable, por ejemplo 2 s, 4 s, 8 s.
- Errores reintentables: timeout, error transitorio del proveedor, fallo
  transitorio de S3.
- Errores no reintentables: payload inválido, capacidad no soportada, error
  determinista del proveedor.
- Cada intento registra `attempts` y el último error.
- El estado final siempre queda persistido.

### 9.4 Recuperación tras reinicio

- Al arrancar, el worker reencola jobs `queued`.
- Si `WORKER_RECOVER_RUNNING_ON_START=true`, los jobs `running` se devuelven a
  `queued` antes de reencolarse.
- Esta recuperación asume **un único worker**. Con múltiples workers, un job
  podría reencolarse mientras otro aún lo ejecuta. La limitación se documenta y
  se revisará al escalar.
- Un job `succeeded` o `failed` no se reprocesa.

### 9.5 Encolado fallido

Si el commit en PostgreSQL tiene éxito pero el encolado en Redis falla:

- El endpoint responde 202 con el job en `queued`.
- Se registra el error en logs sin secretos.
- El worker, al arrancar, o un barrido periódico reencola los jobs `queued`.
- La readiness refleja que Redis no está disponible.

Se prioriza no perder el trabajo persistido sobre fallar la petición.

### 9.6 Timeouts

- Timeout de job en ARQ: configurable, propuesto 120 s.
- Timeout de proveedor fake: no aplica, pero el contrato lo contempla.
- Timeout de S3: configurable y acotado.
- Timeout de polling del frontend: solo mientras el job está activo; se detiene
  en estado terminal y al desmontar el componente.

---

## 10. Estrategia del proveedor ficticio

### 10.1 Contrato

```python
class VideoProvider(Protocol):
    name: str
    capabilities: ProviderCapabilities

    async def submit(self, request: GenerationRequest) -> ProviderSubmission: ...
    async def get_status(self, provider_job_id: str) -> ProviderStatus: ...
    async def fetch_result(self, provider_job_id: str) -> ProviderArtifact: ...
```

Tipos:

- `GenerationRequest`: prompt, duración, seed opcional.
- `ProviderSubmission`: `provider_job_id`, `submitted_at`.
- `ProviderStatus`: `queued|running|succeeded|failed`, progreso opcional, error.
- `ProviderArtifact`: nombre, content type, bytes, metadatos, `simulated`.
- `ProviderCapabilities`: duración, audio, referencias, semilla, resolución,
  todos explícitos y limitados a lo que el fake soporta.

### 10.2 Comportamiento

- Selección por `VIDEO_PROVIDER=fake`; valores desconocidos fallan al arrancar.
- Sin llamadas de red, claves, GPU ni costes.
- `seed` determinista: SHA-256 de `prompt|duration_seconds`.
- `provider_job_id` determinista: `fake-<primeros 16 hex del seed>`.
- `submit` devuelve el identificador; `get_status` devuelve `succeeded`;
  `fetch_result` devuelve el artefacto.
- Retardo configurable (`FAKE_PROVIDER_DELAY_SECONDS`, default 1.0; 0 en tests)
  para observar el estado `running`.
- Artefacto JSON con `simulated: true` y aviso explícito.
- El resultado nunca se presenta como video real generado por IA.

### 10.3 Sustitución futura

Un proveedor real implementará el mismo protocolo y se seleccionará en la
factory. El worker, los servicios y los contratos HTTP no se reescriben. Las
capacidades no soportadas se declararán y se validarán antes de enviar el job.

---

## 11. Diseño del frontend

### 11.1 Páginas

| Ruta | Contenido |
|---|---|
| `/` | Listado de proyectos; botón “Nuevo proyecto” |
| `/projects/new` | Formulario de creación |
| `/projects/[id]` | Detalle, jobs, lanzamiento, polling y resultado |

### 11.2 Componentes

- `AppShell`: cabecera, navegación y contenedor.
- `ProjectList`: tarjetas o filas con nombre, descripción, estado y fechas.
- `ProjectForm`: nombre, descripción, validación y envío.
- `JobList`: historial de jobs con estado y fecha.
- `JobLauncher`: prompt, duración y botón de generación simulada.
- `JobStatusBadge`: `queued`, `running`, `succeeded`, `failed`.
- `JobResult`: aviso de simulación, metadatos y descarga.
- `JobError`: mensaje comprensible sin trazas.
- `LoadingState`, `EmptyState`, `ErrorState`.

### 11.3 Estados

- Carga inicial: skeleton o mensaje.
- Vacío: explicación y llamada a la acción.
- Error: mensaje comprensible y reintento.
- Envío: botón deshabilitado y feedback.
- Job activo: polling cada 2 s.
- Job terminal: detener polling y mostrar resultado o error.
- Proyecto inexistente: mensaje 404 amigable.

### 11.4 Cliente API

- `src/lib/api.ts` con base URL `NEXT_PUBLIC_API_BASE_URL`.
- Tipos importados desde `@ai-studio/contracts`.
- Función común que parsea el error uniforme y lanza un error tipado.
- Sin claves ni lógica de proveedores en el navegador.
- Sin simular la ejecución del job en el frontend.

### 11.5 Polling

- Hook `useJobPolling(jobId)` con `setInterval` de 2 s.
- Se detiene en `succeeded` o `failed`.
- Cancela el intervalo al desmontar.
- Evita actualizaciones después de desmontar.
- Muestra `aria-live="polite"` para cambios de estado.

### 11.6 Estilo

- CSS propio con variables.
- Tema claro sobrio, responsive, sin animaciones complejas.
- Tipografía del sistema, sin fuentes remotas.
- Contraste y foco visibles; labels asociados a inputs.
- Sin sistema de diseño completo.

### 11.7 Pruebas

- Vitest + Testing Library + user-event + jsdom.
- Lista con datos, vacío y error.
- Formulario válido e inválido; error de API.
- Lanzamiento y polling hasta `succeeded`.
- Visualización de `failed` y del aviso de simulación.
- No se implementa Playwright en este hito por requerir navegadores y un stack
  completo; se documenta como pendiente.

---

## 12. Infraestructura local

### 12.1 Servicios de Docker Compose

| Servicio | Imagen propuesta | Función | Health check |
|---|---|---|---|
| `postgres` | `postgres:17-alpine` | Base de datos | `pg_isready` |
| `redis` | `redis:7.4-alpine` | Cola ARQ | `redis-cli ping` |
| `minio` | `quay.io/minio/minio` (tag fijado) | Object storage | `mc ready local` |
| `minio-init` | `quay.io/minio/mc` (tag fijado) | Crear bucket privado | comando idempotente |
| `migrate` | imagen de `api` | `alembic upgrade head` | exit code 0 |
| `api` | `apps/api/Dockerfile` | FastAPI | GET `/api/v1/health` |
| `worker` | misma imagen que `api` | ARQ worker | `arq --check ...` |
| `web` | `apps/web/Dockerfile` | Next.js standalone | fetch HTTP a `/` |

### 12.2 Dependencias entre servicios

- `migrate` depende de `postgres` healthy.
- `api` depende de `postgres`, `redis` y `minio` healthy, de `minio-init`
  completado y de `migrate` completado.
- `worker` depende de las mismas condiciones.
- `web` depende de `api` healthy.

No se marca un servicio como saludable solo porque el contenedor esté en
ejecución.

### 12.3 Volúmenes

- `postgres_data`: datos de PostgreSQL.
- `redis_data`: persistencia AOF de Redis.
- `minio_data`: objetos de MinIO.

`docker compose down -v` elimina estos volúmenes y **pierde los datos**. Se
advertirá en la documentación.

### 12.4 Puertos

Publicados solo en `127.0.0.1`:

- Web: 3000
- API: 8000
- PostgreSQL: 5432
- Redis: 6379
- MinIO API: 9000
- MinIO Console: 9001

### 12.5 Imágenes y seguridad de contenedores

- Python `3.12-slim`, usuario no root.
- Node `24-alpine`, usuario no root.
- Dependencias de producción con `uv sync --frozen --no-dev` y `npm ci`.
- `output: "standalone"` en Next.js.
- Sin secretos en imágenes ni en argumentos de build.
- `.dockerignore` para excluir `.env`, `.git`, tests y artefactos locales.
- Tags fijados; no `latest` en servicios críticos.

### 12.6 Comandos documentados

Preparación:

```powershell
Copy-Item .env.example .env
```

Build y arranque:

```powershell
docker compose build
docker compose up -d
docker compose ps
```

Migraciones (si no se ejecutan por el servicio `migrate`):

```powershell
docker compose run --rm migrate
```

Comprobaciones:

```powershell
Invoke-RestMethod http://localhost:8000/api/v1/health
Invoke-RestMethod http://localhost:8000/api/v1/ready
Start-Process http://localhost:3000
```

Pruebas dentro de contenedores:

```powershell
docker compose run --rm api pytest
docker compose run --rm web npm test
```

Logs:

```powershell
docker compose logs -f api worker web
```

Detener:

```powershell
docker compose down
```

Eliminar entorno y datos (destructivo, requiere confirmación):

```powershell
docker compose down -v
```

También se documentarán equivalentes para bash.

### 12.7 Configuración de MinIO

- Bucket `novela-assets`, creado de forma idempotente.
- Política privada por defecto; sin acceso anónimo.
- Credenciales de ejemplo solo para desarrollo.
- La API comprueba la existencia del bucket en readiness.
- No se exponen claves al frontend.

### 12.8 FFmpeg

No se incluye servicio ni dependencia de FFmpeg. La estructura del worker deja
preparada la incorporación futura de un worker de render aislado.

---

## 13. Seguridad

### 13.1 Modelo de seguridad de este hito

- Aplicación de desarrollo local, sin autenticación ni autorización.
- Endpoints públicos en localhost.
- No se presenta como lista para producción.
- No se implementa autenticación ficticia.

### 13.2 Secretos

- Solo en `.env`, nunca en código, logs, fixtures ni commits.
- `.env` ignorado por Git.
- `.env.example` con valores locales no reales.
- `SecretStr` para contraseñas y claves.
- No se imprimen settings completos al arrancar.

### 13.3 CORS

- Orígenes explícitos desde `API_CORS_ORIGINS`; por defecto
  `http://localhost:3000`.
- Sin `*`.
- Métodos y cabeceras limitados a los necesarios.

### 13.4 Validación y errores

- Validación Pydantic en todos los endpoints.
- Consultas parametrizadas mediante ORM.
- Errores internos: 500 genérico con `request_id`; traza solo en logs del servidor.
- No se filtran credenciales, URLs internas ni respuestas de proveedores.

### 13.5 Logs

- JSON estructurado a stdout.
- Campos: timestamp, level, service, request_id, method, path, status, duration.
- En worker: job_id, attempt, status, provider, duration.
- Sin contraseñas, tokens, cookies, prompts completos ni cuerpos de request.
- Nivel configurable.

### 13.6 Red y almacenamiento

- Puertos ligados a `127.0.0.1`.
- MinIO privado; descarga vía API.
- Sin URLs prefirmadas en este hito.
- Sin endpoints administrativos.
- Swagger habilitado en desarrollo; se documentará su desactivación futura.

### 13.7 Limitaciones conocidas

- Sin auth: cualquier proceso local puede acceder a la API.
- Sin cifrado en reposo más allá del que aporten las imágenes locales.
- Sin rate limiting ni cuotas.
- Sin auditoría de acciones.
- Sin backups.
- No apto para producción ni para exposición a Internet.

---

## 14. Plan de pruebas

### 14.1 Backend

| Área | Casos |
|---|---|
| Configuración | Env válido; proveedor desconocido; URL inválida; límites fuera de rango |
| Schemas | Nombre vacío/largo; descripción larga; prompt vacío; duración inválida |
| Proyectos | Crear, listar, consultar, 404, validación |
| Jobs | Crear 202, consultar, 404, 409 activo, replay idempotente, conflicto |
| Health | Liveness sin dependencias; readiness 200 y 503 |
| Artefacto | Descarga correcta; job sin artefacto; job inexistente |
| Errores | Formato uniforme; 500 sin trazas |

### 14.2 Worker y proveedor

| Área | Casos |
|---|---|
| Fake provider | Determinismo; `simulated: true`; capacidades |
| Processor | `queued → running → succeeded`; artefacto persistido |
| Retry | Fallo transitorio → reintento → éxito |
| Fallo final | Agota intentos → `failed` con error estructurado |
| Duplicado | Segunda ejecución no reprocesa |
| Recuperación | Jobs `queued` y `running` huérfanos al arrancar |
| Cola | Encolado con ARQ/fakeredis; deduplicación por id |

### 14.3 Frontend

| Área | Casos |
|---|---|
| Lista | Datos, vacío, error, reintento |
| Formulario | Válido, inválido, error de API |
| Detalle | Proyecto inexistente; carga |
| Jobs | Lanzamiento, polling, éxito, fallo |
| Resultado | Aviso de simulación y enlace de descarga |

### 14.4 Integración

- Flujo nativo: proyecto → job → processor → succeeded → artefacto.
- Flujo HTTP con `TestClient` y dependencias fake.
- Flujo opcional con PostgreSQL/Redis reales mediante `TEST_DATABASE_URL` y
  `TEST_REDIS_URL`; se omite si no están configurados.
- Flujo Compose completo: requiere Docker.

### 14.5 Pruebas no viables en este entorno

- `docker compose build/up` y health checks reales: bloqueadas sin Docker.
- Playwright E2E: requiere navegadores y stack completo; se pospone.
- Migración sobre PostgreSQL real: bloqueada sin PostgreSQL/Docker; se intentará
  una verificación equivalente en CI o cuando el usuario habilite Docker.

No se declarará una prueba como aprobada sin haberla ejecutado.

---

## 15. Plan de ejecución por etapas

Estados posibles: `pending`, `in_progress`, `blocked`, `completed`.

### Etapa A — Base y configuración

#### PLAN-001 — Clonar repositorio y preparar rama

- **Descripción:** clonar `ai-studio` en el directorio de trabajo y crear la rama
  de documentación.
- **Motivo/resultado:** base reproducible sin tocar `main`.
- **Archivos/componentes:** `.git` (solo metadatos).
- **Dependencias:** ninguna.
- **Pruebas:** `git status`, `git log -1`.
- **Criterio de aceptación:** HEAD en `9cfbfd2`; rama activa; sin cambios.
- **Riesgo:** bajo.
- **Estado:** completed.

#### PLAN-002 — Documento de plan

- **Descripción:** crear este documento con alcance, arquitectura, tareas,
  riesgos y criterios.
- **Motivo/resultado:** permitir revisión y aprobación antes de implementar.
- **Archivos/componentes:** `docs/implementation/PHASE_0_1_IMPLEMENTATION_PLAN.md`.
- **Dependencias:** PLAN-001.
- **Pruebas:** revisión de contenido y enlaces.
- **Criterio de aceptación:** las 20 secciones exigidas están presentes.
- **Riesgo:** bajo.
- **Estado:** completed.

#### PLAN-003 — Skeleton del monorepo y tooling

- **Descripción:** crear workspaces uv/npm, `.dockerignore`, `.editorconfig` y
  estructura de carpetas.
- **Motivo/resultado:** base común para backend, worker y web.
- **Archivos/componentes:** `pyproject.toml`, `package.json`, `uv.lock`,
  `package-lock.json`, `apps/*`, `packages/contracts`, `infra/`.
- **Dependencias:** PLAN-002.
- **Pruebas:** `uv sync --all-packages`, `npm ci`.
- **Criterio de aceptación:** ambos gestores resuelven sin errores.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-004 — Configuración central y `.env.example`

- **Descripción:** settings Pydantic, validación al arrancar y plantilla de
  variables completa.
- **Motivo/resultado:** configuración única y fallo temprano.
- **Archivos/componentes:** `core/config.py`, `.env.example`, README.
- **Dependencias:** PLAN-003.
- **Pruebas:** tests de settings válidos e inválidos.
- **Criterio de aceptación:** configuración inválida impide arrancar; no se
  registran secretos.
- **Riesgo:** bajo.
- **Estado:** pending.

### Etapa B — Backend, persistencia y API

#### PLAN-005 — Modelos, sesión e índices

- **Descripción:** modelos `Project` y `GenerationJob`, enums, sesión y
  restricciones.
- **Motivo/resultado:** persistencia tipada y consultable.
- **Archivos/componentes:** `db/`, `models/`.
- **Dependencias:** PLAN-004.
- **Pruebas:** creación de metadata en SQLite; constraints.
- **Criterio de aceptación:** tablas e índices definidos y documentados.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-006 — Alembic y migración inicial

- **Descripción:** configurar Alembic y generar la migración inicial.
- **Motivo/resultado:** entornos nuevos reproducibles.
- **Archivos/componentes:** `alembic.ini`, `alembic/`.
- **Dependencias:** PLAN-005.
- **Pruebas:** `alembic upgrade head` sobre base vacía.
- **Criterio de aceptación:** migración aplicada y reversible.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-007 — Logging, errores, health y readiness

- **Descripción:** logging JSON, request ID, manejadores de error y endpoints de
  salud.
- **Motivo/resultado:** operación observable y errores uniformes.
- **Archivos/componentes:** `core/logging.py`, `core/errors.py`, rutas de health.
- **Dependencias:** PLAN-004.
- **Pruebas:** health 200; readiness 200/503; formato de error.
- **Criterio de aceptación:** no se filtran trazas ni secretos.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-008 — Proyectos

- **Descripción:** repositorio, servicio, schemas y endpoints de proyectos.
- **Motivo/resultado:** crear, listar y consultar proyectos.
- **Archivos/componentes:** `repositories/projects.py`,
  `services/projects.py`, `api/v1/routes/projects.py`.
- **Dependencias:** PLAN-006, PLAN-007.
- **Pruebas:** creación, listado, detalle, 404, validación.
- **Criterio de aceptación:** contratos y códigos HTTP correctos.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-009 — Jobs: API e idempotencia

- **Descripción:** creación y consulta de jobs, reglas de job activo y replay
  idempotente.
- **Motivo/resultado:** iniciar y consultar trabajos de forma segura.
- **Archivos/componentes:** `repositories/jobs.py`, `services/jobs.py`,
  `api/v1/routes/jobs.py`.
- **Dependencias:** PLAN-008.
- **Pruebas:** 202, 404, 409, replay, conflicto de payload.
- **Criterio de aceptación:** no se crean duplicados accidentales.
- **Riesgo:** alto.
- **Estado:** pending.

#### PLAN-010 — Storage S3 y descarga de artefactos

- **Descripción:** contrato de storage, adaptador boto3 y endpoint de descarga.
- **Motivo/resultado:** separar metadatos de binarios y servir el resultado.
- **Archivos/componentes:** `storage/`, ruta de artefacto.
- **Dependencias:** PLAN-007.
- **Pruebas:** put/get/head; descarga; 404.
- **Criterio de aceptación:** bucket privado; sin URLs prefirmadas.
- **Riesgo:** medio.
- **Estado:** pending.

### Etapa C — Cola y worker

#### PLAN-011 — Cola ARQ

- **Descripción:** contrato de cola, adaptador ARQ y encolado tras commit.
- **Motivo/resultado:** procesamiento asíncrono desacoplado.
- **Archivos/componentes:** `queue/`, servicio de jobs.
- **Dependencias:** PLAN-004.
- **Pruebas:** encolado y deduplicación con fakeredis/ARQ.
- **Criterio de aceptación:** un job se encola una sola vez por id.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-012 — Runtime del worker

- **Descripción:** `WorkerSettings`, health check ARQ y recuperación al arrancar.
- **Motivo/resultado:** worker persistente y recuperable.
- **Archivos/componentes:** `apps/worker/`, `jobs/recovery.py`.
- **Dependencias:** PLAN-011.
- **Pruebas:** `arq --check`; reencolado de `queued`; recuperación de `running`.
- **Criterio de aceptación:** el worker arranca y responde al health check.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-013 — Processor, estados y reintentos

- **Descripción:** reclamación atómica, invocación al proveedor, subida y
  transiciones con retry/backoff.
- **Motivo/resultado:** ejecución fiable y estados persistentes.
- **Archivos/componentes:** `jobs/processor.py`, `jobs/tasks.py`.
- **Dependencias:** PLAN-009, PLAN-012.
- **Pruebas:** éxito, retry, fallo final, duplicado.
- **Criterio de aceptación:** estados y errores siempre persistidos.
- **Riesgo:** alto.
- **Estado:** pending.

### Etapa D — Proveedor fake

#### PLAN-014 — Contrato de proveedor

- **Descripción:** protocolo tipado, capacidades y modelos de resultado.
- **Motivo/resultado:** desacoplar el dominio de proveedores reales.
- **Archivos/componentes:** `providers/base.py`.
- **Dependencias:** PLAN-004.
- **Pruebas:** conformidad de tipos; capacidades explícitas.
- **Criterio de aceptación:** ningún acoplamiento a APIs externas.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-015 — Proveedor fake determinista

- **Descripción:** implementación fake, seed y artefacto JSON simulado.
- **Motivo/resultado:** ejecutar el flujo sin claves ni costes.
- **Archivos/componentes:** `providers/fake.py`, `providers/factory.py`.
- **Dependencias:** PLAN-010, PLAN-014.
- **Pruebas:** determinismo; aviso de simulación; sin red.
- **Criterio de aceptación:** mismo input produce mismo resultado.
- **Riesgo:** bajo.
- **Estado:** pending.

### Etapa E — Frontend

#### PLAN-016 — Scaffold web y contratos

- **Descripción:** Next.js 16, TypeScript estricto, layout, cliente API y tipos
  generados desde OpenAPI.
- **Motivo/resultado:** frontend tipado y compilable.
- **Archivos/componentes:** `apps/web/`, `packages/contracts/`.
- **Dependencias:** PLAN-008.
- **Pruebas:** `npm run lint`, `npm run typecheck`, `npm run build`.
- **Criterio de aceptación:** build sin errores.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-017 — Listado y creación de proyectos

- **Descripción:** página principal, formulario y estados.
- **Motivo/resultado:** crear y ver proyectos desde la UI.
- **Archivos/componentes:** `src/app/page.tsx`, `projects/new/`, componentes.
- **Dependencias:** PLAN-016.
- **Pruebas:** lista con datos/vacío/error; formulario.
- **Criterio de aceptación:** flujo funciona contra la API.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-018 — Detalle, jobs y polling

- **Descripción:** detalle de proyecto, lanzamiento de job, polling y resultado.
- **Motivo/resultado:** recorrido completo desde la UI.
- **Archivos/componentes:** `projects/[id]/`, hooks y componentes de job.
- **Dependencias:** PLAN-009, PLAN-016.
- **Pruebas:** polling hasta éxito; fallo; descarga.
- **Criterio de aceptación:** estado y resultado visibles sin recargar.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-019 — Estilo y accesibilidad

- **Descripción:** CSS responsive, foco, labels y `aria-live`.
- **Motivo/resultado:** interfaz usable y sobria.
- **Archivos/componentes:** estilos globales y componentes.
- **Dependencias:** PLAN-017, PLAN-018.
- **Pruebas:** revisión manual y checks básicos.
- **Criterio de aceptación:** navegación y lectura correctas en móvil y desktop.
- **Riesgo:** bajo.
- **Estado:** pending.

### Etapa F — Docker e integración

#### PLAN-020 — Dockerfiles

- **Descripción:** imágenes de API/worker y web, no root, standalone.
- **Motivo/resultado:** artefactos reproducibles.
- **Archivos/componentes:** `apps/api/Dockerfile`, `apps/web/Dockerfile`,
  `.dockerignore`.
- **Dependencias:** PLAN-003.
- **Pruebas:** `docker build` (bloqueado sin Docker).
- **Criterio de aceptación:** imágenes construidas y arrancables.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-021 — Docker Compose

- **Descripción:** servicios, health checks, volúmenes, dependencias y puertos
  locales.
- **Motivo/resultado:** entorno local reproducible.
- **Archivos/componentes:** `docker-compose.yml`.
- **Dependencias:** PLAN-020.
- **Pruebas:** `docker compose config` y `up` (bloqueado sin Docker).
- **Criterio de aceptación:** servicios healthy y orden de arranque correcto.
- **Riesgo:** alto.
- **Estado:** pending.

#### PLAN-022 — Inicialización de MinIO

- **Descripción:** crear bucket privado de forma idempotente.
- **Motivo/resultado:** storage listo sin pasos manuales.
- **Archivos/componentes:** `infra/minio/init-bucket.sh`, Compose.
- **Dependencias:** PLAN-021.
- **Pruebas:** bucket existe; acceso anónimo deshabilitado.
- **Criterio de aceptación:** API readiness confirma storage.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-023 — Smoke test E2E

- **Descripción:** script o procedimiento para crear proyecto, job y verificar
  resultado.
- **Motivo/resultado:** demostrar el recorrido completo.
- **Archivos/componentes:** `scripts/`, `docs/DEVELOPMENT.md`.
- **Dependencias:** PLAN-021, PLAN-022.
- **Pruebas:** ejecución contra Compose.
- **Criterio de aceptación:** job termina `succeeded` y artefacto descargable.
- **Riesgo:** alto.
- **Estado:** pending.

### Etapa G — Pruebas y calidad

#### PLAN-024 — Pruebas backend

- **Descripción:** unitarias y de API con SQLite/fakes.
- **Motivo/resultado:** cubrir contratos y errores.
- **Archivos/componentes:** `apps/api/tests/`.
- **Dependencias:** PLAN-008, PLAN-009, PLAN-010.
- **Pruebas:** `uv run pytest`.
- **Criterio de aceptación:** suite verde.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-025 — Pruebas worker/proveedor/cola

- **Descripción:** processor, retries, duplicados, recuperación y fake.
- **Motivo/resultado:** validar comportamiento asíncrono.
- **Archivos/componentes:** `apps/api/tests/worker/`, `apps/worker/tests/`.
- **Dependencias:** PLAN-011, PLAN-013, PLAN-015.
- **Pruebas:** `uv run pytest`.
- **Criterio de aceptación:** estados y reintentos verificados.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-026 — Pruebas frontend

- **Descripción:** componentes y flujo principal.
- **Motivo/resultado:** evitar regresiones de UI.
- **Archivos/componentes:** `apps/web/tests/`.
- **Dependencias:** PLAN-017, PLAN-018, PLAN-019.
- **Pruebas:** `npm test`.
- **Criterio de aceptación:** interacción principal cubierta.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-027 — Integración nativa y opcional

- **Descripción:** flujo completo con fakes; variante Postgres/Redis si hay
  entorno.
- **Motivo/resultado:** demostrar el recorrido sin Docker y preparar CI.
- **Archivos/componentes:** `tests/integration/`.
- **Dependencias:** PLAN-024, PLAN-025, PLAN-026.
- **Pruebas:** `uv run pytest -m integration`; variante `-m postgres`.
- **Criterio de aceptación:** proyecto → job → succeeded → artefacto.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-028 — Lint, formato y tipos

- **Descripción:** Ruff, mypy, ESLint y `tsc --noEmit`.
- **Motivo/resultado:** calidad estática.
- **Archivos/componentes:** configuración y código.
- **Dependencias:** PLAN-024, PLAN-025, PLAN-026.
- **Pruebas:** comandos de checks.
- **Criterio de aceptación:** sin errores.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-029 — Validación de Compose

- **Descripción:** build, up, migraciones, health checks, logs y E2E real.
- **Motivo/resultado:** verificar reproducibilidad.
- **Archivos/componentes:** Compose, scripts, documentación.
- **Dependencias:** PLAN-021, PLAN-023.
- **Pruebas:** ejecución real.
- **Criterio de aceptación:** servicios healthy y flujo completo.
- **Riesgo:** alto; **bloqueado si no hay Docker**.
- **Estado:** pending.

### Etapa H — Documentación y cierre

#### PLAN-030 — Documentación principal

- **Descripción:** actualizar README, AGENTS, ARCHITECTURE y ROADMAP.
- **Motivo/resultado:** instalación y estado real documentados.
- **Archivos/componentes:** `README.md`, `AGENTS.md`, `docs/ARCHITECTURE.md`,
  `docs/ROADMAP.md`.
- **Dependencias:** PLAN-029.
- **Pruebas:** revisión de comandos y enlaces.
- **Criterio de aceptación:** otra persona puede levantar el proyecto.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-031 — API, desarrollo y ADR

- **Descripción:** `docs/API.md`, `docs/DEVELOPMENT.md`, ADR 0002 y 0003.
- **Motivo/resultado:** contratos y decisiones registrados.
- **Archivos/componentes:** docs indicados.
- **Dependencias:** PLAN-029.
- **Pruebas:** revisión de coherencia.
- **Criterio de aceptación:** documentación sin contradicciones.
- **Riesgo:** bajo.
- **Estado:** pending.

#### PLAN-032 — CI mínimo

- **Descripción:** workflow de lint, typecheck, tests y `docker compose config`.
- **Motivo/resultado:** proteger el scaffold.
- **Archivos/componentes:** `.github/workflows/ci.yml`.
- **Dependencias:** PLAN-028.
- **Pruebas:** validación de sintaxis; ejecución remota pendiente.
- **Criterio de aceptación:** workflow válido; no se afirma ejecutado.
- **Riesgo:** medio.
- **Estado:** pending.

#### PLAN-033 — Revisión final

- **Descripción:** revisar diff, buscar secretos, comprobar artefactos generados
  y emitir informe.
- **Motivo/resultado:** cierre verificable.
- **Archivos/componentes:** todo el repositorio.
- **Dependencias:** PLAN-030, PLAN-031, PLAN-032.
- **Pruebas:** `git diff`, búsqueda de secretos, ejecución final de checks.
- **Criterio de aceptación:** sin secretos; resultados reales informados.
- **Riesgo:** bajo.
- **Estado:** pending.

### 15.1 Resumen por etapas

| Etapa | Tareas | Entregable |
|---|---|---|
| A | PLAN-001..004 | Base y configuración |
| B | PLAN-005..010 | API + PostgreSQL + migraciones |
| C | PLAN-011..013 | Cola + worker |
| D | PLAN-014..015 | Proveedor fake |
| E | PLAN-016..019 | Frontend |
| F | PLAN-020..023 | Docker, MinIO y smoke test |
| G | PLAN-024..029 | Pruebas y validación |
| H | PLAN-030..033 | Documentación, CI y cierre |

---

## 16. Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Docker no instalado | Alta | Alto | Decidir instalación; si no, validar nativo y marcar Compose como pendiente |
| OneDrive y bloqueos de archivos | Media | Medio | Ignorar `node_modules`/`.venv`; recomendar mover el repo fuera de OneDrive |
| Incompatibilidad Next 16/React 19/TS 5.9 | Media | Medio | Versiones fijadas; build temprano; fallback a versiones estables |
| Diferencias SQLite/PostgreSQL | Media | Medio | Tipos genéricos; pruebas Postgres en CI |
| Fragilidad ARQ/fakeredis | Media | Medio | Abstracción de cola; probar processor directamente |
| Healthcheck de MinIO | Media | Medio | `mc ready local`; tag fijado verificado |
| Complejidad de workspaces en Docker | Media | Medio | Build temprano; fallback documentado |
| Scope creep a dominio narrativo | Media | Alto | Congelar modelo a Project + GenerationJob |
| Ausencia de auth malinterpretada | Media | Alto | Documentación explícita “solo desarrollo local” |
| Conflictos de puertos | Baja | Bajo | Variables y troubleshooting |
| CI no ejecutable localmente | Media | Bajo | Validar sintaxis; ejecutar en remoto tras push |
| Fuga de secretos | Baja | Alto | `.env` ignorado; revisión de diff; sin claves reales |

---

## 17. Criterios de aceptación

| # | Criterio | Verificación prevista |
|---|---|---|
| 1 | Monorepo creado y documentado | Árbol y README |
| 2 | Frontend funcional | Build, tests y recorrido manual |
| 3 | API arranca y expone OpenAPI | `/api/v1/docs` |
| 4 | PostgreSQL persiste proyectos y jobs | Tests de integración y Compose |
| 5 | Migraciones en entorno nuevo | `alembic upgrade head` |
| 6 | Redis y worker ejecutan jobs | Tests y Compose |
| 7 | Proveedor fake sin claves | Tests y configuración |
| 8 | Crear proyecto desde UI | Test y manual |
| 9 | Iniciar job desde UI | Test y manual |
| 10 | Procesamiento en segundo plano | Estado `queued/running → succeeded` |
| 11 | Estado y resultado consultables | API, polling y descarga |
| 12 | Docker Compose reproducible | `docker compose up` real |
| 13 | Pruebas ejecutadas | Salidas de pytest, vitest y checks |
| 14 | `.env.example` coherente | Diff contra settings |
| 15 | Documentación suficiente | Revisión del quickstart |
| 16 | Sin proveedores reales | Auditoría de código y dependencias |

**Nota sobre el criterio 12:** sin Docker instalado no puede verificarse. Se
reportará como limitación explícita; no se declarará cumplido.

---

## 18. Archivos que se crearán o modificarán

### 18.1 Crear

- `apps/api/**`
- `apps/worker/**`
- `apps/web/**`
- `packages/contracts/**`
- `infra/minio/init-bucket.sh`
- `docker-compose.yml`
- `apps/api/Dockerfile`
- `apps/web/Dockerfile`
- `.dockerignore`
- `.editorconfig`
- `package.json`, `package-lock.json`
- `pyproject.toml`, `uv.lock`
- `docs/API.md`
- `docs/DEVELOPMENT.md`
- `docs/adr/0002-async-jobs-arq.md`
- `docs/adr/0003-fake-provider-and-artifacts.md`
- `.github/workflows/ci.yml`
- tests de backend, worker y frontend

### 18.2 Modificar

- `.env.example`
- `.gitignore`
- `README.md`
- `AGENTS.md`
- `docs/ARCHITECTURE.md`
- `docs/ROADMAP.md`

### 18.3 No modificar

- `docs/PRODUCT_REQUIREMENTS.md` salvo contradicción detectada.
- `docs/LTX_DESKTOP_ASSESSMENT.md`.
- `docs/adr/0001-initial-architecture.md`.

---

## 19. Requisitos del entorno

### 19.1 Para desarrollo y pruebas nativas

- Windows, macOS o Linux.
- Node.js ≥ 20.9 (se usará 24).
- npm 11.
- Python 3.12.
- uv 0.12.
- Git 2.54.
- Espacio libre para dependencias y artefactos.
- Conexión a npm y PyPI para instalar dependencias.

### 19.2 Para validación completa

- Docker Desktop 4.x con Compose v2.
- WSL2 en Windows.
- Permisos de administrador para la instalación.
- 4 GB de RAM libres como mínimo.
- Puertos 3000, 8000, 5432, 6379, 9000 y 9001 disponibles.

### 19.3 No requerido

- GPU.
- Claves de proveedores.
- FFmpeg.
- Servicios cloud.

---

## 20. Preguntas abiertas

1. **Aprobación del plan:** ¿se aprueba este documento como referencia de
   implementación? Sin aprobación explícita no se inicia la Etapa B.
2. **Docker:** ¿se instalará Docker Desktop para validar Compose, o se acepta
   entregar Compose implementado pero con la validación marcada como pendiente?
3. **Rama:** la documentación se publicó en `docs/phase-0-1-plan`. ¿Se mantiene
   esa rama para la implementación o se integra antes en `main`?
4. **CI y contratos:** ¿se incluye el workflow de CI y la generación de tipos
   TypeScript desde OpenAPI? La recomendación es sí.
5. **Alcance del corte vertical:** ¿se acepta limitar el hito a `Project` y
   `GenerationJob`, sin story bible, personajes, escenas ni assets?
6. **Autorización:** ¿se acepta explícitamente que este hito no incluye
   autenticación y que la aplicación es solo para desarrollo local?

---

## 21. Coherencia con la documentación existente

| Documento | Coherencia | Observación |
|---|---|---|
| `PRODUCT_REQUIREMENTS.md` | Alta | Cubre proveedor fake, jobs persistentes, S3 y Compose sin claves |
| `ARCHITECTURE.md` | Alta | Respeta monolito modular, PostgreSQL, Redis, S3 y contrato de proveedor |
| `ROADMAP.md` | Media-alta | Implementa Fase 1 y un corte mínimo de Fases 2/3; se registrará la desviación |
| `LTX_DESKTOP_ASSESSMENT.md` | Alta | No se integra LTX ni se copia código |
| `ADR 0001` | Alta | No se reescribe; se añaden ADR 0002 y 0003 |
| `AGENTS.md` | Media | La autorización por recurso queda explícitamente fuera de alcance en este hito |

---

## 22. Protección del repositorio durante la planificación

- No se modificó `main`.
- No se eliminaron archivos.
- No se ejecutaron migraciones ni se alteraron bases de datos.
- No se instalaron dependencias de aplicación.
- No se modificaron archivos de dependencias.
- No se desplegaron servicios externos.
- No se consumieron APIs de pago.
- No se generó contenido audiovisual real.
- No se hizo commit ni push de código de aplicación.

Este documento se publica en una rama de documentación para revisión.

---

## 23. Aprobación

- [ ] El usuario aprueba el alcance.
- [ ] El usuario aprueba la arquitectura y las decisiones técnicas.
- [ ] El usuario resuelve la estrategia de Docker.
- [ ] El usuario aprueba el plan de pruebas.
- [ ] El usuario autoriza el inicio de la Etapa B.

**Sin las marcas anteriores no se inicia la implementación.**
