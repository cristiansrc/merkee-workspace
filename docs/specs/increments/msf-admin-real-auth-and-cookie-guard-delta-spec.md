# Delta Spec — msf-admin-real-auth-and-cookie-guard

## Status: planning
**Lifecycle status:** `planning`
**Incremento padre:** `merkee-shop-foundation` (`validated-not-executed`, revalidación focalizada pendiente)
**Creado:** 2026-08-20 · **Planner:** activo · **Spec Validator:** `pending`
**Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/msf-admin-real-auth-and-cookie-guard-sdd-context.md`
**Deuda técnica origen:** nueva `TD-ADMIN-AUTH-001` (cookie-parser ausente + JWT guard sin verificación + logout sin limpieza de cookie); se relaciona con `TD-NEW-ROP-SIGN` (interseccion de archivos en `login.use-case.ts`/`logout.use-case.ts`) sin fusionarse.

> Estado `planning`: artefactos en preparación. **No hay `verdict: ready` de Spec Validator, no hay `## Human Plan Approval: approved_by_user`, no hay task board, no hay handoff a Task Decomposer/Executor.** No se modificó código, OpenAPI, Prisma, migraciones, `package.json`, runtime ni se ejecutó Git. No se declaran resueltas deudas activas preexistentes.

## 1. Propósito y alcance

Habilitar el flujo real de autenticación admin (login → cookie de refresh HttpOnly → JWT en memoria → guard con verificación de firma → logout con limpieza de cookie) y, con ello, resolver el síntoma de "pantalla blanca" en `/` del admin cuando el guard no recibe un actor/sesión válido. El incremento **no introduce endpoints, módulos, migraciones, schemas OpenAPI, parámetros ni reglas de negocio nuevos**: respeta los paths, statuses, claims JWT, DTOs y tablas ya declarados.

### 1.1 Alcance exhaustivo y exclusivo

- **Server API (`projects/merkee-shop-api/`)**: añadir `cookie-parser` middleware; reescribir `TransportAuthGuard` para inyectar `JwtPort`, verificar la firma del JWT y asignar `req.user = { id, sessionId, role }`; emitir `Set-Cookie` de limpieza en `POST /v1/auth/logout` (204) tras éxito o no-op idempotente; mantener `POST /v1/auth/login` y `POST /v1/auth/refresh` emitiendo/rotando la cookie `merkee_refresh_session` con la semántica ya implementada; actualizar los tests afectados.
- **Hallazgo del admin (`projects/merkee-shop-admin/`)** documentado: la causa raíz probable de la "pantalla blanca" en `/` es la ausencia de un actor verificable en el backend. La corrección definitiva del comportamiento del guard visual del admin queda **fuera del alcance** de este incremento (es responsabilidad del proyecto admin, no del API), pero se documenta como Q-04 para que el equipo admin decida si requiere un fix menor en `authSlice.ts`/`AuthGuard.tsx` (p. ej., inicializar `loading: true` y resolver tras `fetchProfile`, o un guard contra Refine). **No** se modifica el admin en este incremento.

### 1.2 Precedencia aplicada

(1) Solicitud explícita del usuario (hallazgos confirmados: cookie-parser, JWT guard, logout cookie). (2) Código implementado, migraciones, OpenAPI y runtime configuration del repositorio (verificado en disco 2026-08-20). (3) Specs activas (`validated-not-executed`/`planning`). No hay conflicto con OpenAPI ni con migraciones (no se tocan). No hay conflicto con `iss`/`aud`/`typ` del JWT (no se modifican).

### 1.3 Hallazgos confirmados (código real verificado en disco 2026-08-20)

