# Shared Context — msf-id-rop-sign-cleanup

**Lifecycle status:** `planning`
**Actualizado:** 2026-08-16

## Current status

Current status: `planning`. Incremento de cleanup arquitectónico para eliminar el `try/catch` técnico de la capa `application` del módulo `identity` en `register`/`login`/`refresh-session`/`logout`, alineando los adapters de salida a `Result`/`DomainError` (ADR-017 / Master Spec §ROP), patrón ya aplicado en `provision`/`bootstrap`/`activate-admin` (STAB-B1) y `jwt.verify` (STAB-B5). No introduce endpoints, migraciones, OpenAPI ni reglas de negocio nuevos.

La delta spec y este shared context están en `planning`. **No hay `verdict: ready` de Spec Validator, no hay `## Human Plan Approval: approved_by_user`, no hay task board, no hay handoff a Task Decomposer/Executor.** No se modificó código ni se ejecutó Git.

**Determinaciones del Planner (con evidencia, sujetas a confirmación):**
- `logout` pertenece a TD-NEW-ROP-SIGN (mismo `try/catch` técnico, mismo módulo, comparte `SessionRepositoryPort`/`CartReservationPort`).
- `purge` NO pertenece a TD-NEW-ROP-SIGN (job con `try/catch` operacional + retry, devuelve `void`); se registra `TD-NEW-ROP-PURGE` separada.
- Decisión unit-of-work (Opción A interfaces separadas vs Opción B wrapper) **pendiente de consulta formal al `solution-architect`** (DEC-07); bloquea `validated-not-executed`.

## Canonical artifacts

- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/msf-id-rop-sign-cleanup-delta-spec.md` (esta delta, `planning`)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` (`revision-needed`) — §ROP, §Identidad
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` (`revision-needed`) — ADR-017 (ROP estricto + catálogo `DomainError`), ADR-002/004/010/015/018
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` (canónico; **sin cambios en este incremento**)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md` (global; TD-NEW-ROP-SIGN a actualizar, TD-NEW-ROP-PURGE a registrar)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md` (`revision-needed`)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` (`closed`; patrón ROP modelo B1/B5)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` (`closed`; trazabilidad)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/.dependency-cruiser.cjs` (5 reglas `forbidden`; sin cambios)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` (commit `89bcd155`; **blocked para frescura** sin Git; 1,245 nodos / 2,603 aristas, sin ciclos)

**Código autoritativo verificado en disco (NO modificado por el Planner):**
- `projects/merkee-shop-api/src/modules/identity/application/use-cases/{register,login,refresh-session,logout,activate-admin,provision-admin-user,bootstrap-initial-admin,purge-idempotency-records}.use-case.ts`
- `projects/merkee-shop-api/src/modules/identity/domain/ports/{jwt,user-repository,session-repository,password-hasher,cookie-token,cart-reservation,clock,activate-admin-unit-of-work,provision-unit-of-work,bootstrap-unit-of-work}.port.ts`
- `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/{jwt,prisma-user-repository,prisma-session-repository,argon2-password-hasher,cookie-token,noop-cart-reservation,system-clock,prisma-activate-admin-unit-of-work,prisma-provision-unit-of-work,prisma-bootstrap-unit-of-work}.adapter.ts`
- `projects/merkee-shop-api/src/modules/identity/{identity.controller,identity.module,identity.tokens}.ts`
- `projects/merkee-shop-api/src/shared/domain/{result,domain-error}.ts`
- `projects/merkee-shop-api/src/shared/http/{result-projector,domain-error-mapper}.ts`
- `projects/merkee-shop-api/src/modules/identity/domain/identity-errors.ts`

## Artifact evidence

