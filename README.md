# merkee-workspace

Monorepo de documentación y especificaciones del ecosistema **merkee.shop**, un
supermercado digital para **Colombia** con interfaz en **es-CO** y precios en
**COP** (pesos colombianos, enteros sin decimales).

Este repositorio es el **workspace de especificación y documentación**. El código
de los tres componentes vive en repositorios/hermanos dentro de `projects/`.

## Repositorios del ecosistema

| Repositorio | Propósito | Enlace |
|---|---|---|
| `merkee-workspace` (este) | Specs, contratos y documentación canónica | https://github.com/cristiansrc/merkee-workspace |
| `merkee-shop-api` | API backend (NestJS, hexagonal/ROP) | https://github.com/cristiansrc/merkee-shop-api |
| `merkee-shop-storefront` | Portal de tienda (React SPA) | https://github.com/cristiansrc/merkee-shop-storefront |
| `merkee-shop-admin` | Panel administrativo (React + Refine SPA) | https://github.com/cristiansrc/merkee-shop-admin |

## Estructura de ramas

Los tres repositorios de código (`merkee-shop-api`, `merkee-shop-storefront`,
`merkee-shop-admin`) siguen el mismo esquema de ramas:

- `main` — rama principal de integración y posible despliegue a producción.
- `qa` — rama de control de calidad y validación.
- `developer` — rama de desarrollo activo (target de las tareas del task board).

El **workspace** (`merkee-workspace`) mantiene únicamente la rama `main` como
fuente de documentación y specs (no usa `qa`/`developer`).

## Commits y push recientes (GitHub)

Últimos commits/push confirmados en los repositorios remotos (no constituyen
declaración de producción estable):

| Repositorio | Commit | Nota |
|---|---|---|
| `merkee-workspace` | `484bbeb` | Documentación / specs. |
| `merkee-shop-api` | `89acf70` | API backend. |
| `merkee-shop-storefront` | `9325ae5` | Portal de tienda. |
| `merkee-shop-admin` | `83fe06a` | Panel administrativo. |

Los hashes corresponden al HEAD remoto más reciente conocido y pueden avanzar.

## Propósito del sistema

merkee.shop es un supermercado online colombiano compuesto por:

- un **portal storefront** (React SPA) para visitantes, clientes y su flujo de
  compra (catálogo, carrito servidor con reserva, checkout Wompi/Mercado Pago,
  órdenes, perfil, recuperación de contraseña);
- un **panel admin** (React + Refine SPA) para administración de catálogo,
  media, stock auditado y lectura de órdenes, con RBAC estricto y sin capacidad
  de compra;
- una **API** (NestJS, monolito modular hexagonal/ROP) que concentra toda la
  lógica de negocio, persistencia (PostgreSQL/Prisma) y la integración con los
  proveedores de pago.

Todos los importes son COP enteros; toda la UI y comunicación al usuario es
`es-CO`. El catálogo se traduce desde el inicio (sin literales en inglés salvo
marcas/términos técnicos).

## Spec Driven Development (SDD)

El workspace sigue desarrollo dirigido por especificación. La **fuente de verdad**
es, en orden de precedencia:

1. `docs/specs/master_spec.md` (Master Spec).
2. `docs/api/openapi.yaml` (contrato del API).
3. `projects/merkee-shop-api/prisma/schema.prisma` + migraciones (contrato Prisma).
4. `docs/specs/architecture-decisions.md` (ADRs).
5. `docs/specs/.working/merkee-shop-foundation-sdd-context.md` (shared context).

**Flujo:** el Planner redacta/actualiza specs e incrementos → el Executor
implementa contra el contrato → el Spec Validator y el Reviewer validan
código↔documentación → la aprobación humana habilita la siguiente tarea. No se
modifica OpenAPI, Prisma ni código fuera de lo autorizado por la spec vigente.

**Estado de validación (revisado 2026-08-17):** la Master Spec está en
`validated-not-executed`. El Spec Validator emitió `ready` sobre el conjunto de
artefactos de ese momento, pero ese veredicto quedó **invalidado** por la
implementación de MSF-ID-003 (password reset + migración 014) y por la
actualización documental del replay de `POST /auth/password-change` (ADR-020);
requiere **revalidación focalizada** antes de cualquier handoff o cierre. La
aprobación de plan humana está recibida y habilita la siguiente tarea documental,
pero **no** constituye declaración de producción lista.

