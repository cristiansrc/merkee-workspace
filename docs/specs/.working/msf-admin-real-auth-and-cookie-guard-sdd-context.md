# Shared Context — msf-admin-real-auth-and-cookie-guard

**Lifecycle status:** `planning`
**Actualizado:** 2026-08-20

## Current status

`planning`: Incremento de cierre del transporte de autenticación admin (cookie-parser + JWT guard + limpieza de cookie en logout). **No hay `verdict: ready` de Spec Validator, no hay `## Human Plan Approval: approved_by_user`, no hay task board, no hay handoff a Task Decomposer/Executor.** No se modificó código, OpenAPI, Prisma, migraciones, `package.json`, runtime ni se ejecutó Git. Las deudas activas preexistentes (`TD-NEW-ROP-SIGN`, `TD-NEW-HTTP-SEC`, `TD-NEW-COV`, `TD-MSF-ID-002-01/02/03`, `TD-AWS-RDS-PUBLIC`, `TD-AWS-SWAGGER-DNS`, `TD-AWS-OBSERVABILITY`, `TD-AWS-ECS-VALIDATION`, `TD-MSF-API-001`, `TD-MSF-API-003` reabierto, `TD-MSF-ID-003-01`) **no** se declaran resueltas por este incremento. La nueva deuda `TD-ADMIN-AUTH-001` (cookie-parser + JWT guard + logout) se registra como `active` y se cierra tras la implementación verificada del incremento. Los gates de producción TD-MSF-ID-002-02 (legal/contable) y TD-MSF-ID-002-03 (AWS) permanecen abiertos. La decisión JWT STAB-DEC-12 (`iss`/`aud`/`typ` y proveedor) sigue pendiente.

## Canonical artifacts

- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/msf-admin-real-auth-and-cookie-guard-delta-spec.md` (este incremento, `planning`)
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` (`validated-not-executed`, revalidación focalizada pendiente) — §Identidad, §ROP, §Decomposition Contract, §Replay seguro de `POST /auth/password-change` (ADR-020).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` (canónico; **sin cambios en este incremento**). Endpoints afectados: `POST /auth/login` (200/400/401/429/500), `POST /auth/refresh` (200/401/403/500), `POST /auth/logout` (204/401/500).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` — ADR-004 (sesiones), ADR-017 (ROP), ADR-020 (replay seguro password-change). **No se añade ADR nuevo** (DEC-09).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md` (`validated-not-executed`; trazabilidad B1-B5 + MSF-ID-003 + ADR-020).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` (`closed`; B1-B5 verificados).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/msf-id-rop-sign-cleanup-sdd-context.md` (`planning`; intersección de archivos en `login.use-case.ts`/`logout.use-case.ts` sin fusión de alcance).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md` (sin cambios; 001–014 vigentes).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md` (global; nueva `TD-ADMIN-AUTH-001` a añadir tras este incremento).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local; misma entrada).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md` (`revision-needed`; añadir sección "Inicio del incremento `msf-admin-real-auth-and-cookie-guard` (2026-08-20)" tras Planner).
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/graphify-out/GRAPH_REPORT.md` (commit `89bcd155`; **blocked para frescura** sin Git; 1,245 nodos / 2,603 aristas, sin ciclos).

**Código autoritativo verificado en disco (NO modificado por el Planner; verificado 2026-08-20):**
- `projects/merkee-shop-api/src/main.ts` (10 líneas; sin `cookieParser`).
- `projects/merkee-shop-api/src/shared/http/transport-auth.guard.ts` (46 líneas; solo regex de presencia).
- `projects/merkee-shop-api/src/shared/http/transport-auth.guard.spec.ts` (46 líneas; 3 casos de presencia/sintaxis).
- `projects/merkee-shop-api/src/shared/http/result-projector.ts` (52 líneas; reusado por DEC-02).
- `projects/merkee-shop-api/src/shared/http/domain-error-mapper.ts` (208 líneas; mapeo canónico código→status).
- `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` (391 líneas; `getActor`, `readCookie`, `login`/`refresh`/`logout`/`passwordChange`).
- `projects/merkee-shop-api/src/modules/identity/identity.controller.spec.ts` (980 líneas; suite de `POST /auth/logout` sin verificación de `Set-Cookie`).
- `projects/merkee-shop-api/src/modules/identity/identity.module.ts` (371 líneas; providers ya cablean `IDENTITY_TOKENS.JWT → JwtAdapter`).
- `projects/merkee-shop-api/src/modules/identity/domain/ports/jwt.port.ts` (`JwtPayload{sub, session_id, role}`; `verify → Result<JwtPayload, DomainError>` por STAB-B5).
- `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts` (75 líneas; `verify` captura `TokenExpiredError`/`JsonWebTokenError`/`NotBeforeError` → `sessionNotFoundOrExpired`; fail-fast del secreto por STAB-B3).
- `projects/merkee-shop-api/src/modules/identity/domain/identity-errors.ts` (`authenticationRequired`, `sessionNotFoundOrExpired`, `technicalFailure`).
- `projects/merkee-shop-api/src/modules/identity/application/use-cases/{login,logout,refresh-session}.use-case.ts` (sin cambios; firma pública intacta).
- `projects/merkee-shop-api/package.json` (sin `cookie-parser`).
- `projects/merkee-shop-admin/src/components/AuthGuard.tsx` (51 líneas; redirige si `isAuthenticated: false`; estado inicial `loading: false`).
- `projects/merkee-shop-admin/src/store/authSlice.ts` (140 líneas; `USE_MOCKS` flag; `loading: false` por defecto).
- `projects/merkee-shop-admin/src/api/client.ts` (341 líneas; `withCredentials: true`; interceptor 401 redirige a `/login` con `window.location.href`).
- `projects/merkee-shop-admin/src/providers/authProvider.ts` (110 líneas; `check()` invoca `fetchProfile` si no está autenticado).

## Artifact evidence

| Ruta | Campo/flujo verificado | Resultado | Estado |
|---|---|---|---|
| `projects/merkee-shop-api/src/main.ts` (10 líneas) | Sin `cookieParser` middleware | `req.cookies` siempre `undefined` en runtime. Hallazgo H3. | fail |
| `projects/merkee-shop-api/package.json` líneas 24-35 | Sin `cookie-parser` en dependencies | `req.cookies` no se popula; bloqueante funcional para `POST /auth/refresh` y limpieza de cookie en logout. Hallazgo H3. | fail |
| `projects/merkee-shop-api/src/shared/http/transport-auth.guard.ts` líneas 23-46 | Solo regex `^Bearer\s+\S+$` | Acepta cualquier cadena `Bearer xyz` sin verificar firma. Hallazgo H1. | fail |
| `projects/merkee-shop-api/src/shared/http/transport-auth.guard.spec.ts` líneas 14-46 | Solo verifica presencia y sintaxis | Tests insuficientes; faltan casos de verificación JWT. Hallazgo H1. | fail |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` líneas 84-88 | `getActor(req)` lee `req.user` | Como H1 nunca asigna, `actor` siempre `null`. Hallazgo H2. | fail |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` líneas 187-190, 281-296, 257-259, 314-324 | `userIdFromGuard: actor ? actor.id : null`, `actorId: actor ? actor.id : ''`, `sessionId: actor ? actor.sessionId : ''` | Endpoints autenticados reciben `null`/`''`; backend no identifica al principal. Hallazgo H2. | fail |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` líneas 246-264 | `POST /auth/logout` no emite `Set-Cookie` de limpieza | Cookie `merkee_refresh_session` persiste hasta `expires` natural tras logout. Hallazgo H4. | fail |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.spec.ts` líneas 942-979 | `describe('POST /auth/logout')` no verifica `clearCookie` ni `Set-Cookie` con `Max-Age=0` | Tests insuficientes. Hallazgo H4. | fail |
| `projects/merkee-shop-api/src/modules/identity/domain/ports/jwt.port.ts` líneas 6-29 | `JwtPayload{sub, session_id, role}`; `verify → Result<JwtPayload, DomainError>` | Puerto canónico; sin cambios requeridos. STAB-B5 verificado. | pass |
| `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts` líneas 54-74 | `verify` captura `TokenExpiredError`/`JsonWebTokenError`/`NotBeforeError` → `sessionNotFoundOrExpired`; otros → `technicalFailure` | ROP correcto; sin causa/PII. STAB-B5 verificado. | pass |
| `projects/merkee-shop-api/src/modules/identity/domain/identity-errors.ts` líneas 17-39, 73-88 | `authenticationRequired`, `sessionNotFoundOrExpired`, `technicalFailure`, `invalidCredentials` | Catálogo suficiente para DEC-02. | pass |
| `projects/merkee-shop-api/src/shared/http/result-projector.ts` líneas 19-52 | `buildErrorResponse` + `projectResult` lanza `HttpException` con `ApiErrorResponse` | Reusable por DEC-02. | pass |
| `projects/merkee-shop-api/src/shared/http/domain-error-mapper.ts` líneas 46-74 | Mapeo `AUTHENTICATION_REQUIRED: 401`, `TECHNICAL_DEPENDENCY_FAILURE: 500` | Coincide con DEC-02. | pass |
| `projects/merkee-shop-api/src/modules/identity/identity.module.ts` líneas 66-69, 339-369 | `jwtProvider` registrado con `IDENTITY_TOKENS.JWT → JwtAdapter`; `IdentityModule` exporta el provider vía el módulo HTTP (a confirmar por Executor) | DEC-06 asume disponibilidad de `JwtPort` para `TransportAuthGuard`; wiring exacta es del Executor. | pass (con caveat) |
| `docs/api/openapi.yaml` | `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout` declarados con statuses 200/400/401/429/500, 200/401/403/500, 204/401/500 respectivamente | Canónico intacto; este incremento no modifica OpenAPI (DEC-05). | pass |
| `docs/api/openapi.yaml` líneas 386-425 (schemas) | `LoginRequest{email, password}`, `SessionResponse{access_token, expires_at, user}`, `UserResponse{id, display_name, email, role, must_change_password, phone}` | Canónicos; sin cambios. | pass |
| `docs/specs/master_spec.md` §Identidad (líneas 83-100) | JWT ≤10 min en memoria; token opaco hashado en cookie HttpOnly, Secure (prod), SameSite=Lax, rotado al refresh; Argon2id; CSRF/CSP/HSTS/nosniff/rate-limits | DEC-01/DEC-03 alineados; CSRF/HSTS/rate-limit siguen bajo `TD-NEW-HTTP-SEC`. | pass |
| `docs/specs/master_spec.md` §ROP (líneas 36-42) | Adapters capturan/traducen; controllers consumen `Result` del puerto; `domain`/`application` sin captura técnica | DEC-02 coherente con el patrón vigente. | pass |
| `docs/specs/master_spec.md` §Decomposition Contract (líneas 142-145) | Rutas canónicas sin `/v1` (prefijo del `servers.url`) | Paths del incremento: `/auth/login`, `/auth/refresh`, `/auth/logout`, `/me`. Sin cambios. | pass |
| `docs/specs/architecture-decisions.md` ADR-017 | ROP estricto + catálogo `DomainError`; adapters traducen; controller consume `Result` | DEC-02 aplica este patrón. | pass |
| `docs/specs/architecture-decisions.md` ADR-004 | Sesión servidor 10 min; JWT max 10 min memoria; refresh token opaco HttpOnly rotado | DEC-01/DEC-03 alineados. | pass |
| `docs/specs/architecture-decisions.md` ADR-020 | Replay seguro password-change con minimización de credenciales; sin `iss`/`aud`/`typ` fijados (STAB-DEC-12 pendiente) | DEC-07 coherente. | pass |
| `projects/merkee-shop-admin/src/components/AuthGuard.tsx` líneas 18-49 | Redirige a `/login` si `!isAuthenticated`; muestra spinner si `loading`; respeta `mustChangePassword` | Si H1-H4 se corrigen y el login funciona, el admin debería redirigir correctamente. H5: síntoma puede requerir fix adicional en admin. | pass (con Q-04) |
| `projects/merkee-shop-admin/src/store/authSlice.ts` líneas 14-20 | `loading: false` por defecto | El guard no muestra spinner infinito en arranque. El síntoma de "pantalla blanca" **no** se explica por esto directamente. H5. | pass |
| `projects/merkee-shop-admin/src/api/client.ts` líneas 47-62 | Interceptor 401 → `setAccessToken(null)` + `window.location.href = '/login'` | Si el guard del API devuelve 401 (H1 hace lo opuesto, pero la combinación de H1 con backend roto puede causar loops). H5. | pass (con Q-04) |
| `projects/merkee-shop-api/.env.example` (20 líneas) | Documenta `JWT_SECRET` (mín. 32 bytes) y `INITIAL_ADMIN_PASSWORD` | Coherente con STAB-B3; sin cambios. | pass |
| `graphify-out/GRAPH_REPORT.md` (1,245 nodos / 2,603 aristas, sin ciclos; commit `89bcd155`) | Contenido verificado en disco | Frescura respecto a `HEAD` **blocked para frescura** sin Git. Recomendar `graphify update .` y `git rev-parse HEAD` antes de la revalidación como verificación defensiva. | pass (contenido) / blocked (frescura) |
| `docs/specs/technical_debt.md` (global) | Sin entrada `TD-ADMIN-AUTH-001` | Se añade en este incremento tras Planner (DEC-implícita). | fail (pendiente de Planner) |
| `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local) | Sin entrada `TD-ADMIN-AUTH-001` | Idem global. | fail (pendiente de Planner) |