| ID | Hallazgo | Evidencia | Contrato canónico que valida la corrección |
|---|---|---|---|
| **H1** | `TransportAuthGuard` (`shared/http/transport-auth.guard.ts`) solo verifica la presencia sintáctica del header `Authorization: Bearer <token>` (regex); **no** invoca `JwtPort.verify`, **no** decodifica el payload, **no** asigna `req.user`. Devuelve `true` con cualquier cadena `Bearer xyz`, por lo que cualquier endpoint con `@UseGuards(TransportAuthGuard)` acepta tokens sin firma. | `projects/merkee-shop-api/src/shared/http/transport-auth.guard.ts` líneas 23-46; `shared/http/transport-auth.guard.spec.ts` líneas 14-46 (solo presencia/sintaxis). | Master Spec §ROP (catálogo `AUTHENTICATION_REQUIRED`/`TECHNICAL_DEPENDENCY_FAILURE`); §Identidad "JWT de acceso ≤10 min"; ADR-017 (adapters traducen, controller consume `Result`). |
| **H2** | `IdentityController.getActor(req)` (`identity.controller.ts` líneas 84-88) lee `req.user`, pero como H1 nunca asigna, `actor` siempre es `null`. En `GET /me` (línea 187-190), `userIdFromGuard: actor ? actor.id : null` siempre es `null`, por lo que `GetMyProfileUseCase` recibe `userId=null`; en `PATCH /me`, `actorId: actor ? actor.id : ''` queda `''`; en `POST /auth/logout` (línea 257-259), `sessionId: actor ? actor.sessionId : ''` queda `''`; en `POST /auth/password-change` (líneas 318-324), `actorId`/`currentSessionId` quedan vacíos. **El backend no identifica al principal autenticado** porque el guard no extrae `id`/`session_id` del JWT. | `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` líneas 84-88 y métodos `getMyProfile`/`updateProfile`/`logout`/`changePassword`. | Master Spec §Identidad (roles exclusivos `admin`/`cliente`); §Decomposition Contract (rutas autenticadas protegidas por actor). |
| **H3** | `cookie-parser` **no** está en `package.json` (`@nestjs/platform-express` + `rawBody: true` solamente). `main.ts` no monta `app.use(cookieParser())`. Por tanto `req.cookies` es `undefined` y `readCookie(req, REFRESH_COOKIE_NAME)` (controller línea 91-95) siempre devuelve `undefined` en runtime (los tests del controller usan `cookieRequest` mock que sí popula `req.cookies`). Resultado: **`POST /v1/auth/refresh` siempre devuelve 401 `AUTHENTICATION_REQUIRED` en condiciones reales** aunque la cookie exista en el navegador, y `POST /v1/auth/logout` no puede leer la cookie para revocar la sesión del lado servidor si la lógica dependiera de ella (en la implementación actual solo usa `sessionId` del JWT, pero el síntoma de refresh roto sigue). | `projects/merkee-shop-api/package.json` líneas 24-35 (sin `cookie-parser`); `src/main.ts` (10 líneas, sin `app.use(cookieParser())`). | Master Spec §Identidad "token opaco hashado en cookie `HttpOnly; Secure; SameSite=Lax`, rotado al refresh". |
| **H4** | `POST /v1/auth/logout` (`identity.controller.ts` líneas 246-264) **no** emite `Set-Cookie` para limpiar `merkee_refresh_session` en el navegador. Tras éxito (204) la cookie persiste hasta su `expires` natural (10 min). El test del logout (spec líneas 942-979) no verifica `Set-Cookie` con `Max-Age=0`. | `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` líneas 246-264; `identity.controller.spec.ts` líneas 942-979. | Master Spec §Identidad "JWT de acceso ≤10 min solo en memoria; token opaco hashado en cookie HttpOnly rotado al refresh" + semántica de revocación de sesión. |
| **H5** | El admin (`projects/merkee-shop-admin/`) tiene `AuthGuard` con `loading: false` por defecto y redirige correctamente cuando `isAuthenticated: false`. La "pantalla blanca en `/`" reportada por el usuario **no es directamente explicada por H1-H4** (esos rompen el flujo post-login), pero sí podría ser un síntoma secundario: tras un login que el admin cree exitoso, las llamadas a `GET /me` u otros recursos autenticados devolverían 401 (porque el guard del API nunca verifica la firma → en realidad H1 hace lo opuesto, devuelve `true` con cualquier token — pero el controller recibe `actor=null` y devuelve error de "sesión no encontrada"). En cualquier caso, la corrección del API (H1-H4) elimina la causa raíz; el síntoma visual del admin puede requerir un fix independiente documentado como Q-04. | `projects/merkee-shop-admin/src/components/AuthGuard.tsx` líneas 18-49; `src/store/authSlice.ts` líneas 14-20 (estado inicial `loading: false`). | Master Spec §Decomposition Contract + flujo canónico admin. |

### 1.4 Fuera de alcance (explícito)

- Cambiar OpenAPI, Prisma, migraciones, endpoints, schemas, parámetros, reglas de negocio.
- Inventar claims nuevos del JWT (`iss`/`aud`/`typ` y proveedor siguen como decisión pendiente STAB-DEC-12; no se eligen aquí).
- Reemplazar el `TransportAuthGuard` actual por un guard que valide también cookies: el flujo actual del admin usa `Authorization: Bearer <jwt>` en memoria (no la cookie de refresh). La cookie de refresh se valida únicamente en `POST /v1/auth/refresh`.
- Modificar el admin (proyecto `merkee-shop-admin`): el fix del síntoma visual de pantalla blanca es del proyecto admin; este incremento solo lo deja documentado (Q-04) y elimina la causa raíz en el backend.
- HTTP security headers (helmet/CSP/HSTS/nosniff/CSRF/rate-limit): siguen bajo `TD-NEW-HTTP-SEC`; este incremento NO los aplica (no bloquea desarrollo local).
- `iss`/`aud`/`typ` y proveedor JWT (STAB-DEC-12) — sigue pendiente.
- `try/catch` técnico en `application` de `register`/`login`/`refresh-session` (deuda `TD-NEW-ROP-SIGN`) — sigue activa; este incremento solo toca la **superficie HTTP** (controller + guard + middleware), no la lógica de negocio de los use cases.
- Operaciones de Git, task board, handoff, producción.

## 2. Decisiones

### 2.1 DEC-01 — `cookie-parser` como middleware global

- **Decisión:** añadir `cookie-parser` (`^1.4.6`) y `@types/cookie-parser` (`^1.4.7`, devDependency) a `package.json`. En `main.ts`, montar `app.use(cookieParser())` **después** de `rawBody: true` (necesario para webhooks) y **antes** de `setGlobalPrefix('v1')`/`enableCors(corsOptions)`/`listen`. El middleware popula `req.cookies` para `refresh` y `logout`.
- **Motivo:** los handlers ya leen `req.cookies[name]` vía `readCookie` (controller líneas 91-95); sin `cookie-parser` ese helper siempre devuelve `undefined` en runtime. La corrección mínima es habilitar el middleware que popula la propiedad; no requiere cambios en el dominio ni en OpenAPI.
- **No cubre:** la firma criptográfica de cookies (CSRF/state cookies); eso queda bajo `TD-NEW-HTTP-SEC`.