## Documentación y su ubicación

| Ruta | Descripción |
|---|---|
| `docs/Wompi FullStack Test.pdf` | Prueba original (Wompi FullStack Test). *Nota: este agente no pudo parsear el PDF; la documentación se construyó desde los artefactos canónicos derivados de la prueba.* |
| `docs/api/openapi.yaml` | Contrato OpenAPI — fuente de verdad del API (paths, schemas, códigos de error `ApiErrorResponse`). Solo lectura para los incrementos de estabilización. |
| `docs/specs/master_spec.md` | Master Spec — fuente de verdad del sistema (propósito, criterios de aceptación, ROP, identidad, datos, carrito/checkout/pagos, deuda). |
| `docs/specs/requirements/merkee-shop-foundation-requirements-brief.md` | Brief funcional consolidado (actores, permisos, requisitos aprobados, fuera de alcance). |
| `docs/specs/architecture-decisions.md` | ADRs (ADR-001 a ADR-020): SPA/Redux, NestJS hexagonal, Prisma, sesiones, pagos, S3/CloudFront, ECS, reservas, IVA, bootstrap, stock, locking, ROP, idempotencia, password reset, replay. |
| `docs/specs/prisma-migration-contract.md` | Contrato de migraciones Prisma (secuencia 001–014, purga, preflight). |
| `docs/specs/technical_debt.md` | Registro global de deuda técnica (espejo en `projects/merkee-shop-api/docs/specs/technical_debt.md`). |
| `docs/specs/.working/merkee-shop-foundation-sdd-context.md` | Shared context SDD (estado, artefactos canónicos, aprobaciones, hallazgos). |
| `docs/specs/tasks/merkee-shop-foundation-task-board.md` | Task board con tareas, dependencias y evidencia de ejecución. |
| `docs/specs/workspace_changes.md` | Bitácora de cambios de workspace y sincronización de deuda. |
| `docs/specs/increments/` | Incrementos (p. ej. `merkee-shop-foundation-stabilization`, `msf-id-rop-sign-cleanup`). |

## Arquitectura general y límites

```
                 ┌────────────────────┐        ┌────────────────────┐
 Visitante/      │  merkee-shop-       │        │  merkee-shop-       │
 Cliente ───────▶|  storefront (SPA)   │        │  admin (SPA)        │
                 └─────────┬──────────┘        └─────────┬──────────┘
                           │  OpenAPI (cliente gen.)      │  OpenAPI (cliente gen.)
                           │                              │  RBAC admin, solo lectura
                           ▼                              ▼
                 ┌──────────────────────────────────────────────┐
                 │            merkee-shop-api (NestJS)            │
                 │  monolito modular hexagonal / ROP              │
                 │  identity · media · catalog · cart-reservation│
                 │  orders · payments · checkout · admin-query   │
                 └───────────────┬──────────────────────────────┘
                                 │
                    PostgreSQL (Prisma Migrate) · S3 privado (media)
                                 │
                    Wompi / Mercado Pago (tokenizado, webhooks firmados)
```

**Límites:**

- La **API** es la única dueña de la lógica de negocio, persistencia y
  contratos. Los controladores/webhooks son adapters de entrada que validan
  transporte/firma, invocan un caso de uso y mapean `Result` a `ApiErrorResponse`;
  no contienen reglas de negocio ni acceden a Prisma.
- El **storefront** es un cliente delgado del OpenAPI para el flujo de compra de
  visitante/cliente (catálogo, carrito servidor, checkout, órdenes, perfil).
  Redux contiene solo vista derivada; **ningún token ni carrito se persiste en
  navegador**.
- El **admin** es un cliente delgado con RBAC para administración de catálogo,
  media, stock y lectura de órdenes. **El admin no compra**; recibe `403` en
  carrito/checkout/órdenes propias.
- Storefront y admin **no comparten componentes de negocio**; solo comparten los
  tipos del cliente generado desde OpenAPI.
- DAG de módulos: `identity→media→catalog→cart-reservation→orders→payments→checkout`,
  con `checkout→cart-reservation` directa y `admin-query` solo lectura.

## Infraestructura AWS — estado de la infraestructura (un solo ambiente, us-east-1)

