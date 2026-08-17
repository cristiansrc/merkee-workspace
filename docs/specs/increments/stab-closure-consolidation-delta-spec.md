# Delta Spec — stab-closure-consolidation

## Status: closed

**Lifecycle status:** `closed`
**Incremento padre:** `merkee-shop-foundation` (`revision-needed`)
**Predecesor directo cerrado:** `merkee-shop-foundation-stabilization` (`closed` por SCC-A1, 2026-08-16)
**Creado:** 2026-08-16 · **Cerrado:** 2026-08-16 · **Planner:** activo · **Spec Validator:** `ready` (trazabilidad histórica conservada)
**Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/stab-closure-consolidation-sdd-context.md`

> **Cierre del incremento (2026-08-16):** las siete acciones documentales SCC-A1–A7 están implementadas y verificadas en disco mediante el task board `docs/specs/tasks/stab-closure-consolidation-task-board.md` (estado superior `done`; las siete tareas `done` con `verification_result: PASS`). Evidencia verificada en disco 2026-08-16: (SCC-A1) delta spec y shared context de estabilización `closed` (líneas 3-4 y 3/7 respectivamente), aprobaciones históricas conservadas; (SCC-A2) Master Spec §ROP (línea 44) consolida B1 (activate-admin ROP) y B5 (jwt.verify ROP) como regla durable con TD-NEW-ROP-SIGN referenciada como deuda activa, y §Identidad (línea 83) consolida B3 (fail-fast del secreto JWT en producción) con STAB-DEC-12 (`iss`/`aud`/`typ`/proveedor) explícitamente pendiente; `architecture-decisions.md` no reeditado; (SCC-A3) los tres artefactos de estabilización permanecen in-place, sin `docs/specs/archive/` (glob sin resultados); (SCC-A4) TD-NEW-ROP-SIGN/HTTP-SEC/COV registradas `active` en ambos registros de deuda (global y espejo local, líneas 21-23), contenido idéntico, ninguna `resolved`; (SCC-A5) TD-NEW-STAB-LIFECYCLE registrada `resolved-by-verified-implementation` en "Cambios completados" de ambos registros (línea 37); (SCC-A6) `workspace_changes.md` (líneas 7-15) registra cierre/consolidación, deudas TD-NEW-* y gates abiertos, sin rutas/endpoints/schemas/dependencias cross-service nuevos; (SCC-A7) TD-MSF-ID-002-02/03 `active` como gates de producción en ambos registros (líneas 12-13) y `workspace_changes.md`. Las deudas TD-NEW-ROP-SIGN/HTTP-SEC/COV permanecen `active` (no resueltas); los gates legal/AWS (TD-MSF-ID-002-02/03) permanecen abiertos; la decisión JWT (STAB-DEC-12) sigue pendiente. El `## Spec Validator Approval` (`verdict: ready`) y la `## Human Plan Approval: approved_by_user` se conservan como trazabilidad histórica en el shared context. No se modificó código, OpenAPI, Prisma, `package.json`, runtime config ni se ejecutó Git.

## Propósito y alcance

Incremento **exclusivamente documental** de cierre y consolidación del incremento `merkee-shop-foundation-stabilization`. Su objetivo es: (1) alinear el lifecycle del incremento de estabilización y sus artefactos con `implemented`/`closed` usando la evidencia de implementación verificada en disco (B1-B5); (2) consolidar las decisiones durables B1-B5 en la Master Spec sin duplicar el historial de estabilización; (3) marcar/archivar como `superseded`/`closed` los artefactos temporales obsoletos cuando corresponda; (4) registrar formalmente cuatro nuevas deudas técnicas en los registros global y local; (5) actualizar `workspace_changes.md`; y (6) mantener abiertos los gates de producción legal/AWS.

**No introduce ni modifica código, OpenAPI, Prisma (schema ni migraciones), `package.json`, runtime config, tests, scripts, UI, API handlers, queries ni automatización.** Toda acción es de edición de artefactos documentales SDD (Master Spec, ADRs, shared context, delta spec, task board, `technical_debt.md`, `workspace_changes.md`). El Planner es el owner de estas ediciones documentales; tras `verdict: ready` de Spec Validator + aprobación humana formal, las acciones se ejecutan como tareas documentales (Task Decomposer descompone; el Planner o un agente de documentación ejecuta).

Precedencia aplicada: (1) solicitud explícita del usuario; (2) código implementado, migraciones, OpenAPI y runtime config del repositorio (evidencia en disco de B1-B5 verificada 2026-08-16); (3) specs activas `revision-needed`/`planning`. Donde la documentación y el código entran en conflicto, este incremento alinea la documentación al estado verificado del código sin tocar el código.

## Principio rector: exclusivamente documental, sin bypass de gates