### 2.2 DEC-02 — `TransportAuthGuard` con verificación JWT y asignación de actor

- **Decisión:** reescribir `TransportAuthGuard` (`shared/http/transport-auth.guard.ts`) para:
  1. Inyectar `JwtPort` vía DI (`@Inject(IDENTITY_TOKENS.JWT) private readonly jwt: JwtPort`).
  2. Extraer el header `Authorization: Bearer <token>` con la misma regex actual (presencia sintáctica).
  3. Llamar `await this.jwt.verify(token)`.
  4. Si `Result.ok`: asignar `request.user = { id: payload.sub, sessionId: payload.session_id, role: payload.role }` y devolver `true`.
  5. Si `Result.fail` con `code === AUTHENTICATION_REQUIRED` (token inválido/expirado): lanzar `UnauthorizedException` con cuerpo `ApiErrorResponse` mapeado (status 401, code `AUTHENTICATION_REQUIRED`).
  6. Si `Result.fail` con `code === TECHNICAL_DEPENDENCY_FAILURE` (error técnico inesperado): lanzar `HttpException` con cuerpo `ApiErrorResponse` (status 500, code `TECHNICAL_DEPENDENCY_FAILURE`). No se revela causa/PII.
- **Motivo:** STAB-B5 ya alineó `JwtPort.verify → Result<JwtPayload, DomainError>` y el adapter traduce a códigos del catálogo existente (sin inventar nuevos). El guard debe consumir ese `Result` (no `try/catch` técnico ni un nuevo try alrededor de `jwt.verify`). Es exactamente el patrón declarado en Master Spec §ROP "El controller de identidad inyecta el puerto `JwtPort` (no el adapter concreto) y proyecta el rail `Failure`".
- **Mapeo de errores:** se reutiliza `projectResult`/`buildErrorResponse` (`shared/http/result-projector.ts`); el guard lanza `HttpException` con el body completo (`timestamp`, `status`, `error`, `code`, `message`, `path`, `trace_id`). El `path` se completa con `req.originalUrl ?? req.url ?? '/'`; `trace_id` con `req.headers['x-request-id']` si existe.
- **No cubre:** reasignación de roles ni revocación en línea; `must_change_password` se proyecta a 403 por el endpoint downstream (`GET /me`, `PATCH /me`, etc.) cuando el controller lo requiera.

### 2.3 DEC-03 — `logout` emite `Set-Cookie` con `Max-Age=0` en éxito

- **Decisión:** tras `projectResult` exitoso en `IdentityController.logout` (línea 246-264), emitir `res.clearCookie(REFRESH_COOKIE_NAME, { path: '/', httpOnly: true, secure: NODE_ENV==='production', sameSite: 'lax' })`. El método `clearCookie` de Express usa `Expires=Thu, 01 Jan 1970 00:00:00 GMT; Max-Age=0` por defecto, lo que provoca que el navegador elimine la cookie inmediatamente. Si el resultado es `Failure`, **no** se limpia la cookie (la sesión no se revocó y el navegador debe intentar de nuevo).
- **Motivo:** Master Spec §Identidad exige que el token opaco rote al refresh; por simetría, al logout debe invalidarse en el navegador. Sin limpieza, una cookie obsoleta puede ser reusada por código malicioso hasta su expiración natural (10 min).
- **Idempotencia:** la revocación de sesión ya es idempotente (`logout.use-case.ts` líneas 47-49). La limpieza de cookie solo se emite cuando el resultado es `Success` o cuando la sesión ya estaba revocada (`ok(undefined)` retornado por idempotencia). Si la sesión nunca existió (`fail(sessionNotFoundOrExpired())` → 401), no se limpia cookie (defensa: no exponer superficie de ataque).

### 2.4 DEC-04 — Tests actualizados sin nuevos archivos

- **Decisión:** actualizar `transport-auth.guard.spec.ts` (4 casos: sin header → 401; header `Basic ...` → 401; token válido → éxito + `req.user` asignado; token expirado/inválido → 401 vía `verify` mock que devuelve `fail(sessionNotFoundOrExpired())`; error técnico → 500 vía `verify` mock que devuelve `fail(technicalFailure())`). Actualizar `identity.controller.spec.ts` para añadir un caso `POST /auth/logout` que verifique `clearCookie` (o equivalente `res.cookie` con `Max-Age=0`) en éxito y que verifique que **no** se llama `clearCookie` en failure.
- **No se crean** archivos de spec nuevos; se modifican los existentes.

### 2.5 DEC-05 — Sin cambios en OpenAPI, Prisma, claims JWT ni DTOs

- **Decisión:** este incremento NO modifica `docs/api/openapi.yaml`, `projects/merkee-shop-api/prisma/schema.prisma`, las migraciones 001–014, los DTOs (`SessionResponse`, `LoginRequest`, etc.) ni los claims del JWT (`sub`, `session_id`, `role`). Los handlers existentes (`POST /v1/auth/login`, `/v1/auth/refresh`, `/v1/auth/logout`) ya están declarados en OpenAPI con sus statuses correctos; este incremento solo ajusta su **comportamiento de transporte** (cookie + verificación de firma + limpieza).
- **Motivo:** la precedencia (OpenAPI > código) exige no relajar ni inventar contrato. La semántica ya está cubierta.