> **Línea base histórica (revisado 2026-08-18):** el párrafo siguiente describe el
> estado al cierre de la entrega del lunes 2026-08-18 y la captura de incidente del
> 2026-08-18. **No es el estado vigente.** El estado verificado posterior (2026-08-21)
> — ECS estable, ECR publicado, CORS allowlist + PUT, media real S3+CloudFront
> `images.merkee.shop` + OAC, prefijo `/v1`, JWT guard real, cart guest→cliente —
> está en la sección **Estado de entrega y trabajo postentrega**.

> Estado real al 2026-08-18 (histórico): **AWS estaba configurado** en una cuenta de
> aprendizaje, región `us-east-1`, **un único ambiente**. No se afirmó despliegue
> productivo terminado. Avances: **ECR** y **CI/CD** progresaban (imagen y pipelines
> en construcción/validación). El servicio **ECS** presentó fallos de arranque
> relacionados con **Prisma**, **RDS** y **puertos**; el **Security Group
> ECS→RDS** ya estaba aplicado. **Pendiente en esa fecha:** verificación final de
> `api.merkee.shop` y estabilidad del servicio en **1/1** (running + health check
> `/health`). No se incluían credenciales ni valores de secretos. La
> configuración de infra no invalidaba las specs vigentes (Master Spec en
> `validated-not-executed`); era estado operativo adicional.

| Componente AWS | Propósito | Estado |
|---|---|---|
| **ECS Fargate** | Contenedor de la API | *Histórico 2026-08-18:* task definition `merkee-backend-task` **revision 2** con `taskRole` (`merkee-backend-task-role`), mapeo `secrets` JSON y health check `/health`; servicio `merkee-backend-service` en despliegue/pendiente de verificación. *Verificado 2026-08-21:* servicio estable vía CI/CD, `api.merkee.shop/health` 200, ECS `running=desired=1` (ver §Estado de entrega y trabajo postentrega; commits `7fdb009`, `215b36b`, `932a71a`). |
| **ECR** | Registro de imágenes | *Histórico 2026-08-18:* repositorio `merkee-backend-api` existía; Dockerfile multi-stage no-root con **Prisma** y **OpenSSL**, endpoint `/health`, build local validado; push pendiente (auditoría: 0 imágenes). *Verificado 2026-08-21:* imagen publicada vía CI/CD y servicio ECS estable (ver §Estado de entrega y trabajo postentrega). |
| **RDS PostgreSQL** | Base de datos gestionada | `merkee-db` existe (PostgreSQL). **Riesgo pendiente:** auditoría indicó `PubliclyAccessible=True`; no se afirma corrección (TD-AWS-RDS-PUBLIC — **gate abierto**). |
| **S3 privado + CloudFront/OAC** | Hosting de las SPAs y media | Buckets `merkee-frontend-client` (CloudFront `E32P11SX9DFU82` → `merkee.shop`) y `merkee-frontend-admin` (CloudFront `E119IKP00L5RU` → `admin.merkee.shop`) desplegados. *Verificado 2026-08-21:* bucket media privado + distribución **`images.merkee.shop`** con **OAC** + **CORS S3** (allowlist `merkee.shop`/`admin.merkee.shop`); imágenes resuelven `https://images.merkee.shop/<key>` (commits `91ed871`, `02167cd`). `aws-s3-tickets-images` **no pertenece** al proyecto y queda excluido. |
| **Secrets Manager** | Secretos de aplicación | Secreto `merkee/app` creado y referenciado por la task definition (mapeo JSON `secrets`); no se exponen valores. Nombres de variables inyectadas: ver `projects/merkee-shop-api/README.md` (solo nombres). |
| **CloudWatch** | Logs y métricas | Log group `/ecs/merkee-backend-task` creado. **Pendiente:** alarms/observabilidad no configuradas (TD-AWS-OBSERVABILITY — **gate abierto**). Localmente el bridge de métricas usa `prom-client` (Prometheus). |
| **Route53 / ACM** | DNS y certificados TLS | DNS gestionado en **Spaceship** (no Route53); `api.merkee.shop`, `merkee.shop`, `www.merkee.shop` y `admin.merkee.shop` existen. *Verificado 2026-08-21:* `merkee.shop` redirige 301 a `www.merkee.shop`; `www.merkee.shop` y `admin.merkee.shop` responden 200; `api.merkee.shop/health` 200; `images.merkee.shop` resuelve vía CloudFront OAC. `swagger.merkee.shop` **pendiente** de distribución/origen (TD-AWS-SWAGGER-DNS). ACM wildcard `*.merkee.shop` válido. |
| **IAM / OIDC** | Identidad de carga de trabajo y acceso | Role IAM OIDC de GitHub `merkee-github-actions-deploy` con trust ajustado a subjects GitHub reales (con IDs). Task role `merkee-backend-task-role` creado. Perfil local AWS CLI `merkee` (Agent Toolkit/MCP). |
| **GitHub Actions** | CI/CD | Workflows de API/storefront/admin migrados a OIDC; validación CI antes de deploy. *Verificado 2026-08-21:* runs exitosos post-2026-08-18 (ver §Estado de entrega y trabajo postentrega; commits `7fdb009`/`215b36b`/`932a71a`/`8948426`/`fe0b121`). CI anterior falló por OIDC/permissions y secreto CloudFront vacío; fixes aplicados. No se declara producción lista (TD-AWS-ECS-VALIDATION sigue gate documental). |
| **SNS/SQS/DLQ selectivo** | Efectos desacoplados (outbox, reintentos) | Pendiente de configuración AWS. La lógica de reintentos/idempotencia es local; sin Step Functions (ADR-007). |
| **KMS** | Cifrado en reposo / sobres de secreto | Pendiente de configuración AWS. |