## Spec Validator Approval

- verdict: pending
- reviewed_at: —
- validator_agent: spec-validator
- artifact_set_reviewed: —
- summary: —
- invalidated_by_changes_since: none

## Decisions locked

- **DEC-01:** `cookie-parser` middleware global en `main.ts` (DEC-01); pin `^1.4.6` y `@types/cookie-parser ^1.4.7`; montado antes de `setGlobalPrefix`/`enableCors`/`listen`. Sin firma criptográfica de cookies (CSRF queda bajo `TD-NEW-HTTP-SEC`).
- **DEC-02:** `TransportAuthGuard` reescrito para inyectar `JwtPort` por `IDENTITY_TOKENS.JWT`, llamar `verify(token)`, mapear `Result` a `HttpException` con `ApiErrorResponse` (`AUTHENTICATION_REQUIRED` 401 / `TECHNICAL_DEPENDENCY_FAILURE` 500), asignar `req.user = { id: payload.sub, sessionId: payload.session_id, role: payload.role }` en éxito. Reutiliza `projectResult`/`buildErrorResponse`. Sin causa/PII.
- **DEC-03:** `POST /v1/auth/logout` emite `res.clearCookie(REFRESH_COOKIE_NAME, { path: '/', httpOnly: true, secure: NODE_ENV==='production', sameSite: 'lax' })` **solo en éxito** (204). En failure NO se limpia cookie (defensa).
- **DEC-04:** Tests actualizados (no nuevos archivos). `transport-auth.guard.spec.ts`: 5 casos (sin header → 401; `Basic ...` → 401; token válido → éxito + `req.user`; token expirado → 401; error técnico → 500). `identity.controller.spec.ts`: 2 casos nuevos en `describe('POST /auth/logout')` (clearCookie en éxito; sin clearCookie en failure).
- **DEC-05:** Sin cambios en OpenAPI, Prisma, migraciones, DTOs, claims JWT.
- **DEC-06:** `IDENTITY_TOKENS.JWT` se reutiliza; el guard se monta en módulo que provea `JwtPort` (wiring exacta del Executor).
- **DEC-07:** `iss`/`aud`/`typ` y proveedor JWT siguen pendientes (STAB-DEC-12); este incremento NO los fija.
- **DEC-08:** El fix del síntoma de pantalla blanca del admin queda documentado (H5) y registrado como Q-04; **no** se modifica el admin en este incremento.
- **DEC-09:** No se introduce patrón GoF nuevo (continuidad del patrón ROP); no se requiere consulta formal al `solution-architect`.
- **DEC-implícita:** Se registra `TD-ADMIN-AUTH-001` (cookie-parser ausente + JWT guard sin verificación + logout sin limpieza de cookie) como `active` en `docs/specs/technical_debt.md` y su espejo local; responsable = Planner + Executor de este incremento; condición de cierre = implementación verificada de DEC-01/DEC-02/DEC-03 con `npm test` PASS, `npm run depcruise` sin violaciones y los tests manuales E2E pasando; evidencia = diff de archivos listados en §6.2 y suite de tests actualizada.

