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
> estado al cierre de la entrega del lunes 2026-08-17 y la captura de incidente del
> 2026-08-18. **No es el estado vigente.** El estado verificado posterior (2026-08-20)
> — ECS estable, `api.merkee.shop/health` 200, dominios resueltos, CORS y `/v1`
> corregidos — está en la sección **Estado de entrega y trabajo postentrega**.

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
| **ECS Fargate** | Contenedor de la API | *Histórico 2026-08-18:* task definition `merkee-backend-task` **revision 2** con `taskRole` (`merkee-backend-task-role`), mapeo `secrets` JSON y health check `/health`; servicio `merkee-backend-service` en despliegue/pendiente de verificación. *Verificado 2026-08-20:* servicio estable vía CI/CD, `api.merkee.shop/health` 200 (ver sección **Estado de entrega y trabajo postentrega**). |
| **ECR** | Registro de imágenes | *Histórico 2026-08-18:* repositorio `merkee-backend-api` existía; Dockerfile multi-stage no-root con **Prisma** y **OpenSSL**, endpoint `/health`, build local validado; push pendiente. *Verificado 2026-08-20:* imagen publicada vía CI/CD y servicio ECS estable (ver sección **Estado de entrega y trabajo postentrega**). |
| **RDS PostgreSQL** | Base de datos gestionada | `merkee-db` existe (PostgreSQL). **Riesgo pendiente:** auditoría indicó `PubliclyAccessible=True`; no se afirma corrección (TD-AWS-RDS-PUBLIC). |
| **S3 privado + CloudFront/OAC** | Hosting de las SPAs y media | Buckets `merkee-frontend-client` (CloudFront `E32P11SX9DFU82` → `merkee.shop`) y `merkee-frontend-admin` (CloudFront `E119IKP00L5RU` → `admin.merkee.shop`) desplegados. `aws-s3-tickets-images` **no pertenece** al proyecto y queda excluido. |
| **Secrets Manager** | Secretos de aplicación | Secreto `merkee/app` creado y referenciado por la task definition (mapeo JSON `secrets`); no se exponen valores. Nombres de variables inyectadas: ver `projects/merkee-shop-api/README.md` (solo nombres). |
| **CloudWatch** | Logs y métricas | Log group `/ecs/merkee-backend-task` creado. **Pendiente:** alarms/observabilidad no configuradas (TD-AWS-OBSERVABILITY). Localmente el bridge de métricas usa `prom-client` (Prometheus). |
| **Route53 / ACM** | DNS y certificados TLS | DNS gestionado en **Spaceship** (no Route53); `api.merkee.shop`, `merkee.shop`, `www.merkee.shop` y `admin.merkee.shop` existen. *Verificado 2026-08-20:* `merkee.shop` redirige 301 a `www.merkee.shop`; `www.merkee.shop` y `admin.merkee.shop` responden 200; `api.merkee.shop/health` 200. `swagger.merkee.shop` **pendiente** de distribución/origen (TD-AWS-SWAGGER-DNS). ACM wildcard `*.merkee.shop` válido. |
| **IAM / OIDC** | Identidad de carga de trabajo y acceso | Role IAM OIDC de GitHub `merkee-github-actions-deploy` con trust ajustado a subjects GitHub reales (con IDs). Task role `merkee-backend-task-role` creado. Perfil local AWS CLI `merkee` (Agent Toolkit/MCP). |
| **GitHub Actions** | CI/CD | Workflows de API/storefront/admin migrados a OIDC; validación CI antes de deploy. CI anterior falló por OIDC/permissions y secreto CloudFront vacío; fixes aplicados, pero **no se afirma** que el deploy final terminó (TD-AWS-ECS-VALIDATION). |
| **SNS/SQS/DLQ selectivo** | Efectos desacoplados (outbox, reintentos) | Pendiente de configuración AWS. La lógica de reintentos/idempotencia es local; sin Step Functions (ADR-007). |
| **KMS** | Cifrado en reposo / sobres de secreto | Pendiente de configuración AWS. |

