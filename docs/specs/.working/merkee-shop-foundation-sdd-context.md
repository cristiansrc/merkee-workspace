# Shared Context — merkee-shop-foundation

**Lifecycle status:** `validated-not-executed`  
**Actualizado:** 2026-08-17

## Current status

`validated-not-executed`: Spec Validator emitió `ready` el 2026-08-17 sobre el conjunto actual de artefactos. MSF-ID-003 está implementada y la siguiente tarea habilitada es MSF-CAT-001. Las tres deudas TD-MSF-ID-002-01/02/03 permanecen activas y no bloquean desarrollo local; TD-MSF-ID-002-02 y TD-MSF-ID-002-03 siguen siendo gates antes de producción. No se modificaron contratos ni criterios; las referencias previas a `revision-needed`/`pending`, al ready invalidado y a la revalidación requerida quedan únicamente como trazabilidad histórica **superseded**.

## Canonical artifacts

- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` (ADR-010, ADR-018 y addendum, ADR-019)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` (solo lectura para este incremento padre; el incremento de estabilización aplicó cambios aprobados: `404` en `provisionAdminUser` STAB-DRIFT-01 Opción A, descripción del `204` aclarada en webhooks STAB-DRIFT-02 Opción A, 10 `x-idempotency` STAB-A7; la actualización documental 2026-08-17 del replay de password-change ajustó solo descripciones de `POST /auth/password-change` — sin statuses/rutas/schemas)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/tasks/merkee-shop-foundation-task-board.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` (evidencia de grafo en disco; solo lectura; frescura respecto a HEAD no verificable sin Git — blocked para frescura)
- Migraciones canónicas evidenciadas en el task board: `007_idempotency_records`, `008_idempotency_records_purge_index`, `009_idempotency_records_response_json_rename`, `010_idempotency_records_response_json_backfill`, `011_idempotency_records_response_json_normalize`, `012_idempotency_records_response_json_validate`, `013_idempotency_records_response_json_strict_validate` y `014_password_reset_tokens_active_unique_index`.

## Artifact evidence

| Ruta | Campo/flujo verificado | Resultado | Estado |
|---|---|---|---|
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` | §Corrección MSF-ID-002: snapshot y clasificación de purga | Define snapshot de cuatro campos sin PII, precedencia exclusiva, protección mínima de 24 h, retención general de 30 días y prioridad de `replay_active`; requiere revalidación. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md` | Secuencia 007–013 y §013 | Registra rename/backfill/normalización/validación; 013 exige UUID v1–v8 con variante RFC 4122, snapshot exacto y derivación solo desde token vigente. Aclara que 008 es solo índice histórico/de soporte y que la ventana operativa surge de la evaluación contractual de 24 h/30 días. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` | ADR-018 y addendum | Fija snapshot mínimo, purga exclusiva, bridge productivo y scheduler local; confirma evidencia 007–013 y ROP pendiente de revalidación. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md` | lifecycle, aprobación y guardas | Registra el cambio posterior y mantiene `revision-needed`/`pending`. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/tasks/merkee-shop-foundation-task-board.md` | MSF-ID-002 y migración 013 | Marca superseded la evidencia de snapshot de tres campos y de deploy con 8 migraciones; conserva como única evidencia vigente 34 suites/249 tests, deploy 007–013, snapshot de cuatro claves con `body_hash`, diff sin drift e integración PostgreSQL. MSF-ID-003 implementada (42 suites/297 tests) y pendiente de revalidación; el tablero global sigue bloqueado por validación pendiente. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` | Grafo actual y dependencias | Reporte fechado 2026-08-16: 1,245 nodos, 2,603 aristas, 87 comunidades, sin ciclos de importación; construido desde el commit `89bcd155`. La igualdad con `HEAD` no es verificable sin Git (blocked para frescura); no se afirma que el grafo está actualizado respecto a `HEAD`. Relaciona el flujo MSF-ID-002 con el conjunto canónico. No se modificó. | pass (contenido) / blocked (frescura) |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md` y `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md` | TD-MSF-ID-002-01/02/03 | Registros espejo con responsable, condición de cierre, evidencia requerida y estado idénticos; TD-01 no inventa scopes, TD-02 requiere evidencia legal y TD-03 no declara AWS configurado. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md` | Sincronización descendente de deuda y completados | Separa las tres deudas activas de los cambios completados; fija TD-02/03 como gates de producción y conserva el scheduler/métricas locales como implementados. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/prisma/migrations/014_password_reset_tokens_active_unique_index/migration.sql` | Migración 014 | `CREATE UNIQUE INDEX IF NOT EXISTS idx_password_reset_tokens_active_per_user ON password_reset_tokens (user_id) WHERE used_at IS NULL`; no edita 001–013; comentario de expand/contract y preflight de duplicados. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/application/use-cases/request-password-reset.use-case.ts` | Request password reset | 202 neutro; token opaco 32 bytes; hash SHA-256; invalidación + creación en UoW atómico; email después del commit; sin token/PII en logs. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/application/use-cases/reset-password.use-case.ts` | Reset password | Consumo atómico del token; hash Argon2id fuera de transacción; revoca TODAS las sesiones; 422 neutro para token inválido/expirado/usado. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/noop-email.adapter.ts` | Email/outbox seguro | No registra token, email ni PII en ningún entorno; token en claro solo al `EmailPort`. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/identity.controller.ts` | Rutas de password reset | `POST /auth/password-reset-requests` (202) y `POST /auth/password-resets` (204) implementadas; sin cambios OpenAPI. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` | `POST /auth/password-change` (descripciones) | Descripción, `x-idempotency` y descripción del `204`/`Set-Cookie` documentan replay seguro: 204 sin Set-Cookie en replay, 409 divergente, detección de replay antes de `current_password`, sin credenciales en `idempotency_records`/logs/métricas. Statuses/rutas/schemas intactos. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` | §Replay seguro de `POST /auth/password-change` | Nueva subsección con primera ejecución, replay equivalente, replay divergente, orden de validación, minimización de credenciales y concurrencia; AC-07 y tabla ROP actualizadas. | pass |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` | ADR-020 | Documenta la decisión de replay seguro de password-change con minimización de credenciales; sin patrón GoF nuevo; no requiere nueva consulta de Solution Architect. | pass |

## Spec Validator Approval

- verdict: ready
- reviewed_at: 2026-08-17
- validator_agent: spec-validator
- artifact_set_reviewed: master spec, Prisma migration contract (007-014), ADR-018 addendum, ADR-019, ADR-020, shared context, task board and current OpenAPI descriptions
- summary: Spec Validator confirmó `ready` sobre el conjunto actual de artefactos, sin hallazgos pendientes para la ejecución documental. La aprobación habilita continuar con MSF-CAT-001; no aprueba readiness de producción ni declara resuelta la deuda activa.
- invalidated_by_changes_since: none

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
- Password reset (MSF-ID-003): `POST /auth/password-reset-requests` devuelve 202 neutro (no revela existencia); token opaco 32 bytes, hash SHA-256, expiración 30 min; invalidación del token activo previo + creación del nuevo en una única transacción `SERIALIZABLE` (`RequestPasswordResetUnitOfWorkPort`) con rollback total; email enviado después del commit vía `EmailPort` (v1 `NoopEmailAdapter`, sin PII/token en logs).
- `POST /auth/password-resets` devuelve 204; consumo atómico del token (`markAsUsed`), hash Argon2id fuera de transacción, revocación de TODAS las sesiones del usuario en una única transacción `SERIALIZABLE` (`ResetPasswordUnitOfWorkPort`); token inválido/expirado/usado → mismo `422 PASSWORD_RESET_TOKEN_INVALID_OR_EXPIRED` sin filtrar estado.
- Migración 014 (`password_reset_tokens_active_unique_index`): índice parcial único `idx_password_reset_tokens_active_per_user ON password_reset_tokens(user_id) WHERE used_at IS NULL`; máximo un token no usado por usuario; no edita 001–013; requiere preflight PostgreSQL si existen tokens activos duplicados preexistentes.
- Email/outbox seguro: el token en claro solo se entrega al `EmailPort` para el enlace del email; nunca se loguea, metrica, traza ni expone en response. El canal de entrega productivo (outbox/SES/SMTP) es decisión operacional pendiente, no implementada en v1.
- Sin cambios OpenAPI para MSF-ID-003: los endpoints de password reset ya estaban declarados en `docs/api/openapi.yaml`.
- Replay seguro de `POST /auth/password-change` (ADR-020, decisión aprobada 2026-08-17): primera ejecución exitosa valida `current_password`, cambia hash, limpia `must_change_password`, revoca las demás sesiones, rota cookie y responde `204` + `Set-Cookie`; replay equivalente (mismo principal/clave/cuerpo) responde `204` sin `Set-Cookie` y sin repetir mutación/rotación/revocación; replay divergente responde `409 IDEMPOTENCY_KEY_REUSED`; la detección de replay precede a la validación de `current_password` (una solicitud nueva sí la valida, `422 CURRENT_PASSWORD_INVALID`); `idempotency_records`, logs y métricas nunca persisten refresh token, hash de sesión, contraseña, secreto ni credenciales derivadas (snapshot mínimo `status 204` + `body_hash`); concurrencia con retry/relectura determinista sin `500` espurio ni doble rotación.

## Historical Validator findings (superseded)

- Revalidar que las migraciones 007-013, incluido el rename/backfill/normalización/validación, concuerdan con el snapshot mínimo, UUID versión/variante y derivación solo desde token vigente sin PII.
- Revalidar que ADR-018 y su addendum concuerdan con el bridge productivo de métricas y el scheduler diario local.
- Revalidar que la corrección MSF-ID-002 conserva el no-op de bootstrap exclusivamente para el correo canónico con `role=admin`.
- Confirmar que ninguna aprobación histórica `ready` o humana se interpreta como autorización para el siguiente handoff mientras `verdict: pending`.
- Revalidar que la precedencia de clasificación está idéntica en Master Spec, contrato Prisma y ADR-018, y que no se interpreta `retention_not_elapsed` como una ventana adicional.
- Revalidar el **último cambio de código** `idempotency_responsejson_rop_purge_note (2026-08-16)`: (a) drift canónico `responseJson`/`response_json` eliminado en `domain/ports/idempotency.port.ts`, `infrastructure/adapters/prisma-idempotency.adapter.ts` y `application/use-cases/provision-admin-user.use-case.ts` (ningún sitio conserva la forma legacy `response`); (b) `PurgeIdempotencyRecordsUseCase.execute()` devuelve `Promise<Result<void, DomainError>>` y `application` no contiene `try/catch` técnico; el adapter captura y traduce las excepciones técnicas en su límite. Sin migraciones nuevas y sin cambios OpenAPI.
- **Nuevos hallazgos MSF-ID-003 (2026-08-17):** revalidar que la migración 014 (`password_reset_tokens_active_unique_index`) concuerda con el contrato Prisma §014 y la Master Spec (índice parcial único `WHERE used_at IS NULL`, sin editar 001–013); revalidar que `request-password-reset.use-case.ts` y `reset-password.use-case.ts` cumplen ROP (sin `try/catch` técnico en application, adapters traducen a `TECHNICAL_DEPENDENCY_FAILURE` sin PII); revalidar que `NoopEmailAdapter` no registra token/email/PII; revalidar que el controller expone `POST /auth/password-reset-requests` (202) y `POST /auth/password-resets` (204) sin cambios OpenAPI; revalidar que el riesgo de tokens activos duplicados preexistentes está registrado con preflight PostgreSQL.
- **Nuevos hallazgos replay de password-change (2026-08-17):** revalidar que ADR-020, la Master Spec (§Replay seguro de `POST /auth/password-change`) y las descripciones OpenAPI del endpoint concuerdan: replay equivalente `204` sin `Set-Cookie` y sin repetir mutación; replay divergente `409 IDEMPOTENCY_KEY_REUSED`; detección de replay antes de re-exigir `current_password`; `idempotency_records`/logs/métricas sin refresh token, hash de sesión, contraseña, secreto ni credenciales derivadas; concurrencia determinista sin `500` espurio ni doble rotación. Verificar que OpenAPI no cambió statuses/rutas/schemas (solo descripciones).

## Resolved findings

- Contract drift on full PII-capable response storage is replaced by minimum snapshot and replay reconstruction.
- Semántica documental de purga resuelta: `retention_not_elapsed` es la razón general residual de no superar 30 días; `replay_active` es la razón específica prioritaria y cada candidato recibe una sola clasificación.
- Metric names, production/test adapter boundaries and local scheduler ownership are explicit.
- Bootstrap no-op semantics now require role validation.
- Contradicción «password reset fuera de alcance» resuelta: la implementación de MSF-ID-003 incluye request/reset; las afirmaciones previas quedan HISTÓRICO/SUPERSEDED en el task board.
- Claim previo de `idempotency_concurrent_blocker_fix_note` («`responseJson` almacena la respuesta contractual completa para replay fiel») queda **superseded para `POST /auth/password-change`**: el snapshot de replay vigente es mínimo (status `204` + `body_hash`) y nunca persiste refresh token, hash de sesión, contraseña, secreto ni credenciales derivadas; el replay no rota cookie ni re-valida `current_password`.

## Historical open questions (superseded)

- Legal/contable review of the general retention/anonimization policy remains required before production; it is not a waiver for the no-PII snapshot.
- TD-MSF-ID-002-03 requiere decisión operativa AWS antes de producción si AWS participa; no existe configuración AWS ni evidencia de coordinación/reemplazo.
- No hay blocker técnico de las tres deudas para desarrollo local: el task board registra pruebas de build, 249 tests (estado **HISTÓRICO** registrado tras `bootstrap_rop_migration013`, anterior al cambio `idempotency_responsejson_rop_purge_note` 2026-08-16 que añadió 3 tests estáticos de detección de drift), Prisma y PostgreSQL para la implementación 007–013. Permanecen la revalidación de especificación (que ahora incluye el último cambio de código) y los gates de producción TD-MSF-ID-002-02/03.
- **MSF-ID-003:** el canal de entrega productivo del email de reset (outbox/SES/SMTP) es decisión operacional pendiente, no implementada en v1 (`NoopEmailAdapter`).
- **MSF-ID-003:** la migración 014 puede fallar si existen tokens activos duplicados preexistentes en `password_reset_tokens`; requiere preflight PostgreSQL antes de `prisma migrate deploy` (registrado en contrato Prisma §014 y `technical_debt.md`).
- **MSF-ID-003:** revalidación focalizada de Spec Validator pendiente sobre el conjunto MSF-ID-002 + MSF-ID-003 (incluida la migración 014); sin nuevo `verdict: ready` no hay handoff ni cierre.
- **Replay de password-change (2026-08-17):** la decisión aprobada (ADR-020) queda pendiente de revalidación focalizada de Spec Validator junto con el conjunto MSF-ID-002 + MSF-ID-003; sin nuevo `verdict: ready` no hay handoff ni cierre.

## Human Plan Approval: approved_by_user

Aprobación humana recibida para continuar con la siguiente tarea permitida. No sustituye el veredicto del Spec Validator ni autoriza modificar código, OpenAPI, Prisma, scripts o Git. La aprobación humana anterior se conserva como historial **superseded**.

## Operational AWS state (2026-08-18)

Estado operativo de infraestructura, **adicional a la spec** y sin alterar el lifecycle (`validated-not-executed`) ni el veredicto de Spec Validator (`ready` histórico, revalidación pendiente). No constituye declaración de producción lista.

- Cuenta de aprendizaje, región `us-east-1`, un único ambiente. Perfil local AWS CLI `merkee` (Agent Toolkit/MCP). No se incluyen credenciales.
- IAM OIDC GitHub `merkee-github-actions-deploy` (trust a subjects GitHub reales con IDs); task role `merkee-backend-task-role` creado.
- Secrets Manager `merkee/app` creado y referenciado por la task definition (mapeo `secrets` JSON); no se exponen valores.
- ECS task definition `merkee-backend-task` revision 2 con `taskRole` y mapeo `secrets` JSON; servicio `merkee-backend-service` **en despliegue / pendiente de verificación** (running/health check por confirmar). No se afirma despliegue terminado.
- Dockerfile multi-stage no-root de la API creado y build local validado; ECR `merkee-backend-api` existe.
- S3/CloudFront: `merkee-frontend-client`/`E32P11SX9DFU82`→`merkee.shop`, `merkee-frontend-admin`/`E119IKP00L5RU`→`admin.merkee.shop`; `aws-s3-tickets-images` excluido.
- DNS en Spaceship; `api.merkee.shop` y `admin.merkee.shop` existen; `swagger.merkee.shop` pendiente.
- RDS `merkee-db` existe; auditoría indicó `PubliclyAccessible=True` (riesgo pendiente, no afirmado como corregido).
- Deuda AWS: TD-AWS-RDS-PUBLIC, TD-AWS-SWAGGER-DNS, TD-AWS-OBSERVABILITY, TD-AWS-ECS-VALIDATION (ver `technical_debt.md`). TD-MSF-ID-002-03 permanece `active`.

Esta sección no modifica contratos, criterios de aceptación ni el veredicto de Spec Validator; es trazabilidad operativa.

## Stale terms guard

Forbidden: `response` JSONB, respuesta original/completa o snapshot de tres campos como regla/evidencia vigente (HISTÓRICO/SUPERSEDED); full `AdminUserProvisionResponse` in idempotency storage; PII/token/password/hash in `response_json`, logs or metrics; replay from stored PII; UUID fuera de v1–v8/RFC 4122 en `resource_id`; derivar `activation_expires_at` desde token usado/expirado; usar 008 para fijar la ventana operativa en lugar de la evaluación contractual 24 h/30 días; purge of `replay_active` or unknown scope; inventar scope, terminalidad o prueba de purga para TD-MSF-ID-002-01; declarar cerrada TD-MSF-ID-002-02 sin evidencia legal/contable; afirmar AWS configurado o cerrada TD-MSF-ID-002-03 sin decisión de coordinación/reemplazo, ownership, alarmas y prevención de doble ejecución; treating `retention_not_elapsed` as a separate retention window or purge behavior; in-memory metrics in production; AWS-only scheduler without local driving adapter; bootstrap no-op for non-admin role; new endpoint/OpenAPI change; `POST /auth/initial-password-change`; afirmar que el commit del grafo (`89bcd155`) es igual a `HEAD` o que el grafo está actualizado respecto a `HEAD` sin verificación con Git (blocked para frescura); **afirmar «password reset fuera de alcance» o «no implementado» como estado vigente (HISTÓRICO/SUPERSEDED — MSF-ID-003 implementado)**; declarar MSF-ID-003 `done`/cerrada sin nuevo `verdict: ready` de Spec Validator; token en claro en logs/métricas/trazas/responses del flujo de reset; aplicar la migración 014 sin preflight PostgreSQL de duplicados activos; editar migraciones aplicadas 001–013; **persistir refresh token, hash de sesión, contraseña, secreto o credenciales derivadas en `idempotency_records`, logs o métricas del flujo de password-change**; replay de password-change que re-valide `current_password`, rote cookie o repita la mutación; `500` espurio o doble rotación por carrera concurrente en password-change; afirmar que el snapshot de replay de password-change almacena la respuesta contractual completa o credenciales (HISTÓRICO/SUPERSEDED por ADR-020).

## Next action

Desbloquear MSF-CAT-001 como siguiente tarea documentalmente validada. MSF-CAT-002 permanece esperando MSF-CAT-001. Mantener las deudas activas TD-MSF-ID-002-01/02/03 y sus gates de producción; no declararlas resueltas.