- **No modificar código/OpenAPI/Prisma:** este incremento no edita `projects/merkee-shop-api/src/**`, `docs/api/openapi.yaml`, `projects/merkee-shop-api/prisma/**`, `package.json`, `.env*`, ni runtime config. Las cuatro nuevas deudas (TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC, TD-NEW-COV, TD-NEW-STAB-LIFECYCLE) se **registran**; solo TD-NEW-STAB-LIFECYCLE se declara resuelta con evidencia (la alineación del lifecycle); las otras tres permanecen `active` y se resuelven en incrementos futuros.
- **No cerrar gates de producción:** TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen `active` como gates antes de producción; este incremento no los toca ni los declara resueltos.
- **No ejecutar Git:** no se ejecutan commits, branches, PRs ni operaciones de Git. La frescura de Graphify respecto a `HEAD` permanece **blocked para frescura**.
- **No crear task board todavía:** el usuario instruyó no crear task board en esta fase; se crea solo tras `verdict: ready` + aprobación humana (`## Human Plan Approval: approved_by_user`), y su creación es responsabilidad de Task Decomposer para las acciones SCC-A1–A7 (SCC-DEC-08).
- **Gate de validación satisfecho:** Spec Validator emitió `verdict: ready` sobre el conjunto canónico + esta delta; permanece requerido `## Human Plan Approval: approved_by_user` antes de crear el board o ejecutar acciones.

## Hallazgos y acciones documentales

### (A) Alineación de lifecycle del incremento de estabilización

| ID | Hallazgo | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A1 | El incremento `merkee-shop-foundation-stabilization` está **implementado y verificado en disco** (B1-B5 verificados 2026-08-16; task board `done`; TD-MSF-STAB-001/002/003/004 `resolved-by-verified-implementation`), pero sus encabezados de lifecycle permanecen en `awaiting-human-plan-approval` (delta spec línea 3-4; shared context línea 3), inconsistentes con `Current status: implemented` (shared context línea 7). Esta inconsistencia es la deuda TD-NEW-STAB-LIFECYCLE. | `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` (encabezado `## Status` y `**Lifecycle status:**`); `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` (encabezado `**Lifecycle status:**`) | Alinear el lifecycle a `closed` en ambos artefactos, conservando como trazabilidad histórica el `verdict: ready` (2026-08-16), `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` del incremento de estabilización. El estado final distingue: `implemented` (B1-B5 implementados y verificados) → `closed` (incremento cerrado y consolidado por este incremento). Resuelve TD-NEW-STAB-LIFECYCLE con evidencia. |

### (B) Consolidación de decisiones durables B1-B5 en Master Spec

| ID | Hallazgo | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A2 | Las decisiones durables de B1-B5 están parcialmente en la Master Spec (AC-05 ampliado por STAB-A5 con `must_change_password: true` en replay; §ROP aclarado por STAB-DEC-10/11 con 404 de provision y `DUPLICATE_WEBHOOK_EVENT` como clasificación interna; §NC-08 aclarado por STAB-A6 con el scheduler como driving adapter local). **Falta consolidar de forma durable y concisa:** (B1) confirmación de que `activate-admin` sigue el patrón ROP (puerto `Result`, adapter traduce, application sin `try/catch` técnico); (B3) JWT fail-fast del secreto en producción (`JWT_SECRET` ausente o < 32 bytes); (B5) confirmación de que `jwt.verify` sigue el patrón ROP (puerto `Result`, adapter traduce excepciones a `DomainError` del catálogo existente, controller consume `Result` del puerto sin `try/catch`). La consolidación **no duplica** el historial de estabilización (notas STAB-A*, STAB-B*, STAB-DRIFT-* son historial del incremento, no reglas durables). | `docs/specs/master_spec.md` (§ROP, §Identidad, §Decomposition Contract) | Consolidar de forma durable y concisa: (1) en §ROP o §Identidad, una mención de que `activate-admin` y `jwt.verify` aplican el patrón ROP (puerto `Result`, adapter traduce, application/controller sin `try/catch` técnico) — sin repetir el historial STAB-B1/B5; (2) en §Identidad, una mención durable del fail-fast del secreto JWT en producción (`JWT_SECRET` ausente o < 32 bytes detiene arranque; default solo en desarrollo con advertencia; `.env.example` documenta) — **sin fijar** `iss`/`aud`/`typ` ni el proveedor JWT (siguen como decisión pendiente STAB-DEC-12); (3) verificar que AC-05, §ROP (404 de provision, `DUPLICATE_WEBHOOK_EVENT` clasificación interna) y §NC-08 (scheduler driving adapter) ya reflejan las decisiones durables y no requieren reedición. No añadir secciones nuevas de historial; integrar en secciones existentes. |

### (C) Marcado/archivado de artefactos temporales obsoletos

| ID | Hallazgo | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A3 | El delta spec y el shared context del incremento de estabilización son artefactos temporales del incremento que, tras cierre y consolidación, deben distinguirse como `closed`/históricos (no `superseded`, porque el incremento sí se completó e implementó; no fue reemplazado por una decisión distinta). El workspace no tiene carpeta `docs/specs/archive/`; el usuario no instruyó mover archivos. | `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md`; `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md`; `docs/specs/tasks/merkee-shop-foundation-stabilization-task-board.md` | (1) Delta spec de estabilización → lifecycle `closed` (SCC-A1); se mantiene en `docs/specs/increments/` como referencia inmutable (no se mueve a archive). (2) Shared context de estabilización → lifecycle `closed`/histórico (SCC-A1); se mantiene en `docs/specs/.working/` como contexto histórico del incremento cerrado; el contexto **activo** pasa a ser `stab-closure-consolidation-sdd-context.md`. (3) Task board de estabilización → permanece `done` (estado superior ya `done`); no se modifica su estado de tareas. No se marcan como `superseded` (no fueron reemplazados); se marcan como `closed`/históricos. |

