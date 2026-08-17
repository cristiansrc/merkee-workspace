# Workspace changes — MSF-ID-002

**Lifecycle status:** `revision-needed`  
**Actualizado:** 2026-08-16  
**Spec Validator:** `pending`

## Cierre y consolidación — stab-closure-consolidation (SCC-A1–A7, 2026-08-16)

Incremento exclusivamente documental de cierre/consolidación del incremento `merkee-shop-foundation-stabilization`. **No cambia rutas, endpoints, schemas OpenAPI, parámetros, migraciones Prisma, dependencias cross-service ni reglas de negocio.** No se ejecutaron operaciones de Git; la frescura de Graphify respecto a `HEAD` permanece **blocked para frescura**. No se declara `ready` de Master Spec ni listo para producción.

1. **Cierre del incremento de estabilización (SCC-A1):** el incremento `merkee-shop-foundation-stabilization` transiciona a `closed` (fue `implemented`: B1-B5 verificados en disco 2026-08-16; task board `done`; TD-MSF-STAB-001/002/003/004 `resolved-by-verified-implementation`). La delta spec y el shared context de estabilización se marcaron `closed` in-place (no `superseded`; no se movieron a `docs/specs/archive/`, que no se creó); las aprobaciones históricas `verdict: ready`, `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` se conservan como trazabilidad.
2. **Consolidación durable B1-B5 en Master Spec (SCC-A2):** se consolidaron de forma concisa en secciones existentes, sin duplicar historial STAB-* ni reeditar ADR-017 en `architecture-decisions.md`: (B1) `activate-admin` aplica el patrón ROP (puerto `Result`, adapter traduce, application sin `try/catch` técnico); (B3) fail-fast del secreto JWT en producción (`JWT_SECRET` ausente o < 32 bytes detiene arranque; default solo en desarrollo con advertencia; `.env.example` documenta); (B5) `jwt.verify` aplica el patrón ROP (puerto `Result`, adapter traduce a `DomainError` del catálogo existente, controller consume `Result` del puerto `JwtPort` sin `try/catch`). AC-05, §ROP (404 de provision, `DUPLICATE_WEBHOOK_EVENT` clasificación interna) y §NC-08 (scheduler driving adapter local) ya estaban consolidados y no se reeditaron.
3. **Registro de nuevas deudas técnicas (SCC-A4/A5):** en `docs/specs/technical_debt.md` (global) y `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local), contenido idéntico: TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC y TD-NEW-COV registradas como `active` (no resueltas); TD-NEW-STAB-LIFECYCLE registrada en "Cambios completados — no son deuda activa" como `resolved-by-verified-implementation` con evidencia (task board `done`, B1-B5 verificados, TD-MSF-STAB-001/002/003/004 resueltas, SCC-A1 aplicado). **Únicamente** la deuda de lifecycle se marca resuelta; las tres deudas `active` se resuelven en incrementos futuros.
4. **Gates de producción abiertos (SCC-A7):** TD-MSF-ID-002-02 (revisión legal/contable de retención/anonimización) y TD-MSF-ID-002-03 (AWS scheduler coordinación/reemplazo) permanecen `active` como gates antes de producción; no se declaran resueltos ni se añade evidencia de cierre legal/contable o AWS. No se declara listo para producción.
5. **Decisión JWT pendiente (STAB-DEC-12):** los valores `iss` (issuer), `aud` (audience), `typ` y el proveedor JWT (algoritmo/biblioteca) **no se fijan** canónicamente; siguen pendientes para una delta futura antes de producción. La consolidación de B3 (fail-fast del secreto) y B5 (ROP `jwt.verify`) no cubre `iss`/`aud`/`typ` ni el proveedor JWT; la implementación actual (`jsonwebtoken`/HS256 sin `iss`/`aud`) sigue provisional y no constituye la decisión canónica.

## Cambio descendente vigente

El contrato global de idempotencia exige `idempotency_records.response_json` mínimo sin PII: `resource_id`, `status`, `activation_expires_at`, `body_hash`. El replay de provisión admin reconstruye la respuesta contractual desde el recurso actual. No se cambian rutas ni endpoints. **Sí hubo un cambio contractual aprobado por el usuario (Opción A, 2026-08-16):** se añadió `404 RESOURCE_NOT_FOUND` a `provisionAdminUser` (`POST /admin/users`) en OpenAPI (cambio de contrato de comunicación autorizado); el webhook conserva `204` idempotente a duplicados (sin añadir 409; solo aclaración documental de la descripción del 204).

## Deuda técnica activa sincronizada

| ID | Estado y alcance | Gate |
|---|---|---|
| TD-MSF-ID-002-01 | `active`. Scopes desconocidos permanecen `operation_pending`; no se inventan scopes ni se permite purgarlos. Cada mutación futura debe definir scope, terminalidad, retención y prueba de purga en su delta. | No bloquea desarrollo local. |
| TD-MSF-ID-002-02 | `active`. Falta revisión legal/contable de retención y anonimización. | Gate antes de producción; requiere evidencia legal/contable explícita. |
| TD-MSF-ID-002-03 | `active`. AWS no está configurado; scheduler local y métricas productivas ya están implementados. Solo queda definir si AWS coordina o reemplaza el scheduler, ownership, alarmas y prevención de doble ejecución. | No bloquea desarrollo local; gate antes de producción si se opera con AWS. |

Los responsables, condiciones de cierre y evidencia requerida son canónicos e idénticos en `docs/specs/technical_debt.md` y `projects/merkee-shop-api/docs/specs/technical_debt.md`.

## Cambios completados con evidencia — no son deuda activa

- Secuencia aplicada: `007_idempotency_records`, `008_idempotency_records_purge_index`, `009_idempotency_records_response_json_rename`, `010_idempotency_records_response_json_backfill`, `011_idempotency_records_response_json_normalize`, `012_idempotency_records_response_json_validate` y `013_idempotency_records_response_json_strict_validate`.
- Snapshot canónico de cuatro claves y replay desde recurso vigente; no se persiste respuesta completa ni PII.
- Scheduler diario de purga cableado localmente como driving adapter (`OnApplicationBootstrap`, `setTimeout` + hora `HH:MM` UTC, defecto `02:00`, configurable; no un cron literal) y métricas productivas mediante bridge Prometheus/CloudWatch; el adapter in-memory está limitado a pruebas. AWS es decisión posterior (TD-MSF-ID-002-03).
- Bootstrap del admin inicial: no-op solo para el correo canónico con `role=admin`; otro rol falla sin mutar credenciales.
- Deudas de estabilización TD-MSF-STAB-001/002/003/004 resueltas con `resolved-by-verified-implementation` (verificadas en disco 2026-08-16): STAB-B3 (JWT fail-fast del secreto), STAB-B1 (ROP activate-admin), STAB-B2 (`must_change_password` replay `true`), STAB-B5 (ROP JWT verify) aplicadas y verificadas en `projects/merkee-shop-api/src/`; evidencia registrada en `technical_debt.md` (global y espejo local) bajo "Cambios completados — no son deuda activa".

## Pendientes de validación documental

1. Revalidación focalizada de Spec Validator del conjunto canónico: Master Spec, contrato Prisma 007–013, ADR-018/addendum, shared context y task board. Hasta un veredicto nuevo `ready`, no hay handoff ni ejecución adicional.
2. TD-MSF-ID-002-02 y TD-MSF-ID-002-03 no se declaran resueltas sin su evidencia de cierre respectiva; son gates de producción según la tabla de deuda activa.
3. TD-MSF-STAB-001/002/003/004 (deudas de estabilización) están resueltas con `resolved-by-verified-implementation` (verificadas en disco 2026-08-16): STAB-B3/B1/B2/B5 aplicadas y verificadas en `projects/merkee-shop-api/src/`; evidencia registrada en `technical_debt.md` (global y espejo local) bajo "Cambios completados — no son deuda activa". TD-MSF-ID-002-02 y TD-MSF-ID-002-03 permanecen como gates de producción abiertos.
4. **Drifts resueltos por decisión del usuario (Opción A, 2026-08-16) — ya no `decision-required`:**
   - **STAB-DRIFT-01 (404 de provision) — resuelto Opción A:** el usuario aprobó añadir `404 RESOURCE_NOT_FOUND` a OpenAPI `provisionAdminUser` (`POST /admin/users`). El Planner aplicó el cambio en `docs/api/openapi.yaml` (línea 205) y aclaró la Master Spec §ROP (catálogo + sección provisión admin): en replay idempotente, si el recurso aprovisionado fue eliminado y no existe para reconstruir la respuesta, la provisión devuelve `404 RESOURCE_NOT_FOUND` sin inventar datos. El código ya emitía este código; OpenAPI ahora lo declara. Es un cambio de contrato de comunicación (añade status 404) autorizado. Corrección de código pendiente: `operation-map.ts` debe reflejar el 404 (STAB-B4, Executor). Opción B (eliminar 404 del código) rechazada.
   - **STAB-DRIFT-02 (webhook duplicate semantics) — resuelto Opción A:** el usuario aprobó que el webhook HTTP devuelve `204` idempotente a duplicados (OpenAPI sin cambio de statuses) y `DUPLICATE_WEBHOOK_EVENT` (409) es clasificación interna del rail ROP no proyectada al proveedor. El Planner aplicó la aclaración en la Master Spec §ROP (catálogo: `DUPLICATE_WEBHOOK_EVENT` 409 es clasificación interna, no status HTTP del webhook; los webhooks responden 204 idempotente al proveedor) y en la descripción del `204` de OpenAPI de `/webhooks/wompi` y `/webhooks/mercado-pago` (líneas 343 y 354). No se añadió 409 a los webhooks (statuses 204/400/401/500 intactos). No requiere corrección de código. Opción B (añadir 409 a webhooks) rechazada.
5. **Decisión pendiente JWT (no fijada canónicamente):** los valores `iss` (issuer), `aud` (audience), `typ` y el proveedor JWT (algoritmo/biblioteca) no están definidos canónicamente. La implementación actual (`jsonwebtoken`/HS256, sin `iss`/`aud` explícitos) es provisional y no constituye la decisión canónica. STAB-B3 cubre solo el fail-fast del secreto (`JWT_SECRET`); STAB-B5 cubre solo la traducción ROP de `jwt.verify` (excepciones→`DomainError`/`Result`). **Ninguno cubre** `iss`/`aud`/`typ` ni el proveedor JWT. La decisión se toma en una delta futura antes de producción; no se elige en este incremento. No es código pendiente (B1-B5) ni deuda técnica (TD-MSF-*).

## Coordinación y gate

Estado local afectado: `revision-needed`. No se cambian rutas ni endpoints ni dependencias cross-service nuevas. **Sí existe un cambio de contrato HTTP aprobado por el usuario (Opción A, 2026-08-16):** `404 RESOURCE_NOT_FOUND` añadido a `provisionAdminUser` en OpenAPI (cambio de contrato de comunicación autorizado); el webhook conserva `204` idempotente a duplicados (sin añadir 409; aclaración documental de la descripción del 204). Las cuatro deudas de estabilización (TD-MSF-STAB-001/002/003/004) no bloquean desarrollo local; TD-MSF-ID-002-02 y TD-MSF-ID-002-03 bloquean producción en los términos definidos arriba. La aprobación `ready` y la aprobación humana históricas son trazabilidad **superseded**; no autorizan continuar.

## Controlled Pre-Ready Remediation: approved_by_user

**Autorización del usuario:** 2026-08-16 · **Alcance:** STAB-B1, STAB-B2, STAB-B3, STAB-B4, STAB-B5 (exhaustivo y exclusivo) · **Tipo:** remediación controlada documental previa al veredicto `ready` del Spec Validator · **Estado del gate:** Spec Validator permanece `pending`/`not ready`; lifecycle de `workspace_changes.md` permanece `revision-needed`; incremento de estabilización permanece `planning`.

### Razón — bloqueo circular validator↔implementation

Existe un bloqueo circular entre el gate de Spec Validator y la ejecución de las correcciones de código del incremento de estabilización `merkee-shop-foundation-stabilization`:
- Spec Validator no puede emitir veredicto `ready` mientras los drifts de código STAB-B1/B2/B3/B4/B5 permanezcan sin corregir (son hallazgos bloqueantes).
- El Planner no puede enrutar STAB-B1/B2/B3/B4/B5 a Executor/Task Decomposer sin un veredicto `ready` vigente (regla del sistema: "Planner no debe hacer handoff salvo que el último veredicto de Spec Validator sea exactamente `ready`").

El usuario autoriza esta remediación controlada para romper el bloqueo circular, permitiendo que las correcciones B1-B5 se documenten y preparen como un conjunto excepcional, acotado y trazable, **sin eludir** el gate de Spec Validator: la revalidación focalizada sigue siendo obligatoria tras la verificación de B1-B5. Detalle canónico completo en `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` (`## Controlled Pre-Ready Remediation: approved_by_user`) y en `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` (misma sección).