## Validator findings

(Bloqueado: Spec Validator no ha revisado. La spec está en `planning` con open questions pendientes — ver §Open questions.)

## Resolved findings

(Ninguno resuelto por validación aún. Hallazgos H1-H5 documentados en la delta spec; decisiones DEC-01..DEC-09 registradas.)

## Historical open questions (superseded)

(N/A — este es el primer incremento del flujo `msf-admin-real-auth-and-cookie-guard`.)

## Open questions

1. **Q-01** — ¿El `TransportAuthGuard` debe leer el token solo del header `Authorization: Bearer` o también de una cookie (e.g., para integraciones internas)? Recomendación: solo header (alineado con OpenAPI y admin actual). Confirmar con el usuario.
2. **Q-02** — ¿La limpieza de cookie en logout debe emitirse ANTES o DESPUÉS del `use case`? Recomendación: DESPUÉS del éxito (DEC-03). Confirmar.
3. **Q-03** — ¿Este incremento debe coordinarse con `msf-id-rop-sign-cleanup` (planning)? Recomendación: ejecutar este primero (más pequeño, solo transporte); luego el otro. Confirmar orden o simultaneidad.
4. **Q-04** — La "pantalla blanca" en `/` del admin — ¿se reproduce tras las correcciones H1-H4? Si persiste, requiere fix independiente en `merkee-shop-admin` (proyecto separado). Posibles causas adicionales: `loading: true` que nunca se resuelve, error de import en `DashboardLayout`, loop del interceptor 401 con `window.location.href`. Se solicita al equipo admin verificar.
5. **Q-05** — ¿El `TransportAuthGuard` debe rechazar también JWT firmados pero con sesión revocada? Recomendación: NO (defensa en profundidad; la revocación se valida downstream). Confirmar.
6. **Q-06** — ¿`CORS_ALLOWED_ORIGINS` debe ampliarse para incluir staging (`https://staging.admin.merkee.shop`)? Si no aplica staging, no hacer nada. Confirmar.
7. **Q-07** — ¿El incremento debe emitir también una limpieza defensiva de `merkee_cart_session` cuando admin cierra sesión (Master Spec §Identidad indica que admin no tiene carrito)? Recomendación: NO en este incremento (la cookie de cart session solo se crea para guest; admin nunca la tiene tras login). Confirmar.
8. **Q-08** — ¿La spec debe documentar también el comportamiento esperado del admin ante 403 `INITIAL_PASSWORD_CHANGE_REQUIRED` (admin con `must_change_password=true` debe llamar `/auth/password-change`)? Ya está en Master Spec §Identidad y AuthGuard.tsx; este incremento no lo toca. Confirmar que no requiere acción adicional.

