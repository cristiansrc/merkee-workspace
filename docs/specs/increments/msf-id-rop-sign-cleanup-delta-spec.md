# Delta Spec — msf-id-rop-sign-cleanup

## Status: planning
**Lifecycle status:** `planning`
**Incremento padre:** `merkee-shop-foundation` (`revision-needed`)
**Creado:** 2026-08-16 · **Planner:** activo · **Spec Validator:** `pending`
**Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/msf-id-rop-sign-cleanup-sdd-context.md`
**Deuda técnica origen:** `TD-NEW-ROP-SIGN` (y determinación sobre `logout`/`purge`, ver §Determinación)

> Estado `planning`: artefactos en preparación. No hay `verdict: ready` de Spec Validator, no hay `## Human Plan Approval: approved_by_user`, no hay task board, no hay handoff a Task Decomposer/Executor. No se modificó código ni se ejecutó Git.

## Propósito y alcance

Incremento de cleanup arquitectónico focalizado en eliminar el `try/catch` técnico de la capa `application` del módulo `identity` en los casos de uso de autenticación, alineando los adapters de salida al patrón ROP vigente (ADR-017 / Master Spec §ROP) ya aplicado en `provision`/`bootstrap`/`activate-admin` (STAB-B1) y `jwt.verify` (STAB-B5). No introduce endpoints, módulos, migraciones, schemas OpenAPI, parámetros ni reglas de negocio nuevos. No cambia el contrato de comunicación OpenAPI ni el contrato Prisma.

**Alcance exhaustivo y exclusivo (TD-NEW-ROP-SIGN):**
- Casos de uso objetivo (eliminar `try/catch` técnico de `application`): `register`, `login`, `refresh-session`, `logout` (ver §Determinación logout).
- Puertos de salida a alinear a `Result<Success, DomainError>` para uso directo por la application: `JwtPort.sign`, `UserRepositoryPort`, `SessionRepositoryPort`, `PasswordHasherPort`, `CookieTokenPort`, `CartReservationPort` (ver §Determinación CartReservationPort).
- Adapters concretos a actualizar (capturar/traducir excepciones técnicas a `DomainError`/`Result` en su límite, sin causa/PII): `JwtAdapter.sign`, `PrismaUserRepositoryAdapter`, `PrismaSessionRepositoryAdapter`, `Argon2PasswordHasherAdapter`, `CookieTokenAdapter`, `NoopCartReservationAdapter`.
- Casos de uso afectados por efecto dominó (consumen los puertos vía unit-of-work; deben mantenerse sin `try/catch` técnico): `activate-admin`, `provision-admin-user`, `bootstrap-initial-admin`. Ver §Decisión arquitectónica pendiente (unit-of-work).
- Tests afectados: specs de los casos de uso y adapters anteriores.

**Fuera de alcance (explícito):**
- HTTP security (TD-NEW-HTTP-SEC), cobertura (TD-NEW-COV), perfil/password (MSF-ID-003), módulos no-identity (catalog, cart-reservation, payments, checkout, media, orders, admin-query).
- `purge-idempotency-records` (drift distinto; ver §Determinación purge y nueva entrada `TD-NEW-ROP-PURGE`).
- `ClockPort` (método `now()` síncrono que no lanza; no requiere traducción).
- `JwtPort.verify` (ya alineado en STAB-B5).
- `iss`/`aud`/`typ` y proveedor JWT (decisión pendiente STAB-DEC-12, no se elige aquí).
- Operaciones de Git, task board, handoff.

Precedencia aplicada: (1) solicitud explícita del usuario; (2) código implementado, migraciones, OpenAPI y runtime config del repositorio; (3) specs activas `revision-needed`/`planning`. No hay conflicto con OpenAPI ni migraciones (no se tocan).

## Hallazgos (código real verificado en disco 2026-08-16)

### H1 — `try/catch` técnico en `application` (register/login/refresh-session/logout)

| Archivo | Líneas | Hallazgo |
|---|---|---|
| `src/modules/identity/application/use-cases/register.use-case.ts` | 48-108 | `try { ...lógica completa... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica, incluidas llamadas a `userRepo.findByEmail`, `passwordHasher.hash`, `userRepo.create`, `cookieToken.generate/hash`, `clock.now`, `sessionRepo.create`, `jwt.sign`. |
| `src/modules/identity/application/use-cases/login.use-case.ts` | 55-129 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica, incluidas `userRepo.findByEmail`, `passwordHasher.verify`, `sessionRepo.findById`, `cartReservation.releaseActiveReservations/closeCart`, `sessionRepo.revoke`, `cookieToken.*`, `clock.now`, `sessionRepo.create`, `jwt.sign`. |
| `src/modules/identity/application/use-cases/refresh-session.use-case.ts` | 48-110 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica, incluidas `cookieToken.hash`, `sessionRepo.findByRefreshTokenHash`, `clock.now`, `userRepo.findById`, `cookieToken.generate/hash`, `sessionRepo.updateRefreshToken`, `jwt.sign`. |
| `src/modules/identity/application/use-cases/logout.use-case.ts` | 34-55 | `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica, incluidas `sessionRepo.findById`, `cartReservation.releaseActiveReservations`, `sessionRepo.revoke`. |