### Problemas y pendientes de despliegue (AWS)

- **Alineación de puerto:** API, ALB y target group deben coincidir en el puerto
  **3000** (fallos previos de ECS por desajuste de puertos).
- **Health check:** confirmar que el endpoint `/health` responde y está
  configurado en la target group / task definition.
- **Verificación ECS:** confirmar servicio en estado **1/1** (running estable)
  y `api.merkee.shop` accesible.
- **Deuda AWS:** RDS expuesto públicamente (`PubliclyAccessible=True`),
  distribución DNS de `swagger.merkee.shop` pendiente y **alarmas/observabilidad
  CloudWatch** no configuradas (TD-AWS-RDS-PUBLIC, TD-AWS-SWAGGER-DNS,
  TD-AWS-OBSERVABILITY).

**Implementado localmente (no requiere AWS):** scheduler diario de purga de
`idempotency_records` como driving adapter cableado al arranque (`setTimeout`,
hora configurable UTC, default `02:00`); métricas Prometheus locales; validación
de firma de webhooks sobre raw body; migraciones Prisma reproducibles; Dockerfile
multi-stage no-root de la API con build local validado.

## Estado de entrega y trabajo postentrega

> **Advertencia de frescura:** la evidencia operativa citada abajo fue verificada el
> **2026-08-20** y es **fechada**; el estado de los servicios en AWS puede cambiar sin
> aviso. No constituye declaración de producción lista. La fuente de verdad del sistema
> sigue siendo `docs/specs/master_spec.md` y `docs/api/openapi.yaml`.

### 1. Estado al entregar (lunes 2026-08-17)

El lunes 2026-08-17 se entregaron los componentes (API NestJS, storefront React SPA,
admin React + Refine SPA) e infraestructura base AWS. **No se afirmó estabilidad final
ni despliegue productivo terminado.** Según la documentación histórica (README
§Infraestructura AWS y `docs/DEPLOYMENT_STATUS.md`, captura 2026-08-18):

- El servicio **ECS** presentaba fallos de arranque (Prisma/RDS/puertos) y estaba en
  despliegue/pendiente de verificación (`Running` no confirmado en `1/1`).
- **ECR** tenía la imagen pendiente de push/verificación.
- **Dominios** (`api.merkee.shop`, `admin.merkee.shop`, `merkee.shop`) existían pero su
  resolución y estabilidad estaban pendientes de verificación; el despliegue de
  storefront/admin en CloudFront/S3 estaba reportado como desplegado sin validación de
  extremo a extremo.
- **CORS** y el **prefijo `/v1`** del API figuraban como pendientes/correctivos en
  trabajo posterior.
- El **admin** dependía de datos/mocks de desarrollo y no estaba conectado a la API real
  de producción en ese momento.

### 2. Trabajo postentrega verificado el 2026-08-20

Trabajo de estabilización y cierre de incidentes ejecutado y verificado el 2026-08-20
(evidencias fechadas; pueden cambiar):

- **API ECS estable vía CI/CD:** el servicio `merkee-backend-service` quedó estable por
  el pipeline de GitHub Actions; `api.merkee.shop/health` responde **200**.
- **Storefront y admin en CloudFront/S3:** distribuciones CloudFront sobre buckets S3
  para el portal (`merkee.shop` / `www.merkee.shop`) y el panel (`admin.merkee.shop`).
- **DNS y routing:** `merkee.shop` redirige **301** a `www.merkee.shop`; `www.merkee.shop`
  y `admin.merkee.shop` responden **200**. SPA fallback configurado en ambas SPAs.
- **Corrección pantalla blanca del admin:** se corrigió el fallo de render (pantalla
  blanca) del panel administrativo.
- **Admin conectado a la API real sin mocks:** el panel admin consume la API desplegada
  (`api.merkee.shop`) con datos reales, sin adaptadores mock.
