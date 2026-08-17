# Shared Context — merkee-shop-foundation

**Lifecycle status:** `revision-needed`  
**Actualizado:** 2026-08-16

## Current status

`revision-needed`: las migraciones 010–013, las correcciones ROP, la precisión documental posterior sobre la clasificación exclusiva de purga (`minimum_age_not_elapsed` → `replay_active` → `retention_not_elapsed` → `operation_pending` → `eligible`) y la formalización sincronizada de TD-MSF-ID-002-01/02/03 requieren revalidación focalizada. Las tres deudas no bloquean desarrollo local; TD-MSF-ID-002-02 y TD-MSF-ID-002-03 son gates antes de producción. El veredicto `ready` histórico está **superseded** e invalidado; Spec Validator permanece `pending`. No hay handoff ni nueva ejecución.

## Canonical artifacts

- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` (ADR-010, ADR-018 y addendum)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` (solo lectura para este incremento padre; el incremento de estabilización aplicó cambios aprobados: `404` en `provisionAdminUser` STAB-DRIFT-01 Opción A, descripción del `204` aclarada en webhooks STAB-DRIFT-02 Opción A, 10 `x-idempotency` STAB-A7)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/tasks/merkee-shop-foundation-task-board.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` (evidencia de grafo en disco; solo lectura; frescura respecto a HEAD no verificable sin Git — blocked para frescura)
- Migraciones canónicas evidenciadas en el task board: `007_idempotency_records`, `008_idempotency_records_purge_index`, `009_idempotency_records_response_json_rename`, `010_idempotency_records_response_json_backfill`, `011_idempotency_records_response_json_normalize`, `012_idempotency_records_response_json_validate`, `013_idempotency_records_response_json_strict_validate`.

## Artifact evidence

| Ruta | Campo/flujo verificado | Resultado | Estado |
|---|---|---|---|
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` | §Corrección MSF-ID-002: snapshot y clasificación de purga | Define snapshot de cuatro campos sin PII, precedencia exclusiva, protección mínima de 24 h, retención general de 30 días y prioridad de `replay_active`; requiere revalidación. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md` | Secuencia 007–013 y §013 | Registra rename/backfill/normalización/validación; 013 exige UUID v1–v8 con variante RFC 4122, snapshot exacto y derivación solo desde token vigente. Aclara que 008 es solo índice histórico/de soporte y que la ventana operativa surge de la evaluación contractual de 24 h/30 días. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` | ADR-018 y addendum | Fija snapshot mínimo, purga exclusiva, bridge productivo y scheduler local; confirma evidencia 007–013 y ROP pendiente de revalidación. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md` | lifecycle, aprobación y guardas | Registra el cambio posterior y mantiene `revision-needed`/`pending`. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/tasks/merkee-shop-foundation-task-board.md` | MSF-ID-002 y migración 013 | Marca superseded la evidencia de snapshot de tres campos y de deploy con 8 migraciones; conserva como única evidencia vigente 34 suites/249 tests, deploy 007–013, snapshot de cuatro claves con `body_hash`, diff sin drift e integración PostgreSQL. El tablero global sigue bloqueado por validación pendiente. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` | Grafo actual y dependencias | Reporte fechado 2026-08-16: 1,055 nodos, 2,430 aristas, 68 comunidades, sin ciclos de importación; construido desde el commit `89bcd155`. La igualdad con `HEAD` no es verificable sin Git (blocked para frescura); no se afirma que el grafo está actualizado respecto a `HEAD`. Relaciona el flujo MSF-ID-002 con el conjunto canónico. No se modificó. | pass (contenido) / blocked (frescura) |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md` y `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md` | TD-MSF-ID-002-01/02/03 | Registros espejo con responsable, condición de cierre, evidencia requerida y estado idénticos; TD-01 no inventa scopes, TD-02 requiere evidencia legal y TD-03 no declara AWS configurado. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md` | Sincronización descendente de deuda y completados | Separa las tres deudas activas de los cambios completados; fija TD-02/03 como gates de producción y conserva el scheduler/métricas locales como implementados. | pass |

## Spec Validator Approval

- verdict: pending
- reviewed_at: none
- validator_agent: spec-validator
- artifact_set_reviewed: pending focal revalidation of MSF-ID-002 against master spec, Prisma contract (007-013), ADR-018 addendum, shared context and task board
- summary: Revalidación solicitada: 010-013 endurecieron snapshot/UUID/token vigente; ROP y la precedencia exclusiva de purga se corrigieron después del veredicto histórico; no existe un veredicto ready vigente.
- invalidated_by_changes_since: migrations 010-013, ROP corrections, and exclusive purge classification/retention clarification

## Decisions locked

- `response_json` is the canonical idempotency snapshot: `resource_id`, `status`, `activation_expires_at`, `body_hash`; it never stores PII or a full response.
- Replay validates the request hash and rebuilds the existing contractual DTO from the current resource; no OpenAPI change is authorized.
- La clasificación de purga es exclusiva y ordenada: `minimum_age_not_elapsed` (<24 h), `replay_active` (replay válido dentro de 30 días), `retention_not_elapsed` (dentro de 30 días sin replay válido), `operation_pending` (fuera de retención, pendiente o scope desconocido) y `eligible`. `retention_not_elapsed` es general/residual y no crea comportamiento distinto; `replay_active` es prioritario. `admin-provision` terminal solo puede purgar tras las protecciones.
- Production metrics use Prometheus/CloudWatch bridge; in-memory metrics are test-only. The daily scheduler is locally wired as a driving adapter (`OnApplicationBootstrap`, `setTimeout` + `HH:MM` UTC, configurable; not a literal cron); AWS coordination/replacement is a later decision (TD-MSF-ID-002-03).
- Bootstrap is no-op only for canonical email with `role=admin`; another role fails closed and does not rewrite credentials.
- 013 es la validación final del snapshot: UUID v1–v8 con variante RFC 4122; `activation_expires_at` solo se deriva de token no usado y vigente, seleccionado determinísticamente; de no existir, el registro se elimina sin inventar datos.
- El conjunto real aplicado es: `007_idempotency_records`, `008_idempotency_records_purge_index`, `009_idempotency_records_response_json_rename`, `010_idempotency_records_response_json_backfill`, `011_idempotency_records_response_json_normalize`, `012_idempotency_records_response_json_validate` y `013_idempotency_records_response_json_strict_validate`.
- La migración 008 es únicamente un índice histórico/de soporte; no decide la ventana de selección de purga. La evaluación contractual vigente usa las protecciones de 24 h y retención/replay de 30 días.
- TD-MSF-ID-002-01: scopes desconocidos se conservan como `operation_pending`; no se inventan scopes ni se cierra esta deuda hasta que cada mutación futura defina scope, terminalidad, retención y prueba de purga en una delta.
- TD-MSF-ID-002-02: la revisión legal/contable de retención y anonimización es obligatoria antes de producción; no existe cierre sin evidencia legal explícita.
- TD-MSF-ID-002-03: AWS no está configurado. Scheduler local y métricas productivas ya están implementados; antes de producción con AWS debe decidirse coordinación o reemplazo, ownership, alarmas y prevención de doble ejecución.
- Trazabilidad con el incremento de estabilización (`merkee-shop-foundation-stabilization`, `planning`): la decisión "no OpenAPI change is authorized" (línea anterior sobre replay) se refiere exclusivamente al DTO contractual del replay de MSF-ID-002 (`AdminUserProvisionResponse`): el replay reconstruye el DTO existente sin cambiar sus campos. No prohíbe añadir statuses de error a operaciones existentes en un incremento separado con autorización del usuario. El incremento de estabilización (STAB-DRIFT-01 Opción A, aprobada por el usuario 2026-08-16) añadió `404 RESOURCE_NOT_FOUND` a `provisionAdminUser` en OpenAPI — cambio de contrato de comunicación autorizado, no contradictorio con esta decisión del replay. Ver shared context de estabilización STAB-DEC-10/11.

## Validator findings

- Revalidar que las migraciones 007-013, incluido el rename/backfill/normalización/validación, concuerdan con el snapshot mínimo, UUID versión/variante y derivación solo desde token vigente sin PII.
- Revalidar que ADR-018 y su addendum concuerdan con el bridge productivo de métricas y el scheduler diario local.
- Revalidar que la corrección MSF-ID-002 conserva el no-op de bootstrap exclusivamente para el correo canónico con `role=admin`.
- Confirmar que ninguna aprobación histórica `ready` o humana se interpreta como autorización para el siguiente handoff mientras `verdict: pending`.
- Revalidar que la precedencia de clasificación está idéntica en Master Spec, contrato Prisma y ADR-018, y que no se interpreta `retention_not_elapsed` como una ventana adicional.

## Resolved findings

- Contract drift on full PII-capable response storage is replaced by minimum snapshot and replay reconstruction.
- Semántica documental de purga resuelta: `retention_not_elapsed` es la razón general residual de no superar 30 días; `replay_active` es la razón específica prioritaria y cada candidato recibe una sola clasificación.
- Metric names, production/test adapter boundaries and local scheduler ownership are explicit.
- Bootstrap no-op semantics now require role validation.

## Open questions

- Legal/contable review of the general retention/anonimization policy remains required before production; it is not a waiver for the no-PII snapshot.
- TD-MSF-ID-002-03 requiere decisión operativa AWS antes de producción si AWS participa; no existe configuración AWS ni evidencia de coordinación/reemplazo.
- No hay blocker técnico de las tres deudas para desarrollo local: el task board registra pruebas de build, 249 tests, Prisma y PostgreSQL para la implementación 007–013. Permanecen la revalidación de especificación y los gates de producción TD-MSF-ID-002-02/03.

## Human Plan Approval — HISTÓRICO / SUPERSEDED / INVALIDADO

**Esta aprobación NO está vigente.** Se conserva únicamente como trazabilidad histórica. Queda **superseded e invalidada** para continuar este incremento por los cambios materiales ADR-018, migraciones 010–013, correcciones ROP y purga MSF-ID-002. No autoriza handoff, ejecución, desbloqueo del task board ni continuar con la siguiente tarea. El encabezado canónico `## Human Plan Approval: approved_by_user` NO existe en este contexto como aprobación vigente; tras la revalidación focalizada de Spec Validator y un veredicto `ready`, se requerirá emitir una nueva aprobación humana aplicable antes de continuar. Mientras `verdict: pending`, ninguna aprobación histórica (esta incluida) sustituye el gate de revalidación.