| Ruta | Campo/flujo verificado | Resultado | Estado |
|---|---|---|---|
| `register.use-case.ts` líneas 48-108 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica | Drift ROP: captura técnica en application (H1). | fail |
| `login.use-case.ts` líneas 55-129 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica | Drift ROP: captura técnica en application (H1). | fail |
| `refresh-session.use-case.ts` líneas 48-110 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica | Drift ROP: captura técnica en application (H1). | fail |
| `logout.use-case.ts` líneas 34-55 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica | Drift ROP: captura técnica en application (H1); mismo patrón que register/login/refresh-session. | fail |
| `jwt.port.ts` línea 23 | `sign(payload): Promise<string>` | Puerto sin `Result` (H2); `verify` ya alineado (B5, línea 25). | fail (sign) / pass (verify) |
| `user-repository.port.ts` líneas 9-25 | `findByEmail/...: Promise<User \| null>` etc. | Puerto sin `Result` (H2). | fail |
| `session-repository.port.ts` líneas 9-25 | `create/...: Promise<Session>` etc. | Puerto sin `Result` (H2). | fail |
| `password-hasher.port.ts` líneas 7-11 | `hash/verify: Promise<string/boolean>` | Puerto sin `Result` (H2). | fail |
| `cookie-token.port.ts` líneas 8-12 | `generate/hash: string` (sync) | Puerto sin `Result` (H2). | fail |
| `cart-reservation.port.ts` líneas 11-15 | `releaseActiveReservations/closeCart: Promise<void>` | Puerto sin `Result` (H2); puerto de identity (no de cart-reservation). | fail |
| `prisma-user-repository.adapter.ts` líneas 16-17 | Comentario: "Los errores técnicos de Prisma se capturan y se traducen a DomainError en los casos de uso (no aquí, por simplicidad)" | Adapter no captura/traduce (H2); comentario stale a corregir. | fail |
| `prisma-session-repository.adapter.ts` líneas 11-12 | Comentario: "Los errores técnicos se propagan como excepciones y son capturados por los casos de uso" | Adapter no captura/traduce (H2); comentario stale a corregir. | fail |
| `argon2-password-hasher.adapter.ts` líneas 8-10, 24-28 | `hash` no captura; `verify` captura y devuelve `false` | `hash` sin traducción; `verify` parcial (H2). | fail |
| `jwt.adapter.ts` líneas 40-44 | `sign` no captura (`jwt.sign` puede lanzar) | `sign` sin traducción (H2); `verify` (líneas 46-66) y fail-fast (líneas 24-37) ya alineados (B3/B5). | fail (sign) / pass (verify, fail-fast) |
| `activate-admin.use-case.ts` | Sin `try/catch` técnico; consume `Result` de `unitOfWork.run` (B1 aplicado) | Patrón ROP correcto; efecto dominó por cambio de puertos (H3). | pass (ROP) / blocked (efecto dominó) |
| `provision-admin-user.use-case.ts` | Sin `try/catch` técnico; consume `Result` de `unitOfWork.run` | Patrón ROP correcto; efecto dominó (H3). | pass (ROP) / blocked (efecto dominó) |
| `bootstrap-initial-admin.use-case.ts` | Sin `try/catch` técnico; consume `Result` de `unitOfWork.run` | Patrón ROP correcto; efecto dominó (H3). | pass (ROP) / blocked (efecto dominó) |
| `prisma-provision-unit-of-work.adapter.ts` líneas 42-88 | `run` devuelve `Result`; captura en límite; `$transaction` revierte si callback lanza | Patrón ROP modelo (B1); restricción H4 (repos dentro de callback deben lanzar para rollback). | pass |
| `prisma-activate-admin-unit-of-work.adapter.ts` líneas 35-55 | `run` devuelve `Result`; captura en límite | Patrón ROP modelo (B1); restricción H4. | pass |
| `purge-idempotency-records.use-case.ts` líneas 47-81, 84-101 | `execute(): Promise<void>` (no `Result`); `try/catch` operacional (no relanza) + retry | Drift distinto (job); NO es TD-NEW-ROP-SIGN (Determinación B). | pass (no pertenece a este incremento) |
| `purge-idempotency-records.use-case.spec.ts` líneas 343-357 | `mockRejectedValue → uc.execute() resolves undefined` + métricas error | Confirma manejo operacional de job (no drift ROP de adapter). | pass |
| `result-projector.ts` / `domain-error-mapper.ts` | Proyección `Result`→HTTP; catálogo código→status | Sin cambios en este incremento. | pass |
| `identity.controller.ts` | Inyecta `JwtPort` (B5); proyecta `Result` vía `projectResult` | Sin cambios en este incremento (controllers no capturan excepciones técnicas). | pass |
| `.dependency-cruiser.cjs` | 5 reglas `forbidden` (domain→app/infra, app→infra, app→@nestjs, domain→frameworks, no-circular) | Sin cambios; invariant ya cubierto. | pass |
| `docs/api/openapi.yaml` | Sin cambios en este incremento | Canónico intacto. | pass |
| `graphify-out/GRAPH_REPORT.md` | 1,245 nodos / 2,603 aristas, sin ciclos; commit `89bcd155` | Contenido verificado; frescura respecto a HEAD **blocked** sin Git. | pass (contenido) / blocked (frescura) |