### (D) Registro formal de nuevas deudas técnicas

| ID | Hallazgo (evidencia en disco 2026-08-16) | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A4 | Tres nuevas deudas técnicas activas detectadas, no registradas formalmente: (1) **TD-NEW-ROP-SIGN** — `register.use-case.ts` (líneas 48-108), `login.use-case.ts` (líneas 55-129) y `refresh-session.use-case.ts` (líneas 48-110) contienen `try { ... } catch { return fail(technicalFailure()); }` envolviendo toda la lógica, incluyendo `this.jwt.sign(...)`, `this.userRepo.*`, `this.passwordHasher.*`, `this.sessionRepo.*`, `this.cookieToken.*`. Contradice Master Spec §ROP ("cada adapter de salida captura excepciones... y las traduce a `DomainError`; no las propaga hacia controllers ni dominio"; "No se usa `throw`/`catch` como flujo de negocio"). Es el patrón corregido en `provision`/`bootstrap`/`activate-admin` (B1) pero **no** en estos tres use cases; el shared context de estabilización lo confirma como "hallazgo separado, no parte de B5". (2) **TD-NEW-HTTP-SEC** — `main.ts` (10 líneas) no configura helmet, CSP, HSTS, nosniff, CORS allowlist, CSRF (Origin/double-submit) ni rate limiting; grep de `helmet|csurf|csrf|rate-limit|rateLimit|@nestjs/throttler|cors` en `src/` no encontró resultados. Master Spec §Identidad exige "CSRF Origin/double-submit, CORS allowlist, CSP/HSTS/nosniff, rate limits de login/registro/reset/activación". (3) **TD-NEW-COV** — `package.json` sección `jest` no define `coverageThreshold`; `test:cov: jest --coverage` genera reporte pero no falla por umbral. Master Spec §Estrategia de Test exige "umbrales de fallo (min 85% por archivo testable)"; MSF-TEST-001 (que configuraría esto) está `blocked`. | `docs/specs/technical_debt.md` (global); `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local) | Registrar las tres deudas como `active` (no resueltas) en ambos registros, con responsable, condición de cierre, evidencia requerida y estado. No se declaran resueltas en este incremento. Ver §Nuevas deudas técnicas a registrar para el contenido exacto. |
| SCC-A5 | **TD-NEW-STAB-LIFECYCLE** — la inconsistencia de lifecycle del incremento de estabilización (encabezados `awaiting-human-plan-approval` vs `Current status: implemented`) es una deuda técnica de lifecycle. Se resuelve **con evidencia** en este incremento mediante SCC-A1, porque B1-B5 están implementados y verificados en disco (2026-08-16), el task board de estabilización está `done`, y TD-MSF-STAB-001/002/003/004 están `resolved-by-verified-implementation`. | `docs/specs/technical_debt.md` (global); `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local) | Registrar TD-NEW-STAB-LIFECYCLE y marcarla `resolved-by-verified-implementation` con evidencia: (a) task board de estabilización `done`; (b) B1-B5 verificados en disco 2026-08-16 (ver Artifact evidence del shared context de estabilización); (c) TD-MSF-STAB-001/002/003/004 `resolved-by-verified-implementation`; (d) SCC-A1 alinea el lifecycle a `closed`. La evidencia de cierre es la alineación documental aplicada por SCC-A1. |

### (E) Actualización de workspace_changes

| ID | Hallazgo | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A6 | `workspace_changes.md` no registra el cierre/consolidación del incremento de estabilización ni las nuevas deudas TD-NEW-*. | `docs/specs/workspace_changes.md` | Añadir una sección de cambio descendente que registre: (1) cierre del incremento de estabilización (lifecycle `closed`, B1-B5 implementados y verificados); (2) consolidación de decisiones durables B1-B5 en Master Spec; (3) registro de TD-NEW-ROP-SIGN/HTTP-SEC/COV como `active` y TD-NEW-STAB-LIFECYCLE como `resolved-by-verified-implementation`; (4) gates legal/AWS (TD-MSF-ID-002-02/03) permanecen abiertos; (5) decisión JWT (`iss`/`aud`/`typ`/proveedor) sigue pendiente (STAB-DEC-12). No se cambian rutas, endpoints, schemas ni dependencias cross-service. |

### (F) Gates de producción abiertos

| ID | Hallazgo | Artefacto afectado | Acción |
|---|---|---|---|
| SCC-A7 | TD-MSF-ID-002-02 (revisión legal/contable de retención/anonimización) y TD-MSF-ID-002-03 (AWS scheduler coordinación/reemplazo) son gates antes de producción. Este incremento no los cierra ni los declara resueltos. | `docs/specs/technical_debt.md`; `projects/merkee-shop-api/docs/specs/technical_debt.md`; `docs/specs/workspace_changes.md` | Confirmar que ambas deudas permanecen `active` como gates de producción. No se añade evidencia de cierre legal/contable ni de configuración AWS. No se declara listo para producción. |