### 2.6 DEC-06 — `IDENTITY_TOKENS.JWT` ya existe y se reutiliza

- **Decisión:** `IDENTITY_TOKENS.JWT` (`identity.tokens.ts`) ya está registrado como provider de `JwtAdapter` en `identity.module.ts` línea 66-69. El `TransportAuthGuard` se registra en el módulo HTTP (`shared/http/http.module.ts`) o como provider global con alcance `APP_GUARD`. La DI funciona mientras el guard se monte en un módulo que importe `IdentityModule` o exponga `JwtPort`. La wiring exacta es decisión del Executor; la spec exige que **el guard inyecte `JwtPort` por el símbolo `IDENTITY_TOKENS.JWT`**, no el adapter concreto (continuidad de ADR-017).
- **No se renombra** `IDENTITY_TOKENS.JWT`; no se duplica el provider.

### 2.7 DEC-07 — `iss`/`aud`/`typ` y proveedor JWT siguen pendientes (STAB-DEC-12)

- **Decisión:** este incremento NO fija `iss`/`aud`/`typ` ni la elección del proveedor JWT (algoritmo/biblioteca). El estado actual (`jsonwebtoken`/HS256, `expiresIn: '10m'`, sin `iss`/`aud` explícitos) sigue siendo provisional y no constituye la decisión canónica. STAB-DEC-12 permanece activa.
- **No se modifica** `JwtAdapter` ni `JwtPort`.

### 2.8 DEC-08 — Pantalla blanca del admin queda documentada, no corregida aquí

- **Decisión:** la corrección del síntoma visual en `/` del admin es responsabilidad del proyecto `merkee-shop-admin`, no del API. Este incremento documenta el hallazgo (H5) y la pregunta abierta Q-04 sin tocar `AuthGuard.tsx`, `authSlice.ts`, `authProvider.ts` ni nada del admin. La causa raíz en el backend (H1-H4) se corrige aquí; el comportamiento del guard visual queda para un fix independiente en el admin si la revalidación confirma que el síntoma persiste tras el fix del backend.

### 2.9 DEC-09 — Consulta al `solution-architect`

- **Decisión:** este incremento NO introduce un patrón GoF nuevo. Reutiliza:
  - **Adapter** ya aplicado: `cookie-parser` es middleware externo; `JwtPort.verify` ya devuelve `Result<JwtPayload, DomainError>` (STAB-B5) — el guard es un consumer de ese adapter vía puerto.
  - **Strategy**: NO se introduce (no hay dos algoritmos de verificación que justifiquen variabilidad).
  - **Decorator / Proxy / Chain of Responsibility / State**: NO se introducen.
- **Justificación (skill `design-patterns-standard`):** el cambio es continuidad del patrón ROP (controller/adapter consume `Result` del puerto). No hay variabilidad real que justifique Strategy ni complejidad que justifique Decorator. La introducción de `cookie-parser` es middleware Express estándar, no un patrón de diseño.
- **No se requiere** consulta formal al `solution-architect` (no se introduce módulo/endpoint/integración); el incremento es de cierre de transporte. Se documenta para trazabilidad.

## 3. Criterios de aceptación

| ID | Criterio verificable |
|---|---|
| **AC-AG-01** | `cookie-parser` está declarado en `package.json` (`dependencies.cookie-parser` con versión pinneada o caret), `@types/cookie-parser` en `devDependencies`; `npm install` y `npm run build` finalizan sin error. |
| **AC-AG-02** | `main.ts` invoca `app.use(cookieParser())` antes de `app.listen()` y la prueba de integración con `supertest`/Testcontainers envía `Cookie: merkee_refresh_session=...` y la recibe correctamente poblada en `req.cookies`. |
| **AC-AG-03** | `POST /v1/auth/login` con credenciales válidas responde 200 + `Set-Cookie: merkee_refresh_session=<token>; HttpOnly; Secure (en prod); SameSite=Lax; Path=/; Expires=<iso>` (verificar el header en la respuesta). |
| **AC-AG-04** | `POST /v1/auth/refresh` con cookie `merkee_refresh_session` válida responde 200 + `Set-Cookie` rotada con nuevo token y `expires_at` posterior. Sin cookie o cookie inválida/expirada responde 401 `AUTHENTICATION_REQUIRED` sin `Set-Cookie`. |
| **AC-AG-05** | `POST /v1/auth/logout` con sesión válida responde 204 + `Set-Cookie: merkee_refresh_session=; Max-Age=0; Path=/; HttpOnly; Secure (en prod); SameSite=Lax`. En failure (`AUTHENTICATION_REQUIRED` 401) **no** se emite `Set-Cookie` de limpieza. |
| **AC-AG-06** | `TransportAuthGuard` rechaza con 401 `AUTHENTICATION_REQUIRED` si falta el header `Authorization` o si la sintaxis no es `Bearer <token>`. |
| **AC-AG-07** | `TransportAuthGuard` rechaza con 401 `AUTHENTICATION_REQUIRED` cuando `JwtPort.verify` devuelve `Result.fail` por token inválido o expirado. El cuerpo de la respuesta es un `ApiErrorResponse` con `code=AUTHENTICATION_REQUIRED`, `status=401`, `path` y `trace_id` completados; sin causa/PII. |
| **AC-AG-08** | `TransportAuthGuard` rechaza con 500 `TECHNICAL_DEPENDENCY_FAILURE` cuando `JwtPort.verify` devuelve `Result.fail` por error técnico inesperado. El cuerpo es `ApiErrorResponse` con `code=TECHNICAL_DEPENDENCY_FAILURE`, sin causa/PII. |
| **AC-AG-09** | `TransportAuthGuard` permite la request y asigna `request.user = { id: payload.sub, sessionId: payload.session_id, role: payload.role }` cuando `JwtPort.verify` devuelve `Result.ok`. |
| **AC-AG-10** | `getActor(req)` en `IdentityController` extrae `{ id, sessionId }` de `req.user` correctamente; `GET /me`, `PATCH /me`, `POST /auth/logout` y `POST /auth/password-change` reciben un `actor` válido tras un login exitoso. |
| **AC-AG-11** | `npm run build` OK; `npm test` PASS con suites actualizadas (TransportAuthGuard con 5 casos; IdentityController.logout con caso `Set-Cookie`); `npm run depcruise` sin violaciones (controller y guard inyectan `JwtPort` por símbolo, no `JwtAdapter`). |
| **AC-AG-12** | No se modifican `docs/api/openapi.yaml`, `projects/merkee-shop-api/prisma/schema.prisma`, migraciones 001–014, ni claims/keys del JWT. `git diff` (verificación de Executor, no del Planner) sobre estos archivos debe estar vacío. |