**Contradicción:** Master Spec §ROP ("cada adapter de salida captura excepciones... y las traduce a `DomainError`; no las propaga hacia controllers ni dominio"; "No se usa `throw`/`catch` como flujo de negocio") y ADR-017 ("Adapters de salida convierten excepciones técnicas en `DomainError` aplicable o `TECHNICAL_DEPENDENCY_FAILURE`"). El patrón corregido ya vive en `provision`/`bootstrap`/`activate-admin` (STAB-B1) y `jwt.verify` (STAB-B5).

### H2 — Puertos de salida sin `Result` (uso directo)

| Puerto | Método | Firma actual | ¿Devuelve `Result`? | ¿Adapter captura/traduce? |
|---|---|---|---|---|
| `JwtPort` | `sign` | `Promise<string>` | No | No (`jwt.sign` puede lanzar; `verify` ya alineado en B5) |
| `UserRepositoryPort` | `findByEmail`/`findById`/`create`/`createAdmin`/`updatePassword` | `Promise<User \| null>` / `Promise<User>` | No | No (comentario en `prisma-user-repository.adapter.ts` líneas 16-17: "Los errores técnicos de Prisma se capturan y se traducen a `DomainError` en los casos de uso (no aquí, por simplicidad)") |
| `SessionRepositoryPort` | `create`/`findById`/`findByRefreshTokenHash`/`updateRefreshToken`/`revoke`/`revokeAllForUser` | `Promise<Session>` / `Promise<Session \| null>` / `Promise<void>` | No | No (comentario en `prisma-session-repository.adapter.ts` líneas 11-12: "Los errores técnicos se propagan como excepciones y son capturados por los casos de uso") |
| `PasswordHasherPort` | `hash`/`verify` | `Promise<string>` / `Promise<boolean>` | No | No en `hash` (comentario en `argon2-password-hasher.adapter.ts` líneas 8-10); `verify` ya captura y devuelve `false` (líneas 24-28) |
| `CookieTokenPort` | `generate`/`hash` | `string` (sync) | No | No (crypto rara vez lanza, pero no está alineado al patrón) |
| `CartReservationPort` | `releaseActiveReservations`/`closeCart` | `Promise<void>` | No | No (adapter `noop` no lanza; pero el contrato no exige `Result`) |

### H3 — Efecto dominó en casos de uso ya corregidos (vía unit-of-work)

Los puertos `UserRepositoryPort`/`SessionRepositoryPort` se consumen también DENTRO de callbacks transaccionales de los unit-of-work (`ActivateAdminUnitOfWorkPort`, `ProvisionUnitOfWorkPort`, `BootstrapUnitOfWorkPort`), que ya devuelven `Result<T, DomainError>` y cuyos adapters ya capturan/traducen en su límite (B1 aplicado a activate-admin; provision y bootstrap ya lo hacían):

| Archivo | Uso de puertos dentro de callback transaccional |
|---|---|
| `activate-admin.use-case.ts` líneas 66-81 | `tx.userRepo.updatePassword`, `tx.sessionRepo.revokeAllForUser` (dentro de `unitOfWork.run`) |
| `provision-admin-user.use-case.ts` líneas 143-230 | `tx.userRepo.findById/findByEmail/createAdmin`, `tx.activationTokenRepo.*`, `tx.idempotencyRepo.*` (dentro de `unitOfWork.run`) |
| `bootstrap-initial-admin.use-case.ts` líneas 68-90 | `tx.userRepo.findByEmail/create` (dentro de `unitOfWork.run`) |

Los unit-of-work exponen `tx.userRepo: UserRepositoryPort` y `tx.sessionRepo: SessionRepositoryPort` (interfaces `ActivateAdminTransaction`/`ProvisionTransaction`/`BootstrapTransaction`). Cambiar las firmas de estos puertos afecta necesariamente a estos casos de uso y a los adapters de unit-of-work.

### H4 — Restricción técnica de rollback transaccional (crítica)

Los adapters de unit-of-work usan `this.prisma.$transaction(async (tx) => { ... return await work(transaction); })` (ver `prisma-provision-unit-of-work.adapter.ts` líneas 51-69, `prisma-activate-admin-unit-of-work.adapter.ts` líneas 39-46, `prisma-bootstrap-unit-of-work.adapter.ts`). Prisma revierte la transacción **solo si el callback lanza**. Si los repos dentro del callback devuelven `Result` (capturando excepciones de Prisma), la transacción **no se revierte** ante un fallo técnico, rompiendo la atomicidad (ADR-018 para provision; ADR-010 para bootstrap; MSF-ID-002 para activate-admin).

Por tanto, los repos DENTRO de callbacks transaccionales deben **lanzar** excepciones técnicas (para que Prisma revierta y el adapter del unit-of-work traduzca en su límite), mientras los repos para **uso directo** por `register`/`login`/`refresh-session`/`logout` deben devolver `Result` (para que la `application` no tenga `try/catch`). Esto exige una decisión arquitectónica (ver §Decisión arquitectónica pendiente).

## Determinación logout / purge (con evidencia)

### Determinación A — `logout` pertenece a TD-NEW-ROP-SIGN

**Evidencia:**
1. `logout.use-case.ts` líneas 34-55 contiene el mismo `try { ... } catch { return fail(technicalFailure()); }` técnico que register/login/refresh-session, envolviendo toda la lógica.
2. Comparte `SessionRepositoryPort` con register/login/refresh-session (puerto que se alinea en este incremento).
3. Comparte `CartReservationPort` con login (puerto que se alinea en este incremento).
4. Pertenece al módulo `identity` y al flujo de autenticación (MSF-ID-001).
5. `logout.use-case.spec.ts` líneas 156-171 prueba el mismo patrón `mockRejectedValue → TECHNICAL_DEPENDENCY_FAILURE` que `register.use-case.spec.ts` líneas 298-310.
6. Aislar `logout` dejaría el módulo `identity` parcialmente corregido (inconsistencia ROP) y requeriría tocar los mismos adapters (`SessionRepositoryPort`, `CartReservationPort`) en un incremento separado, duplicando trabajo y riesgo.