## Decisiones

### SCC-DEC-01 — Estado final del incremento de estabilización

**Decisión:** el incremento `merkee-shop-foundation-stabilization` transiciona a `closed` (estado final: implementado y cerrado/consolidado).

- El delta spec de estabilización → `**Lifecycle status:** closed` y `## Status: closed` (conservando `**Spec Validator:** ready` y `**Shared context:**` como trazabilidad histórica del veredicto y aprobación).
- El shared context de estabilización → `**Lifecycle status:** closed` (conservando `## Spec Validator Approval` con `verdict: ready`, `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` como trazabilidad histórica; no se invalidan).
- El task board de estabilización → permanece `done` (sin cambio).
- **No se marcan como `superseded`** (no fueron reemplazados por una decisión distinta); se marcan como `closed`/históricos. La distinción `implemented` (B1-B5 verificados) → `closed` (consolidado por este incremento) sigue la regla SDD "El estado final debe distinguir entre `implemented`, `closed` y `superseded`".
- **OQ-SCC-01 resuelta (aprobación humana 2026-08-16):** el usuario confirmó `closed` como estado final para el delta spec y el shared context de estabilización. No se usa un estado intermedio `implemented` distinto de `closed`; la distinción `implemented` (B1-B5 verificados) → `closed` (consolidado por este incremento) queda registrada como trazabilidad, no como dos estados vigentes simultáneos.

### SCC-DEC-02 — Consolidación sin duplicar historial

**Decisión:** las decisiones durables B1-B5 se consolidan en la Master Spec integrándolas en secciones existentes (§ROP, §Identidad, §Decomposition Contract) de forma concisa, **sin duplicar** el historial del incremento de estabilización (notas STAB-A*, STAB-B*, STAB-DRIFT-*, STAB-DEC-* son historial del incremento, no reglas durables de Master Spec).

- Lo ya consolidado por el incremento de estabilización (no reeditar): AC-05 (`must_change_password: true` en replay, STAB-A5/STAB-DEC-01); §ROP (404 de provision STAB-DEC-10, `DUPLICATE_WEBHOOK_EVENT` clasificación interna STAB-DEC-11); §NC-08 (scheduler driving adapter local STAB-A6/STAB-DEC-06).
- Lo que falta consolidar de forma durable y concisa: (B1) mención de que `activate-admin` aplica el patrón ROP; (B3) mención durable del fail-fast del secreto JWT en producción; (B5) mención de que `jwt.verify` aplica el patrón ROP. Estas menciones se integran en §ROP/§Identidad sin crear una sección "Estabilización" nueva.

### SCC-DEC-03 — Nuevas deudas: registro sin cierre (salvo lifecycle)

**Decisión:** TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC y TD-NEW-COV se registran como `active` (no resueltas); TD-NEW-STAB-LIFECYCLE se registra y se marca `resolved-by-verified-implementation` con evidencia (la alineación del lifecycle por SCC-A1).

- Las tres deudas `active` se resuelven en incrementos futuros: TD-NEW-ROP-SIGN (alinear `register`/`login`/`refresh-session` al patrón ROP, como B1 para `activate-admin`); TD-NEW-HTTP-SEC (configurar helmet/CSP/HSTS/nosniff/CORS/CSRF/rate-limit, probablemente en MSF-OPS-001 o un incremento de seguridad); TD-NEW-COV (configurar `coverageThreshold` en `package.json`/Jest, probablemente en MSF-TEST-001).
- TD-NEW-STAB-LIFECYCLE se resuelve en este incremento porque la evidencia de implementación ya existe (B1-B5 verificados, task board `done`, TD-MSF-STAB-001/002/003/004 resueltas) y la acción SCC-A1 es puramente documental.

### SCC-DEC-04 — Decisión JWT sigue pendiente

**Decisión:** los valores `iss` (issuer), `aud` (audience), `typ` y el proveedor JWT (algoritmo/biblioteca) **no se fijan** en este incremento. Siguen como decisión pendiente (STAB-DEC-12) para una delta futura antes de producción. La consolidación de B3 (fail-fast del secreto) y B5 (ROP `jwt.verify`) **no cubre** `iss`/`aud`/`typ` ni el proveedor JWT. La implementación actual (`jsonwebtoken`/HS256, sin `iss`/`aud` explícitos) sigue siendo provisional y no constituye la decisión canónica.

### SCC-DEC-05 — Graphify: blocked para frescura

**Decisión:** el grafo `graphify-out/GRAPH_REPORT.md` (1,245 nodos / 2,603 aristas, sin ciclos, 269 nodos aislados, construido desde commit `89bcd155`) se referencia como artefacto canónico con contenido verificado en disco, pero la frescura respecto a `HEAD` permanece **blocked para frescura** (no se ejecuta Git ni `graphify --update` en este incremento). Se recomienda ejecutar `graphify update .` y `git rev-parse HEAD` antes de la revalidación de Spec Validator como verificación defensiva. No se afirma que el grafo esté actualizado respecto a `HEAD`.