### Problemas y pendientes de despliegue (AWS) — actualizado 2026-08-21

- **Resueltos 2026-08-21:** alineación de puerto **3000** (app/ALB/target group), health
  check `/health` operativo, CORS allowlist con **PUT** (`main.ts` `credentials:true`),
  prefijo `/v1` (`app.setGlobalPrefix('v1')`), `cookie-parser` + `TransportAuthGuard`
  con `JwtPort.verify` + `clearCookie` en logout, media real vía `images.merkee.shop`
  OAC (commits `7fdb009`, `215b36b`, `932a71a`, `91ed871`, `02167cd`).
- **Deuda AWS vigente (gates abiertos, no bloquean desarrollo, sí producción):**
  RDS expuesto públicamente (`PubliclyAccessible=True`, TD-AWS-RDS-PUBLIC),
  `swagger.merkee.shop` pendiente (TD-AWS-SWAGGER-DNS) y **alarmas/observabilidad
  CloudWatch** no configuradas (TD-AWS-OBSERVABILITY). Ver `docs/specs/technical_debt.md`.
- **Verificación fresca requerida:** cualquier afirmación de "producción estable"
  requiere re-verificación en vivo (ECS running/desired, health, logs, dominios,
  CloudFront media) al momento de la lectura — evidencia fechada 2026-08-21 puede
  cambiar.

**Implementado localmente (no requiere AWS):** scheduler diario de purga de
`idempotency_records` como driving adapter cableado al arranque (`setTimeout`,
hora configurable UTC, default `02:00`); métricas Prometheus locales; validación
de firma de webhooks sobre raw body; migraciones Prisma reproducibles; Dockerfile
multi-stage no-root de la API con build local validado.

## Estado de entrega y trabajo postentrega

> **Advertencia de frescura:** la evidencia operativa citada abajo fue verificada el
> **2026-08-21** y es **fechada**; el estado de los servicios en AWS puede cambiar sin
> aviso. No constituye declaración de producción lista. La fuente de verdad del sistema
> sigue siendo `docs/specs/master_spec.md` y `docs/api/openapi.yaml`.

### 1. Estado al entregar (lunes 2026-08-18)

El lunes 2026-08-18 se entregaron los componentes (API NestJS, storefront React SPA,
admin React + Refine SPA) e infraestructura base AWS. **No se afirmó estabilidad final
ni despliegue productivo terminado.** Según la documentación histórica (README
§Infraestructura AWS y `docs/DEPLOYMENT_STATUS.md`, captura 2026-08-18):

- **API local OK:** NestJS + Prisma + tests locales en verde, pero **AWS no operativo**.
- **ECR 0 imágenes:** repositorio `merkee-backend-api` existía **sin imagen publicada**
  (auditoría 2026-08-18: `0 imágenes`, bloqueante).
- **ECS 1/0:** servicio `merkee-backend-service` `desired 1 / running 0`, en
  rollback/circuit breaker (fallos de arranque Prisma/RDS/puertos).