### Alcance exacto (exhaustivo y exclusivo)

| ID | Corrección de código | Archivos autoritativos (verificados en disco el 2026-08-16; NO modificados por el Planner) |
|---|---|---|
| STAB-B1 | ROP activate-admin: puerto `ActivateAdminUnitOfWorkPort.run` → `Result<T, DomainError>`; adapter `PrismaActivateAdminUnitOfWorkAdapter` captura/traduce a `fail(technicalFailure())` sin causa/PII; `activate-admin.use-case.ts` sin `try/catch` técnico; DI en `identity.module.ts`. | `projects/merkee-shop-api/src/modules/identity/domain/ports/activate-admin-unit-of-work.port.ts`; `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/prisma-activate-admin-unit-of-work.adapter.ts`; `projects/merkee-shop-api/src/modules/identity/application/use-cases/activate-admin.use-case.ts`; `projects/merkee-shop-api/src/modules/identity/identity.module.ts` |
| STAB-B2 | `must_change_password` en replay: `provision-admin-user.use-case.ts` fuerza `true` en `replayFromResource`; `schemas.ts` tipa `must_change_password: true` (literal); OpenAPI `const: true` sin cambio (STAB-DEC-01 Opción A). | `projects/merkee-shop-api/src/modules/identity/application/use-cases/provision-admin-user.use-case.ts`; `projects/merkee-shop-api/src/contract/schemas.ts` |
| STAB-B3 | JWT fail-fast: `jwt.adapter.ts` lanza error de arranque en `NODE_ENV=production` si `JWT_SECRET` ausente o < 32 bytes; default solo en desarrollo con advertencia; `.env.example` documenta `JWT_SECRET`. | `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts`; `projects/merkee-shop-api/.env.example` |
| STAB-B4 | `operation-map.ts` actualiza el mapa de trazabilidad `provisionAdminUser` para reflejar el `404` añadido en OpenAPI (statuses: 201,400,401,403,404,409,429,500). | `projects/merkee-shop-api/src/contract/operation-map.ts` |
| STAB-B5 | ROP JWT verify: `JwtPort.verify` → `Result<JwtPayload, DomainError>`; `JwtAdapter.verify` captura excepciones de `jwt.verify` (`TokenExpiredError`/`JsonWebTokenError`/`NotBeforeError`) y traduce a `DomainError` del catálogo existente (`AUTHENTICATION_REQUIRED` 401 / `TECHNICAL_DEPENDENCY_FAILURE` 500) sin causa/PII, devuelve `fail(...)`; el controller `extractSessionId`/`extractUserId` consume `Result` del puerto `JwtPort` (no del adapter concreto), elimina el `try/catch` y maneja el rail `Failure`; DI en `identity.module.ts` (controller inyecta `JwtPort`, no `JwtAdapter`). **No cubre** `iss`/`aud`/`typ` ni el proveedor JWT (decisión pendiente STAB-DEC-12) ni el fail-fast del secreto (B3). | `projects/merkee-shop-api/src/modules/identity/domain/ports/jwt.port.ts`; `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts` (compartido con B3); `projects/merkee-shop-api/src/modules/identity/identity.controller.ts`; `projects/merkee-shop-api/src/modules/identity/identity.module.ts` |