### SCC-DEC-06 — Artefactos in-place, sin archive (OQ-SCC-02 resuelta)

**Decisión:** los artefactos del incremento de estabilización (delta spec, shared context, task board) se mantienen in-place en sus rutas actuales (`docs/specs/increments/`, `docs/specs/.working/`, `docs/specs/tasks/`) marcados `closed`/históricos. No se crea la carpeta `docs/specs/archive/` ni se mueven artefactos a ella.

- El estado aplicado es `closed` (no `superseded`, porque el incremento de estabilización no fue reemplazado por una decisión distinta — consistente con SCC-DEC-01).
- El workspace no tiene carpeta `docs/specs/archive/`; el usuario confirmó mantener in-place sin crear archive. `documentation-lifecycle` permite "mantener como referencia inmutable".
- SCC-A3 se ajusta en consecuencia: confirmar artefactos `closed`/históricos in-place; ninguna acción de movimiento ni creación de carpeta.

### SCC-DEC-07 — Consolidación solo en Master Spec, sin reeditar ADR-017 (OQ-SCC-03 resuelta)

**Decisión:** la consolidación durable de B1-B5 ocurre únicamente en `docs/specs/master_spec.md` (§ROP/§Identidad). No se reedita ADR-017 en `docs/specs/architecture-decisions.md`.

- Las decisiones durables B1-B5 son aplicación de ADR-017 (ROP) ya existente; ADR-017 no requiere reedición ni notas adicionales.
- SCC-A2 se confirma: consolidar B1 (activate-admin ROP), B3 (JWT fail-fast del secreto) y B5 (jwt.verify ROP) en Master Spec, sin añadir menciones en `architecture-decisions.md`.
- `docs/specs/architecture-decisions.md` queda sin acción en este incremento.

### SCC-DEC-08 — Task Decomposer crea task board de SCC-A1–A7 (OQ-SCC-04 resuelta)

**Decisión:** tras `verdict: ready` de Spec Validator + aprobación humana formal (`## Human Plan Approval: approved_by_user`), Task Decomposer crea un task board para las acciones documentales SCC-A1 a SCC-A7 y las descompone en tareas atómicas ejecutables.

- No se ejecutan las acciones documentales sin un task board previo; el estado superior del task board arranca en `todo` (regla SDD: después de aprobación de Spec Validator y antes de que Executor empiece, el estado superior del task board debe ser `todo`).
- El Planner no crea el task board directamente (regla SDD: Task Decomposer es dueño exclusivo del task board después de `validated-not-executed`).
- El gate de ejecución documental se ajusta para exigir task board previo de Task Decomposer.

## Nuevas deudas técnicas a registrar

Contenido exacto a añadir en `docs/specs/technical_debt.md` (global) y `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local), en la tabla "Deuda activa" (las tres primeras) y en "Cambios completados — no son deuda activa" (TD-NEW-STAB-LIFECYCLE):

### Deuda activa (añadir a la tabla "Deuda activa")

| ID | Descripción e impacto | Responsable | Condición de cierre | Evidencia requerida para cierre | Estado |
|---|---|---|---|---|---|
| TD-NEW-ROP-SIGN | `try/catch` técnico en `application` de `register.use-case.ts`, `login.use-case.ts` y `refresh-session.use-case.ts` envolviendo toda la lógica (incluida `jwt.sign` y otros adapters de salida). Contradice Master Spec §ROP y ADR-017: los adapters de salida deben capturar/traducir excepciones a `DomainError` y la application no debe usar `throw`/`catch` como flujo de negocio. Es el patrón corregido en `provision`/`bootstrap`/`activate-admin` (STAB-B1) pero no en estos tres use cases. Impacto: drift ROP en tres use cases de identidad; los fallos técnicos se capturan en application en lugar del adapter. | Planner + Executor (incremento futuro de ROP sign) | Un incremento futuro alinea `register`/`login`/`refresh-session` al patrón ROP: los adapters de salida (`JwtPort.sign`, `UserRepositoryPort`, `PasswordHasherPort`, `SessionRepositoryPort`, `CookieTokenPort`) capturan/traducen excepciones a `DomainError`/`Result`; la application elimina el `try/catch` técnico y devuelve `Result` solo con reglas de negocio. | Diff sin `try/catch` técnico en los tres use cases; tests del adapter traduciendo fallos técnicos a `TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII; `npm run depcruise` sin violaciones; `npm test` PASS. | active — no bloqueante para desarrollo local; se resuelve en un incremento futuro de ROP sign. |
| TD-NEW-HTTP-SEC | `main.ts` (10 líneas) no configura headers de seguridad HTTP (CSP, HSTS, nosniff), CORS allowlist, CSRF (Origin/double-submit) ni rate limiting de login/registro/reset/activación. Master Spec §Identidad exige "CSRF Origin/double-submit, CORS allowlist, CSP/HSTS/nosniff, rate limits de login/registro/reset/activación". Grep de `helmet|csurf|csrf|rate-limit|rateLimit|@nestjs/throttler|cors` en `src/` no encontró resultados. Impacto: el API no aplica las protecciones de seguridad HTTP exigidas por la Master Spec; riesgo si se despliega sin configurar. | Plataforma/Seguridad + Executor (incremento futuro de seguridad HTTP o MSF-OPS-001) | Un incremento futuro (o MSF-OPS-001) configura helmet/CSP/HSTS/nosniff, CORS allowlist, CSRF Origin/double-submit y rate limiting de login/registro/reset/activación conforme a Master Spec §Identidad, sin cambiar endpoints, schemas ni reglas de negocio. | Diff de `main.ts`/`app.module.ts` con helmet, CSP/HSTS/nosniff, CORS allowlist, CSRF y rate limiting configurados; tests de los headers y rate limits; `npm run build` OK; `npm test` PASS. | active — no bloqueante para desarrollo local; **gate antes de producción** (no desplegar sin las protecciones HTTP exigidas). |
| TD-NEW-COV | `package.json` sección `jest` no define `coverageThreshold`; `test:cov: jest --coverage` genera reporte de cobertura pero no falla el comando si la cobertura está por debajo del umbral. Master Spec §Estrategia de Test exige "umbrales de fallo (min 85% por archivo testable)"; MSF-TEST-001 (que configuraría esto) está `blocked`. Impacto: no hay gate de cobertura en CI/local; la cobertura puede degradarse sin detección. | Test-Architect (MSF-TEST-001 o incremento futuro de cobertura) | MSF-TEST-001 (o un incremento futuro) configura `coverageThreshold` en `package.json`/Jest con umbral mínimo 85% por archivo testable (excluyendo DTOs pasivos/config/código generado), de forma que `npm run test:cov` falle si la cobertura está por debajo. | Diff de `package.json` con `coverageThreshold` configurado (stmts/branches/funcs/lines ≥ 85% global y por archivo testable); `npm run test:cov` PASS con umbral; reporte de cobertura. | active — no bloqueante para desarrollo local; recomendado antes de producción. |