- **CORS configurado:** el API acepta los orígenes del storefront/admin (habilita la
  comunicación real frontend↔API).
- **Prefijo `/v1` corregido:** el routing del API usa el prefijo `/v1` conforme al
  `servers.url` del OpenAPI (`https://api.merkee.shop/v1`).
- **Workflows GitHub Actions exitosos:** los pipelines de API y admin finalizaron con
  éxito. Commits relevantes del trabajo postentrega:
  - `6330489` (API)
  - `aec6283` (API)
  - `7fdb009` (admin)
  - `215b36b` (admin)

  Enlaces a los workflows (evidencia fechada, puede cambiar):
  - API: https://github.com/cristiansrc/merkee-shop-api/actions
  - Storefront: https://github.com/cristiansrc/merkee-shop-storefront/actions
  - Admin: https://github.com/cristiansrc/merkee-shop-admin/actions

### 3. Estado actual honesto (2026-08-20)

El sistema **no** está declarado listo para producción. Lo verificado hasta ahora:

- El **frontend renderiza** (storefront y admin cargan y responden 200).
- La **API pública** (`/v1/categories`, `/v1/products`) responde **200** pero devuelve
  **listas vacías** (sin datos sembrados/seed productivo ni catálogo poblado).
- El **login real** requiere credenciales válidas; no hay credenciales de prueba
  públicas ni seed de usuario cliente/admin poblado.
- **Pendiente de validación funcional completa:** flujo de sesiones, carrito servidor,
  checkout, pagos (Wompi/Mercado Pago) y órdenes no han sido validados de extremo a
  extremo contra la API desplegada.
- **Bugs conocidos abiertos:** comportamiento de sesiones, carrito y pagos requiere
  verificación/corrección; el seed de datos (catálogo, usuario admin/cliente) está
  pendiente.

**Gates abiertos antes de declarar producción** (no bloquean desarrollo, sí producción):
RDS expuesta públicamente (`PubliclyAccessible=True`, TD-AWS-RDS-PUBLIC), hardening HTTP
de borde (helmet/CSP/HSTS/nosniff, CSRF, rate limiting — TD-NEW-HTTP-SEC; CORS ya
habilitado en postentrega, ver §Seguridad), observabilidad/CloudWatch alarms
(TD-AWS-OBSERVABILITY), email productivo (reemplazo de `NoopEmailAdapter`), proveedores
de pago reales (reemplazo de `FakePaymentProviderAdapter`), retención/anonimización
pendiente de revisión legal/contable (TD-MSF-ID-002-02), decisión operativa AWS del
scheduler (TD-MSF-ID-002-03) y swagger DNS (TD-AWS-SWAGGER-DNS). Véase
`docs/specs/technical_debt.md`.

### 4. Enlaces y advertencia de frescura

- Workflows (evidencia fechada 2026-08-20, puede cambiar):
  - API: https://github.com/cristiansrc/merkee-shop-api/actions
  - Storefront: https://github.com/cristiansrc/merkee-shop-storefront/actions
  - Admin: https://github.com/cristiansrc/merkee-shop-admin/actions
- Estado de despliegue histórico: `docs/DEPLOYMENT_STATUS.md` (captura 2026-08-18, marcada
  como línea base histórica; el incidente ECS allí descrito quedó resuelto según la
  verificación 2026-08-20).
- **La evidencia operativa es fechada y puede cambiar.** Cualquier afirmación de
  "producción estable" requiere verificación en vivo renovada (ECS running/desired,
  health check, logs, dominios) en el momento de la lectura.

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
  **CORS allowlist:** habilitado en el trabajo postentrega (2026-08-20) para aceptar los
  orígenes del storefront/admin, lo que permitió conectar el admin a la API real sin
  mocks; el registro de deuda `TD-NEW-HTTP-SEC` aún lista "sin CORS allowlist" y requiere
  reconciliación documental (no se edita aquí por lifecycle del registro). El resto del
  hardening HTTP de borde sigue pendiente.

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