### Límites

- Esta remediación controlada se limita **exclusivamente** a STAB-B1, STAB-B2, STAB-B3, STAB-B4 y STAB-B5. Cualquier otro hallazgo o corrección (incluida la deuda técnica C y los gates de producción) queda fuera de alcance.
- El Planner solo actualiza **documentación** del incremento de estabilización (shared context, delta spec y este `workspace_changes.md`). El Planner no crea, edita ni parchea código, Prisma, `package.json`, runtime, tests, migraciones, scripts, UI, API handlers, queries ni automatización.
- Las correcciones de código B1-B5 son propiedad del Executor; el Planner no las ejecuta ni las descompone en tareas (Task Decomposer es dueño de la descomposición tras el gate).
- Esta autorización es una remediación controlada **previa al veredicto `ready`**: no constituye un veredicto `ready`, no sustituye la revalidación focalizada de Spec Validator y no elimina el gate de aprobación humana (`## Human Plan Approval: approved_by_user`).

### Prohibiciones

- **No declarar `ready`**: Spec Validator permanece `pending`/`not ready`; el lifecycle del incremento de estabilización permanece `planning` y el de `workspace_changes.md` permanece `revision-needed`.
- **No hacer handoff** a Task Decomposer, Executor ni Architect Executor sin veredicto `ready` de Spec Validator + aprobación humana (`## Human Plan Approval: approved_by_user`).
- **No autorizar nuevas funcionalidades**: no se crean endpoints, módulos, migraciones, schemas, parámetros, reglas de negocio ni integraciones nuevas.
- **No autorizar task board general**: el task board global permanece `blocked` (TD-MSF-API-006); esta remediación no desbloquea tareas fuera de B1-B5.
- **No autorizar producción**: los gates TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen; no se despliega ni se opera en producción.
- **No autorizar Git**: no se ejecutan commits, branches, PRs ni operaciones de Git; la frescura de Graphify respecto a `HEAD` permanece **blocked para frescura**.
- **No modificar** código, Prisma (`schema.prisma`, migraciones), `package.json`, runtime config, `.env` real, ni ningún archivo de implementación.