- **Puertos/health desalineados:** app escuchando en **3000** (`main.ts`/`EXPOSE 3000`),
  task definition mapeada a **80**, target group a **8080**; health check del target
  group no apuntaba a `/health`.
- **DNS/CORS/media fake:** `api.merkee.shop` sin CNAME estable al ALB (verificado
  301 a `www.merkee.shop`); **CORS sin allowlist** (bloqueaba `merkee.shop` y
  `admin.merkee.shop`); media con `FakeS3MediaStorageAdapter` (S3 real no configurado,
  imágenes con `url` vacía).
- **Admin mocks forzados:** panel admin con `VITE_USE_MOCKS=true` forzado, desacoplado
  de la API real; pantalla blanca por `TransportAuthGuard` sin verificación JWT
  (`a13625e`/`83fe06a` previos — solo presencia regex `Bearer`).
- **Carrito guest roto:** `TransportAuthGuard` solo regex, `cookie-parser` ausente en
  `package.json`/`main.ts` (`req.cookies` `undefined` en runtime), `actor=null` en
  `GET /me`/`PATCH /me`/`logout`/`password-change`; `POST /cart` y `GET /cart` guest
  devolvían **401/GONE** sin creación de `merkee_cart_session`.
- **Checkout stub:** `POST /checkouts` stub sin snapshots completos (IVA/entrega/total),
  sin transferencia de carrito guest→cliente, sin idempotencia operativa ni validación
  de `CHECKOUT_PENDING` vs `ACTIVE`.
- **Imágenes url vacía:** `GET /v1/products` y `GET /v1/categories` devolvían
  `images[].url = ""` (sin resolución S3→CloudFront); `PUT /cart/items/{productId}`
  sin soporte completo.
- **Sesión 10m:** tanto reservas `ACTIVE`/carrito como sesiones autenticadas
  expiraban a **10 minutos** (sin distinción guest 10m vs cliente 30m).
- **Categorías sin relación:** `adminUpdateProduct`/`adminCreateProduct` perdía la
  relación `category` (faltaba `include` Prisma → `category: null` en respuesta).
- **Banners desactivados:** listado de banners sin filtro `active`/`display_order`,
  toggle `active` incompleto y sin paginación consistente.

### 2. Trabajo postentrega verificado el 2026-08-21

Trabajo de estabilización y cierre de incidentes ejecutado y verificado el **2026-08-21**
(evidencias fechadas; pueden cambiar). **No se declara producción lista** — ver gates
abiertos abajo.

- **ECS estable + ECR publicado:** servicio `merkee-backend-service` estable vía CI/CD,
  `api.merkee.shop/health` responde **200** (`{"status":"ok"}` sin auth ni BD);
  imagen publicada en `merkee-backend-api` (continúa lo verificado 2026-08-19/20). Commits
  base ya visibles: `7fdb009` (allow frontend origins with credentials), `215b36b`
  (expose routes under `v1` prefix).
- **CORS allowlist + PUT:** `main.ts` con `origin` allowlist (`merkee.shop`,
  `www.merkee.shop`, `admin.merkee.shop`, `localhost`) + `credentials:true` +
  `methods` incluyendo **PUT** (fix `PUT /cart/items/{productId}` y preflight storefront).
  Ver `932a71a` (enable real session authentication) y `7fdb009`.
- **Media real S3+CloudFront `images.merkee.shop` + OAC + CORS S3:** bucket media
  privado + distribución CloudFront `images.merkee.shop` con OAC, CORS S3
  (`GET/PUT` desde orígenes allowlist), `BucketOwnerEnforced` sin ACL. Imágenes ahora
  resuelven `https://images.merkee.shop/<key>` en lugar de `url` vacía. Commits
  verificados: `91ed871` (remove ACL from bucket-owner-enforced uploads),
  `02167cd` (resolve image URLs through CloudFront).
- **Prefijo `/v1`:** routing del API bajo `app.setGlobalPrefix('v1')` conforme a
  `servers.url` del OpenAPI (`https://api.merkee.shop/v1`). Commit `215b36b`.
- **JWT guard real + cookie-parser + clearCookie:** `cookie-parser` añadido a
  `package.json`/`main.ts` (`app.use(cookieParser())`), `TransportAuthGuard`
  reescrito para invocar `JwtPort.verify` → `Result<JwtPayload, DomainError>` y
  asignar `req.user`; `POST /v1/auth/logout` emite `clearCookie` (`Max-Age=0`)
  para `merkee_refresh_session`. Commits `932a71a`, `9e3ad3e` (persist session
  activity timestamp).