## Stale terms guard

Forbidden: `response` JSONB, respuesta original/completa o snapshot de tres campos como regla/evidencia vigente (HISTÓRICO/SUPERSEDED); full `AdminUserProvisionResponse` in idempotency storage; PII/token/password/hash in `response_json`, logs or metrics; replay from stored PII; UUID fuera de v1–v8/RFC 4122 en `resource_id`; derivar `activation_expires_at` desde token usado/expirado; usar 008 para fijar la ventana operativa en lugar de la evaluación contractual 24 h/30 días; purge of `replay_active` or unknown scope; inventar scope, terminalidad o prueba de purga para TD-MSF-ID-002-01; declarar cerrada TD-MSF-ID-002-02 sin evidencia legal/contable; afirmar AWS configurado o cerrada TD-MSF-ID-002-03 sin decisión de coordinación/reemplazo, ownership, alarmas y prevención de doble ejecución; treating `retention_not_elapsed` as a separate retention window or purge behavior; in-memory metrics in production; AWS-only scheduler without local driving adapter; bootstrap no-op for non-admin role; new endpoint/OpenAPI change; `POST /auth/initial-password-change`; afirmar que el commit del grafo (`89bcd155`) es igual a `HEAD` o que el grafo está actualizado respecto a `HEAD` sin verificación con Git (blocked para frescura).

## Next action

Solicitar revalidación focalizada de Spec Validator para MSF-ID-002 contra la Master Spec, contrato Prisma 007–013, ADR-018, este contexto, los registros espejo de deuda y el task board, verificando snapshot estricto, UUID/version/variant, token vigente, ROP, la precedencia exclusiva `minimum_age_not_elapsed` → `replay_active` → `retention_not_elapsed` → `operation_pending` → `eligible` y la separación entre desarrollo local y gates de producción TD-MSF-ID-002-02/03. No declarar `ready`, no hacer handoff y no continuar con la siguiente tarea mientras `verdict: pending`; las aprobaciones históricas no eliminan este gate.