### Evidencia requerida (para cerrar B1-B5 y habilitar revalidación)

- STAB-B1: test estático sin `try/catch` técnico en `application`; test del adapter traduciendo fallo técnico a `TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII; `npm run depcruise` sin violaciones.
- STAB-B2: test de replay tras activación devuelve `must_change_password: true`; diff de `schemas.ts` con literal `true`; OpenAPI `const: true` sin cambio.
- STAB-B3: test de fail-fast en producción (`NODE_ENV=production`, `JWT_SECRET` ausente o < 32 bytes); diff de `.env.example` con `JWT_SECRET` documentado.
- STAB-B4: diff de `operation-map.ts` con `provisionAdminUser` statuses `201,400,401,403,404,409,429,500` consistente con OpenAPI.
- STAB-B5: `JwtPort.verify` devuelve `Result<JwtPayload, DomainError>`; test del adapter traduciendo `TokenExpiredError`/`JsonWebTokenError`/`NotBeforeError` a `DomainError` del catálogo existente (`AUTHENTICATION_REQUIRED` 401 / `TECHNICAL_DEPENDENCY_FAILURE` 500) sin causa/PII; el controller `extractSessionId`/`extractUserId` consume `Result` del puerto `JwtPort` (no del adapter concreto) sin `try/catch`; DI actualizada en `identity.module.ts`.
- Global: `npm run build` OK; `npm test` PASS con suites actualizadas (activate-admin, provision replay, JWT fail-fast, JWT verify ROP); `npm run depcruise` sin violaciones; sin `HttpException` ni `try/catch` técnico en `domain`/`application`; el controller no captura excepciones técnicas de `jwt.verify` (consume `Result`).

### Lo que esta autorización NO autoriza

Esta remediación controlada **no autoriza**: (1) nuevas funcionalidades, endpoints, módulos, migraciones, schemas, parámetros ni reglas de negocio; (2) el desbloqueo del task board general (permanece `blocked`); (3) despliegue u operación en producción; (4) operaciones de Git (commits, branches, PRs); (5) la modificación de código, Prisma, `package.json` ni runtime por parte del Planner; (6) handoff a otros agentes sin veredicto `ready` + aprobación humana; (7) declarar `ready` ni transicionar el incremento fuera de `planning`/`revision-needed`.