### Cambio completado (añadir a la tabla "Cambios completados — no son deuda activa")

| ID | Evidencia | Estado |
|---|---|---|
| TD-NEW-STAB-LIFECYCLE | Lifecycle del incremento `merkee-shop-foundation-stabilization` alineado a `closed` con evidencia (2026-08-16): task board de estabilización `done`; B1-B5 (ROP activate-admin, `must_change_password` replay `true`, JWT fail-fast del secreto, `operation-map.ts` 404, ROP JWT verify) implementados y verificados en disco; TD-MSF-STAB-001/002/003/004 `resolved-by-verified-implementation`; delta spec y shared context de estabilización marcados `closed` por SCC-A1. | resolved-by-verified-implementation |

## Artefactos a actualizar

| Artefacto | Cambio | Owner | Estado documental |
|---|---|---|---|
| `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` | SCC-A1: alinear `## Status` y `**Lifecycle status:**` a `closed`; conservar `**Spec Validator:** ready` y `**Shared context:**` como trazabilidad histórica. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` | SCC-A1: alinear `**Lifecycle status:**` a `closed`; conservar `## Spec Validator Approval` (`verdict: ready`), `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` como trazabilidad histórica. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `docs/specs/master_spec.md` | SCC-A2: consolidar de forma durable y concisa (B1 activate-admin ROP, B3 JWT fail-fast del secreto, B5 jwt.verify ROP) en §ROP/§Identidad sin duplicar historial; verificar AC-05/§ROP/§NC-08 ya consolidados. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `docs/specs/technical_debt.md` | SCC-A4/A5/A7: añadir TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC, TD-NEW-COV como `active`; añadir TD-NEW-STAB-LIFECYCLE a "Cambios completados" como `resolved-by-verified-implementation`; confirmar TD-MSF-ID-002-02/03 `active`. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `projects/merkee-shop-api/docs/specs/technical_debt.md` | SCC-A4/A5/A7: espejo local idéntico al global. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `docs/specs/workspace_changes.md` | SCC-A6: añadir sección de cambio descendente de cierre/consolidación. | Planner | ejecutado (SCC-A1–A7 `done`, verificado en disco 2026-08-16 mediante `docs/specs/tasks/stab-closure-consolidation-task-board.md`). |
| `docs/specs/tasks/merkee-shop-foundation-stabilization-task-board.md` | SCC-A3: permanece `done` (sin cambio de estado de tareas); no se modifica. | — | sin acción (permanece `done`). |
| `docs/specs/architecture-decisions.md` | Sin acción (SCC-DEC-07): las decisiones durables B1-B5 son aplicación de ADR-017 (ROP) ya existente; no se reedita ADR-017. La consolidación de B1-B5 ocurre solo en Master Spec. | — | sin acción. |

## Criterios de aceptación