- **Register cliente `must_change_password=false`:** `POST /v1/auth/register`
  público crea exclusivamente `cliente` con `must_change_password=false` (no `true`
  como admin), alineado con `operation-map.ts`. Commit `f62cee4` (expose public
  customer registration) + `57c95b1` (align specs with cart transfer contracts).
- **Cart guest→cliente transfer + checkout real:** `POST /v1/auth/login` promueve
  carrito guest→cliente (transfiere items/reservas del `merkee_cart_session` al
  carrito del cliente en una única transacción); `POST /v1/checkouts` acepta
  `guest_session_id` (cookie o body) y convierte `ACTIVE→CHECKOUT_PENDING` con
  snapshots `items_subtotal_cop` + `delivery_fee_cop:5000` + `iva_cop` HALF_UP.
  Commits `9df9187` (support anonymous guest sessions), `400a711` (support guest
  checkout and complete item details), `8948426` (enable guest cart transfer for
  checkout), `fe0b121` (complete cart transfer and payment checkout contracts),
  `57c95b1`.
- **Cart session 30m inactividad:** toda acción válida de usuario autenticado
  (admin/cliente) renueva su sesión a **30 minutos** de inactividad (antes 10m);
  reservas `ACTIVE`/carrito siguen a **10m**; invitados mantienen 10m; JWT de
  acceso sigue ≤10 min solo en memoria. Webhooks no renuevan. Commit `580ff8f`
  (extend inactivity session to 30 minutes) + `9e3ad3e` (`last_active_at`).
- **CloudFront OAC media + S3 CORS:** distribución media con OAC verificada y
  política de bucket permitiendo solo CloudFront; CORS de bucket permite
  `GET/PUT` desde dominios allowlist. Ya incluido en `91ed871`/`02167cd`.
- **Workspace `openapi.yaml` guest anonymous:** paths `/cart` (`GET`), `/cart/items`
  (`POST`/`PUT`/`DELETE`) declaran `security: [{bearerAuth: []}, {cartSessionCookie: []}, {}]` —
  documenta sesión anónima que crea `merkee_cart_session` vía `Set-Cookie` cuando
  no existe (commit `d4b438f` workspace + `9df9187`/`57c95b1` en API).
- **Eliminación soft-delete operativa + imágenes resueltas:** `DELETE /admin/products/{id}`
  y `DELETE /admin/categories/{id}` con soft-delete real (no stub), bloqueo de
  categoría con productos activos → 409, reconstrucción de imágenes con URL
  CloudFront. Commits `2601833` (preserve product category relationship),
  `02167cd`.
- **Auth admin/storefront:** admin conecta a API real (`VITE_USE_MOCKS=false`,
  `api.merkee.shop/v1`), pantalla blanca corregida (`6330489` mount router before
  refine, `aec6283` use production api instead of forced mocks), storefront con
  `refresh` silencioso y preservación de auth (`04cdeaf` storefront), `recover
  expired sessions` (`0090288`), `logout` (`a82f8d5`), `checkout continue`
  (`31074e5`/`b37b280`/`acd8cbc` send guest session on auth), `b7febd7` admin
  preserve product category selection.
- **Banners toggle:** `adminCreateBanner`/`adminUpdateBanner` con `active` +
  `display_order` + `target_path`, toggle operativo y `GET /banners` filtra `active`
  por `display_order` (ver `technical_debt.md` pendiente de gate legal/observabilidad,
  no de banners).
- **CI runs verificados:** workflows GitHub Actions (API/storefront/admin) con runs
  exitosos post-2026-08-18 (OIDC `merkee-github-actions-deploy`). Evidencia fechada
  2026-08-21, puede cambiar:
  - API: https://github.com/cristiansrc/merkee-shop-api/actions — commits
    `7fdb009`, `215b36b`, `932a71a`, `9e3ad3e`, `91ed871`, `02167cd`, `2601833`,
    `580ff8f`, `9df9187`, `400a711`, `f62cee4`, `9faad46`, `8948426`, `fe0b121`,
    `57c95b1`
  - Storefront: https://github.com/cristiansrc/merkee-shop-storefront/actions —
    `b37b280`, `0090288`, `a82f8d5`, `acd8cbc`
  - Admin: https://github.com/cristiansrc/merkee-shop-admin/actions — `6330489`,
    `aec6283`, `b7febd7`

