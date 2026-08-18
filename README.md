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

Cada repositorio sigue el mismo esquema de ramas:

- `main` — rama principal de integración y posible despliegue a producción.
- `qa` — rama de control de calidad y validación.
- `developer` — rama de desarrollo activo (target de las tareas del task board).

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

## Infraestructura AWS propuesta (un solo ambiente)

> Estado real: **AWS no está configurado**. Lo siguiente es la propuesta
> arquitectónica (ADR-006/007) y su grado de avance local.

| Componente AWS | Propósito | Estado |
|---|---|---|
| **ECS Fargate** | Contenedor de la API y (opcional) jobs | Pendiente de configuración AWS. Localmente la API corre como proceso NestJS (`npm run start:dev` / `node dist/main`). |
| **RDS PostgreSQL** | Base de datos gestionada | Pendiente de configuración AWS. Localmente se usa PostgreSQL en Docker vía Prisma Migrate. |
| **S3 privado + CloudFront/OAC** | Hosting de las SPAs y media privada | Decisiones fijadas (ADR-006) pero **no configurado**. Buckets privados, solo CloudFront lee por OAC; CSP/CORS allowlist. |
| **Secrets Manager** | `JWT_SECRET`, `INITIAL_ADMIN_PASSWORD`, credenciales | Pendiente de configuración AWS. Localmente se inyectan vía `.env` (no versionado). |
| **CloudWatch** | Logs y métricas | Pendiente de configuración AWS. Localmente el bridge de métricas usa `prom-client` (Prometheus) y está cableado para exponer las métricas canónicas; el destino CloudWatch no está conectado. |
| **SNS/SQS/DLQ selectivo** | Efectos desacoplados (outbox, reintentos) | Pendiente de configuración AWS. La lógica de reintentos/idempotencia es local; sin Step Functions (ADR-007). |
| **Route53 / ACM** | DNS y certificados TLS | Pendiente de configuración AWS. |
| **KMS** | Cifrado en reposo / sobres de secreto | Pendiente de configuración AWS. |
| **IAM / OIDC** | Identidad de carga de trabajo y acceso | Pendiente de configuración AWS. |

**Implementado localmente (no requiere AWS):** scheduler diario de purga de
`idempotency_records` como driving adapter cableado al arranque (`setTimeout`,
hora configurable UTC, default `02:00`); métricas Prometheus locales; validación
de firma de webhooks sobre raw body; migraciones Prisma reproducibles.

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
- **Pendiente (gate antes de producción):** protecciones HTTP (helmet/CSP/HSTS/
  nosniff, CORS allowlist, CSRF Origin/double-submit, rate limiting de
  login/registro/reset/activación) aún no aplicadas en `main.ts` (TD-NEW-HTTP-SEC).

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

## Pendientes de decisión

- **PDF no legible por el agente:** `docs/Wompi FullStack Test.pdf` no pudo ser
  parseado por el modelo. La documentación se basó en los artefactos canónicos
  derivados de la prueba (requirements brief, master spec, ADRs, task board,
  deuda técnica). Se recomienda revisar manualmente el PDF para confirmar
  cualquier requisito no reflejado en dichos artefactos.
- **Deuda TD-MSF-API-005 reconciliada (2026-08-18):** el registro indicaba que
  storefront/admin "contienen únicamente README y .gitignore"; en disco ambos
  tienen código, `package.json`, `node_modules` y `dist`. La entrada pasó a
  `HISTÓRICO/SUPERSEDED` en `docs/specs/technical_debt.md` y su espejo local.
- **`.env.example` del admin:** apunta a `https://api.merkee.shop/vite`; parece
  un typo de `/v1` (el prefijo canónico del server OpenAPI es `/v1`). Confirmar
  antes de usar.