## 4. Pruebas funcionales exigidas

### 4.1 Pruebas unitarias (mínimas)

- `transport-auth.guard.spec.ts`:
  - `permite la request y asigna req.user cuando verify devuelve ok` (mock `JwtPort.verify → ok({ sub, session_id, role })`).
  - `rechaza 401 AUTHENTICATION_REQUIRED sin header Authorization`.
  - `rechaza 401 AUTHENTICATION_REQUIRED con header no Bearer`.
  - `rechaza 401 AUTHENTICATION_REQUIRED cuando verify devuelve fail con sessionNotFoundOrExpired`.
  - `rechaza 500 TECHNICAL_DEPENDENCY_FAILURE cuando verify devuelve fail con technicalFailure`.
- `identity.controller.spec.ts`:
  - Caso nuevo en `describe('POST /auth/logout')`: `emite Set-Cookie de limpieza con Max-Age=0 en éxito` (mock `res.clearCookie`).
  - Caso nuevo: `NO emite clearCookie cuando el use case devuelve Failure`.

### 4.2 Pruebas de integración (Testcontainers + supertest)

- Login → recibe 200 + `Set-Cookie merkee_refresh_session=...` + `access_token` en body.
- Refresh con la cookie recibida (sin `Authorization` Bearer) → 200 + cookie rotada.
- Refresh sin cookie → 401 `AUTHENTICATION_REQUIRED`.
- `GET /me` con `Authorization: Bearer <access_token>` → 200 + perfil.
- `GET /me` con token forjado (`Bearer not-a-jwt`) → 401 `AUTHENTICATION_REQUIRED` (no 500; `JwtAdapter.verify` debe traducir `JsonWebTokenError` a `sessionNotFoundOrExpired`).
- Logout con sesión válida → 204 + cookie limpiada (verificar que la siguiente request `/auth/refresh` falla con 401 por cookie expirada/inexistente).

### 4.3 Pruebas manuales E2E (curl)

```bash
# 1. Login (capturar cookie en cookie-jar)
curl -i -c cookies.txt -X POST https://api.merkee.shop/v1/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"<admin>","password":"<pwd>"}'
# Esperado: 200 + Set-Cookie merkee_refresh_session=...

# 2. Refresh con cookie (sin Bearer)
curl -i -b cookies.txt -c cookies.txt -X POST https://api.merkee.shop/v1/auth/refresh
# Esperado: 200 + Set-Cookie merkee_refresh_session=<nuevo>

# 3. GET /me con Bearer
curl -i -b cookies.txt -H "Authorization: Bearer <access_token>" https://api.merkee.shop/v1/me
# Esperado: 200 + perfil

# 4. GET /me con Bearer falso
curl -i -H "Authorization: Bearer fake" https://api.merkee.shop/v1/me
# Esperado: 401 AUTHENTICATION_REQUIRED

# 5. Logout
curl -i -b cookies.txt -X POST https://api.merkee.shop/v1/auth/logout
# Esperado: 204 + Set-Cookie merkee_refresh_session=; Max-Age=0

# 6. Refresh con cookie limpiada
curl -i -b cookies.txt -X POST https://api.merkee.shop/v1/auth/refresh
# Esperado: 401 AUTHENTICATION_REQUIRED
```

### 4.4 Pruebas manuales con navegador (admin)

- `https://admin.merkee.shop/` sin sesión → redirige a `/login` (verificar HTTP 200 en `/login`, no pantalla blanca).
- Tras login válido con credenciales reales → redirige a `/` y muestra el dashboard.
- Click en "Cerrar Sesión" → redirige a `/login`; inspección de `Application > Cookies` muestra `merkee_refresh_session` eliminada o expirada (`Max-Age=0`).
- Refrescar `https://admin.merkee.shop/` con DevTools cerrado y luego reabrir la pestaña → debe redirigir a `/login` (la cookie está expirada).