## Stale terms guard

Forbidden:
- "guard verifica solo la presencia de Bearer" como patrón vigente.
- "cookie-parser ausente" como estado aceptado (se cierra con DEC-01).
- "logout no limpia cookie" como patrón vigente (se cierra con DEC-03).
- "actor=null en endpoints autenticados" como estado aceptado (se cierra con DEC-02).
- Modificar OpenAPI, Prisma, migraciones, claims del JWT para "arreglar" este incremento.
- Aplicar `TransportAuthGuard` globalmente con `APP_GUARD` sin excluir endpoints públicos.
- Inyectar `JwtAdapter` concreto en `TransportAuthGuard` (debe ser `JwtPort`).
- `try/catch` técnico en `application` de `login`/`logout`/`refresh-session` (deuda activa `TD-NEW-ROP-SIGN`, separada, NO se resuelve aquí).
- Fijar `iss`/`aud`/`typ` o el proveedor JWT canónicamente (decisión pendiente STAB-DEC-12; este incremento NO la elige).
- Declarar listo para producción (gates TD-MSF-ID-002-02 legal/contable y TD-MSF-ID-002-03 AWS abiertos; TD-AWS-RDS-PUBLIC, TD-AWS-SWAGGER-DNS, TD-AWS-OBSERVABILITY, TD-AWS-ECS-VALIDATION activas; TD-NEW-HTTP-SEC activa; TD-MSF-API-001 activa; TD-MSF-ID-003-01 activa).
- Declarar `verdict: ready` de Spec Validator antes de la revalidación.
- Handoff a Task Decomposer/Executor sin `verdict: ready` + aprobación humana `## Human Plan Approval: approved_by_user`.
- Operaciones de Git (commits, branches, PRs).
- Afirmar que el grafo está actualizado respecto a `HEAD` sin verificación con Git (frescura **blocked**).
- Declarar el incremento `superseded` (no fue reemplazado por otro; es `planning`).
- Mezclar la causa raíz del backend (H1-H4) con el síntoma del admin (H5); son dos correcciones distintas.
- Aplicar este incremento en producción antes de cerrar los gates TD-MSF-ID-002-02 y TD-MSF-ID-002-03.

## Next action

Esperar revalidación focalizada de Spec Validator. Si emite `verdict: ready` con los campos exactos (`verdict`, `reviewed_at`, `validator_agent`, `artifact_set_reviewed`, `summary`, `invalidated_by_changes_since: none`), registrarlo en `## Spec Validator Approval` y solicitar aprobación humana del plan (`## Human Plan Approval: approved_by_user`). Si Spec Validator emite hallazgos, remediarlos (DEC-04, §Pruebas, §Decomposition Contract de la delta spec) antes del handoff. Tras la aprobación humana, Task Decomposer (no el Planner) crea el task board y descompone las tareas; el Planner no marca tareas como `done` ni crea task boards directamente.