| ID | Criterio verificable |
|---|---|
| AC-SCC-01 | El delta spec `merkee-shop-foundation-stabilization-delta-spec.md` declara `## Status: closed` y `**Lifecycle status:** closed`; conserva `**Spec Validator:** ready` y `**Shared context:**` como trazabilidad histórica; no se eliminan las secciones `## Controlled Pre-Ready Remediation: approved_by_user` ni `## Human Plan Approval: approved_by_user` del delta spec. |
| AC-SCC-02 | El shared context `merkee-shop-foundation-stabilization-sdd-context.md` declara `**Lifecycle status:** closed`; conserva `## Spec Validator Approval` con `verdict: ready`, `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` como trazabilidad histórica; `## Current status` refleja `closed` (incremento implementado y consolidado). |
| AC-SCC-03 | La Master Spec consolida de forma durable y concisa: (B1) `activate-admin` aplica patrón ROP; (B3) JWT fail-fast del secreto en producción (`JWT_SECRET` ausente o < 32 bytes detiene arranque; default solo en desarrollo con advertencia; `.env.example` documenta); (B5) `jwt.verify` aplica patrón ROP (puerto `Result`, adapter traduce, controller consume `Result` del puerto). No se duplica el historial STAB-*; no se fijan `iss`/`aud`/`typ` ni el proveedor JWT (siguen pendientes STAB-DEC-12). |
| AC-SCC-04 | `docs/specs/technical_debt.md` y `projects/merkee-shop-api/docs/specs/technical_debt.md` registran TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC y TD-NEW-COV como `active` con responsable, condición de cierre, evidencia requerida y estado idénticos en ambos registros; ninguna de las tres se declara resuelta. |
| AC-SCC-05 | `docs/specs/technical_debt.md` y `projects/merkee-shop-api/docs/specs/technical_debt.md` registran TD-NEW-STAB-LIFECYCLE en "Cambios completados — no son deuda activa" como `resolved-by-verified-implementation` con evidencia (task board `done`, B1-B5 verificados, TD-MSF-STAB-001/002/003/004 resueltas, SCC-A1 aplicado). |
| AC-SCC-06 | `docs/specs/workspace_changes.md` registra el cierre/consolidación del incremento de estabilización, las nuevas deudas TD-NEW-* y la permanencia de los gates legal/AWS; no se cambian rutas, endpoints, schemas ni dependencias cross-service. |
| AC-SCC-07 | TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen `active` como gates antes de producción en ambos registros de deuda; no se declaran resueltos ni se añade evidencia de cierre legal/contable o AWS. |
| AC-SCC-08 | No se modifica código (`projects/merkee-shop-api/src/**`), OpenAPI (`docs/api/openapi.yaml`), Prisma (`projects/merkee-shop-api/prisma/**`), `package.json`, `.env*` ni runtime config. Verificable por diff: ningún archivo fuera de `docs/specs/**` y `projects/merkee-shop-api/docs/specs/**` es modificado. |
| AC-SCC-09 | No se ejecutan operaciones de Git (commits, branches, PRs); la frescura de Graphify respecto a `HEAD` permanece **blocked para frescura**. |
| AC-SCC-10 | Spec Validator emite veredicto sobre el conjunto canónico + esta delta; el incremento no transiciona a ejecución de acciones documentales sin `verdict: ready` + aprobación humana (`## Human Plan Approval: approved_by_user`). |
| AC-SCC-11 | La decisión JWT (`iss`/`aud`/`typ` y proveedor) sigue documentada como pendiente (STAB-DEC-12) en Master Spec §Identidad; la consolidación de B3/B5 no la fija canónicamente. |
| AC-SCC-12 | No se crea la carpeta `docs/specs/archive/` ni se mueven artefactos del incremento de estabilización a ella (SCC-DEC-06); el delta spec, el shared context y el task board de estabilización permanecen in-place en sus rutas actuales (`docs/specs/increments/`, `docs/specs/.working/`, `docs/specs/tasks/`) marcados `closed`/históricos. |
| AC-SCC-13 | No se reedita ADR-017 en `docs/specs/architecture-decisions.md` (SCC-DEC-07); la consolidación durable de B1-B5 ocurre únicamente en `docs/specs/master_spec.md` (§ROP/§Identidad), sin añadir secciones nuevas de historial STAB-* ni menciones en `architecture-decisions.md`. |
| AC-SCC-14 | Tras `verdict: ready` + aprobación humana (`## Human Plan Approval: approved_by_user`), Task Decomposer crea un task board para SCC-A1 a SCC-A7 (SCC-DEC-08); no se ejecutan las acciones documentales sin un task board previo, y el estado superior del task board arranca en `todo`. |

## Decomposition Contract