## 5. Decomposition Contract (para Task Decomposer)

Fuentes autoritativas de este incremento: la presente delta spec, `docs/specs/master_spec.md` §Identidad, §ROP y §Decomposition Contract; `docs/api/openapi.yaml` (sin cambios); `docs/specs/architecture-decisions.md` ADR-004/ADR-017; `projects/merkee-shop-api/src/shared/http/transport-auth.guard.ts` y `src/modules/identity/identity.controller.ts` (código actual). Rutas OpenAPI canónicas (sin cambios): `/auth/login`, `/auth/refresh`, `/auth/logout`, `/me` (PATCH), `/me` (GET), `/auth/password-change`. DTOs canónicos (sin cambios): `LoginRequest`, `SessionResponse`, `UserResponse`. Tabla canónica (sin cambios): `sessions` con columnas `id`, `user_id`, `refresh_token_hash`, `expires_at`, `revoked_at`, `last_activity_at`, `session_kind`. **Allowed task order** sugerido: (T1) instalar `cookie-parser` + `@types/cookie-parser`; (T2) montar `cookieParser()` en `main.ts`; (T3) reescribir `TransportAuthGuard` con DI de `JwtPort` + 4 tests; (T4) emitir `Set-Cookie` de limpieza en `logout` + 1 test; (T5) `npm run build` + `npm test` + `npm run depcruise` + `npm run test:cov`. **Forbidden stale terms:** "guard solo verifica presencia de header Bearer" como patrón vigente; "cookie-parser ausente" como estado aceptado; "logout no limpia cookie" como patrón vigente; "actor=null" como estado aceptado en endpoints autenticados; modificar OpenAPI/Prisma/claims; declarar producción lista; `iss`/`aud`/`typ` fijados; JWT default en producción; `try/catch` técnico en `application` (deuda activa TD-NEW-ROP-SIGN, separada). **Archivos autoritativos** (no modificables por Task Decomposer; sí modificables por Executor tras approval): ver §6.

## 6. Archivos a crear/actualizar (propiedad del Executor; el Planner solo los enumera)

### 6.1 Crear (no aplica — solo actualizar)

No se crean archivos de spec incrementales fuera de `docs/specs/increments/msf-admin-real-auth-and-cookie-guard-delta-spec.md` (este) y `docs/specs/.working/msf-admin-real-auth-and-cookie-guard-sdd-context.md` (propiedad del Planner).

### 6.2 Actualizar por el Executor (código y configuración)

| Archivo | Cambio | Riesgo |
|---|---|---|
| `projects/merkee-shop-api/package.json` | Añadir `"cookie-parser": "^1.4.6"` a `dependencies` y `"@types/cookie-parser": "^1.4.7"` a `devDependencies`. | Bajo; requiere `npm install`. |
| `projects/merkee-shop-api/package-lock.json` | Regenerado por `npm install`. | Bajo; CI debe pasar. |
| `projects/merkee-shop-api/src/main.ts` | Importar `cookieParser` y montar `app.use(cookieParser())` antes de `setGlobalPrefix`/`enableCors`/`listen`. | Bajo; tests de integración con cookies deben seguir pasando. |
| `projects/merkee-shop-api/src/shared/http/transport-auth.guard.ts` | Reescribir para inyectar `JwtPort`, llamar `verify`, mapear `Result` a `HttpException` (`ApiErrorResponse`) y asignar `req.user` en éxito. | Medio; rompe API existente del guard si los tests no se actualizan en paralelo. |
| `projects/merkee-shop-api/src/shared/http/transport-auth.guard.spec.ts` | Añadir 3 casos (token válido, token expirado, error técnico) y mantener los 2 existentes. | Bajo. |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.ts` | En `logout` (línea 246-264), emitir `res.clearCookie(REFRESH_COOKIE_NAME, { path: '/', httpOnly: true, secure: NODE_ENV==='production', sameSite: 'lax' })` solo en éxito. | Bajo. |
| `projects/merkee-shop-api/src/modules/identity/identity.controller.spec.ts` | Añadir 1 caso que verifique `clearCookie` en éxito y 1 caso que verifique que NO se llama en failure. | Bajo. |
| `projects/merkee-shop-api/src/shared/http/http.module.ts` (o donde se monte el guard global) | Si el guard se aplica globalmente con `APP_GUARD`, asegurar que `JwtPort` está disponible en ese módulo (importar `IdentityModule` o exponer el token). | Bajo; decisión del Executor. |

### 6.3 Actualizar por el Planner (documentación)

| Archivo | Cambio |
|---|---|
| `docs/specs/increments/msf-admin-real-auth-and-cookie-guard-delta-spec.md` | Este archivo (creado). |
| `docs/specs/.working/msf-admin-real-auth-and-cookie-guard-sdd-context.md` | Crear el shared context del incremento (ver §Open questions, §Decisions locked, §Artifact evidence). |
| `docs/specs/technical_debt.md` (global) y `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local) | Añadir entrada `TD-ADMIN-AUTH-001` (cookie-parser ausente + JWT guard sin verificación + logout sin limpieza); marcarla `active` con responsable, condición de cierre y evidencia. |
| `docs/specs/workspace_changes.md` | Añadir sección "Inicio del incremento `msf-admin-real-auth-and-cookie-guard` (2026-08-20)" indicando: alcance exclusivo (H1-H5), no cambia OpenAPI/Prisma, deudas activas preexistentes no se cierran, gates abiertos. |