### 3. Estado actual honesto (2026-08-21)

El sistema **no** está declarado listo para producción. Lo verificado hasta ahora
(fresco al 2026-08-21):

- **Frontends renderizan:** storefront (`merkee.shop` 301→`www.merkee.shop` 200) y
  admin (`admin.merkee.shop` 200) cargan vía CloudFront/S3 con SPA fallback.
- **API pública:** `GET https://api.merkee.shop/health` **200**; `GET /v1/categories`
  y `GET /v1/products` **200** con imágenes resueltas a `https://images.merkee.shop/<key>`
  (no más `url` vacía), categorías con relación preservada.
- **Auth real:** `POST /v1/auth/register` (cliente `must_change_password=false`),
  `POST /v1/auth/login` (guest→cliente transfer), `POST /v1/auth/refresh` (cookie
  `HttpOnly`), `POST /v1/auth/logout` (`clearCookie`), `TransportAuthGuard` con
  `JwtPort.verify` y `cookie-parser`.
- **Carrito/checkout:** guest anónimo crea `merkee_cart_session` (`GET /cart` sin
  cookie → 200 + `Set-Cookie`), `PUT /cart/items/{productId}` operativo con
  `Idempotency-Key`, sesión autenticada renueva a **30m** (guest 10m), checkout con
  guest transfer y `delivery_fee_cop:5000` + `iva_cop` HALF_UP.
- **Pendiente de validación E2E completa:** pagos Wompi/Mercado Pago (webhooks
  firmados + reconciliación 15m) y órdenes requieren validación E2E contra la API
  desplegada con datos poblados (seed productivo pendiente; catálogo dummy local
  6 categorías/15 productos/3 banners no equivale a producción).

**Gates abiertos antes de declarar producción** (no bloquean desarrollo, sí producción):
`PubliclyAccessible=True` de RDS `merkee-db` (TD-AWS-RDS-PUBLIC), hardening HTTP de
borde (helmet/CSP/HSTS/nosniff, CSRF Origin/double-submit, rate limiting —
TD-NEW-HTTP-SEC; **CORS allowlist + PUT ya habilitado en postentrega**, ver §Seguridad),
observabilidad/CloudWatch alarms (TD-AWS-OBSERVABILITY), email productivo (reemplazo
de `NoopEmailAdapter`), proveedores de pago reales (reemplazo de
`FakePaymentProviderAdapter` — lógica de reintentos/refunds es local), retención/
anonimización pendiente de revisión legal/contable (TD-MSF-ID-002-02), decisión
operativa AWS del scheduler (TD-MSF-ID-002-03) y swagger DNS
(TD-AWS-SWAGGER-DNS). Véase `docs/specs/technical_debt.md`. **Nada se declara
producción lista.**

### 4. Enlaces y advertencia de frescura

- Workflows (evidencia fechada **2026-08-21**, puede cambiar):
  - API: https://github.com/cristiansrc/merkee-shop-api/actions
  - Storefront: https://github.com/cristiansrc/merkee-shop-storefront/actions
  - Admin: https://github.com/cristiansrc/merkee-shop-admin/actions
- Estado de despliegue: `docs/DEPLOYMENT_STATUS.md` (trazabilidad completa:
  captura histórica 2026-08-18 + estado verificado 2026-08-21; el incidente ECS
  allí descrito quedó resuelto según la verificación 2026-08-21).
- **La evidencia operativa es fechada y puede cambiar.** Cualquier afirmación de
  "producción estable" requiere verificación en vivo renovada (ECS running/desired,
  health check, logs, dominios, CloudFront media) en el momento de la lectura.

## Seguridad

- **Pagos tokenizados:** el PAN/CVV/fecha de tarjeta **nunca** se aceptan ni
  almacenan. El pago se tokeniza/hospeda en el proveedor (Wompi / Mercado Pago);
  el webhook es autoritativo y su firma se valida sobre el raw body antes de
  persistir. (No se incluyen credenciales ni secretos del PDF de la prueba.)