## Spec Validator Approval

verdict: pending
reviewed_at: —
validator_agent: spec-validator
artifact_set_reviewed: —
summary: —
invalidated_by_changes_since: none

## Decisions locked

- DEC-01: `logout` pertenece a TD-NEW-ROP-SIGN (Determinación A, con evidencia).
- DEC-02: `purge` NO pertenece a TD-NEW-ROP-SIGN; se registra `TD-NEW-ROP-PURGE` separada (Determinación B, con evidencia).
- DEC-03: No se inventan códigos `DomainError` nuevos (ADR-017); adapters traducen a `TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII; `null` de `findByEmail`/`findById`/`findByRefreshTokenHash` son `ok(null)` (negocio legítimo).
- DEC-04: No se cambia OpenAPI, Prisma, migraciones, endpoints, schemas ni parámetros. No se elige `iss`/`aud`/`typ` ni proveedor JWT (STAB-DEC-12 pendiente).
- DEC-05: `ClockPort` (sync, no lanza), `JwtPort.verify` (ya B5) y `purge` (DEC-02) quedan fuera.

## Validator findings

(Bloqueado: Spec Validator no ha revisado. La spec está en `planning` con open questions pendientes.)

## Resolved findings

(Ninguno resuelto por validación aún. Hallazgos H1-H4 documentados en la delta spec; determinaciones A/B con evidencia.)

## Open questions

1. **`logout` en alcance (DEC-01):** confirmar inclusión en TD-NEW-ROP-SIGN. Recomendación: SÍ.
2. **`CartReservationPort` alineado a `Result` (DEC-06):** confirmar que NO cuenta como "carrito" fuera de alcance (es puerto de identity; adapter noop → `ok(undefined)`). Recomendación: incluir.
3. **Decisión unit-of-work (DEC-07):** Opción A (interfaces separadas) vs Opción B (wrapper). **Requiere consulta formal al `solution-architect`**. Recomendación Planner: Opción A.
4. **`purge` (DEC-02):** confirmar que queda fuera y se registra `TD-NEW-ROP-PURGE`. ¿Jobs exentos de invariant `Result` por no tener rail HTTP? Recomendación: entrada separada.
5. **`CookieTokenPort` sync `Result`:** confirmar alineación a `Result<string, DomainError>` sync. Recomendación: alinear.

## Stale terms guard

Forbidden: "try/catch técnico en application" como patrón vigente; `mockRejectedValue` para fallos técnicos en tests de caso de uso (debe ser `mockResolvedValue(fail(technicalFailure()))`); "los errores técnicos se capturan en los casos de uso" (comentarios stale en `prisma-user-repository.adapter.ts` y `prisma-session-repository.adapter.ts`); "JwtAdapter propaga excepciones de jwt.sign" como patrón vigente; afirmar que `purge` pertenece a TD-NEW-ROP-SIGN; afirmar que `CartReservationPort` es del módulo cart-reservation (es de identity); afirmar que la decisión unit-of-work está locked sin consulta al Solution Architect; afirmar que `iss`/`aud`/`typ` o proveedor JWT están fijados (STAB-DEC-12 pendiente); declarar `ready` sin Spec Validator; handoff sin `## Human Plan Approval: approved_by_user`; crear task board antes de `validated-not-executed`; modificar código/Prisma/`package.json`/runtime por el Planner; operaciones de Git; afirmar que el grafo está actualizado respecto a HEAD sin verificación (blocked para frescura); incluir HTTP security/cobertura/perfil/password/carrito-implementación/otros módulos en este incremento.

## Next action

1. **Consultar al usuario las open questions 1, 2, 4, 5** (confirmación de alcance: logout, CartReservationPort, purge, CookieTokenPort).
2. **Consultar formalmente al `solution-architect` la open question 3** (decisión unit-of-work Opción A vs B) y registrar la decisión acordada como DEC-07 locked en esta spec y shared context.
3. Tras confirmaciones, actualizar la delta spec y este shared context (a `draft`), sincronizar `technical_debt.md` (global y local) con DEC-01/DEC-02, y solicitar revisión de Spec Validator.
4. No crear task board ni hacer handoff hasta `verdict: ready` + `## Human Plan Approval: approved_by_user`.
5. Recomendar `graphify update .` + `git rev-parse HEAD` antes de la revalidación de Spec Validator (frescura del grafo).