### 6.4 NO modificar (verificación)

| Archivo | Razón |
|---|---|
| `docs/api/openapi.yaml` | Endpoints/statuses/claims/DTOs ya declarados; este incremento es de comportamiento de transporte. |
| `projects/merkee-shop-api/prisma/schema.prisma` y migraciones 001–014 | Sin cambios de esquema ni de datos. |
| `projects/merkee-shop-api/src/modules/identity/domain/ports/jwt.port.ts` | `JwtPort.verify` ya devuelve `Result<JwtPayload, DomainError>` (STAB-B5). |
| `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts` | Fail-fast (B3) y traducción `verify` (B5) ya aplicados. |
| `projects/merkee-shop-api/src/modules/identity/application/use-cases/{login,logout,refresh-session}.use-case.ts` | Firma pública intacta; este incremento no toca la lógica de negocio (la deuda `TD-NEW-ROP-SIGN` queda activa y se resolverá en `msf-id-rop-sign-cleanup`). |
| `docs/specs/architecture-decisions.md` | No requiere ADR nuevo (no introduce módulo/endpoint/integración/patrón GoF nuevo). |

## 7. Plan de rollback

### 7.1 Rollback del incremento (si falla antes de producción)

1. **Revertir `package.json`/`package-lock.json`:** `git checkout` o `npm uninstall cookie-parser @types/cookie-parser` (commit previo).
2. **Revertir `main.ts`:** quitar `app.use(cookieParser())`.
3. **Revertir `TransportAuthGuard`:** restaurar la versión que solo verifica la presencia de `Bearer` (H1). Riesgo: el backend vuelve a aceptar tokens sin firma; **NO** debe quedarse así en producción más de lo estrictamente necesario para diagnosticar.
4. **Revertir `identity.controller.ts` logout:** quitar la llamada a `res.clearCookie`.
5. **Revertir tests** a la versión anterior.

### 7.2 Hotfix defensivo en producción (si TrasportAuthGuard rompe endpoints autenticados)

- **Síntoma:** todos los endpoints con `@UseGuards(TransportAuthGuard)` devuelven 401.
- **Causa probable:** `JwtAdapter.verify` lanza excepción no capturada (e.g., por cambio en la clave o por una versión incompatible de `jsonwebtoken`).
- **Hotfix:** revertir `transport-auth.guard.ts` a la versión "solo presencia de Bearer" temporalmente, **solo** si el problema se aísla al guard. Confirmar que el resto del stack no depende de `req.user` (todos los `actor ? actor.id : ''` caerían al fallback de cadena vacía, lo cual **rompería** `GET /me`/`PATCH /me`/`logout`/`password-change`).
- **Mitigación:** mientras el hotfix esté activo, **deshabilitar** los endpoints autenticados críticos (logout/password-change/profile) con un feature flag de emergencia, o permitir `actor.id === ''` solo para endpoints sin requisito de identificación.

### 7.3 Mitigación si la limpieza de cookie en logout falla en producción

- **Síntoma:** usuarios reportan cookies obsoletas tras logout.
- **Causa probable:** la lógica de limpieza no se invoca por una condición de carrera o por un failure que se ignora.
- **Mitigación:** añadir un middleware/interceptor de seguridad que **siempre** emita `Set-Cookie merkee_refresh_session=; Max-Age=0; Path=/` en cualquier respuesta 2xx de `/auth/logout`, independientemente del resultado del use case. Esto es red de seguridad; la lógica principal sigue siendo "solo en éxito" (DEC-03).

### 7.4 Compatibilidad con incremento `msf-id-rop-sign-cleanup` (planning)

- **Conflicto potencial:** ambos incrementos tocan `identity.controller.ts` (este: logout) y `login.use-case.ts`/`logout.use-case.ts`/`refresh-session.use-case.ts` (el otro: try/catch técnico).
- **Estrategia:** este incremento NO modifica los use cases; solo el controller (logout) y el guard. La intersección con `msf-id-rop-sign-cleanup` se limita al controller. **Recomendación:** ejecutar este incremento primero (más pequeño, sin ROP); luego `msf-id-rop-sign-cleanup` puede ajustar el controller sin colisión con el cambio de `clearCookie`.
- **Si el orden se invierte:** no hay conflicto técnico siempre que los diffs se mantengan en líneas no superpuestas (DEC-03: líneas 246-264 del controller; `msf-id-rop-sign-cleanup`: lógica interna del use case).