- **Sesiones:** JWT de acceso ≤10 min **solo en memoria**; refresh/cart token
  opaco `HttpOnly; Secure; SameSite=Lax`, rotado. Argon2id para contraseñas.
- **RBAC:** roles `admin` / `cliente` exclusivos; el admin no accede a carrito,
  checkout ni compra; provisión de admin solo por admin con contraseña ya
  cambiada, con token de activación opaco de un uso.
- **ROP:** todo caso de uso devuelve `Result<Success, DomainError>`; los
  errores de negocio no lanzan excepciones; `TECHNICAL_DEPENDENCY_FAILURE` (500)
  no revela causa/PII.
- **Pendiente (gate antes de producción):** protecciones HTTP de borde (helmet/CSP/HSTS/
  nosniff, CSRF Origin/double-submit, rate limiting de
  login/registro/reset/activación) aún no aplicadas en `main.ts` (TD-NEW-HTTP-SEC).
  **CORS allowlist + PUT:** habilitado en el trabajo postentrega **2026-08-21**
  (`7fdb009`/`932a71a`: allowlist `merkee.shop`/`www`/`admin` + `credentials:true` +
  `PUT`; media CORS S3 `91ed871`) — permitió conectar admin/storefront a la API real
  sin mocks y habilitar `PUT /cart/items/{productId}`; el registro de deuda
  `TD-NEW-HTTP-SEC` aún lista el resto del hardening HTTP y requiere reconciliación
  documental (no se edita aquí por lifecycle del registro). El resto del hardening
  de borde sigue pendiente.
- **Sesión postentrega:** toda acción válida autenticada renueva **30m inactividad**
  (`580ff8f`/`9e3ad3e`), guest y reservas `ACTIVE`/carrito 10m, JWT ≤10 min.

## Estado real (no es producción lista)

El sistema **no** está listo para producción. Gates abiertos antes de producción:

- **TD-MSF-ID-002-02:** revisión legal/contable de retención y anonimización
  pendiente.
- **TD-MSF-ID-002-03:** decisión operativa AWS (coordinar o reemplazar el
  scheduler, ownership, alarmas, prevención de doble ejecución) pendiente.
- **TD-NEW-HTTP-SEC:** protecciones HTTP de borde pendientes.
- **Módulos de la API:** `identity`, `catalog` (incl. stock auditado),
  `media`, `cart-reservation`, `orders`, `payments`, `checkout` y `admin-query`
  tienen implementación local funcional (casos de uso ROP + adapters + pruebas).
  **Pendientes de producción:** email noop, `FakeS3MediaStorageAdapter` y
  `FakePaymentProviderAdapter` en dev (AWS/proveedores no configurados),
  `CartReservationPort` noop en identity (TD-MSF-API-001) y protecciones HTTP de
  borde (TD-NEW-HTTP-SEC). No se declara producción lista.
- Revalidación focalizada de Spec Validator pendiente tras MSF-ID-003.

## Medición de cobertura del API (local)

Última medición local confirmada por el executor (`npm run test:cov`, exit code
0). No corresponde a un reporte de CI externo ni a un entorno de producción:

- **125 suites / 1232 tests PASS**
- **Statements:** 93.36%
- **Branches:** 84.43%
- **Functions:** 93.01%
- **Lines:** 93.57%

El detalle por módulo y la evidencia están en
`projects/merkee-shop-api/README.md`. Esta medición **no altera** el estado de
"no producción lista" del sistema (ver sección anterior). Los portales
storefront y admin mantienen su cobertura completa independiente pendiente de
medición (no se afirman porcentajes para ellos).

## Cómo navegar

- Para el API: ver `projects/merkee-shop-api/README.md`.
- Para el portal: ver `projects/merkee-shop-storefront/README.md`.
- Para el panel: ver `projects/merkee-shop-admin/README.md`.
- **Estado de despliegue AWS e incidente actual:** ver `docs/DEPLOYMENT_STATUS.md`
  (avances, AWS configurado, acciones realizadas, incidente ECS, próximos pasos
  exactos, seguridad y enlaces de ramas). No se afirma producción estable.
- **Auditoría AWS (no versionada):** `AUDITORIA_AWS_MERKEE.md` documenta el estado
  de la cuenta AWS; **no está versionada** en el repositorio por sensibilidad
  (referencias a recursos/identificadores de cuenta). No se incluye su contenido
  aquí ni se afirma su estado.