**Decisión:** `logout` se incluye en el alcance de TD-NEW-ROP-SIGN. La entrada de deuda `TD-NEW-ROP-SIGN` se actualiza para reflejar el alcance real (register/login/refresh-session/**logout**).

### Determinación B — `purge` NO pertenece a TD-NEW-ROP-SIGN (entrada separada TD-NEW-ROP-PURGE)

**Evidencia:**
1. `purge-idempotency-records.use-case.ts` `execute()` devuelve `Promise<void>` (no `Result<Success, DomainError>`), a diferencia de register/login/refresh-session/logout que devuelven `Result`.
2. Es un **job controlado** invocado por un driving adapter (`ScheduledIdempotencyPurgeAdapter`, `OnApplicationBootstrap`), no un caso de uso HTTP con un rail `Result` que proyectar a `ApiErrorResponse`.
3. El `try/catch` de `execute` (líneas 52-80) es **manejo operacional intencional**: captura, registra métricas/logs (`recordRun('error')`, `logger.error('idempotency_records.purge_failed')`) y **no relanza** (comentario línea 79: "No relanzar: el job deja el trabajo para el siguiente ciclo"). No es captura técnica de adapter para traducir a `DomainError`.
4. El `try/catch` de `processBatchWithRetries` (líneas 90-101) es **lógica de retry** (1/5/15 s, `MAX_BATCH_ATTEMPTS=3`), no captura técnica de adapter.
5. `purge-idempotency-records.use-case.spec.ts` líneas 343-357 confirma la semántica: `mockRejectedValue(new Error('db down'))` → `uc.execute()` resuelve `undefined` (no relanza) + registra métricas de error. Líneas 359-379 confirman el retry.
6. El drift de ADR-017 aquí es distinto: un job devuelve `void` (no `Result`), pero el `try/catch` no es el anti-patrón "capturar en application lo que el adapter debería traducir". Es manejo de error a nivel job.

**Decisión:** `purge` queda fuera de este incremento. Se registra la nueva entrada `TD-NEW-ROP-PURGE` en `technical_debt.md` (global y espejo local) con la evidencia anterior, para resolver en un incremento separado (o decidir que los jobs están exentos del invariant `Result` por no tener rail HTTP). Ver §Open questions.

## Contratos objetivo (a implementar por Executor tras aprobación)

### C1 — Puertos de salida (uso directo) → `Result`

Firmas objetivo (sin causa/PII en `DomainError`; `Success` es el valor de dominio o `void`):

| Puerto | Método | Firma objetivo |
|---|---|---|
| `JwtPort` | `sign(payload: JwtPayload)` | `Promise<Result<string, DomainError>>` |
| `UserRepositoryPort` | `findByEmail(email)` | `Promise<Result<User \| null, DomainError>>` |
| `UserRepositoryPort` | `findById(id)` | `Promise<Result<User \| null, DomainError>>` |
| `UserRepositoryPort` | `create(data)` | `Promise<Result<User, DomainError>>` |
| `UserRepositoryPort` | `createAdmin(data)` | `Promise<Result<User, DomainError>>` |
| `UserRepositoryPort` | `updatePassword(userId, passwordHash)` | `Promise<Result<User, DomainError>>` |
| `SessionRepositoryPort` | `create(data)` | `Promise<Result<Session, DomainError>>` |
| `SessionRepositoryPort` | `findById(id)` | `Promise<Result<Session \| null, DomainError>>` |
| `SessionRepositoryPort` | `findByRefreshTokenHash(hash)` | `Promise<Result<Session \| null, DomainError>>` |
| `SessionRepositoryPort` | `updateRefreshToken(sessionId, hash, expiresAt)` | `Promise<Result<void, DomainError>>` |
| `SessionRepositoryPort` | `revoke(sessionId)` | `Promise<Result<void, DomainError>>` |
| `SessionRepositoryPort` | `revokeAllForUser(userId)` | `Promise<Result<void, DomainError>>` |
| `PasswordHasherPort` | `hash(plain)` | `Promise<Result<string, DomainError>>` |
| `PasswordHasherPort` | `verify(plain, hash)` | `Promise<Result<boolean, DomainError>>` |
| `CookieTokenPort` | `generate()` | `Result<string, DomainError>` (sync) |
| `CookieTokenPort` | `hash(token)` | `Result<string, DomainError>` (sync) |
| `CartReservationPort` | `releaseActiveReservations(sessionId)` | `Promise<Result<void, DomainError>>` |
| `CartReservationPort` | `closeCart(sessionId)` | `Promise<Result<void, DomainError>>` |

**Catálogo de `DomainError` aplicable:** no se inventan códigos nuevos (ADR-017). Los adapters traducen excepciones técnicas a `TECHNICAL_DEPENDENCY_FAILURE` (`technicalFailure()`). Los `null` de `findByEmail`/`findById`/`findByRefreshTokenHash` son **resultado de negocio legítimo** (ausencia de recurso), no fallo técnico: se devuelven como `ok(null)` (el caso de uso decide si es `Failure` de negocio, p. ej. `invalidCredentials()`/`sessionNotFoundOrExpired()`). `PasswordHasherPort.verify` devuelve `ok(boolean)` (el caso de uso decide `invalidCredentials()` si `false`); un fallo técnico de argon2 se traduce a `fail(technicalFailure())`.

### C2 — Adapters concretos (capturar/traducir en su límite)

Cada adapter envuelve su llamada técnica en `try { ... } catch (error) { logger.warn(<tipo código, sin mensaje/PII>); return fail(technicalFailure()); }` y devuelve `ok(<valor>)` en éxito. Patrones de referencia ya verificados: `prisma-provision-unit-of-work.adapter.ts` líneas 71-85, `prisma-activate-admin-unit-of-work.adapter.ts` líneas 48-54, `jwt.adapter.ts` `verify` líneas 46-66.

| Adapter | Cambio |
|---|---|
| `JwtAdapter.sign` | Capturar excepciones de `jwt.sign`; devolver `Result`. Mantener fail-fast del secreto (STAB-B3) y `verify` (STAB-B5) intactos. |
| `PrismaUserRepositoryAdapter` | Capturar `Prisma.PrismaClientKnownRequestError`/`Prisma.PrismaClientUnknownRequestError` y otras; registrar solo `code` (sin mensaje/PII); devolver `fail(technicalFailure())`. `findByEmail`/`findById` devuelven `ok(null)` si la fila no existe. |
| `PrismaSessionRepositoryAdapter` | Igual patrón. |
| `Argon2PasswordHasherAdapter` | `hash`: capturar fallos de argon2; devolver `Result`. `verify`: actualmente captura y devuelve `false` (líneas 24-28); alinear a `Result<boolean, DomainError>`: fallo técnico → `fail(technicalFailure())`, hash no verificable → `ok(false)` (no revela existencia). |
| `CookieTokenAdapter` | `generate`/`hash` sync: envolver `randomBytes`/`createHash` en try/catch; en la práctica casi nunca lanzan, pero alinear al patrón; devolver `Result` sync. |
| `NoopCartReservationAdapter` | `releaseActiveReservations`/`closeCart` devuelven `ok(undefined)` (noop; no lanza). |

### C3 — Casos de uso objetivo (eliminar `try/catch` técnico)

`register`, `login`, `refresh-session`, `logout`: eliminar el `try { ... } catch { return fail(technicalFailure()); }` envolvente. La application consume `Result` de cada puerto: ante `isFailure(repoResult)`, propaga el `Failure` (`return repoResult` o `return fail(repoResult.error)`); ante `ok(value)`, aplica la regla de negocio. No hay `throw`/`catch` técnico en `application`. Las reglas de negocio (`emailAlreadyRegistered`, `invalidCredentials`, `sessionNotFoundOrExpired`) se devuelven por el rail `Failure` como hoy.

### C4 — Casos de uso afectados por efecto dominó (mantener sin `try/catch` técnico)

`activate-admin`, `provision-admin-user`, `bootstrap-initial-admin`: ya están sin `try/catch` técnico. Deben actualizarse para consumir las nuevas firmas `Result` de los puertos DENTRO de los callbacks transaccionales, según la decisión arquitectónica de §Decisión arquitectónica pendiente. En ningún caso se reintroduce `try/catch` técnico en `application`.

## Decisión arquitectónica pendiente (unit-of-work) — REQUIERE CONSULTA A SOLUTION ARCHITECT

**Problema (H4):** los repos dentro de callbacks transaccionales deben lanzar para que Prisma revierta; los repos de uso directo deben devolver `Result`. No se puede usar la misma interfaz con un solo comportamiento sin romper la atomicidad transaccional o el invariant "application sin try/catch".

**Opciones documentadas:**

| Opción | Descripción | Pros | Contras |
|---|---|---|---|
| **A (interfaces separadas)** | `UserRepositoryPort` (devuelve `Result`, uso directo) + `TransactionalUserRepository` (devuelve valores directos, lanza, uso transaccional). Los unit-of-work exponen `TransactionalUserRepository` en sus `*Transaction`. Dos adapters o un adapter que implementa ambas interfaces. | Separa contextos directo vs transaccional; respeta rollback y "application sin try/catch"; patrón Adapter justificable (dos variantes de uso confirmadas). | Duplica interfaces de dominio; más boilerplate de tipos. |
| **B (adapter wrapper Result)** | `PrismaUserRepositoryAdapter` lanza (como hoy). Un wrapper `ResultUserRepositoryAdapter` (infrastructure) envuelve al prisma adapter y devuelve `Result`. Register/login/refresh-session/logout inyectan el wrapper; los unit-of-work inyectan el prisma adapter directo. `UserRepositoryPort` (interfaz del dominio) devuelve `Result`; el wrapper la implementa; el prisma adapter implementa una interfaz transaccional que lanza. | Reutiliza el adapter Prisma existente; añade solo el wrapper; rollback intacto. | Requiere igualmente una interfaz transaccional separada para el prisma adapter (o el wrapper invierte la traducción); dos capas de adapter. |
| **C (repos devuelven Result; callback señaliza fallo y unit-of-work revierte explícito)** | No viable: Prisma `$transaction` interactivo no expone rollback explícito; se logra lanzando. Reintroduciría `throw` como flujo. | — | Viola ADR-017 y rompe atomicidad. **Rechazada.** |

**Recomendación Planner:** Opción A (interfaces separadas) por claridad arquitectónica y respeto estricto de ADR-017 + rollback transaccional. Aplicar `design-patterns-standard` (patrón Adapter, dos variantes de uso confirmadas: directo vs transaccional). **Decisión final: pendiente de consulta formal al `solution-architect`** (regla: "Al diseñar una Delta Spec que introduzca flujos complejos, el planner DEBE consultar formalmente al solution-architect"). La spec no puede pasar a `validated-not-executed` sin esta decisión locked.

## Determinación CartReservationPort (sub-decisión)

`CartReservationPort` vive en `identity/domain/ports/` (es un puerto de salida de identity que abstrae operaciones de carrito; el adapter concreto `NoopCartReservationAdapter` vive en identity). `login` y `logout` lo usan. Para eliminar el `try/catch` de `login`/`logout`, `CartReservationPort.releaseActiveReservations`/`closeCart` deben devolver `Result` (o garantizar no-lanzamiento). El adapter noop no lanza, pero el contrato debe exigir `Result` para que el adapter real (MSF-CART-001 futuro) cumpla el invariant.

**Recomendación:** alinear `CartReservationPort` a `Result` en este incremento. **No implementa carrito** (MSF-CART-001 sigue pendiente); solo alinea el contrato del puerto de salida de identity. El adapter noop se actualiza trivialmente a `ok(undefined)`. **Confirmar con el usuario** (instrucción "no incluyas carrito" puede interpretarse restrictivamente). Ver §Open questions.

## Archivos afectados (autoritativos, verificados en disco; NO modificados por el Planner)

**Puertos (domain):**
- `src/modules/identity/domain/ports/jwt.port.ts` (solo `sign`; `verify` ya alineado)
- `src/modules/identity/domain/ports/user-repository.port.ts`
- `src/modules/identity/domain/ports/session-repository.port.ts`
- `src/modules/identity/domain/ports/password-hasher.port.ts`
- `src/modules/identity/domain/ports/cookie-token.port.ts`
- `src/modules/identity/domain/ports/cart-reservation.port.ts`
- (Opción A) nuevos puertos transaccionales `TransactionalUserRepository`/`TransactionalSessionRepository` (o equivalentes) — decisión pendiente.

**Adapters (infrastructure):**
- `src/modules/identity/infrastructure/adapters/jwt.adapter.ts` (solo `sign`; `verify` y fail-fast intactos)
- `src/modules/identity/infrastructure/adapters/prisma-user-repository.adapter.ts`
- `src/modules/identity/infrastructure/adapters/prisma-session-repository.adapter.ts`
- `src/modules/identity/infrastructure/adapters/argon2-password-hasher.adapter.ts`
- `src/modules/identity/infrastructure/adapters/cookie-token.adapter.ts`
- `src/modules/identity/infrastructure/adapters/noop-cart-reservation.adapter.ts`
- (Opción A) adapters transaccionales o `PrismaUserRepositoryAdapter` implementando ambas interfaces.
- (Opción B) wrapper `ResultUserRepositoryAdapter`/`ResultSessionRepositoryAdapter`.

**Casos de uso (application):**
- `src/modules/identity/application/use-cases/register.use-case.ts`
- `src/modules/identity/application/use-cases/login.use-case.ts`
- `src/modules/identity/application/use-cases/refresh-session.use-case.ts`
- `src/modules/identity/application/use-cases/logout.use-case.ts`
- `src/modules/identity/application/use-cases/activate-admin.use-case.ts` (efecto dominó)
- `src/modules/identity/application/use-cases/provision-admin-user.use-case.ts` (efecto dominó)
- `src/modules/identity/application/use-cases/bootstrap-initial-admin.use-case.ts` (efecto dominó)

**Unit-of-work ports y adapters (efecto dominó, según decisión arquitectónica):**
- `src/modules/identity/domain/ports/activate-admin-unit-of-work.port.ts` (`ActivateAdminTransaction`)
- `src/modules/identity/domain/ports/provision-unit-of-work.port.ts` (`ProvisionTransaction`)
- `src/modules/identity/domain/ports/bootstrap-unit-of-work.port.ts` (`BootstrapTransaction`)
- `src/modules/identity/infrastructure/adapters/prisma-activate-admin-unit-of-work.adapter.ts`
- `src/modules/identity/infrastructure/adapters/prisma-provision-unit-of-work.adapter.ts`
- `src/modules/identity/infrastructure/adapters/prisma-bootstrap-unit-of-work.adapter.ts`

**DI:**
- `src/modules/identity/identity.module.ts` (factories de los use cases y adapters; posibles nuevos tokens/providers según Opción A/B).

**Tests:**
- `src/modules/identity/application/use-cases/register.use-case.spec.ts`
- `src/modules/identity/application/use-cases/login.use-case.spec.ts`
- `src/modules/identity/application/use-cases/refresh-session.use-case.spec.ts`
- `src/modules/identity/application/use-cases/logout.use-case.spec.ts`
- `src/modules/identity/application/use-cases/activate-admin.use-case.spec.ts`
- `src/modules/identity/application/use-cases/provision-admin-user.use-case.spec.ts`
- `src/modules/identity/application/use-cases/bootstrap-initial-admin.use-case.spec.ts`
- `src/modules/identity/infrastructure/adapters/jwt.adapter.spec.ts` (o equivalente; `sign`)
- `src/modules/identity/infrastructure/adapters/prisma-user-repository.adapter.spec.ts` (crear si no existe; traducción de excepciones Prisma)
- `src/modules/identity/infrastructure/adapters/prisma-session-repository.adapter.spec.ts` (crear si no existe)
- `src/modules/identity/infrastructure/adapters/argon2-password-hasher.adapter.spec.ts`
- `src/modules/identity/infrastructure/adapters/cookie-token.adapter.spec.ts` (crear si no existe)
- `src/modules/identity/infrastructure/adapters/noop-cart-reservation.adapter.spec.ts` (crear si no existe)
- `src/modules/identity/infrastructure/adapters/identity-module-wiring.spec.ts`
- `src/modules/identity/identity.controller.spec.ts`

**No afectados:** `docs/api/openapi.yaml`, `prisma/schema.prisma`, migraciones, `src/contract/*`, `src/shared/*` (result-projector y domain-error-mapper no cambian), otros módulos.

## Tests (estrategia)

### T1 — Tests de adapter (traducción de excepción técnica → `DomainError`)
Por cada adapter de salida: un test que fuerce una excepción técnica (mock de Prisma/argon2/jwt/crypto que lanza) y verifique que el adapter devuelve `fail(technicalFailure())` **sin causa/PII** en el `DomainError` (solo `code=TECHNICAL_DEPENDENCY_FAILURE`, `kind=technical`, `messageKey=technical.dependency_failure`). Un test de éxito que devuelva `ok(<valor>)`. Para `findByEmail`/`findById`/`findByRefreshTokenHash`: test de `ok(null)` cuando la fila no existe (negocio legítimo, no fallo).

### T2 — Tests de caso de uso (sin `try/catch` técnico, consume `Result`)
- `register`/`login`/`refresh-session`/`logout`: test de happy path (`ok`), test de cada `Failure` de negocio (`EMAIL_ALREADY_REGISTERED`, `INVALID_CREDENTIALS`, `AUTHENTICATION_REQUIRED`/`sessionNotFoundOrExpired`), y test de fallo técnico donde el stub del puerto devuelve `fail(technicalFailure())` (no `mockRejectedValue`) → el caso de uso propaga `fail(technicalFailure())`. **Cambio clave:** los stubs dejan de usar `mockRejectedValue` para fallos técnicos y pasan a `mockResolvedValue(fail(technicalFailure()))`.
- `activate-admin`/`provision-admin-user`/`bootstrap-initial-admin`: tests existentes adaptados a las nuevas firmas `Result` de los repos dentro de callbacks (según decisión arquitectónica). Mantienen cobertura de outcomes (`actorNotAuthorized`, `initialPasswordChangeRequired`, `emailTaken`, `conflict`, `resourceMissing`, `replay`, `created`, `noop`, `roleMismatch`).

### T3 — Tests de arquitectura (dependency-cruiser)
`npm run depcruise` sin violaciones. Las reglas existentes (`no-domain-to-application-or-infrastructure`, `no-application-to-infrastructure`, `no-application-framework-imports`, `no-domain-framework-imports`, `no-circular`) ya cubren el invariant. No se requieren reglas nuevas en este incremento (ver §Dependency-cruiser).

### T4 — Tests de wiring (DI)
`identity-module-wiring.spec.ts` y `identity.controller.spec.ts` actualizados para verificar que los use cases y controllers se construyen con los nuevos adapters/puertos.

### Cobertura
No se introduce `coverageThreshold` en este incremento (TD-NEW-COV está fuera de alcance). Se mantiene la cobertura existente; los tests nuevos/actualizados deben mantener `npm test` PASS.

## Dependency-cruiser

Configuración actual: `projects/merkee-shop-api/.dependency-cruiser.cjs` (5 reglas `forbidden`). Las reglas existentes ya imponen:
- `domain` no depende de `application`/`infrastructure`.
- `application` no depende de `infrastructure`.
- `application` no importa `@nestjs`.
- `domain` no importa `@nestjs`/`prisma`/`@prisma`/`express`/`axios`/`aws-sdk`.
- No circular.

**Este incremento no añade reglas nuevas.** El invariant ROP "application sin `try/catch` técnico" no es verificable por dependency-cruiser (es un patrón de código, no de importaciones). Se verifica por inspección (test estático manual / lint personalizado futuro, fuera de alcance). `npm run depcruise` debe seguir PASS.

**Verificación estática adicional (evidencia de cierre, no regla depcruise):** grep de `try {` / `catch` en `src/modules/identity/application/use-cases/{register,login,refresh-session,logout}.use-case.ts` debe retornar 0 ocurrencias (los casos de uso ya corregidos `activate-admin`/`provision`/`bootstrap` no tienen `try/catch` técnico; este incremento extiende ese estado a los cuatro objetivo).

## Riesgos

| ID | Riesgo | Mitigación |
|---|---|---|
| R1 | Efecto dominó en `activate-admin`/`provision`/`bootstrap` al cambiar firmas de puertos compartidos. | Decisión arquitectónica unit-of-work locked antes de implementar (Opción A/B); tests existentes de esos casos de uso deben seguir PASS; sin `try/catch` técnico reintroducido. |
| R2 | Romper atomicidad transaccional si los repos dentro de callbacks devuelven `Result` (no revierten). | Opción A/B preserva lanzamiento dentro de callbacks; el adapter del unit-of-work sigue capturando en su límite. Prohibida la Opción C. |
| R3 | Duplicación de interfaces (Opción A) o capas de adapter (Opción B) añade complejidad. | Consulta al Solution Architect para elegir la opción de menor coste manteniendo invariantes. |
| R4 | `CookieTokenPort` sync → `Result` sync añade boilerplate en use cases (`generate`/`hash` consumidos en register/login/refresh-session). | Aceptado por consistencia ADR-017 (TD-NEW-ROP-SIGN lista `CookieTokenPort`). El adapter traduce; en práctica crypto casi nunca lanza. |
| R5 | `CartReservationPort` alineado a `Result` puede interpretarse como "tocar carrito" (fuera de alcance). | Confirmar con el usuario (§Open questions). Es un puerto de identity, no implementación de carrito. |
| R6 | Drift con `purge` (devuelve `void`, no `Result`) queda sin resolver. | Registrado como `TD-NEW-ROP-PURGE` (entrada separada); no se resuelve aquí. |
| R7 | Frescura de Graphify respecto a HEAD no verificable sin Git. | Graphify registrado como `blocked para frescura` (commit `89bcd155`); recomendar `graphify update .` + `git rev-parse HEAD` antes de revalidación. |

## Decisiones locked

- **DEC-01:** `logout` pertenece a TD-NEW-ROP-SIGN (Determinación A, con evidencia). La entrada de deuda se actualiza para reflejar el alcance real.
- **DEC-02:** `purge` NO pertenece a TD-NEW-ROP-SIGN; se registra `TD-NEW-ROP-PURGE` como entrada separada (Determinación B, con evidencia).
- **DEC-03:** No se inventan códigos nuevos del catálogo `DomainError` (ADR-017); los adapters traducen fallos técnicos a `TECHNICAL_DEPENDENCY_FAILURE` (`technicalFailure()`) sin causa/PII. Los `null` de `findByEmail`/`findById`/`findByRefreshTokenHash` son `ok(null)` (negocio legítimo).
- **DEC-04:** No se cambia OpenAPI, contrato Prisma, migraciones, endpoints, schemas ni parámetros. No se elige `iss`/`aud`/`typ` ni proveedor JWT (STAB-DEC-12 pendiente).
- **DEC-05:** `ClockPort` queda fuera (sync, no lanza). `JwtPort.verify` queda fuera (ya alineado B5). `purge` queda fuera (DEC-02).
- **DEC-06 (pendiente de confirmación usuario):** alinear `CartReservationPort` a `Result` (puerto de identity, no implementación de carrito).
- **DEC-07 (pendiente de consulta Solution Architect):** decisión unit-of-work (Opción A vs B). No se locka sin consulta formal.

## Open questions

1. **`logout` en alcance (DEC-01):** confirmar que `logout` se incluye en TD-NEW-ROP-SIGN (mismo drift, mismo módulo, comparte `SessionRepositoryPort`/`CartReservationPort`). Recomendación: SÍ.
2. **`CartReservationPort` alineado a `Result` (DEC-06):** confirmar que alinear el puerto de identity `CartReservationPort` a `Result` (adapter noop → `ok(undefined)`) NO cuenta como "carrito" fuera de alcance. Recomendación: incluir (no implementa carrito; solo alinea contrato). Alternativa: dejar `Promise<void>` y documentar que el adapter noop garantiza no-lanzamiento (rompe invariant cuando MSF-CART-001 implemente adapter real).
3. **Decisión unit-of-work (DEC-07):** Opción A (interfaces separadas) vs Opción B (wrapper). Requiere consulta formal al `solution-architect`. Recomendación Planner: Opción A.
4. **`purge` (DEC-02):** confirmar que `purge` queda fuera de este incremento y se registra `TD-NEW-ROP-PURGE` separada. ¿Debe `purge` alinearse a `Result` en un incremento futuro, o los jobs están exentos del invariant `Result` por no tener rail HTTP? Recomendación: entrada separada; resolver después.
5. **`CookieTokenPort` sync `Result`:** confirmar que `generate`/`hash` devuelven `Result<string, DomainError>` sync (TD-NEW-ROP-SIGN los lista). Alternativa: dejar `string` y garantizar no-lanzamiento (excepción al invariant). Recomendación: alinear a `Result`.

## Criterios de aceptación

| ID | Criterio verificable |
|---|---|
| AC-01 | `register.use-case.ts`, `login.use-case.ts`, `refresh-session.use-case.ts`, `logout.use-case.ts` no contienen `try {`/`catch` técnico (grep retorna 0 ocurrencias); la `application` consume `Result` de cada puerto y propaga `Failure` sin `throw`/`catch`. |
| AC-02 | `JwtPort.sign`, `UserRepositoryPort`, `SessionRepositoryPort`, `PasswordHasherPort`, `CookieTokenPort` (y `CartReservationPort` si DEC-06 confirmada) devuelven `Result<Success, DomainError>` para uso directo; sus adapters concretos capturan excepciones técnicas en su límite, registran solo el tipo/código (sin causa/PII) y devuelven `fail(technicalFailure())`. |
| AC-03 | `findByEmail`/`findById`/`findByRefreshTokenHash` devuelven `ok(null)` cuando la fila no existe (negocio legítimo); los casos de uso mapean `null` a `Failure` de negocio (`emailAlreadyRegistered`/`invalidCredentials`/`sessionNotFoundOrExpired`) según corresponda. |
| AC-04 | `activate-admin`, `provision-admin-user`, `bootstrap-initial-admin` siguen sin `try/catch` técnico en `application`; consumen las nuevas firmas dentro de callbacks transaccionales según la decisión unit-of-work locked (DEC-07); la atomicidad transaccional (rollback de Prisma) se preserva. |
| AC-05 | Tests de adapter (T1) demuestran traducción de excepción técnica → `fail(technicalFailure())` sin causa/PII, y `ok(<valor>)`/`ok(null)` en éxito/ausencia. |
| AC-06 | Tests de caso de uso (T2) demuestran happy path, cada `Failure` de negocio y propagación de `fail(technicalFailure())` (stubs usan `mockResolvedValue(fail(...))`, no `mockRejectedValue`). |
| AC-07 | `npm run build` OK; `npm test` PASS con suites actualizadas; `npm run depcruise` sin violaciones. |
| AC-08 | No se cambia `docs/api/openapi.yaml`, `prisma/schema.prisma`, migraciones, `src/contract/*`, ni `src/shared/*`. No se inventan códigos `DomainError` nuevos. |
| AC-09 | `TD-NEW-ROP-SIGN` se actualiza en `technical_debt.md` (global y espejo local) para reflejar el alcance real (register/login/refresh-session/logout + puertos alineados); `TD-NEW-ROP-PURGE` se registra como entrada separada activa. |
| AC-10 | La decisión unit-of-work (DEC-07) está locked en la spec y el shared context tras consulta al `solution-architect`, antes de cualquier handoff. |

## Decomposition Contract

**Fuentes autoritativas:** esta delta spec, Master Spec §ROP, ADR-017, shared context, código existente. **Rutas canónicas (sin cambio):** `POST /auth/register`, `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout` (paths OpenAPI sin prefijo `/v1`; el prefijo pertenece al `servers.url`). **Sin endpoints nuevos.** **Sin migraciones.** **Sin cambios OpenAPI.**

**DTOs/schema names (sin cambio):** `RegisterRequest`, `LoginRequest`, `SessionDto`, `RegisterResult`, `LoginResult`, `RefreshSessionResult`, `LogoutCommand` (void). **Tipos transversales:** `Result<Success, DomainError>`, catálogo §ROP (sin códigos nuevos).

**Tablas canónicas (sin cambio):** `users`, `sessions`. Columnas afectadas: ninguna (solo firmas de puertos/adapters en código).

**Allowed task order (orientativo, Task Decomposer decide tras `validated-not-executed`):**
1. Decisión unit-of-work locked (DEC-07) — precondición.
2. Puertos de salida (domain) → `Result` (incluye interfaces transaccionales si Opción A).
3. Adapters concretos (infrastructure) → capturar/traducir.
4. Casos de uso objetivo (register/login/refresh-session/logout) → eliminar `try/catch`.
5. Casos de uso efecto dominó (activate-admin/provision/bootstrap) → consumir nuevas firmas.
6. DI (`identity.module.ts`) → actualizar factories/providers/tokens.
7. Tests (T1/T2/T4) → actualizar y añadir.
8. Verificación: `npm run build`, `npm test`, `npm run depcruise`, grep sin `try/catch` técnico en los cuatro use cases objetivo.
9. Actualizar `technical_debt.md` (global y local): cerrar `TD-NEW-ROP-SIGN` con evidencia; `TD-NEW-ROP-PURGE` permanece activa.

**Forbidden stale terms:** "try/catch técnico en application" como patrón vigente; `mockRejectedValue` para fallos técnicos en tests de caso de uso (debe ser `mockResolvedValue(fail(technicalFailure()))`); "los errores técnicos se capturan en los casos de uso" (comentario stale en `prisma-user-repository.adapter.ts` líneas 16-17 y `prisma-session-repository.adapter.ts` líneas 11-12); "JwtAdapter propaga excepciones de jwt.sign" como patrón vigente; afirmar que `purge` pertenece a TD-NEW-ROP-SIGN; afirmar que `CartReservationPort` es del módulo cart-reservation (es de identity); afirmar que la decisión unit-of-work está locked sin consulta al Solution Architect.

**Archivos autoritativos:** listados en §Archivos afectados.

## Gates

1. **Gate de decisión arquitectónica (precondición para `validated-not-executed`):** DEC-07 (unit-of-work Opción A/B) debe estar locked tras consulta formal al `solution-architect`. Sin esta decisión, la spec no puede pasar a `validated-not-executed` (no es descomponible sin decisiones de arquitectura).
2. **Gate de confirmación humana (open questions 1-5):** el usuario confirma logout (DEC-01), CartReservationPort (DEC-06), purge (DEC-02), CookieTokenPort (open question 5). El Planner no enruta sin `## Human Plan Approval: approved_by_user`.
3. **Gate de Spec Validator:** veredicto `ready` sobre el conjunto canónico (esta delta, Master Spec §ROP, ADR-017, shared context, código existente). Hasta `ready`, no hay handoff.
4. **Gate de ejecución (post-aprobación):** Executor implementa tras `ready` + aprobación humana. Evidencia: AC-01 a AC-10.
5. **Gates de producción existentes:** TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen; no se cierran aquí. TD-NEW-HTTP-SEC y TD-NEW-COV permanecen activas (fuera de alcance).

## Dependencias

- Autónomo dentro del módulo `identity`. No depende de MSF-ID-003 ni de módulos no-identity.
- Depende de la decisión unit-of-work (DEC-07) para completar el contrato.
- `TD-NEW-ROP-PURGE` (nueva) es independiente y se resuelve en incremento separado.
- No depende de TD-NEW-HTTP-SEC ni TD-NEW-COV (fuera de alcance).