## 8. Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| `cookie-parser` incompatible con `@nestjs/platform-express` 10.x | Baja | Build falla | Pin versión `^1.4.6`; `npm run build` lo detecta. |
| `TransportAuthGuard` global rompe endpoints públicos (`/health`, `/categories`, `/products`) | Media | 401 en endpoints públicos | NO aplicar globalmente; `@UseGuards(TransportAuthGuard)` solo en endpoints autenticados (uso actual). |
| Cambio de firma de `TransportAuthGuard` afecta `identity.controller.spec.ts` (mock de `req.user`) | Baja | Tests fallan | Actualizar `actorRequest` helper para incluir `req.user`; verificar. |
| `logout` con `clearCookie` interfiere con otros middlewares de cookie (e.g., CSRF) | Baja | Cookie no se limpia | `clearCookie` de Express usa los mismos atributos; verificar que no haya otro middleware que reescriba `Set-Cookie`. |
| El admin no recibe el cambio de cookie por CORS | Baja | Cookie no se persiste en navegador | CORS ya configurado (`credentials: true`, `cors.config.ts`); origins `https://www.merkee.shop` y `https://admin.merkee.shop` en allowlist. Verificar `Access-Control-Allow-Credentials: true` en respuesta. |
| Replay de `refresh` con cookie vieja antes de la rotación | Baja | Doble sesión | La rotación atómica en `RefreshSessionUseCase` (`refresh-session.use-case.ts` líneas 84-89) ya lo evita; la limpieza de cookie en logout no afecta esa lógica. |

## 9. Stale terms guard

Forbidden:
- "guard verifica solo la presencia de Bearer" como patrón vigente.
- "cookie-parser ausente" como estado aceptado.
- "logout no limpia cookie" como patrón vigente.
- "actor=null en endpoints autenticados" como estado aceptado.
- "el admin debe seguir mostrando pantalla blanca tras la corrección del backend" como verdad permanente (debe verificarse tras el fix).
- Modificar OpenAPI/Prisma/migraciones/claims para "arreglar" este incremento.
- Aplicar el guard globalmente con `APP_GUARD` sin verificar que los endpoints públicos lo excluyan.
- Inyectar `JwtAdapter` concreto en `TransportAuthGuard` (debe ser `JwtPort`).
- `try/catch` técnico en `application` de `login`/`logout`/`refresh-session` (deuda activa `TD-NEW-ROP-SIGN`, separada).
- Fijar `iss`/`aud`/`typ` o el proveedor JWT canónicamente (decisión pendiente STAB-DEC-12).
- Declarar listo para producción (gates TD-MSF-ID-002-02 legal/contable y TD-MSF-ID-002-03 AWS abiertos).
- Declarar `verdict: ready` de Spec Validator antes de la revalidación.
- Handoff a Task Decomposer/Executor sin `verdict: ready` + aprobación humana.
- Operaciones de Git (commits, branches, PRs).

## 10. Preguntas abiertas (a confirmar con el usuario)

- **Q-01:** ¿El `TransportAuthGuard` debe seguir leyendo el token solo del header `Authorization: Bearer`, o también de una cookie (e.g., para integraciones internas)? En este incremento **se mantiene solo header** (alineado con el flujo actual del admin y OpenAPI). Confirmar.
- **Q-02:** ¿La limpieza de cookie en logout debe emitirse ANTES o DESPUÉS del `use case`? Recomendación: **DESPUÉS del éxito** (DEC-03). Si el `use case` falla, no se limpia la cookie (defensa). Confirmar.
- **Q-03:** ¿Este incremento debe coordinarse con `msf-id-rop-sign-cleanup` (planning) para evitar colisión de archivos? Recomendación: ejecutar este primero (más pequeño, solo transporte); luego el otro puede ajustar el controller sin colisión. Confirmar orden o simultaneidad.
- **Q-04:** La "pantalla blanca" reportada en `/` del admin — ¿se reproduce en el navegador del usuario tras las correcciones H1-H4? Si persiste, requiere un fix independiente en el admin (proyecto `merkee-shop-admin`). El síntoma puede tener causas adicionales (e.g., `loading: true` que nunca se resuelve, error de import en `DashboardLayout`, o un loop del interceptor 401). Se solicita al equipo admin verificar tras el fix del backend y, si aplica, abrir un incremento separado en el proyecto admin.
- **Q-05:** ¿El `TransportAuthGuard` debe rechazar también tokens que pertenezcan a una sesión revocada (más allá de la verificación de firma)? Actualmente `JwtPort.verify` solo valida firma/expiración; la revocación se valida en `GetMyProfileUseCase` o implícitamente al usar el `sessionId`. ¿Es aceptable que un JWT firmado pero con sesión revocada pase el guard y falle downstream? Recomendación: **sí, mantener** (defensa en profundidad; coherente con la separación hexagonal). Confirmar.
- **Q-06:** ¿`CORS_ALLOWED_ORIGINS` debe ampliarse para incluir un eventual entorno de staging del admin (e.g., `https://staging.admin.merkee.shop`)? `cors.config.ts` ya es configurable vía env var. Si no aplica, no hacer nada. Confirmar si hay un entorno staging.

## 11. Historial superseded

- (Vacío en este incremento; este es el primer incremento del flujo `msf-admin-real-auth-and-cookie-guard`.)

## 12. Próximo paso

Esperar revalidación focalizada de Spec Validator. Si emite `verdict: ready`, registrarlo en `## Spec Validator Approval` del shared context con los campos exactos (`verdict`, `reviewed_at`, `validator_agent`, `artifact_set_reviewed`, `summary`, `invalidated_by_changes_since: none`). Solicitar aprobación humana del plan (`## Human Plan Approval: approved_by_user`) antes de habilitar handoff a Task Decomposer/Executor. Si Spec Validator emite hallazgos, remediarlos antes del handoff (DEC-04, §Pruebas, §Decomposition Contract).