- **Rutas canónicas:** sin cambio (no se tocan endpoints). Rutas canónicas vigentes: `/admin/users`, `/auth/admin-activations`, `/auth/password-change`, etc. (prefijo `/v1` es del `servers.url`).
- **DTOs afectados:** sin cambio. `AdminUserProvisionResponse.must_change_password: true` (literal) ya alineado a OpenAPI `const: true` (STAB-DEC-01 Opción A).
- **Puertos afectados:** sin cambio. `ActivateAdminUnitOfWorkPort.run` → `Result<T, DomainError>` (B1, ya implementado); `JwtPort.verify` → `Result<JwtPayload, DomainError>` (B5, ya implementado); `JwtPort.sign` → `Promise<string>` (TD-NEW-ROP-SIGN: pendiente de alineación ROP en incremento futuro, no en este).
- **Tablas/columnas:** sin cambio (no hay migración). `idempotency_records.response_json` sin cambio.
- **OpenAPI:** sin cambio. Este incremento no edita `docs/api/openapi.yaml`.
- **Prisma:** sin cambio. Este incremento no edita `projects/merkee-shop-api/prisma/**`.
- **Orden permitido:** (SCC-A1 alinear lifecycle estabilización → `closed`) → (SCC-A2 consolidar B1-B5 en Master Spec) → (SCC-A3 confirmar artefactos closed/históricos) → (SCC-A4 registrar TD-NEW-ROP-SIGN/HTTP-SEC/COV `active`) → (SCC-A5 registrar TD-NEW-STAB-LIFECYCLE `resolved-by-verified-implementation`) → (SCC-A6 actualizar workspace_changes) → (SCC-A7 confirmar gates legal/AWS abiertos). Todas las acciones son documentales y pueden secuenciarse o paralelizarse salvo SCC-A2 (Master Spec) que debe coordinarse con SCC-A1 (referencia al estado cerrado).
- **Términos stale prohibidos en este incremento:** declarar TD-NEW-ROP-SIGN/HTTP-SEC/COV resueltas (son `active`); declarar TD-MSF-ID-002-02/03 resueltas (son gates abiertos); fijar `iss`/`aud`/`typ` o el proveedor JWT canónicamente (decisión pendiente STAB-DEC-12); afirmar que el grafo está actualizado respecto a `HEAD` sin Git (blocked para frescura); modificar código/OpenAPI/Prisma/`package.json`/runtime; ejecutar Git; crear task board antes de `verdict: ready` + aprobación humana; declarar `ready` sin Spec Validator; marcar el incremento de estabilización como `superseded` (no fue reemplazado; es `closed`); eliminar las aprobaciones históricas `ready`/humanas del incremento de estabilización (se conservan como trazabilidad); crear `docs/specs/archive/` o mover artefactos de estabilización a ella (SCC-DEC-06: mantener in-place); reeditar ADR-017 en `architecture-decisions.md` (SCC-DEC-07: consolidar solo en Master Spec); ejecutar acciones documentales SCC-A1–A7 sin task board previo de Task Decomposer (SCC-DEC-08).
- **Archivos autoritativos:** esta delta spec, `docs/specs/master_spec.md`, `docs/specs/architecture-decisions.md` (ADR-017 ROP), `docs/specs/technical_debt.md`, `projects/merkee-shop-api/docs/specs/technical_debt.md`, `docs/specs/workspace_changes.md`, `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md`, `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md`, `docs/specs/tasks/merkee-shop-foundation-stabilization-task-board.md`, `docs/specs/.working/stab-closure-consolidation-sdd-context.md`.
- **No autorizado:** crear endpoints, migraciones, schemas, parámetros o reglas de negocio; modificar código/OpenAPI/Prisma; ejecutar Git; crear task board antes del gate; declarar `ready` sin Spec Validator; cerrar gates legal/AWS; fijar `iss`/`aud`/`typ`/proveedor JWT.

## Gates

1. **Gate de Spec Validator (evalúa el contrato documental):** solicitar revalidación focalizada de Spec Validator sobre el conjunto canónico + esta delta — consistencia entre artefactos documentales, decisiones locked, alcance exclusivamente documental y descomponibilidad de las acciones documentales. El gate NO exige que las acciones documentales estén ejecutadas para emitir `verdict: ready` (evalúa el contrato, no la ejecución). Hasta `verdict: ready`, no hay ejecución de acciones ni handoff.
2. **Gate de aprobación humana:** tras `ready` de Spec Validator, el incremento transiciona a `awaiting-human-plan-approval`; el Planner no enruta a otros agentes sin el encabezado `## Human Plan Approval: approved_by_user`.
3. **Gate de ejecución documental (post-aprobación):** tras `ready` + aprobación humana, Task Decomposer crea un task board para las acciones documentales SCC-A1 a SCC-A7 (SCC-DEC-08) y las descompone en tareas atómicas ejecutables; el Planner o un agente de documentación ejecuta las tareas desde el task board. La evidencia de ejecución es el diff de los artefactos documentales (lifecycle `closed`, Master Spec consolidada, deudas registradas, `workspace_changes.md` actualizado). No se ejecutan las acciones documentales sin task board previo.
4. **Gates de producción existentes (no se cierran):** TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen `active`; no se despliega ni se opera en producción.
5. **Gate de frescura Graphify (no se cierra):** la frescura del grafo respecto a `HEAD` permanece **blocked para frescura**; se recomienda ejecutar `graphify update .` + `git rev-parse HEAD` antes de la revalidación de Spec Validator como verificación defensiva.

## Open questions

- Ninguna abierta. OQ-SCC-01, OQ-SCC-02, OQ-SCC-03 y OQ-SCC-04 están resueltas por aprobación humana (2026-08-16) y registradas como decisiones locked: OQ-SCC-01 → SCC-DEC-01 (confirmada, estado final `closed`); OQ-SCC-02 → SCC-DEC-06 (mantener in-place, sin archive); OQ-SCC-03 → SCC-DEC-07 (consolidar solo en Master Spec, sin reeditar ADR-017); OQ-SCC-04 → SCC-DEC-08 (Task Decomposer crea task board de SCC-A1–A7).
