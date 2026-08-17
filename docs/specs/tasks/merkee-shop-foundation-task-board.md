# Task Board — merkee-shop-foundation

## Control

- **Estado:** `awaiting-human-plan-approval`
- **Rama objetivo:** `developer` (API, Storefront y Admin)
- **Spec canónica:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` (`awaiting-human-plan-approval`)
- **Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md`
- **Contratos:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`, `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md`, `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md`
- **Regla de ejecución:** no editar OpenAPI; no inventar endpoints; aplicar ROP y el catálogo `DomainError` de la Master Spec. Los paths indicados son áreas probables porque los tres repositorios contienen únicamente documentación inicial.
- **Spec validation:** `pending` / `revision-needed`. Las migraciones 010–013, las correcciones ROP y la precisión posterior de la clasificación exclusiva de purga (`minimum_age_not_elapsed` → `replay_active` → `retention_not_elapsed` → `operation_pending` → `eligible`) invalidan el `ready` histórico, que queda **superseded**. La evidencia histórica de tarea `done` y la aprobación humana histórica se conservan solo como trazabilidad: no autorizan nuevo handoff ni continuar con la siguiente tarea. Requiere revalidación focalizada de Spec Validator para MSF-ID-002 antes de desbloquear el tablero. **Dependencia del incremento de estabilización (`merkee-shop-foundation-stabilization`):** el desbloqueo de este tablero depende de que la delta de estabilización complete las correcciones de código pendientes STAB-B1 (ROP activate-admin), STAB-B2 (`must_change_password` replay `true`), STAB-B3 (JWT fail-fast) y STAB-B4 (`operation-map.ts` debe reflejar el `404` añadido a `provisionAdminUser` en OpenAPI). Los drifts STAB-DRIFT-01 (404 de provision) y STAB-DRIFT-02 (webhook duplicate semantics) están **resueltos por decisión del usuario (Opción A, 2026-08-16)** y aplicados en disco (OpenAPI + Master Spec §ROP); son decisiones históricas/resueltas y ya no son `decision-required` ni bloquean por decisión pendiente. Hasta que se completen B1-B4 y Spec Validator emita veredicto `ready`, el tablero permanece `blocked`. **Último cambio de código (`idempotency_responsejson_rop_purge_note` 2026-08-16, MSF-ID-002):** corrigió el drift del nombre canónico `responseJson`/`response_json` en `domain/ports/idempotency.port.ts`, `infrastructure/adapters/prisma-idempotency.adapter.ts` y `application/use-cases/provision-admin-user.use-case.ts` (Blocker 1) y alineó `PurgeIdempotencyRecordsUseCase` al patrón ROP estricto de Master Spec §ROP / ADR-017 — `execute()` ahora devuelve `Promise<Result<void, DomainError>>` y el `try/catch` técnico se eliminó de `application`. Sin cambios OpenAPI, sin migraciones nuevas, sin tocar otros módulos. **Este cambio también invalida el veredicto vigente y exige una nueva revalidación focalizada de Spec Validator antes de cualquier `ready`/handoff**; no declarar `ready` mientras `verdict: pending` y las aprobaciones previas son **superseded** (no autorizan continuar).
- **Evidencia MSF-ID-002:** `done`; la evidencia vigente única registra 34 suites/249 tests, build, dependency-cruiser, `prisma validate`, `prisma migrate deploy` de la secuencia 007–013, diff sin drift e integración PostgreSQL. El conjunto aplicado es `007_idempotency_records`, `008_idempotency_records_purge_index`, `009_idempotency_records_response_json_rename`, `010_idempotency_records_response_json_backfill`, `011_idempotency_records_response_json_normalize`, `012_idempotency_records_response_json_validate` y `013_idempotency_records_response_json_strict_validate`; el snapshot vigente contiene exactamente `resource_id`, `status`, `activation_expires_at` y `body_hash`. **HISTÓRICO/SUPERSEDED:** toda evidencia de snapshot de tres campos o de `migrate deploy` con 8 migraciones.
- **Deuda técnica sincronizada:** TD-MSF-ID-002-01/02/03 está `active` en los registros global y local. No bloquea desarrollo local ni altera el bloqueo actual del tablero, que sigue siendo la revalidación documental. TD-01 conserva scopes desconocidos como `operation_pending` y no inventa scopes; TD-02 requiere evidencia legal/contable de retención/anonimización antes de producción; TD-03 reconoce scheduler local y métricas productivas implementados, mientras AWS no está configurado y debe definir coordinación o reemplazo, ownership, alarmas y prevención de doble ejecución antes de producción si AWS participa. Ninguna de las tres deudas se declara resuelta sin la evidencia de cierre registrada en `technical_debt.md`.
- **Blocked:** todas las tareas no ejecutadas permanecen `blocked` exclusivamente por la revalidación documental `Spec validation pending/revision-needed`; los campos históricos `blocker: none` de tareas bloqueadas no son autorización de ejecución. No existe autorización para Executor, Task Decomposer ni handoff hasta un nuevo veredicto `ready` y la aprobación humana aplicable.

## Tareas

### API — scaffolding NestJS modular hexagonal/ROP + contrato/schema

#### MSF-API-001 — Crear el esqueleto NestJS modular y los boundaries hexagonales
- **agent:** executor
- **spec_refs:** Master Spec §8, §29, §35-42; ADR-002, ADR-013, ADR-017; CA-09, CA-11
- **goal:** dejar ejecutable el esqueleto del API sin reglas de negocio.
- **scope:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src`; configuración NestJS/TypeScript; módulos `identity`, `media`, `catalog`, `cart-reservation`, `orders`, `payments`, `checkout`, `admin-query`.
- **out_of_scope:** endpoints funcionales, Prisma, OpenAPI, AWS y decisiones de arquitectura.
- **inputs:** fuentes canónicas del encabezado; DAG de Master Spec §29.
- **implementation_notes:** separar `domain`, `application`, `infrastructure`; exponer puertos explícitos; controllers como adapters de entrada; `Result<Success, DomainError>` y catálogo estable como tipos compartidos de aplicación/dominio.
- **edge_cases:** imports prohibidos `domain→application|infrastructure` y `application→infrastructure`; ningún `HttpException` en dominio/aplicación.
- **done_criteria:** build base arranca; módulos y boundaries son visibles; no hay acceso Prisma desde controllers.
- **verification:** revisión de árbol/imports y prueba mínima de arranque; no modificar OpenAPI.
- **dependencies:** none
- **handoff_context:** siguiente tarea añade la proyección HTTP y esquemas sin cambiar el contrato.
- **source_of_truth:** Master Spec §35-42 y ADR-017.
- **stale_terms_guard:** no Next.js, Strapi, CQRS, excepciones de negocio ni controllers con Prisma.
- **status:** `done`
- **executor_notes:** Esqueleto NestJS modular hexagonal/ROP creado: 8 módulos (identity, media, catalog, cart-reservation, orders, payments, checkout, admin-query) con boundaries domain/application/infrastructure; puertos explícitos; controllers vacíos; `Result<Success, DomainError>` y catálogo `DomainError` en `src/shared/domain` (TypeScript puro, sin NestJS/Prisma/HTTP); dependency-cruiser bloquea `domain→application|infrastructure` y `application→infrastructure`; prueba de arranque y tests ROP. Sin reglas de negocio, sin endpoints funcionales, sin Prisma, sin OpenAPI modificado, sin secretos.
- **verification_result:** `npm run build` OK; `npm test` 3 suites / 7 tests PASS; `npm run depcruise` "no dependency violations found (56 modules, 72 dependencies)"; arranque real `node dist/main` inicializa los 8 módulos y 8 controllers (HTTP 404 en `/` esperado, sin rutas); cobertura global 64.62% stmts / 62.88% lines (shared/domain 100%; adapters de infraestructura son placeholders "no implementado" que se cubrirán en tareas posteriores). Sin `HttpException` en domain/application; `@nestjs/common` solo en controllers/modules/infrastructure.
- **blocker:** `none`

#### MSF-API-002 — Implementar validación de transporte y proyección ROP a ApiErrorResponse
- **agent:** executor
- **spec_refs:** OpenAPI `components/schemas/ApiErrorResponse` y respuestas; Master Spec §43-58; CA-09, CA-11
- **goal:** centralizar validación sintáctica, autenticación de transporte y mapping de `Result` a HTTP.
- **scope:** adapters HTTP, pipes/guards/interceptors y mapper `DomainError→ApiErrorResponse` en `merkee-shop-api/src`.
- **out_of_scope:** reglas de negocio, acceso a Prisma, nuevos códigos/endpoints y autenticación completa de identidad.
- **inputs:** OpenAPI real; catálogo de códigos de Master Spec §47-58.
- **implementation_notes:** completar `path` y `trace_id` solo en adapter; no revelar causas, secretos, hashes ni PII; reservar 400 para sintaxis y 429 para rate limit.
- **edge_cases:** `TECHNICAL_DEPENDENCY_FAILURE` debe producir 500; código incompatible con el status no debe serializarse.
- **done_criteria:** mapper cubre todos los códigos aprobados y respuestas OpenAPI existentes; controllers futuros pueden invocar un único puerto.
- **verification:** tests unitarios del mapper y tests HTTP de status/code; revisión de que no se lanza excepción como flujo de negocio.
- **dependencies:** MSF-API-001
- **handoff_context:** identity y módulos de dominio reutilizan este adapter sin duplicarlo.
- **source_of_truth:** OpenAPI y Master Spec §43-58.
- **stale_terms_guard:** no inventar códigos ni endpoints; no `HttpException` desde application/domain.
- **status:** `done`
- **executor_notes:** Capa HTTP compartida creada en `merkee-shop-api/src/shared/http` (MSF-API-002): `api-error-response.ts` (tipos `ApiErrorResponse`/`ApiErrorDetail` alineados a OpenAPI + type guard), `domain-error-mapper.ts` (mapper puro `DomainError→{status,error,code,message,details}` con tabla canónica código→status de Master Spec §ROP, mensajes `es-CO` por `messageKey` con fallback por código, sanitización de `details` que descarta entradas con claves extra/secretos/PII, y degradación de códigos desconocidos a `500 TECHNICAL_DEPENDENCY_FAILURE`), `result-projector.ts` (único puerto que controllers invocan: `Success`→valor, `Failure`→`HttpException` con `ApiErrorResponse` completando `path`/`trace_id`), `http-exception.filter.ts` (filtro global registrado vía `HttpModule`/`APP_FILTER`: reenvía `ApiErrorResponse` proyectados, convierte `HttpException` de transporte a código de transporte y traduce excepciones técnicas inesperadas a `500 TECHNICAL_DEPENDENCY_FAILURE` sin filtrar causa/secretos/PII), `transport-validation.pipe.ts` (400 sintaxis `INVALID_DOMAIN_INPUT`), `transport-auth.guard.ts` (401 transporte `AUTHENTICATION_REQUIRED`) y `transport-error.ts` (códigos de transporte `RATE_LIMITED`/`INVALID_DOMAIN_INPUT`). `HttpModule` añadido a `AppModule`. Sin endpoints funcionales, sin autenticación de negocio completa, sin Prisma, sin OpenAPI modificado, sin códigos/endpoints inventados; dominio/aplicación no lanzan `HttpException`.
- **verification_result:** `npm run build` OK; `npm test` 8 suites / 35 tests PASS (mapper unitario cubre Success/Failure y todos los códigos del catálogo: 400, 401, 403, 404, 409, 410, 422, 500; tests de adapter del filtro cubren 400 sintaxis, 401, 403, 409, 410, 422, 429 y 500, incluida la no-filtración de secretos en 500); `npm run depcruise` "no dependency violations found (70 modules, 108 dependencies)"; arranque real `node dist/main` inicializa `HttpModule` + 8 módulos y 8 controllers sin errores. Sin `HttpException` en domain/application; `@nestjs/common` solo en la capa de transporte (`shared/http`) y controllers/modules/infrastructure.
- **blocker:** `none`

#### MSF-API-003 — Alinear DTOs de entrada/salida y tipos de contrato OpenAPI
- **agent:** executor
- **spec_refs:** OpenAPI paths/schemas; Master Spec §97-99
- **goal:** disponer de schemas/DTOs TypeScript alineados al contrato vigente.
- **scope:** `merkee-shop-api/src/**/dto`, tipos de aplicación y configuración de generación/validación local del contrato.
- **out_of_scope:** editar `docs/api/openapi.yaml`, agregar operaciones o implementar casos de uso.
- **inputs:** OpenAPI real, especialmente `CreateAdminUserRequest`, `AdminActivationRequest`, `PasswordChangeRequest`, `UpdateProfileRequest`, paginación y errores.
- **implementation_notes:** conservar nombres, required, enums, headers, statuses e idempotencia; usar validación de transporte separada de validación semántica de dominio.
- **edge_cases:** `If-Match` numérico, `Idempotency-Key` UUID, raw body de webhooks y campos write-only.
- **done_criteria:** DTOs compilables y trazables a operationId/schema; ninguna divergencia contractual.
- **verification:** comparación manual campo por campo con OpenAPI y tests de serialización/validación.
- **dependencies:** MSF-API-001, MSF-API-002
- **handoff_context:** las tareas de persistencia y módulos consumen estos tipos, sin modificar contrato.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`.
- **stale_terms_guard:** no crear `POST /auth/initial-password-change`, `base_fee_cop` ni dirección de perfil.
- **status:** `done`
- **executor_notes:** Contrato OpenAPI tipado creado en `merkee-shop-api/src/contract` (TypeScript puro, sin NestJS/Prisma/HTTP): `schemas.ts` (DTOs de transporte de entrada/salida alineados campo por campo con `components/schemas`: `CreateAdminUserRequest`, `AdminActivationRequest`, `PasswordChangeRequest`, `UpdateProfileRequest`, `RegisterRequest`, `LoginRequest`, `PasswordResetRequest/Confirm`, `DeliveryAddressRequest`, `CreateCheckoutRequest`, `CartItemMutationRequest`, `SetCartItemQuantityRequest`, `CategoryWriteRequest`, `ProductWriteRequest/Update`, `StockAdjustmentRequest`, `BannerWriteRequest`, `CreateUploadUrlRequest`, `ProviderWebhookPayload` y todas las respuestas: `UserResponse`, `AdminUserProvisionResponse`, `SessionResponse`, `ImageResponse`, `CategoryResponse`, `ProductResponse`, `BannerResponse`, `CartItemResponse`, `CartResponse`, `OrderItemResponse`, `PaymentResponse`, `PaymentRefundResponse`, `OrderResponse` con snapshots `delivery_*`, `CheckoutResponse`, `StockAdjustmentResponse`, `UploadUrlResponse`, `PageMeta`, `PagedProductResponse`, `PagedOrderResponse`, `ApiErrorDetail`, `ApiErrorResponse`; enums `Role`, `PaymentProvider`, `OrderStatus`, `PaymentStatus`, `RefundStatus`, `CartStatus`, `ReservationStatus`, `UploadContentType`), `parameters.ts` (headers `Idempotency-Key`/`If-Match`/firmas de webhook, query `page`/`size`/`q`/`category_id`/`status` y path UUIDs), `operation-map.ts` (mapa de trazabilidad operationId→método/path/schemas/status/headers para las 40 operaciones del contrato), `application/dto.ts` (DTOs de aplicación `Success` del rail ROP: `SessionDto`, `UserDto`, `AdminUserProvisionDto`, `CategoryDto`, `ProductDto`, `BannerDto`, `CartDto`, `CheckoutDto`, `OrderDto`, `PagedProductsDto`, `PagedOrdersDto`, `StockAdjustmentDto`, `UploadUrlDto`, `NoContentDto`) y `validation/` (validación sintáctica de transporte separada de la semántica de dominio: `validators.ts` con helpers puros, `request-validators.ts` con un validador por schema de entrada y `header-validators.ts` para `Idempotency-Key`/`If-Match`/firmas). Se conserva ROP: la validación semántica de negocio queda en el dominio vía `Result<Success, DomainError>`; no se añadieron endpoints, códigos ni campos fuera del contrato; los campos secretos/write-only (`token`, `new_password`) nunca se devuelven en respuestas (verificado por aserción de tipo `@ts-expect-error` y serialización). No se editó `docs/api/openapi.yaml` ni otros documentos canónicos.
- **verification_result:** `npm run build` OK; `npm test` 12 suites / 85 tests PASS (4 suites nuevas de contrato: `schemas.spec.ts` con aserciones de tipo write-only y serialización nullable/enums/snapshots, `request-validators.spec.ts` con 22 casos de required/nullable/enum/límites/`additionalProperties:false`, `header-validators.spec.ts` con UUID/If-Match numérico/firmas y `operation-map.spec.ts` con trazabilidad de las 40 operaciones e If-Match/Idempotency-Key); `npm run depcruise` "no dependency violations found (82 modules, 125 dependencies cruised)"; arranque real `node dist/main` inicializa los 8 módulos y 8 controllers sin errores. Cobertura del contrato: `operation-map.ts` 100%, `request-validators.ts` 90.62% stmts, `header-validators.ts` 90.9%, `validators.ts` 72.72%; `schemas.ts`/`application/dto.ts` son solo tipos (sin código runtime). Cobertura global 81.65% stmts / 82.75% lines. Sin `HttpException` en domain/application; `@nestjs/common` solo en la capa de transporte existente.
- **blocker:** `none`

### API — Prisma schema/migraciones PostgreSQL y seeds sin secretos

#### MSF-DATA-001 — Crear Prisma schema y migraciones 001-003
- **agent:** executor
- **spec_refs:** prisma contract §5-13; Master Spec §73-89; CA-04, CA-05, CA-07
- **goal:** persistir identidad, activación, catálogo y media con invariantes PostgreSQL.
- **scope:** `merkee-shop-api/prisma/schema.prisma`, `prisma/migrations/001_*` a `003_*`.
- **out_of_scope:** casos de uso, seeds, pagos, AWS y cambios OpenAPI.
- **inputs:** contrato Prisma y ADR-003, ADR-015.
- **implementation_notes:** UUID, timestamptz, bigint COP, enums, soft delete, `version`, checks, FKs e índice parcial de activación; token solo hash.
- **edge_cases:** email activo case-insensitive; reemisión revoca token expirado no usado antes de insertar; no índice con `now()`.
- **done_criteria:** migraciones reproducibles en PostgreSQL y estructura coincide con objetos 001-003.
- **verification:** inspección SQL generado y prueba de aplicación limpia en Testcontainers cuando el entorno esté preparado.
- **dependencies:** MSF-API-003
- **handoff_context:** identity y catalog pueden mapear entidades sin cambiar DDL.
- **source_of_truth:** prisma-migration-contract.md y Master Spec §81-89.
- **stale_terms_guard:** nunca `db push`, H2, token claro, hard delete v1 o `base_fee_cop`.
- **status:** `done`
- **executor_notes:** Creado `merkee-shop-api/prisma/schema.prisma` (PostgreSQL, Prisma Migrate) y migraciones `001_identity_and_auth`, `002_admin_activation_tokens`, `003_catalog_and_media` en `prisma/migrations/`. 001: enums `UserRole`(admin/cliente) y `SessionKind`(GUEST/AUTHENTICATED), tablas `users`/`sessions`/`password_reset_tokens`, `must_change_password NOT NULL DEFAULT true`, guest con `user_id` nullable, FKs RESTRICT, índice único CI `users_email_ci_key ON users(lower(email))` (SQL crudo, no modelable en Prisma). 002: `admin_activation_tokens` con `token_hash` único, `used_at` nullable, índices `(expires_at,used_at)` y `(user_id,used_at)`, e índice parcial único `admin_activation_tokens_user_id_unused_key ON admin_activation_tokens(user_id) WHERE used_at IS NULL` (sin `expires_at > now()` en el índice; vigencia validada atómicamente en el canje y reemisión revoca el token no usado expirado antes de insertar). 003: `categories`/`products`/`product_images`/`banners` con soft delete (`deleted_at`), una categoría por producto (FK), `version DEFAULT 1` (optimistic locking), checks `regular_price_cop>=0`, `sale_price_cop>=0`, `stock_on_hand>=0`, `stock_reserved>=0`, `stock_reserved<=stock_on_hand`, `position>=0`, `display_order>=0`. Sin secretos, passwords, hashes conocidos ni admin seed. Se añadieron scripts npm `prisma:generate`, `prisma:migrate:deploy`, `prisma:migrate:dev` y `DATABASE_URL` en `.env.example`. Prisma 5.22.0 instalado como devDependency.
- **verification_result:** `prisma validate` OK. Aplicación limpia 001→003 con `prisma migrate deploy` contra PostgreSQL 17 (Docker) exitosa y reproducible (2 aplicaciones limpias). Inspección SQL y `\d` confirman tablas, enums, FKs RESTRICT, checks e índices. Pruebas funcionales de invariantes en PostgreSQL: email CI rechaza `Admin@Example.com` vs `admin@example.com` (`users_email_ci_key`); índice parcial rechaza 2 tokens no usados por el mismo admin (`admin_activation_tokens_user_id_unused_key`) y permite usado+no usado; `stock_reserved>stock_on_hand` rechazado (`products_stock_reserved_le_on_hand_check`); precio negativo rechazado (`products_regular_price_cop_check`). `npm run build` OK; `npm run depcruise` "no dependency violations found (82 modules, 125 dependencies)"; `npm test` 12 suites / 85 tests PASS (sin regresiones). `prisma generate` OK.
- **blocker:** `none`

#### MSF-DATA-002 — Crear migraciones 004-006 de carrito, órdenes, pagos y auditoría
- **agent:** executor
- **spec_refs:** prisma contract §14-16, §20-27; Master Spec §81-95; CA-08, CA-10
- **goal:** persistir reservas, snapshots, pagos, webhooks, refunds, outbox y ajustes auditados.
- **scope:** `merkee-shop-api/prisma/migrations/004_*` a `006_*` y schema Prisma correspondiente.
- **out_of_scope:** lógica de checkout/pagos, scheduled jobs, proveedor externo y seed.
- **inputs:** OpenAPI schemas de carrito/checkout/order/payment; contrato Prisma 004-006.
- **implementation_notes:** 1:1 carrito/sesión, reserva/item, estados terminales; snapshots `delivery_*` NOT NULL; idempotencias únicas; auditoría inmutable.
- **edge_cases:** `CHECKOUT_PENDING` no expira por reaper; suma de reservas y contador agregado; ajuste no modifica `stock_reserved`.
- **done_criteria:** checks, índices, FKs y restricciones representan las invariantes documentadas.
- **verification:** migración limpia 001→006 y pruebas de constraints/índices en PostgreSQL/Testcontainers.
- **dependencies:** MSF-DATA-001
- **handoff_context:** cart, checkout y payments implementan transacciones sobre puertos, no sobre Prisma en controllers.
- **source_of_truth:** prisma-migration-contract.md §14-16 y Master Spec §87, §93-95.
- **stale_terms_guard:** no hold de 10 minutos durante checkout, no snapshots opcionales, no mutar `stock_reserved` desde catalog.
- **status:** `done`
- **executor_notes:** Completado `merkee-shop-api/prisma/schema.prisma` y migraciones `004_cart_reservations`, `005_orders_payments_outbox`, `006_product_stock_adjustments`. 004: enums `CartStatus`(ACTIVE/CHECKOUT_PENDING/CLOSED/EXPIRED) y `ReservationStatus`(ACTIVE/CHECKOUT_PENDING/CONSUMED/RELEASED/EXPIRED); `carts` 1:1 sesión (`session_id` UNIQUE) con totales COP persistidos, `cart_items` único carrito/producto (`UNIQUE cart_id+product_id`), `stock_reservations` 1:1 item (`cart_item_id` UNIQUE); checks `delivery_fee_cop=5000`, `tax_rate_basis_points=1900`, `total=subtotal+entrega+iva`, `quantity>=1` y `CHECKOUT_PENDING` sin expiración (`reservation_expires_at`/`expires_at` NULL). 005: enums `OrderStatus`, `PaymentStatus`, `RefundStatus`, `PaymentProvider`, `WebhookEventStatus`, `OutboxStatus`; `orders` con snapshots `delivery_*` NOT NULL, `order_number` UNIQUE, `cart_id` UNIQUE (orden única por carrito), checks de IVA/entrega/total; `payments` con `idempotency_key` UNIQUE; `payment_refunds` único por payment (`payment_id` UNIQUE) e idempotente; `payment_webhook_events` dedupe `UNIQUE(provider, provider_event_id)`; `outbox_events` con índice `(status, created_at)`. 006: `product_stock_adjustments` append-only con checks `quantity_delta<>0`, `after=before+delta`, `after>=reserved`, `stock_available=after-reserved`, `idempotency_key` UNIQUE, FKs RESTRICT e inmutabilidad vía trigger `product_stock_adjustments_immutable` (bloquea UPDATE/DELETE). Sin casos de uso, scheduled jobs, proveedores, AWS ni seeds; sin SQLite/H2/db push.
- **verification_result:** `prisma validate` OK; `prisma generate` OK; `prisma migrate status` "Database schema is up to date!"; `prisma migrate diff --from-migrations --to-schema-datamodel` "No difference detected" (sin drift). Aplicación limpia 001→006 con `prisma migrate deploy` contra PostgreSQL 17 (Docker) exitosa y reproducible. 20 pruebas de constraints/invariantes en PostgreSQL (savepoints): carrito 1:1 sesión, delivery_fee=5000, ecuación total, CHECKOUT_PENDING sin expiración (cart y reserva), item único carrito/producto, quantity>=1, reserva 1:1 item, snapshot delivery NOT NULL, tax_rate=1900, orden única por carrito, payment idempotency única, refund único por payment, webhook dedupe, delta!=0, after=before+delta, after>=reserved, idempotency única y UPDATE/DELETE bloqueados por trigger — todas rechazadas con el error esperado. `npm run build` OK; `npm run depcruise` "no dependency violations found (82 modules, 125 dependencies)"; `npm test` 12 suites / 85 tests PASS (sin regresiones).
- **blocker:** `none`

#### MSF-DATA-003 — Crear seed no productivo seguro
- **agent:** executor
- **spec_refs:** prisma contract §17; ADR-010; Master Spec §77
- **goal:** poblar datos dummy sin credenciales ni secretos.
- **scope:** `merkee-shop-api/prisma/seed*` y documentación local de ejecución.
- **out_of_scope:** bootstrap del admin inicial, passwords, hashes, tokens y datos productivos.
- **inputs:** entidades de migraciones 001-006.
- **implementation_notes:** seed solo categorías/productos/media/banners no sensibles; generar UUIDs/datos deterministas si procede.
- **edge_cases:** ejecución repetida idempotente; ningún admin creado.
- **done_criteria:** seed no contiene secretos, contraseñas, hashes, activation token ni PAN/CVV.
- **verification:** revisión de contenido y ejecución contra base PostgreSQL de prueba si Executor dispone del entorno.
- **dependencies:** MSF-DATA-002
- **handoff_context:** bootstrap seguro queda separado y depende de configuración operacional posterior.
- **source_of_truth:** prisma contract §17 y ADR-010.
- **stale_terms_guard:** prohibido sembrar admin, contraseña conocida o token en claro.
- **status:** `done`
- **executor_notes:** Creado seed NO PRODUCTIVO seguro e idempotente en `merkee-shop-api/prisma/seed.ts` (MSF-DATA-003) + documentación local `prisma/README-seed.md`. Puebla solo catálogo dummy: 6 categorías, 15 productos, 15 imágenes de producto y 3 banners, con datos explícitamente ficticios en `es-CO` y precios COP enteros (BigInt) para probar catálogo, paginación, carrito y stock/reservas. Idempotencia por UUIDs deterministas fijos + `upsert` sobre `id` (re-ejecución no duplica filas ni rompe checks). NO crea admin, usuario real, contraseña, hash, token de activación, secreto, PAN/CVV ni datos productivos; no incluye credenciales; no modifica OpenAPI ni migraciones; no usa `db push`, SQLite ni H2. Añadidos script `prisma:seed` y config `prisma.seed` en `package.json`. El seed vive en `prisma/` (fuera de `src/`), por lo que no participa del build Nest ni de dependency-cruiser.
- **verification_result:** `npm run prisma:seed` OK contra PostgreSQL 17 de prueba (Docker) con migraciones 001-006 aplicadas; ejecutado 3 veces consecutivas y conteos estables (6 categorías, 15 productos, 15 imágenes, 3 banners) confirmando idempotencia; `users` permanece en 0 (ningún admin/usuario creado). Invariantes verificadas en BD: 0 productos con `stock_reserved > stock_on_hand` o stock negativo; 0 productos con precios negativos. Escaneo de secretos en `seed.ts`/`README-seed.md`: solo menciones descriptivas de lo que el seed NO contiene; sin contraseñas, hashes, tokens, PAN/CVV ni credenciales reales. `npm run build` OK; `npm test` 12 suites / 85 tests PASS (sin regresiones); `npm run depcruise` "no dependency violations found (82 modules, 125 dependencies cruised)".
- **blocker:** `none`

### API — identity

#### MSF-ID-001 — Implementar dominio, sesiones y registro/login/refresh/logout
- **agent:** executor
- **spec_refs:** OpenAPI Auth; Master Spec §73-80, §62-65; ADR-004, ADR-014
- **goal:** habilitar sesiones guest/cliente/admin y registro público exclusivamente cliente.
- **scope:** `merkee-shop-api/src/modules/identity/{domain,application,infrastructure}` para register, login, refresh, logout y sesión.
- **out_of_scope:** provisión/activación, password flows, carrito, AWS y UI.
- **inputs:** migraciones 001-002, DTOs y ROP.
- **implementation_notes:** Argon2id; access JWT ≤10m solo memoria; refresh/cookie opaco hashado, HttpOnly/Secure/SameSite=Lax, rotación y CSRF/CORS según ADR.
- **edge_cases:** login guest→cliente conserva carrito; login guest→admin delega liberación a puerto cart-reservation; admin no obtiene carrito; credenciales inválidas neutras.
- **done_criteria:** endpoints existentes cumplen status/cookies y roles; todos los casos de uso retornan `Result`.
- **verification:** unitarias Success/Failure, tests HTTP y concurrencia de promoción guest→admin.
- **dependencies:** MSF-DATA-001, MSF-API-002, MSF-API-003
- **handoff_context:** password y provisioning reutilizan sesiones/ports y guardas de must_change_password.
- **source_of_truth:** OpenAPI `/auth/*`, Master Spec §73-80, ADR-004/014.
- **stale_terms_guard:** no auto-registro admin, JWT persistido en navegador ni admin comprador.
- **status:** `done`
- **executor_notes:** Implementación completa de dominio, aplicación, infraestructura y controller para register, login, refresh y logout (MSF-ID-001). **Dominio** (TypeScript puro, sin NestJS/Prisma/HTTP): modelos `User`/`Session`, puertos `UserRepositoryPort`, `SessionRepositoryPort`, `JwtPort`, `CookieTokenPort`, `CartReservationPort` + fábrica de `DomainError` (`emailAlreadyRegistered`, `invalidCredentials`, `sessionNotFoundOrExpired`, `technicalFailure`). **Aplicación** (casos de uso ROP): `RegisterUseCase` (crea cliente + sesión AUTHENTICATED + JWT ≤10min + refresh token hashado), `LoginUseCase` (email/password con credenciales inválidas neutras; guest→cliente conserva carrito revocando guest; guest→admin libera ACTIVE + cierra carrito vía `CartReservationPort` + revoca guest), `RefreshSessionUseCase` (rota cookie opaca hashada, valida expiración/revocación, emite nuevo JWT), `LogoutUseCase` (revoca sesión + libera ACTIVE vía puerto; idempotente si ya revocada; no toca CHECKOUT_PENDING). **Infraestructura**: `PrismaService`, `PrismaUserRepositoryAdapter`, `PrismaSessionRepositoryAdapter`, `Argon2PasswordHasherAdapter` (argon2id, 64MiB, timeCost=3), `JwtAdapter` (HS256, 10min, secret de entorno), `CookieTokenAdapter` (randomBytes 32 + SHA-256), `SystemClockAdapter`, `NoopCartReservationAdapter` (placeholder hasta MSF-CART-001). **Controller**: `POST /auth/register` (201 + Set-Cookie), `POST /auth/login` (200 + Set-Cookie + clearCookie guest), `POST /auth/refresh` (200 + Set-Cookie rotada), `POST /auth/logout` (204 + clearCookie). Cookies HttpOnly/Secure/SameSite=Lax, 10min maxAge. Sin secretos/tokens/PII en logs. Sin modificar OpenAPI ni documentos canónicos. Sin `any` (salvo `_sessionId` en noop adapter justificado como placeholder). **Archivos creados**: `domain/models/{user,session,index}.ts`, `domain/ports/{user-repository,session-repository,jwt,cookie-token,cart-reservation}.port.ts`, `domain/identity-errors.ts`, `application/use-cases/{register,login,refresh-session,logout}.use-case.ts`, `infrastructure/prisma.service.ts`, `infrastructure/adapters/{prisma-user-repository,prisma-session-repository,argon2-password-hasher,jwt,cookie-token,system-clock,noop-cart-reservation}.adapter.ts`, `identity.tokens.ts`. **Archivos actualizados**: `identity.controller.ts`, `identity.module.ts`. **Archivos eliminados**: `domain/ports/session.port.ts`, `infrastructure/adapters/session.adapter.ts` (placeholders reemplazados). **Dependencias añadidas**: `argon2`, `jsonwebtoken`, `@types/jsonwebtoken`.
- **verification_result:** `npm run build` OK; `npm test` 17 suites / 125 tests PASS (5 suites nuevas de identity: `identity-errors.spec.ts` 6 tests, `register.use-case.spec.ts` 8 tests, `login.use-case.spec.ts` 10 tests, `refresh-session.use-case.spec.ts` 9 tests, `logout.use-case.spec.ts` 7 tests — cubren Success/Failure, credenciales neutras, rotación de token, idempotencia de logout, promoción guest→cliente y guest→admin); `npm run depcruise` "no dependency violations found (110 modules, 255 dependencies cruised)". Sin `HttpException` en domain/application; `@nestjs/common` solo en controller/module/infrastructure. Sin regresiones en los 85 tests existentes.
- **blocker:** `none`

#### MSF-ID-002 — Implementar bootstrap seguro, provisión y activación de admin
- **agent:** executor
- **spec_refs:** OpenAPI `/admin/users`, `/auth/admin-activations`; Master Spec AC-05/06, §77-80; ADR-010/015
- **goal:** crear y activar admins mediante token opaco de un solo uso.
- **scope:** casos de uso/controllers/adapters identity para bootstrap, `provisionAdminUser` y `activateAdmin`.
- **out_of_scope:** canal de entrega del token, email/notificaciones, nuevos endpoints y modificación de OpenAPI.
- **inputs:** migraciones 001-002; hash/clock/idempotency/session ports; secreto externo de bootstrap.
- **implementation_notes:** solo admin con `must_change_password=false`; token hashado 24h, consumo atómico; respuesta no contiene token/password; bootstrap no-op tras cambio de contraseña.
- **edge_cases:** provisión repetida misma clave/cuerpo devuelve original; divergente 409; activación concurrente solo un éxito; expirado se revoca al reemitir.
- **done_criteria:** activation deja `must_change_password=false`; no secreto en logs/traces/metrics/outbox.
- **verification:** unitarias, HTTP status/code, concurrencia de canje y prueba de no filtración.
- **dependencies:** MSF-ID-001, MSF-DATA-001, MSF-API-003
- **handoff_context:** el canal operativo seguro queda como decisión operacional pendiente, no se implementa en API.
- **source_of_truth:** Master Spec §77-80 y prisma contract §12.
- **stale_terms_guard:** no contraseña/token en claro, no auto-registro, no `POST /auth/initial-password-change`.
- **status:** `done`
- **executor_notes:** Corrección de MSF-ID-002 (2026-08-15) para cumplir ADR-018 y la política aprobada de transacción atómica, retención y purga de `idempotency_records`. **Frontera transaccional real (provisión):** nuevo puerto `ProvisionUnitOfWorkPort` (`domain/ports/provision-unit-of-work.port.ts`) y adapter `PrismaProvisionUnitOfWorkAdapter` (`infrastructure/adapters`) que ejecuta usuario admin + token de activación + `idempotency_records` en una única transacción PostgreSQL `SERIALIZABLE` con advisory lock transaccional `SHA-256(scope || 0x00 || idempotency_key)` (bigint con signo de 64 bits, pasado como string y casteado `::bigint`), `FOR UPDATE` del registro idempotente (`IdempotencyPort.findForUpdate`), rollback total ante fallo y hasta tres reintentos 50/100/200 ms ante abortos de serialización (P2034) o conflicto de único (P2002). `UNIQUE(scope,idempotency_key)` es la salvaguarda final: tras conflicto, el reintento relee con `FOR UPDATE` y resuelve replay (hash igual) o `409 IDEMPOTENCY_KEY_REUSED` (hash distinto), sin duplicar admin/token. `ProvisionAdminUserUseCase` se reescribió para delegar toda la escritura al puerto; la autorización (actor admin con `must_change_password=false`) se valida en lectura fuera de la transacción. Los adapters `PrismaUserRepositoryAdapter`, `PrismaAdminActivationTokenRepositoryAdapter` y `PrismaIdempotencyAdapter` aceptan `Prisma.TransactionClient | PrismaService` (con `@Inject(PrismaService)` para DI) y se reutilizan dentro de la transacción sin llevar Prisma a domain/application. **Purga (job/caso de uso):** `PurgeIdempotencyRecordsUseCase` (`application/use-cases`) con batches ≤500, cutoff de 30 días desde `created_at`, protección mínima de 24 h, exclusión de replay vigente y de scopes con operación pendiente; reintentos de batch 1/5/15 s y rollback total (eliminación atómica `deleteMany`); bucle que evita iteración infinita si un batch completo se salta. Puertos `IdempotencyPurgeRepositoryPort`, `IdempotencyScopeEvaluatorPort`, `PurgeMetricsPort`, `PurgeLoggerPort`; adapters `PrismaIdempotencyPurgeRepositoryAdapter` (`FOR UPDATE SKIP LOCKED` en `READ COMMITTED`), `AdminProvisionScopeEvaluatorAdapter` (admin-provision es terminal al confirmar; scope desconocido se trata como pendiente → no se purga, conservador), `NoopPurgeMetricsAdapter` y `ConsolePurgeLoggerAdapter` (sin PII), y `ScheduledIdempotencyPurgeAdapter` (driving adapter con `run()`; el cron real se empaqueta en despliegue). **Migración** `008_idempotency_records_purge_index` (índice `idempotency_records_created_at_idx` sobre `created_at` para la selección de purga) + `@@index([createdAt])` en el modelo Prisma; no altera el único contractual ni OpenAPI. Bootstrap inicial NO implementado (fuera de alcance). Sin secretos/tokens/contraseñas/hashes en logs, métricas ni respuestas; sin `HttpException` en domain/application; sin Prisma en domain/application; sin endpoints nuevos ni cambios OpenAPI.
- **verification_result:** `npm run build` OK; `npm test` 23 suites / 172 tests PASS (suites nuevas: `prisma-provision-unit-of-work.adapter.spec.ts` 5 tests — transacción SERIALIZABLE + advisory lock, rollback total ante fallo, reintento P2034/P2002 y agotamiento de reintentos; `purge-idempotency-records.use-case.spec.ts` 9 tests — borrado tras 30d/24h, skips `retention_not_elapsed`/`minimum_age_not_elapsed`/`replay_active`/`operation_pending`, multi-batch, detención ante batch completo saltado, error→`purge_failed` sin relanzar, reintento transitorio; `admin-provision-scope-evaluator.adapter.spec.ts` 2 tests; `prisma-idempotency-purge-repository.adapter.spec.ts` 3 tests; `provision-admin-user.use-case.spec.ts` reescrito 14 tests con unitOfWork); `npm run depcruise` "no dependency violations found (136 modules, 368 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `008_idempotency_records_purge_index` contra PostgreSQL 17 (Docker) y `\d idempotency_records` confirma el índice `idempotency_records_created_at_idx`; `prisma migrate status` "Database schema is up to date!" (8 migraciones) **[HISTÓRICO/SUPERSEDED — la evidencia vigente de MSF-ID-002 es 13 migraciones 007–013; este conteo refleja el estado al cierre de la nota de activación 2026-08-15 (post 008)]**; arranque real `node dist/main` OK (8 módulos, rutas existentes, sin endpoints nuevos). **Integración real contra PostgreSQL** (`scripts/integration-msf-id-002.ts`): provisión inicial OK; replay misma clave/cuerpo devuelve el mismo id; replay divergente → 409; dos provisiones concurrentes con la misma clave → 2 success y 1 solo admin; rollback total ante email duplicado (sin efectos parciales); invariantes 1 usuario / 2 tokens / 2 registros. **Integración real de purga** (`scripts/integration-purge-msf-id-002.ts`): respeta 24h/30d (registros de 12h y 29d no se purgan) y scopes pendientes (scope desconocido de 31d no se purga); solo purga el de 31d admin-provision. Sin PII en métricas/logger/evaluador; sin `HttpException` ni Prisma en domain/application.
- **documentation_sync_note:** MSF-ID-002 permanece `done` tras la corrección. El 2026-08-15 se implementó la frontera transaccional real y el job de purga de `idempotency_records` conforme a ADR-018 y la política aprobada; se añadió la migración `008_idempotency_records_purge_index`. No se modificó OpenAPI ni se añadieron endpoints. La política de purga para scopes desconocidos es conservadora (no se purgan) hasta que una delta defina su semántica de operación pendiente; para `admin-provision:{actorId}` la operación es terminal al confirmar la transacción de provisión, por lo que no hay blocker para el scope actual. El veredicto histórico de Spec Validator sigue invalidado por la sincronización documental previa y requiere revalidación antes de cualquier handoff o nueva ejecución.
- **bootstrap_note (2026-08-15):** Subtarea acotada de bootstrap seguro del admin inicial (ADR-010) implementada en `identity` sin tocar purga ni otros hallazgos. Nuevo caso de uso ROP `BootstrapInitialAdminUseCase` (`application/use-cases/bootstrap-initial-admin.use-case.ts`) que crea/valida únicamente `cristiansrc@gmail.com` como `admin` con `must_change_password=true`; la contraseña inicial llega SOLO por referencia externa (`INITIAL_ADMIN_PASSWORD` vía `InitialAdminSecretPort`), nunca hardcodeada; si el secreto falta, falla de forma segura (`TECHNICAL_DEPENDENCY_FAILURE`) antes de crear usuario; si el admin ya existe, es no-op sin reescribir contraseña/hash/flag; creación/validación atómica vía `BootstrapUnitOfWorkPort` (`PrismaBootstrapUnitOfWorkAdapter`, transacción Prisma con rollback total). Puertos nuevos: `initial-admin-secret.port.ts`, `bootstrap-unit-of-work.port.ts`; adapters: `env-initial-admin-secret.adapter.ts`, `prisma-bootstrap-unit-of-work.adapter.ts`, `bootstrap-initial-admin-on-startup.ts` (driving adapter `OnApplicationBootstrap`, habilitable/deshabilitable vía `BOOTSTRAP_INITIAL_ADMIN_ENABLED`, default true, nunca rompe arranque y registra advertencia sin PII). Sin endpoints nuevos, sin cambios OpenAPI, sin migraciones, sin provisión adicional ni purga. Verificación: `npm run build` OK; `npm test` 27 suites / 189 tests PASS (nuevas: `bootstrap-initial-admin.use-case.spec.ts` 7 tests — secreto ausente, creación, hash Argon2id, no-op existente con flag true/false, revalidación atómica, error técnico; `bootstrap-initial-admin-on-startup.spec.ts` 4 tests — disabled, outcome sin PII, fallo sin lanzar, excepción sin detalles crudos; `env-initial-admin-secret.adapter.spec.ts` 3 tests; `argon2-password-hasher.adapter.spec.ts` 3 tests — prefijo `$argon2id$`); `npm run depcruise` "no dependency violations found (146 modules, 407 dependencies cruised)". Sin contraseña/hash/secreto/token/PII en logs, métricas ni respuestas; sin `HttpException` ni Prisma en domain/application.
- **activation_note (2026-08-15):** Subtarea acotada de activación atómica de admin implementada en `identity` sin tocar purga, bootstrap, provisión ni documentación. Nuevo puerto `ActivateAdminUnitOfWorkPort` (`domain/ports/activate-admin-unit-of-work.port.ts`) y adapter `PrismaActivateAdminUnitOfWorkAdapter` (`infrastructure/adapters`) que ejecuta en una única transacción PostgreSQL: consumo atómico del token (`used_at IS NULL AND expires_at > now()`), actualización de `password_hash`/`must_change_password` y revocación de las demás sesiones del admin, con rollback total ante fallo. `ActivateAdminUseCase` se adaptó para delegar toda la escritura al puerto; la búsqueda del token por hash es lectura fuera de la transacción y el hash de contraseña se calcula fuera (cómputo puro) para mantener corta la transacción; el consumo atómico es la salvaguarda de concurrencia (solo una activación gana; la otra devuelve 422 neutro). `PrismaSessionRepositoryAdapter` ahora acepta `Prisma.TransactionClient | PrismaService` (con `@Inject(PrismaService)`) para participar en la transacción sin llevar Prisma a domain/application. DI: token `IDENTITY_ACTIVATE_ADMIN_UNIT_OF_WORK` y provider en `identity.module.ts`. Sin endpoints nuevos, sin cambios OpenAPI, sin migraciones. Verificación: `npm run build` OK; `npm test` 28 suites / 192 tests PASS (nuevas: `prisma-activate-admin-unit-of-work.adapter.spec.ts` 3 tests — transacción única con consumo+contraseña+revocación, rollback total ante fallo temprano y tras escrituras; `activate-admin.use-case.spec.ts` reescrito 9 tests — éxito en transacción, hash fuera, token por hash, token inexistente/usado/expirado 422, concurrencia, mensaje neutro, fallo de transacción→`TECHNICAL_DEPENDENCY_FAILURE`); `npm run depcruise` "no dependency violations found (149 modules, 418 dependencies cruised)". Sin contraseña/hash/token/PII en logs, métricas ni respuestas; sin `HttpException` ni Prisma en domain/application.
- **privacy_replay_note (2026-08-15):** **HISTÓRICO/SUPERSEDED — reemplazado por snapshot de cuatro campos `{resource_id, status, activation_expires_at, body_hash}` y replay desde recurso actual (ver `snapshot_idempotency_blockers_note` y `response_json_replay_note`).** Corrección acotada de privacidad de errores y replay de idempotencia de provisión admin (sin tocar purga, bootstrap ni activación). **Privacidad de errores:** `emailAlreadyRegistered()` ya no recibe ni persiste el email (mensaje/código neutro, sin metadata); `technicalFailure()` ya no transporta la causa al rail HTTP (sin `metadata.cause`); la causa se registra únicamente en logging interno sanitizado de infrastructure (`PrismaProvisionUnitOfWorkAdapter` loguea solo el `code` Prisma, nunca el mensaje que puede contener PII). **Replay de idempotencia:** `PrismaProvisionUnitOfWorkAdapter.isRetryable` ahora distingue el P2002 de `(scope, idempotency_key)` (target array `['scope','idempotency_key']` o nombre de restricción `idempotency_records_scope_idempotency_key_key`) — reintentable para releer con `FOR UPDATE` y resolver replay (mismo `body_hash`) o `409` divergente — de cualquier otro P2002 (p. ej. email único) que se propaga sin ocultarse como replay. **Minimización de `response_json`:** `ProvisionAdminUserUseCase` persiste una instantánea mínima `{ resourceId, status, activationExpiresAt }` (sin email/display_name/phone ni secretos) y reconstruye la respuesta contractual segura en el replay desde el comando entrante (mismo `body_hash`). Verificación: `npm run build` OK; `npm test` PASS (nuevos casos de no filtración de email/causa y de replay de carrera); `npm run depcruise` sin violaciones. Sin cambios OpenAPI, sin migraciones, sin Git/push.
- **purge_note (2026-08-15):** Corrección acotada de la purga de `idempotency_records` (sin tocar bootstrap, activación, provisión ni privacidad). **Unidad transaccional única:** `IdempotencyPurgeRepositoryPort` ahora expone un único `purgeBatch(now, limit, evaluate)` que ejecuta el batch completo en una sola transacción PostgreSQL `READ COMMITTED` cubriendo de forma consistente: advisory lock transaccional global (`pg_advisory_xact_lock` con clave fija "MSFID002", exclusión distribuida), timeout de 5 s (`SET LOCAL lock_timeout`/`statement_timeout` vía `Prisma.raw`), selección con retención vencida (`created_at < now() - 30 days`) con `FOR UPDATE SKIP LOCKED`, evaluación de cada candidato con el callback `evaluate` y eliminación atómica (`deleteMany`) de los elegibles; si algo falla, la transacción se revierte íntegramente (rollback total). El adapter ya no selecciona en una transacción y borra fuera: selección, evaluación, delete y métricas de resultado comparten la misma frontera transaccional. **Caso de uso:** `PurgeIdempotencyRecordsUseCase` delega la unidad transaccional al adapter, registra métricas de resultado tras el commit (consistentes con el estado confirmado), mantiene reintentos de batch 1/5/15 s y el guard que evita iteración infinita cuando un lote completo se omite (`hasMore = batch.hasMore && batch.deleted > 0`). **Métricas reales:** se reemplazó `NoopPurgeMetricsAdapter` por `InMemoryPurgeMetricsAdapter` (adapter real testeable, acumula contadores/timestamps sin PII y expone `snapshot()` inmutable); registrado en `identity.module.ts`. **Tests:** `purge-idempotency-records.use-case.spec.ts` reescrito (evaluación 24h/30d, replay, scope pendiente, métricas tras commit, multi-batch, guard anti-bucle, error→`purge_failed`, reintento transitorio); `prisma-idempotency-purge-repository.adapter.spec.ts` reescrito (transacción única READ COMMITTED con advisory lock + timeout + FOR UPDATE SKIP LOCKED + delete, skips por razón, batch 500, rollback ante fallo de evaluación); nuevo `in-memory-purge-metrics.adapter.spec.ts`. **Integración real PostgreSQL** (`scripts/integration-purge-msf-id-002.ts`): cutoff 24h/30d + scope pendiente/desconocido OK; métricas reales sin PII OK; batch 500 (600 filas purgadas en batches) OK; dos jobs concurrentes (advisory lock + SKIP LOCKED, 40 filas sin doble procesamiento) OK; rollback total ante fallo de evaluación OK. Verificación: `npm run build` OK; `npm test` 29 suites / 205 tests PASS; `npm run depcruise` "no dependency violations found (150 modules, 421 dependencies cruised)"; `prisma validate` OK; `prisma migrate status` "Database schema is up to date!" (8 migraciones). Sin cambios OpenAPI, sin endpoints nuevos, sin migraciones, sin Git/push.
- **bootstrap_role_note (2026-08-15):** Corrección acotada de la validación de rol en el bootstrap del admin inicial (ADR-010), sin tocar purga, métricas, scheduler, provisión ni activación. `BootstrapInitialAdminUseCase` ahora es **no-op solo si el correo canónico (`cristiansrc@gmail.com`) existe con `role=admin`**; si existe con otro rol (p. ej. `cliente`), **falla de forma segura** (`TECHNICAL_DEPENDENCY_FAILURE`) sin reescribir contraseña, hash ni flag, tanto en el pre-chequeo fuera de transacción como en la revalidación dentro de la transacción atómica (carrera). Se añadieron 2 tests de rol mismatch (pre-chequeo y carrera transaccional) que verifican que no se llama a `create`/`updatePassword`/`hash`. Verificación: `npm run build` OK; `npm test` 29 suites / 207 tests PASS; `npm run depcruise` "no dependency violations found (150 modules, 421 dependencies cruised)". Sin cambios OpenAPI, sin migraciones, sin Git/push.
- **metrics_scheduler_note (2026-08-15):** Corrección acotada de métricas operativas y scheduler diario de la purga de `idempotency_records` (ADR-018 addendum), sin tocar bootstrap, activación, provisión, privacidad ni migraciones. **Métricas productivas:** se reemplazó el registro productivo de `InMemoryPurgeMetricsAdapter` por `PrometheusPurgeMetricsAdapter` (`infrastructure/adapters/prometheus-purge-metrics.adapter.ts`), que implementa `PurgeMetricsPort` emitiendo en prom-client los nombres canónicos `idempotency_records_purge_runs_total{outcome}`, `idempotency_records_purge_deleted_total`, `idempotency_records_purge_skipped_total{reason}`, `idempotency_records_purge_errors_total` e `idempotency_records_purge_last_success_timestamp_seconds` (gauge), sin PII; usa el registry global por defecto (inyectable en tests) y reutiliza métricas ya registradas para evitar duplicados. `InMemoryPurgeMetricsAdapter` queda solo para tests/integración. **Scheduler diario cableado:** `ScheduledIdempotencyPurgeAdapter` ahora es un driving adapter `OnApplicationBootstrap`/`OnApplicationShutdown` que programa la ejecución diaria del caso de uso a la hora configurable (default `02:00` UTC) vía token `IDENTITY_PURGE_SCHEDULE_CONFIG` (env `IDEMPOTENCY_PURGE_SCHEDULE_ENABLED`/`IDEMPOTENCY_PURGE_SCHEDULE_TIME`); `start()` es idempotente (no duplica jobs) y `enabled:false` lo deshabilita en tests. **replay_active:** el evaluador del caso de uso produce `replay_active` para registros dentro de la ventana de 30 días (razón explícita de skip que bloquea purga) y `minimum_age_not_elapsed` para los de <24 h, alineado al addendum ADR-018. Se mantienen READ COMMITTED, advisory lock, FOR UPDATE SKIP LOCKED, timeout 5 s, batch 500 y reintentos 1/5/15 s. **Tests:** `prometheus-purge-metrics.adapter.spec.ts` (nombres/valores exactos, sin PII, HELP/TYPE), `scheduled-idempotency-purge.adapter.spec.ts` (disabled/enabled, hora configurable, start idempotente, shutdown), `identity-module-wiring.spec.ts` (PURGE_METRICS→Prometheus no in-memory, config default 02:00, scheduler deshabilitable) y `purge-idempotency-records.use-case.spec.ts` actualizado (replay_active). Verificación: `npm run build` OK; `npm test` 32 suites / 218 tests PASS; `npm run depcruise` "no dependency violations found (155 modules, 436 dependencies cruised)"; `prisma validate` OK; `prisma migrate status` "Database schema is up to date!" (8 migraciones); arranque real `node dist/main` OK; integración real `scripts/integration-purge-msf-id-002.ts` contra PostgreSQL OK (cutoff 24h/30d, scope pendiente, batch 500, concurrencia advisory lock+SKIP LOCKED, rollback total, métricas sin PII). Sin cambios OpenAPI, sin migraciones, sin Git/push.
- **response_json_replay_note (2026-08-15):** Corrección acotada de MSF-ID-002 (sin tocar purga, bootstrap, activación ni OpenAPI). **1) Drift `response` vs contrato `response_json`:** se renombró la columna al nombre canónico `response_json` en `prisma/schema.prisma` (`responseJson Json @map("response_json")`) y se creó la migración expand/contract `009_idempotency_records_response_json_rename` (`ALTER TABLE ... RENAME COLUMN "response" TO "response_json"`); NO se editó la migración aplicada `007`. Se alinearon `PrismaIdempotencyAdapter` (raw SQL `response_json`, `create` con `responseJson`) y el script de integración de purga. **2) Snapshot canónico sin PII:** la instantánea persistida en `response_json` usa nombres canónicos `{ resource_id, status, activation_expires_at }` (snake_case) y nunca persiste email/display_name/phone/token/password/hash. **3) Replay desde DB:** `ProvisionAdminUserUseCase` reconstruye `AdminUserProvisionResponse` desde el recurso vigente en DB vía puertos (`userRepo.findById` + nuevo `activationTokenRepo.findActiveByUserId`), nunca desde el comando de replay; si el recurso ya no existe devuelve error seguro `RESOURCE_NOT_FOUND` (`provisionedResourceNotFound()`) sin datos falsos. **4) Application pura sin NestJS:** se eliminaron `@nestjs/common` (`@Injectable`/`@Inject`/`@Optional`) y `IDENTITY_TOKENS` de los 8 use cases de `application`; la DI se movió a factories en `identity.module.ts` sobre los tokens de infrastructure. Se añadió regla dependency-cruiser `no-application-framework-imports` (application→`@nestjs`). **Tests:** `idempotency-response-rename.spec.ts` (schema `response_json`, migración 009 rename, 007 intacta), snapshot sin PII, replay desde DB y recurso ausente. Verificación: `npm run build` OK; `npm test` 33 suites / 222 tests PASS; `npm run depcruise` "no dependency violations found (158 modules, 422 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `009` y `prisma migrate status` "Database schema is up to date!" (9 migraciones). Sin cambios OpenAPI, sin Git/push.
- **purge_blockers_note (2026-08-15):** Corrección acotada de los blockers restantes de purga de `idempotency_records` (sin tocar bootstrap, activación, provisión, privacidad, métricas, scheduler ni migraciones). **1) Selección desde la ventana mínima de 24 h:** `IdempotencyPurgeRepositoryPort.purgeBatch` ahora recibe `minimumAgeCutoff` y el adapter selecciona `created_at < minimumAgeCutoff` (24 h) en lugar de `created_at < now()-30d`, haciendo alcanzables y metricables dentro de la transacción tanto `replay_active` (24 h–30 d) como `minimum_age_not_elapsed` (<24 h); el caso de uso pasa el cutoff de 24 h. Se mantienen READ COMMITTED, advisory lock transaccional, `FOR UPDATE SKIP LOCKED`, timeout 5 s, batch ≤500, rollback total y reintentos 1/5/15 s. **2) Scope evaluator estricto:** `AdminProvisionScopeEvaluatorAdapter` solo considera terminal `admin-provision:<UUID válido>` (regex UUID v1–v8 en minúsculas); cualquier scope desconocido o mal formado (actor no-UUID, UUID truncado/con llaves/mayúsculas, prefijo sin actor, cadena vacía) produce `operation_pending` y no se purga. **3) Protecciones conservadas:** `replay_active` para registros dentro de la ventana 30 d y protección <24 h. **Tests:** `prisma-idempotency-purge-repository.adapter.spec.ts` (nuevo test de selección 24 h que evalúa 29 d→`replay_active` y 31 d→delete), `purge-idempotency-records.use-case.spec.ts` (nuevo test de paso del cutoff 24 h), `admin-provision-scope-evaluator.adapter.spec.ts` (UUID válido terminal; mal formado/desconocido pendiente). Integración real `scripts/integration-purge-msf-id-002.ts` actualizada a UUIDs válidos y verifica 24h/30d, replay_active metricable, scope no-UUID/desconocido pendiente, batch 500, dos jobs concurrentes y rollback. Verificación: `npm run build` OK; `npm test` 33 suites / 226 tests PASS; `npm run depcruise` "no dependency violations found (158 modules, 422 dependencies cruised)"; `prisma validate` OK; `prisma migrate status` "Database schema is up to date!" (9 migraciones) **[HISTÓRICO/SUPERSEDED — la evidencia vigente de MSF-ID-002 es 13 migraciones 007–013; este conteo refleja el estado al cierre del `purge_blockers_note` 2026-08-15 (post 009, sin nuevas migraciones)]**; integración purga real contra PostgreSQL OK (todas las comprobaciones). Sin cambios OpenAPI, sin migraciones, sin Git/push.
- **bootstrap_transaction_note (2026-08-15):** Corrección acotada del blocker del bootstrap del admin inicial (ADR-010), sin tocar purga, métricas, scheduler, provisión, activación, privacidad ni migraciones. `BootstrapInitialAdminUseCase` ya **no hace un no-op definitivo fuera de la unidad transaccional**: se eliminó el pre-chequeo `findByEmail` previo a la transacción que devolvía `noop`/fallo de forma prematura. Toda la validación (relectura con lock) y la decisión `created`/`noop`/`roleMismatch` se resuelven **dentro de la misma transacción atómica** vía `BootstrapUnitOfWorkPort`: si el correo canónico (`cristiansrc@gmail.com`) existe con `role=admin` → `noop` sin reescribir contraseña/hash/flag; si existe con otro rol → falla segura (`TECHNICAL_DEPENDENCY_FAILURE`) sin modificar nada; si no existe y el secreto externo está presente → se hashea con Argon2id y se crea el admin con `must_change_password=true` en la transacción. El hash Argon2id se calcula **solo en la rama de creación** (no en no-op/roleMismatch), manteniendo corta la transacción y evitando cómputo innecesario. Si el secreto falta, falla de forma segura antes de crear usuario. Se conservan ROP, ausencia de secretos/PII en logs y los tests de carrera role mismatch. Verificación: `npm run build` OK; `npm test` 33 suites / 226 tests PASS; `npm run depcruise` "no dependency violations found (158 modules, 422 dependencies cruised)". Sin cambios OpenAPI, sin migraciones, sin Git/push.
- **snapshot_idempotency_blockers_note (2026-08-15):** Corrección acotada de los dos blockers de snapshot/idempotencia de MSF-ID-002 (sin tocar purga, bootstrap, activación, privacidad ni OpenAPI). **1) Snapshot canónico con `status` entero y `body_hash`:** `ProvisionAdminUserUseCase` persiste en `response_json` exactamente `{ resource_id, status, activation_expires_at, body_hash }`; `status` es el entero HTTP `201` (creación/replay contractual, constante `PROVISION_CREATED_STATUS`); `body_hash` coincide con la columna/hash usado para idempotencia (se persiste en la instantánea y se valida en el replay). El replay contractual valida que el `body_hash` de la instantánea coincida con el hash del cuerpo entrante (si diverge → `409 IDEMPOTENCY_KEY_REUSED`) y reconstruye la respuesta desde el recurso vigente en DB vía puertos, nunca desde el comando. Sin PII (email/display_name/phone) ni secretos (token/password/hash de credencial). **2) Migración expand/contract `010_idempotency_records_response_json_backfill`:** backfillea los registros históricos de `response_json` al snapshot mínimo sin copiar PII; elimina las claves prohibidas del JSON existente; conserva solo `resource_id`/`status`/`activation_expires_at`/`body_hash` cuando pueden determinarse (`resource_id` desde `resource_id`|`resourceId`|`id`; `status` normalizado a entero HTTP 201; `activation_expires_at` desde `activation_expires_at`|`activationExpiresAt`; `body_hash` siempre desde la columna). Si un registro histórico no puede reconstruirse de forma segura (sin `resource_id` determinable), se retiene de forma conservadora con `unrecoverable: true` y SIN PII (decisión documentada en la migración), para que el replay devuelva error seguro. No se editaron migraciones aplicadas (007/009 intactas). **Tests:** snapshot exacto `{resource_id,status:201,activation_expires_at,body_hash}`, body_hash en instantánea, replay igual/divergente (incluido body_hash de instantánea divergente → 409), migración 010 backfill sin PII (spec `idempotency-response-rename.spec.ts`). Verificación: `npm run build` OK; `npm test` 33 suites / 228 tests PASS; `npm run depcruise` "no dependency violations found (158 modules, 422 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `010` y `prisma migrate status` "Database schema is up to date!" (10 migraciones); backfill real contra PostgreSQL verificado (PII eliminada, camelCase legacy normalizado, status→201, `unrecoverable` sin PII); integración real `scripts/integration-msf-id-002.ts` OK (provisión, replay igual/divergente, concurrencia, rollback, invariantes). Sin cambios OpenAPI, sin Git/push.
- **three_findings_fix_note (2026-08-15):** Corrección de los tres hallazgos actuales de MSF-ID-002 (sin tocar bootstrap, activación, privacidad, métricas, scheduler ni OpenAPI). **1) Migración 010 → compensatoria `011_idempotency_records_response_json_normalize`:** garantiza que `response_json` quede EXACTAMENTE con las cuatro claves canónicas `{ resource_id, status, activation_expires_at, body_hash }`; elimina la quinta clave `unrecoverable` y las claves opcionales. Política segura documentada por código para legacy no reconstruible: si `resource_id` no puede determinarse (ni `resource_id`/`resourceId`/`id`) el registro NO puede cumplir el replay contractual → se ELIMINA durante el backfill; si `activation_expires_at` no es determinable desde el JSON se deriva del token de activación (`admin_activation_tokens.expires_at`) y si tampoco existe token se elimina. `status` normalizado a entero HTTP 201; `body_hash` siempre desde la columna. No se editaron migraciones aplicadas (007/009/010 intactas). **2) Replay de provisión:** `ProvisionAdminUserUseCase.replayFromResource` reconstruye `AdminUserProvisionResponse` desde el usuario vigente en DB usando `user.mustChangePassword` (no fuerza `true` tras la activación); se amplió `AdminUserProvisionResponse.must_change_password` a `boolean` en `schemas.ts`. Se mantienen body_hash en la instantánea, ausencia de PII en `response_json` y `409 IDEMPOTENCY_KEY_REUSED` divergente. **3) Purga:** `PrismaIdempotencyPurgeRepositoryAdapter` selecciona ahora `created_at < now` (sin filtrar antes las filas necesarias) para que la evaluación dentro de la transacción alcance `minimum_age_not_elapsed` (<24 h), `replay_active` (24 h–30 d) y eligible (>30 d) sin dejar registros elegibles sin purgar; se conservan READ COMMITTED, advisory lock transaccional, `FOR UPDATE SKIP LOCKED`, timeout 5 s, batch ≤500, reintentos 1/5/15 s y scopes conservadores. Tests reales añadidos para 24 h (`minimum_age_not_elapsed`), 29 d (`replay_active`) y >30 d (eligible purged) en el adapter y el caso de uso.
- **verification_result (three_findings_fix):** `npm run build` OK; `npm test` 33 suites / 233 tests PASS (nuevos: replay tras activación con `must_change_password=false`, selección desde `now` con las tres razones alcanzables, 24 h→`minimum_age_not_elapsed`, 29 d→`replay_active`, >30 d→eligible purged, y test de migración 011 con exactamente cuatro claves sin `unrecoverable`); `npm run depcruise` "no dependency violations found (158 modules, 422 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `011` contra PostgreSQL 17 (Docker) y `prisma migrate status` "Database schema is up to date!" (11 migraciones) **[HISTÓRICO/SUPERSEDED — la evidencia vigente de MSF-ID-002 es 13 migraciones 007–013; este conteo refleja el estado al cierre del `three_findings_fix_note` 2026-08-15 (post 011)]**; `prisma migrate diff --from-migrations --to-schema-datamodel` "No difference detected" (sin drift). **Integración real PostgreSQL:** `scripts/integration-purge-msf-id-002.ts` OK (cutoff 24h/30d, replay_active metricable, scope no-UUID/desconocido pendiente, batch 500, dos jobs concurrentes, rollback) y `scripts/integration-msf-id-002.ts` OK (provisión, replay igual/divergente 409, concurrencia, rollback, invariantes). Verificación manual de la lógica de 011 en transacción: registro no reconstruible eliminado, `activation_expires_at` derivado del token, snapshot exacto de cuatro claves. Sin PII en `response_json`/métricas/logger; sin `HttpException` ni Prisma en domain/application.
- **rop_migration_validate_note (2026-08-15):** Corrección de los dos hallazgos vigentes de MSF-ID-002 (sin tocar purga, bootstrap, activación, privacidad, métricas, scheduler ni OpenAPI). **1) ROP en `provision-admin-user.use-case.ts`:** se eliminó la captura directa de excepciones técnicas en `application` (el `try/catch` que devolvía `technicalFailure()`), que contradecía Master Spec §ROP ("cada adapter de salida captura excepciones... y las traduce a `DomainError`; no las propaga hacia controllers ni dominio"). El puerto `ProvisionUnitOfWorkPort.run` ahora devuelve `Result<T, DomainError>`; el adapter `PrismaProvisionUnitOfWorkAdapter` captura las excepciones en su límite, registra solo el código Prisma (sin causa/PII) y devuelve `fail(technicalFailure())`. La autorización (actor admin con `must_change_password=false`) se movió dentro de la transacción, de modo que la aplicación no necesita `catch` para fallos técnicos de lectura; `application` conserva solo reglas de negocio y devuelve `Result`, sin transportar cause/message/PII. Se eliminaron los repos `userRepo`/`activationTokenRepo`/`idempotency` del constructor (quedaban sin uso) y se actualizó la DI en `identity.module.ts`. Sin regresiones en replay/409/idempotencia (integración real OK). **2) Migración compensatoria `012_idempotency_records_response_json_validate`:** valida estrictamente el snapshot mínimo `{resource_id,status,activation_expires_at,body_hash}`: `resource_id` debe ser UUID v1–v8 (si no, se elimina); `status` debe ser entero HTTP 100–599 (si no, se normaliza a 201 para provisión); `activation_expires_at` debe ser RFC 3339/date-time (si no, se deriva del token de activación y si tampoco existe se elimina); `body_hash` debe ser SHA-256 hexadecimal (64 hex) y coincidir con la columna `body_hash` (si no, se elimina). No conserva claves extra ni PII. No se editaron migraciones aplicadas (007/009/010/011 intactas). **Tests:** `idempotency-response-rename.spec.ts` añade 5 tests de 012 (UUID inválido, status no entero→201, RFC 3339 inválido, body_hash no hex/divergente, snapshot exacto de cuatro claves sin PII); `prisma-provision-unit-of-work.adapter.spec.ts` y `provision-admin-user.use-case.spec.ts` actualizados al contrato `Result` del puerto (traducción técnica en el adapter, sin `catch` en application).
- **verification_result (rop_migration_validate):** `npm run build` OK; `npm test` 33 suites / 238 tests PASS (nuevos: 5 tests de migración 012; adapter traduce fallo técnico a `TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII; use case sin `catch` técnico con autorización en transacción); `npm run depcruise` "no dependency violations found (158 modules, 428 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `012` contra PostgreSQL 17 (Docker) y `prisma migrate status` "Database schema is up to date!" (12 migraciones) **[HISTÓRICO/SUPERSEDED — la evidencia vigente de MSF-ID-002 es 13 migraciones 007–013; este conteo refleja el estado al cierre del `rop_migration_validate_note` 2026-08-15 (post 012)]**; `prisma migrate diff --from-migrations --to-schema-datamodel` "No difference detected" (sin drift). **Integración real PostgreSQL:** `scripts/integration-msf-id-002.ts` OK (provisión, replay igual/divergente 409, concurrencia 2 success/1 admin, rollback, invariantes 1 user/2 tokens/2 records) y `scripts/integration-purge-msf-id-002.ts` OK (sin regresiones). Verificación manual de la lógica de 012 en transacción (rollback): resource_id no-UUID eliminado, status `'created'`→201, `activation_expires_at` no-RFC3339 eliminado, body_hash no-hex eliminado, body_hash divergente de la columna eliminado, snapshot exacto de cuatro claves. Sin PII en `response_json`/métricas/logger; sin `HttpException` ni Prisma en domain/application; sin `catch` técnico en application.
- **exclusive_classification_note (2026-08-16):** Corrección de MSF-ID-002 para materializar la precedencia exclusiva documentada (`minimum_age_not_elapsed` → `replay_active` → `retention_not_elapsed` → `operation_pending` → `eligible`) y la validación determinista de la migración 012 (sin tocar bootstrap, activación, provisión, privacidad, métricas, scheduler ni OpenAPI). **1) Clasificación exclusiva por candidato:** `PurgeEvaluation` ahora devuelve `PurgeClassification = PurgeSkipReason | 'eligible'`; el evaluador del caso de uso devuelve `eligible` EXPLÍCITAMENTE (separado de las razones de skip) y hace alcanzables las cinco clasificaciones: `minimum_age_not_elapsed` (<24 h), `replay_active` (24 h–30 d con expiración específica no vencida o ausente), `retention_not_elapsed` (24 h–30 d con expiración específica `activation_expires_at` ya vencida — razón residual/general, sin ventana ni conducta nueva), `operation_pending` (fuera de retención con scope pendiente/desconocido) y `eligible` (>30 d, sin replay vigente, scope terminal). `PurgeCandidate` incorpora `activationExpiresAt` (leída de `response_json->>'activation_expires_at'`, sin PII) para distinguir `replay_active` de `retention_not_elapsed`; el adapter selecciona esa columna y borra únicamente los clasificados `eligible` (`skipped` solo cuenta las cuatro razones de skip; cada candidato recibe exactamente una clasificación). **2) Migración 012 determinista:** `activation_expires_at` se valida con la regex estricta RFC 3339 Y el cast nativo `::timestamptz` dentro de un bloque con captura de excepción (PostgreSQL rechaza calendarios imposibles como `2026-02-30` o `2026-99-99` que la regex sola aceptaría); si no es RFC 3339 válido se deriva del token de activación y si tampoco existe se ELIMINA de forma conservadora (sin inventar datos). Se añade `jsonb_typeof(response_json) = 'object'` al borrado de no reconstruibles; se conservan resource_id UUID v1–v8, status HTTP 100–599→201, body_hash SHA-256 + columna y snapshot exacto de cuatro claves. No se editaron migraciones aplicadas (007/009/010/011 intactas). **Tests:** `purge-idempotency-records.use-case.spec.ts` (eligible explícito, replay_active sin expiración, retention_not_elapsed con expiración vencida), `prisma-idempotency-purge-repository.adapter.spec.ts` (selección de `activation_expires_at`, retention_not_elapsed, eligible→delete) e `idempotency-response-rename.spec.ts` (cast `::timestamptz` + excepción, `jsonb_typeof`). Verificación: `npm run build` OK; `npm test` 33 suites / 242 tests PASS; `npm run depcruise` "no dependency violations found (158 modules, 428 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó 001→012 contra PostgreSQL 17 (Docker) y `prisma migrate status` "Database schema is up to date!" (12 migraciones). **Integración real PostgreSQL:** `scripts/integration-purge-msf-id-002.ts` OK (12h→minimum_age, 29d→replay_active, 29d con expiración vencida→retention_not_elapsed, 31d→eligible purgado, scopes pendientes, batch 500, concurrencia, rollback) y `scripts/integration-msf-id-002.ts` OK (sin regresiones). Verificación manual de 012 en transacción (rollback): resource_id no-UUID eliminado, `2026-02-30T00:00:00Z` rechazado por el cast y eliminado, body_hash no-hex eliminado, JSON no-objeto eliminado, snapshot con clave extra normalizado a exactamente cuatro claves. Sin PII en `response_json`/métricas/logger; sin `HttpException` ni Prisma en domain/application; sin Git/push.
- **bootstrap_rop_migration013_note (2026-08-16):** Corrección de los dos hallazgos vigentes de MSF-ID-002 (sin tocar purga, activación, provisión, privacidad, métricas, scheduler ni OpenAPI). **1) ROP en `bootstrap-initial-admin.use-case.ts`:** se eliminó el `try/catch` técnico de `application` (contradecía Master Spec §ROP: "cada adapter de salida captura excepciones... y las traduce a `DomainError`"). El puerto `BootstrapUnitOfWorkPort.run` ahora devuelve `Result<T, DomainError>`; el adapter `PrismaBootstrapUnitOfWorkAdapter` captura las excepciones en su límite, registra solo el código Prisma (sin causa/PII) y devuelve `fail(technicalFailure())`. La aplicación conserva solo reglas de negocio (secreto ausente, no-op admin, roleMismatch) y propaga el rail `Failure` sin transportar cause/message/PII. Se eliminó el repo `userRepo` del constructor (quedaba sin uso) y se actualizó la DI en `identity.module.ts`. **2) Migración compensatoria `013_idempotency_records_response_json_strict_validate`** (no se editaron migraciones aplicadas 007/009/010/011/012): corrige dos hallazgos de 011/012. **a) Derivación de `activation_expires_at`:** 011/012 derivaban con `MAX(t.expires_at)` sobre CUALQUIER token (incluidos usados/expirados); 013 deriva SOLO desde tokens vigentes (`used_at IS NULL AND expires_at > now()`) con selección determinista (`ORDER BY t.expires_at DESC LIMIT 1`); si no existe token vigente, el registro se ELIMINA de forma conservadora (sin inventar datos ni derivar de tokens usados/expirados). **b) `resource_id` UUID v1–v8 con variante RFC 4122:** 011/012 aceptaban cualquier nibble de versión/variante; PostgreSQL no valida versión/variante de forma nativa, por lo que 013 valida con la misma regex declarada en `admin-provision-scope-evaluator.adapter.ts` (`^[0-9a-f]{8}-[0-9a-f]{4}-[1-8][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$`); si no cumple, se ELIMINA. Se conservan las validaciones de 012 (objeto JSON, body_hash SHA-256 + columna, status HTTP 100–599→201, RFC 3339 determinista con cast `::timestamptz`). Snapshot final exactamente cuatro claves, sin PII. **Tests:** `bootstrap-initial-admin.use-case.spec.ts` añade test estático de ausencia de `try/catch` en application y test de propagación del fallo técnico traducido por el adapter sin causa/PII; nuevo `prisma-bootstrap-unit-of-work.adapter.spec.ts` (3 tests: transacción OK, fallo del trabajo→`TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII, fallo Prisma P2002 sin propagar mensaje con PII); `idempotency-response-rename.spec.ts` añade 3 tests de 013 (derivación solo desde tokens vigentes con selección determinista, UUID v1–v8 + variante RFC 4122 con eliminación de inválidos, snapshot exacto de cuatro claves sin PII).
- **verification_result (bootstrap_rop_migration013):** `npm run build` OK; `npm test` 34 suites / 249 tests PASS (nuevos: test estático sin `try/catch` en application, propagación de fallo técnico del adapter, 3 tests del adapter de bootstrap, 3 tests de migración 013); `npm run depcruise` "no dependency violations found (159 modules, 437 dependencies cruised)"; `prisma validate` OK; `prisma migrate deploy` aplicó `013` contra PostgreSQL 17 (Docker) y `prisma migrate status` "Database schema is up to date!" (13 migraciones); `prisma migrate diff --from-migrations --to-schema-datamodel` "No difference detected" (sin drift). **Integración real PostgreSQL:** `scripts/integration-msf-id-002.ts` OK (provisión, replay igual/divergente 409, concurrencia 2 success/1 admin, rollback, invariantes) y `scripts/integration-purge-msf-id-002.ts` OK (sin regresiones). Verificación manual de la lógica de 013 en transacción (rollback): registro con solo tokens usados/expirados eliminado (no derivable), resource_id versión inválida (v0) eliminado, resource_id variante inválida (`c`) eliminado, registro con token vigente conservado con `activation_expires_at` derivado, snapshot exacto de cuatro claves sin PII. Sin PII en `response_json`/métricas/logger; sin `HttpException` ni Prisma en domain/application; sin `catch` técnico en application.
- **blocker:** `none`
- **estado_real_msf_id_002 (2026-08-16):** `done`. La evidencia más reciente es la verificación de `013_idempotency_records_response_json_strict_validate`: 34 suites/249 tests, build, dependency-cruiser, `prisma validate`, deploy de 13 migraciones y diff sin drift, más integración PostgreSQL de provisión/replay/concurrencia/rollback y purga. 013 valida snapshot de cuatro campos sin PII, UUID v1–v8 con variante RFC 4122 y derivación de `activation_expires_at` solo desde token no usado y vigente; la corrección ROP de bootstrap deja la traducción técnica en el adapter. Esto no cambia el bloqueo global: el `ready` previo es histórico/superseded y la siguiente acción es revalidación focalizada de Spec Validator, sin handoff.
- **idempotency_responsejson_rop_purge_note (2026-08-16):** Corrección acotada de los dos blockers de código vigentes de MSF-ID-002 (sin tocar bootstrap, activación, provisión, privacidad, métricas, scheduler, migraciones, OpenAPI ni otros módulos). **1) Drift `response` vs contrato `response_json` (Blocker 1):** el campo canónico Prisma `responseJson`/`response_json` se sustituye en todos los sitios donde quedaba la forma legacy `response` en `domain/ports/idempotency.port.ts` (`IdempotencyRecord.responseJson` y `IdempotencyPort.save(responseJson)`), en `infrastructure/adapters/prisma-idempotency.adapter.ts` (mappings dominio↔Prisma: `responseJson: row.response_json`, `responseJson: responseJson as Prisma.InputJsonValue`, `responseJson: row.responseJson`) y en `application/use-cases/provision-admin-user.use-case.ts` (lectura `existing.responseJson as ProvisionResponseSnapshot`). El snapshot persistido sigue siendo el de cuatro claves canónicas `{ resource_id, status, activation_expires_at, body_hash }` sin PII (email/display_name/phone) ni secretos (token/password/hash); el replay contractual se reconstruye desde el recurso vigente en DB a través de los puertos (`userRepo.findById` + `activationTokenRepo.findActiveByUserId`), nunca desde el comando de replay. Se añaden 3 tests estáticos en `idempotency-response-rename.spec.ts` que detectan el drift: el primero verifica que el puerto expone `readonly responseJson: unknown` y que `save(..., responseJson: unknown)` no contiene `response: unknown` legacy; el segundo verifica que el adapter usa `responseJson` en los mappings a `row.response_json` / `row.responseJson` / `responseJson as Prisma.InputJsonValue`; el tercero verifica que el caso de uso lee `existing.responseJson` (no `existing.response`). **2) ROP de purga (Blocker 2):** `PurgeIdempotencyRecordsUseCase` se alinea a ROP estricto (Master Spec §ROP / ADR-017): `execute()` ahora devuelve `Promise<Result<void, DomainError>>` (en lugar de `Promise<void>`) y se elimina el `try/catch` técnico. **3) Fix de selección purga:** `PrismaIdempotencyPurgeRepositoryAdapter.purgeBatch` cambia la selección SQL de `WHERE created_at < ${minimumAgeCutoff}` a `WHERE created_at < ${now}`, permitiendo que los registros menores de 24 h alcancen el evaluador y clasifiquen como `minimum_age_not_elapsed`. El caso de uso mantiene la precedencia exclusiva: `minimum_age_not_elapsed` (<24 h), `replay_active` (24 h–30 d con expiración específica no vencida o ausente), `retention_not_elapsed` (24 h–30 d con expiración específica vencida), `operation_pending` (fuera de retención con scope pendiente/desconocido) y `eligible` (>30 d, sin replay vigente, scope terminal). Solo `eligible` se borra. Se mantienen READ COMMITTED, advisory lock transaccional, `FOR UPDATE SKIP LOCKED`, timeout 5 s, batch ≤500, reintentos 1/5/15 s y rollback total. Tests actualizados para 12 h (`minimum_age_not_elapsed`), 29 d (`replay_active`) y 31 d (`eligible`). Verificación: `npm run build` OK; `npm test` PASS (sin regresiones).minó todo el `try/catch` técnico de la capa `application`; las excepciones se traducen en el límite del adapter `PrismaIdempotencyPurgeRepositoryAdapter`, que captura las excepciones de `$transaction` (incluido el rollback total) y devuelve `fail(technicalFailure())` con logging sanitizado (solo el código Prisma, sin causa/PII) — NUNCA propaga la excepción ni su causa al caso de uso. El puerto `IdempotencyPurgeRepositoryPort.purgeBatch` ahora devuelve `Result<PurgeBatchResult, DomainError>` (sin cambios en la lógica de las cinco clasificaciones, métricas, scheduler ni rollback: la unidad transaccional, el advisory lock, `FOR UPDATE SKIP LOCKED`, el timeout 5 s, los reintentos 1/5/15 s, los cortes `hasMore`/`deleted>0` y la cobertura de `minimum_age_not_elapsed` / `replay_active` / `retention_not_elapsed` / `operation_pending` / `eligible` se conservan íntegramente). El driving scheduler `ScheduledIdempotencyPurgeAdapter.run()` proyecta el `Result` propagado por el caso de uso y registra `recordRun('error')` + `recordError()` + log sanitizado `idempotency_records.purge_failed` con `{ code: 'TECHNICAL_DEPENDENCY_FAILURE' }` en `Failure` (sin filtrar mensaje técnico, scope, PII ni secretos), y deja el siguiente ciclo del scheduler para reintentar; en `Success` no toca métricas de error. Se mantienen 2 tests estáticos más: `purge-idempotency-records.use-case.spec.ts` ("no contiene try/catch técnico en application") y el `idempotency-response-rename.spec.ts` (`el puerto IdempotencyPort expone el campo canónico responseJson (no response)`). El spec del adapter de purga (`prisma-idempotency-purge-repository.adapter.spec.ts`) reescribe el test de rollback: la excepción ya no se propaga, el adapter devuelve `Failure<TECHNICAL_DEPENDENCY_FAILURE>` sin `metadata` y sin filtrar la causa (`'evaluation failed'`); además se añade un segundo test que verifica la traducción de un fallo técnico a nivel de `$transaction` (`code: 'P1001'`). El script `scripts/integration-purge-msf-id-002.ts` se alinea al nuevo contrato ROP: la sección de rollback total ahora valida que `purgeBatch` devuelve `Failure<TECHNICAL_DEPENDENCY_FAILURE>` sin `metadata` y sin filtrar la causa forzada, en lugar de esperar una excepción propagada. Verificación: `npm run build` OK; `npm test` 36 suites / 267 tests PASS (nuevos: 3 tests estáticos de drift `response`/`responseJson` en `idempotency-response-rename.spec.ts`, 1 test de éxito del scheduler sin métricas de error + 1 test de fallo del scheduler con métricas/log sanitizado en `scheduled-idempotency-purge.adapter.spec.ts`, 1 test de `Failure` técnico propagado por el adapter de purga + 1 test de ROP estático sin `try/catch` en `purge-idempotency-records.use-case.spec.ts`, 1 test de traducción de `$transaction` fallido en `prisma-idempotency-purge-repository.adapter.spec.ts`); `npm run depcruise` "no dependency violations found (161 modules, 480 dependencies cruised)"; `prisma validate` OK; `prisma migrate status` "Database schema is up to date!" (13 migraciones, sin cambios); integración real `scripts/integration-msf-id-002.ts` OK (provisión, replay igual/divergente 409, concurrencia 2 success/1 admin, rollback, invariantes) y `scripts/integration-purge-msf-id-002.ts` OK (24h/30d, scope pendiente, métricas, batch 500, dos jobs concurrentes, rollback total vía `Failure` sin causa). Sin PII en `response_json`/métricas/logger; sin `HttpException` ni Prisma en domain/application; sin `catch` técnico en `purge-idempotency-records.use-case.ts`; sin cambios OpenAPI/Prisma/migraciones/`package.json`/Git/push.

#### MSF-ID-003 — Implementar perfil, password-change y password-reset
- **agent:** executor
- **spec_refs:** OpenAPI `/me`, `/auth/password-change`, reset; Master Spec AC-07, §62-66; ADR-016
- **goal:** soportar perfil mínimo y flujos de contraseña con revocación segura.
- **scope:** identity use cases/adapters/controllers para `GET/PATCH /me`, cambio y recuperación.
- **out_of_scope:** dirección de perfil, exportación, administración legal de retención y UI.
- **inputs:** DTOs OpenAPI, sesiones, Argon2id, idempotencia, tablas reset/sessions.
- **implementation_notes:** PATCH solo `display_name`/`phone`; password-change exige current password + Idempotency-Key, rota sesión actual y revoca demás; reset respuesta neutra y token 30m de un uso.
- **edge_cases:** idempotencia divergente 409; admin inicial solo puede profile read/refresh/logout/password-change; Set-Cookie en 204.
- **done_criteria:** email/role/dirección no modificables; respuestas y códigos estables.
- **verification:** unitarias, HTTP, tests de idempotencia concurrente y traducción de error técnico.
- **dependencies:** MSF-ID-001, MSF-ID-002, MSF-API-002
- **handoff_context:** storefront consumirá `/me` sin inventar perfil de dirección.
- **source_of_truth:** OpenAPI operaciones citadas y Master Spec §62-66.
- **stale_terms_guard:** no endpoint inicial histórico, no dirección persistida, no excepciones para errores esperados.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### API — catalog/media

#### MSF-CAT-001 — Implementar media ports y upload URLs S3 privadas
- **agent:** executor
- **spec_refs:** OpenAPI `/media/upload-urls`; Master Spec AC-12, §29, §33; ADR-006
- **goal:** encapsular S3 y exponer únicamente URLs prefirmadas de corta duración para admin.
- **scope:** módulo `media`, ports/adapters S3, caso de uso y controller existente.
- **out_of_scope:** infraestructura AWS desplegada, buckets/CloudFront, componentes UI y edición de catálogo.
- **inputs:** OpenAPI `CreateUploadUrlRequest`/`UploadUrlResponse`; ADR-006.
- **implementation_notes:** adapter traduce errores técnicos; no credenciales S3 ni PAN/CVV en navegador/logs; validar key/media type/tamaño conforme contrato.
- **edge_cases:** idempotencia, URL expirada, admin con password change requerido, bucket privado.
- **done_criteria:** controller invoca un puerto y devuelve el schema contractual; dominio/application no importan SDK.
- **verification:** unitarias de ports/use case, adapter error mapping y HTTP auth/status.
- **dependencies:** MSF-API-002, MSF-API-003, MSF-DATA-001
- **handoff_context:** catalog referencia media key validada sin acoplarse a S3.
- **source_of_truth:** OpenAPI `/media/upload-urls`, ADR-006.
- **stale_terms_guard:** no bucket público, credenciales en frontend ni SDK en dominio.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-CAT-002 — Implementar categorías, productos, banners y paginación pública/admin
- **agent:** executor
- **spec_refs:** OpenAPI Catalog/Admin; Master Spec AC-01, AC-10, §29-33; ADR-012/013
- **goal:** entregar lectura pública y escritura administrativa de catálogo/media con soft delete.
- **scope:** módulo `catalog` y escrituras de banners/media según ownership; endpoints existentes de categorías/productos/banners.
- **out_of_scope:** stock adjustment, carrito, S3 upload implementation y cambios OpenAPI.
- **inputs:** migración 003, DTOs, media port, If-Match/version.
- **implementation_notes:** una categoría por producto; productos públicos activos; paginación page/size; `version` inicia 1 y sube con If-Match; producto/categoría/banner solo soft delete.
- **edge_cases:** If-Match mismatch 409 sin cambio; categoría con producto activo no se elimina; búsqueda q 2..100.
- **done_criteria:** todas las operaciones existentes devuelven schemas y statuses; no hard delete.
- **verification:** unitarias, integración Prisma/Testcontainers, HTTP contract tests y tests concurrentes de version.
- **dependencies:** MSF-CAT-001, MSF-DATA-001, MSF-API-002, MSF-API-003
- **handoff_context:** stock y carrito consumirán puertos de catálogo, no tablas directamente.
- **source_of_truth:** OpenAPI operaciones Catalog/Admin y Master Spec AC-01/09/10.
- **stale_terms_guard:** no Strapi, hard delete, stock en PATCH general, `base_fee_cop`.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-CAT-003 — Implementar ajuste de stock auditado e idempotente
- **agent:** executor
- **spec_refs:** OpenAPI `/admin/products/{productId}/stock-adjustments`; Master Spec AC-09, §62-71; ADR-011
- **goal:** ajustar solo stock físico con auditoría inmutable.
- **scope:** caso de uso, port/adaptador Prisma y controller de stock adjustment.
- **out_of_scope:** reservas, pago aprobado, If-Match y edición general del producto.
- **inputs:** migraciones 003/006, idempotency, actor admin, quantity_delta/reason.
- **implementation_notes:** lock transaccional del producto; exige delta no cero; `after=before+delta`, after≥reserved; inserta actor y snapshots; nunca escribe reserved.
- **edge_cases:** dos ajustes concurrentes serializados; reintento igual devuelve original; clave divergente 409; stock insuficiente 409 según contrato.
- **done_criteria:** auditoría no mutable y respuesta `StockAdjustmentResponse` contractual.
- **verification:** unitarias, integración PostgreSQL, concurrencia y HTTP code/idempotency.
- **dependencies:** MSF-CAT-002, MSF-DATA-002, MSF-API-002
- **handoff_context:** payments será el otro writer autorizado de stock físico al consumir pago.
- **source_of_truth:** ADR-011, OpenAPI operationId `adminCreateProductStockAdjustment`.
- **stale_terms_guard:** no If-Match para ajuste, no mutar `stock_reserved`, no ledger no solicitado.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### API — cart-reservation

#### MSF-CART-001 — Implementar carritos guest/cliente y mutaciones de reserva
- **agent:** executor
- **spec_refs:** OpenAPI `/cart`, `/cart/items`; Master Spec AC-02/03/04/11, §91-95; ADR-008/014
- **goal:** gestionar carrito servidor y reservar stock atómicamente.
- **scope:** módulo `cart-reservation`, casos de uso, puertos/adaptador Prisma y controllers existentes.
- **out_of_scope:** checkout, pagos, UI y login identity.
- **inputs:** migraciones 004, catálogo ports, sesión/auth context, idempotencia.
- **implementation_notes:** carrito guest/cliente 1:1 sesión; lock product+reservation; renueva sesión/cart/ACTIVE a 10m; admin recibe 403 y no crea carrito.
- **edge_cases:** último stock concurrente; remove/set repetido; sesión expirada 410; CHECKOUT_PENDING no se libera por mutación admin.
- **done_criteria:** GET/add/set/delete cumplen schemas, cookies y estados; contador reserved consistente.
- **verification:** unitarias Success/Failure, integración Testcontainers, concurrency last-stock e HTTP 403 admin.
- **dependencies:** MSF-ID-001, MSF-CAT-002, MSF-DATA-002, MSF-API-002
- **handoff_context:** checkout usará ports de este módulo para convertir ACTIVE a CHECKOUT_PENDING.
- **source_of_truth:** OpenAPI cart y ADR-008/014.
- **stale_terms_guard:** no carrito local/Redux persistido, no carrito admin, no hold checkout 10m.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-CART-002 — Implementar reaper y transición guest→admin
- **agent:** executor
- **spec_refs:** Master Spec AC-04/11, §62-71, §93; prisma contract §20-27; ADR-014
- **goal:** liberar ACTIVE expiradas y coordinar promoción guest→admin sin tocar CHECKOUT_PENDING.
- **scope:** scheduled adapter del módulo, caso de uso reaper y puerto de transición guest/admin.
- **out_of_scope:** infraestructura de scheduler AWS, login HTTP y checkout/pagos.
- **inputs:** migración 004, ClockPort, session/cart repositories y lock rules.
- **implementation_notes:** cada minuto, lote ≤500, timeout 5s, retries 1/5/15s; transición condicional e idempotente; guest→admin libera ACTIVE, cierra carrito y revoca guest en transacción.
- **edge_cases:** dos reapers concurrentes; fallo parcial rollback; reservas CHECKOUT_PENDING intactas; métricas/log sin PII.
- **done_criteria:** a lo sumo una liberación por reserva y contador consistente; sesión admin queda sin carrito.
- **verification:** unitarias, integración y pruebas concurrentes/reintentos; evidencia de métricas requeridas.
- **dependencies:** MSF-CART-001, MSF-ID-001, MSF-DATA-002
- **handoff_context:** checkout depende directamente de ports de cart-reservation.
- **source_of_truth:** Master Spec §93 y ADR-008/014.
- **stale_terms_guard:** no liberar CHECKOUT_PENDING, no timeout de checkout, no log con PII.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### API — checkout/orders/payments/refunds/webhooks/reconciliation

#### MSF-PAY-001 — Implementar checkout y orders con cálculo COP contractual
- **agent:** executor
- **spec_refs:** OpenAPI `/checkouts`, `/orders`; Master Spec AC-08, §91-95; ADR-009
- **goal:** convertir reservas ACTIVE en checkout pendiente y crear snapshots de orden sin duplicados.
- **scope:** módulos `checkout` y `orders`, puertos a cart-reservation, persistencia 005 y controllers existentes.
- **out_of_scope:** adaptadores Wompi/Mercado Pago, webhooks, refunds y frontend.
- **inputs:** cart ports, migración 005, `CreateCheckoutRequest`, idempotencia.
- **implementation_notes:** cliente únicamente; lock reservas; snapshots de dirección; IVA `floor((subtotal*19+50)/100)`, entrega 5000, total exacto; orden única por carrito.
- **edge_cases:** reserva expirada/no activa 422; checkout concurrente solo una orden/pago pendiente; admin 403; `CHECKOUT_PENDING` persiste.
- **done_criteria:** response y paginación de órdenes coinciden con OpenAPI; no se crea pago si checkout inválido.
- **verification:** unitarias de cálculo/Failure, integración transaccional, concurrencia e HTTP contract.
- **dependencies:** MSF-CART-001, MSF-DATA-002, MSF-API-002, MSF-API-003
- **handoff_context:** payments consumirá payment intent creado por este flujo y holds pendientes.
- **source_of_truth:** OpenAPI checkout/orders y Master Spec AC-08.
- **stale_terms_guard:** no dirección en perfil, no IVA distinto, no `base_fee_cop`, no checkout admin.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-PAY-002 — Implementar puertos y adapters de Wompi/Mercado Pago
- **agent:** executor
- **spec_refs:** ADR-005; OpenAPI `ProviderWebhookPayload`, payment schemas; Master Spec §91-95, §62-71
- **goal:** encapsular creación de pagos y refund en Strategy/Adapter sin filtrar SDK al dominio.
- **scope:** `payments` ports, estrategias Wompi/Mercado Pago, configuración de timeout/retries y mapping técnico.
- **out_of_scope:** secretos AWS/CI, webhook controller, conciliación y UI.
- **inputs:** checkout output, migración 005, contratos de proveedor ya aprobados en spec.
- **implementation_notes:** timeout 10s; retries 0.5/2/8 solo red/5xx; refunds 1m/5m/15m/1h/6h; no PAN/CVV/fecha.
- **edge_cases:** proveedor no clasificable→`TECHNICAL_DEPENDENCY_FAILURE`; idempotencia por payment/refund; no registrar payload sensible.
- **done_criteria:** application conoce solo ports; ambos adapters son intercambiables y traducen excepciones.
- **verification:** unitarias con fake ports, adapter contract tests y tests de retry/timeouts sin secretos.
- **dependencies:** MSF-PAY-001, MSF-DATA-002, MSF-API-001
- **handoff_context:** webhook y reconciliación invocan los mismos ports.
- **source_of_truth:** ADR-005 y Master Spec §38-41, §95.
- **stale_terms_guard:** no SDK en domain/application, no tarjeta cruda, no reintentos para errores de negocio.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-PAY-003 — Implementar webhooks firmados, deduplicación y consumo de hold/refund
- **agent:** executor
- **spec_refs:** OpenAPI `/webhooks/wompi`, `/webhooks/mercado-pago`; Master Spec §38-41, §62-71; ADR-005
- **goal:** procesar únicamente webhooks autenticados y hacer transición de pago/stock idempotente.
- **scope:** raw-body controllers, signature adapters, payment use cases, webhook/event/refund persistence.
- **out_of_scope:** cambios de contrato, polling de conciliación y notificaciones.
- **inputs:** headers/body OpenAPI, migración 005, payment/cart ports.
- **implementation_notes:** verificar firma raw body antes de persistir; deduplicar provider event; APPROVED consume hold y descuenta stock en transacción; fallo de hold crea refund idempotente sin decremento.
- **edge_cases:** firma inválida 401 sin persistir; evento duplicado 204; transición inválida; hold no consumible `PAYMENT_HOLD_NOT_CONSUMABLE`.
- **done_criteria:** estados y refunds siguen enum/response contract; handlers no contienen reglas ni Prisma.
- **verification:** unitarias, firma raw-body, integración, concurrencia de eventos y prueba de compensación.
- **dependencies:** MSF-PAY-002, MSF-PAY-001, MSF-CART-002, MSF-DATA-002
- **handoff_context:** conciliación reutiliza deduplicación y compensación.
- **source_of_truth:** OpenAPI webhook descriptions y ADR-005.
- **stale_terms_guard:** no aceptar llamadas de cliente, no persistir firma inválida, no doble consumo/descuento/refund.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-PAY-004 — Implementar reconciliación programada y retención técnica
- **agent:** executor
- **spec_refs:** Master Spec §89, §95; prisma contract §20-27; ADR-005/016
- **goal:** reconciliar pagos no terminales y aplicar purgas técnicas permitidas sin borrar evidencia requerida.
- **scope:** scheduled driving adapter de payments, casos de uso de reconciliación, métricas y jobs de retención.
- **out_of_scope:** decisión legal definitiva, despliegue AWS y exportación de datos.
- **inputs:** payment ports, tablas 005, reglas de 15m/retención.
- **implementation_notes:** reconciliación cada 15m; terminaliza solo con proveedor; retries/refunds idempotentes; tokens/carritos terminales 30d, sesiones operativas, logs 90d, órdenes/pagos/auditoría 5 años provisionales.
- **edge_cases:** proveedor caído→failure/retry; snapshots no se eliminan antes de obligación; logs sin PII.
- **done_criteria:** métricas requeridas y reintentos trazables; no purga de producto v1.
- **verification:** tests de reloj, idempotencia, integración y retención sobre datos sintéticos.
- **dependencies:** MSF-PAY-003, MSF-DATA-002
- **handoff_context:** AWS/CI solo empaquetará este adapter en la etapa posterior.
- **source_of_truth:** Master Spec §89, §95 y prisma contract §23.
- **stale_terms_guard:** no hard delete de producto, no retención inventada, no PII en logs.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### Storefront React/Redux es-CO

#### MSF-STORE-001 — Crear shell React/Redux y cliente API generado desde OpenAPI
- **agent:** executor
- **spec_refs:** ADR-001; Master Spec §8, AC-02, §10; OpenAPI completo
- **goal:** establecer SPA storefront y cliente tipado sin persistir tokens/carrito.
- **scope:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-storefront/src` y configuración frontend.
- **out_of_scope:** componentes de negocio admin, modificar OpenAPI, carrito funcional completo y despliegue AWS.
- **inputs:** OpenAPI vigente y endpoints implementados.
- **implementation_notes:** Redux solo estado derivado; access token en memoria; cookie HttpOnly gestionada por browser; catálogo i18n `es-CO`, COP entero.
- **edge_cases:** 401 refresh/logout, 403 admin, 410 carrito expirado, mensajes `ApiErrorResponse` localizados.
- **done_criteria:** shell compilable, cliente generado/validado desde contrato, no localStorage/sessionStorage para token/carrito.
- **verification:** tests de store y cliente; inspección de persistencia y serialización.
- **dependencies:** MSF-API-003
- **handoff_context:** pantallas de catálogo y carrito se implementan en tareas separadas.
- **source_of_truth:** ADR-001, Master Spec AC-02 y OpenAPI.
- **stale_terms_guard:** no Next.js, no carrito local, no literales ingleses visibles.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-STORE-002 — Implementar navegación pública, catálogo y detalle es-CO
- **agent:** executor
- **spec_refs:** OpenAPI `/categories`, `/products`, `/products/{productId}`, `/banners`; Master Spec AC-01, §10
- **goal:** permitir explorar categorías, banners, búsqueda, paginación y producto.
- **scope:** features/pantallas/catalog selectors en storefront.
- **out_of_scope:** checkout, admin, cambios API y persistencia de carrito.
- **inputs:** cliente API de MSF-STORE-001 y schemas Product/Category/Banner.
- **implementation_notes:** paginación contractual; precios COP enteros; accesibilidad y mensajes es-CO; imagen URLs privadas/CloudFront según respuesta.
- **edge_cases:** página vacía, q inválida, producto soft-deleted/404 y error técnico.
- **done_criteria:** UI muestra estados loading/error/empty y datos contractuales sin inventar campos.
- **verification:** unitarias componentes/selectors y E2E de navegación/búsqueda/detalle.
- **dependencies:** MSF-STORE-001, MSF-CAT-002
- **handoff_context:** carrito añade reservas usando la misma ficha de producto.
- **source_of_truth:** OpenAPI catálogo y Master Spec AC-01.
- **stale_terms_guard:** no inventar endpoints/filtros, no decimales COP, no Strapi.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-STORE-003 — Implementar auth, perfil, carrito y checkout del cliente
- **agent:** executor
- **spec_refs:** OpenAPI Auth/Profile/Cart/Checkout/Orders; Master Spec AC-02/03/04/07/08
- **goal:** completar flujos de cliente/guest sin guardar secretos ni carrito local.
- **scope:** features auth, profile, cart, checkout, orders del storefront.
- **out_of_scope:** administración, proveedor de pagos externo, cambios OpenAPI y token delivery.
- **inputs:** API identity/cart/checkout implementadas, cliente tipado.
- **implementation_notes:** guest cookie; Redux vista derivada; login cliente conserva carrito; admin muestra 403; checkout presenta IVA/entrega/total contractual; textos es-CO.
- **edge_cases:** refresh, session expired, reservation expired, idempotency retry, admin forbidden, must-change-password.
- **done_criteria:** flujos críticos funcionan contra API y no hay token/carrito en storage del navegador.
- **verification:** tests unitarios de slices y E2E guest→cart→login cliente→checkout; E2E admin bloqueado.
- **dependencies:** MSF-STORE-002, MSF-ID-001, MSF-ID-003, MSF-CART-001, MSF-PAY-001
- **handoff_context:** test-architect cubre contratos E2E y concurrencia en API; UI no duplica reglas.
- **source_of_truth:** OpenAPI operaciones citadas.
- **stale_terms_guard:** no compra admin, no dirección de perfil, no carrito local, no guardar JWT.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### Admin React/Refine/Redux es-CO

#### MSF-ADMIN-001 — Crear shell Admin React/Refine/Redux y guards de sesión
- **agent:** executor
- **spec_refs:** ADR-001; Master Spec AC-03/05, §73-80; OpenAPI Admin/Auth
- **goal:** establecer SPA admin separada con autorización y estado seguro.
- **scope:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-admin/src` y configuración.
- **out_of_scope:** recursos CRUD, provisión funcional y AWS.
- **inputs:** cliente API OpenAPI, identity endpoints.
- **implementation_notes:** Refine data/provider solo llama API; Redux no persiste tokens/carritos; guards para admin y `must_change_password`; UI es-CO.
- **edge_cases:** 401/403/INITIAL_PASSWORD_CHANGE_REQUIRED, logout, refresh y respuesta de sesión admin.
- **done_criteria:** shell compilable y navegación protegida; ninguna pantalla de compra/carrito.
- **verification:** unitarias de guards/provider y E2E login/forbidden.
- **dependencies:** MSF-API-003, MSF-ID-001, MSF-ID-003
- **handoff_context:** CRUD de catalog/media y consultas/orders se agregan sin crear endpoints.
- **source_of_truth:** ADR-001 y OpenAPI Admin/Auth.
- **stale_terms_guard:** no storefront purchase, no carrito admin, no auto-registro admin.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-ADMIN-002 — Implementar gestión administrativa de catálogo/media/stock
- **agent:** executor
- **spec_refs:** OpenAPI Admin/Media; Master Spec AC-09/10/12; ADR-006, ADR-011, ADR-012
- **goal:** permitir CRUD administrativo contractual con optimistic locking, media y ajuste auditado.
- **scope:** recursos Refine para categorías/productos/banners/media upload/stock adjustment.
- **out_of_scope:** órdenes mutables, checkout, AWS provisioning y cambios OpenAPI.
- **inputs:** APIs catalog/media/stock, cliente generado, If-Match e Idempotency-Key.
- **implementation_notes:** enviar If-Match en edits generales; separar stock adjustment; mostrar conflicto 409 sin sobrescribir; media usa URL prefirmada; COP/es-CO.
- **edge_cases:** soft delete, categoría ocupada, upload expirado, stock below reserved, password change required.
- **done_criteria:** operaciones existentes reflejadas sin campos inventados y errores localizados.
- **verification:** tests de formularios/data provider y E2E If-Match/stock/idempotencia.
- **dependencies:** MSF-ADMIN-001, MSF-CAT-001, MSF-CAT-002, MSF-CAT-003
- **handoff_context:** consultas de órdenes son read-only en tarea siguiente.
- **source_of_truth:** OpenAPI Admin/Media y ADR-011/012.
- **stale_terms_guard:** no editar stock vía PATCH general, no hard delete, no bucket público.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-ADMIN-003 — Implementar provisión/activación admin y consulta read-only de órdenes
- **agent:** executor
- **spec_refs:** OpenAPI `/admin/users`, `/auth/admin-activations`, `/admin/orders`; Master Spec AC-05/06, ADR-013/015
- **goal:** permitir provisionar/activar admins y consultar órdenes sin mutarlas.
- **scope:** pantallas/forms de provisionamiento, activación pública y list/detail de órdenes admin.
- **out_of_scope:** entrega del token por API, mutación de órdenes, compra, notificaciones y nuevos endpoints.
- **inputs:** APIs identity/admin-query, schemas y estados de error.
- **implementation_notes:** nunca mostrar/guardar token o password en logs/UI fuera del canal operativo aprobado; activación consume formulario; órdenes paginadas y filtros contractuales.
- **edge_cases:** idempotency replay/divergente, activation expired/used, admin initial password, 403.
- **done_criteria:** admin-query solo lectura; responses no exponen secretos; UI es-CO.
- **verification:** unitarias y E2E provisión→activación controlada y consulta de orden.
- **dependencies:** MSF-ADMIN-001, MSF-ID-002, MSF-PAY-001
- **handoff_context:** final QA verifica que Admin no tiene rutas de compra.
- **source_of_truth:** OpenAPI operaciones citadas y ADR-013/015.
- **stale_terms_guard:** no auto-registro, no token en claro, no endpoint de mutación de órdenes.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### Pruebas unitarias/integración/concurrencia/E2E/cobertura

#### MSF-TEST-001 — Configurar calidad, arquitectura y cobertura del API
- **agent:** test-architect
- **spec_refs:** Master Spec §60-71; Requirements CA-09..11; testing-strategy
- **goal:** hacer obligatorias pruebas ROP, boundaries y cobertura mínima 85% por archivo testable.
- **scope:** configuración de tests del API, dependency-cruiser, cobertura y fixtures.
- **out_of_scope:** corregir implementación funcional y pruebas E2E frontend.
- **inputs:** API scaffolding y módulos implementados.
- **implementation_notes:** Vitest/Jest según stack NestJS; dependency-cruiser bloquea imports prohibidos; tests de Result Success/Failure y adapter translation; Testcontainers PostgreSQL, nunca H2.
- **edge_cases:** excluir solo DTOs pasivos/config/código generado; no ocultar violaciones de arquitectura.
- **done_criteria:** CI/local test command documentado; umbral 85% y dominio/application idealmente 100%; architecture test falla ante drift.
- **verification:** ejecutar suite y registrar evidencia real de cobertura, boundaries y migraciones.
- **dependencies:** MSF-PAY-004, MSF-CAT-003, MSF-CART-002, MSF-ID-003
- **handoff_context:** resultados bloquean cierre si falta cobertura o hay imports prohibidos.
- **source_of_truth:** testing-strategy §2-5 y Master Spec §71.
- **stale_terms_guard:** no mocks que sustituyan persistencia real, no H2, no excepción de negocio.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-TEST-002 — Cubrir integración y concurrencia de identidad, catálogo y carrito
- **agent:** test-architect
- **spec_refs:** Master Spec §62-68; prisma contract §25-27; CA-04, CA-05, CA-09, CA-10
- **goal:** demostrar invariantes transaccionales y carreras de los módulos previos.
- **scope:** tests API de identity/catalog/cart con PostgreSQL/Testcontainers y pruebas paralelas.
- **out_of_scope:** nueva funcionalidad y pruebas de proveedores externos.
- **inputs:** módulos implementados, fixtures sin secretos, migraciones 001-004.
- **implementation_notes:** activación concurrente un éxito; If-Match; último stock; reaper doble; guest→admin libera ACTIVE pero conserva CHECKOUT_PENDING.
- **edge_cases:** reintentos idempotentes, rollback, sesiones expiradas y contadores.
- **done_criteria:** evidencia de cada caso y sin PII/secrets en logs de test.
- **verification:** suite integración/concurrencia reproducible y cobertura por archivo ≥85%.
- **dependencies:** MSF-TEST-001, MSF-ID-002, MSF-CAT-003, MSF-CART-002
- **handoff_context:** pagos reutiliza infraestructura de fixtures y patrones de concurrencia.
- **source_of_truth:** Master Spec §64-68 y prisma contract §27.
- **stale_terms_guard:** no validar con estado eventual sin transacción; no omitir failures.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-TEST-003 — Cubrir checkout, pagos, webhooks, refunds y reconciliación
- **agent:** test-architect
- **spec_refs:** Master Spec §68-71, §91-95; ADR-005/009; CA-08..10
- **goal:** verificar exactamente una vez, compensación y estados terminales.
- **scope:** tests unitarios/integración/concurrencia/contract de checkout y payments.
- **out_of_scope:** cambiar reglas o proveedores reales en pruebas.
- **inputs:** módulos pagos implementados, Testcontainers PostgreSQL y fakes de provider.
- **implementation_notes:** checkout concurrente uno; firma raw-body; evento duplicado; aprobación consume hold; falta de hold crea refund sin stock decrement; reconciliación 15m.
- **edge_cases:** retries solo técnicos, transición inválida, refund repetido y `CHECKOUT_PENDING` no expirado.
- **done_criteria:** cada Failure catalogado tiene test; statuses/codes OpenAPI verificados; cobertura ≥85%.
- **verification:** suite reproducible con concurrencia y pruebas de contrato HTTP.
- **dependencies:** MSF-TEST-001, MSF-PAY-004
- **handoff_context:** E2E frontend consume escenarios aprobados, no mocks de UI para reglas API.
- **source_of_truth:** Master Spec §68-71 y ADR-005.
- **stale_terms_guard:** no doble descuento, refund no idempotente ni webhook sin firma.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

#### MSF-TEST-004 — Ejecutar matriz E2E de Storefront/Admin y revisión final de aceptación
- **agent:** test-architect
- **spec_refs:** Master Spec AC-01..12; Requirements CA-01..11; OpenAPI completo
- **goal:** validar flujos críticos de ambas SPAs contra API real o entorno efímero equivalente.
- **scope:** Playwright/Puppeteer E2E en storefront/admin, matriz de roles, errores y locale.
- **out_of_scope:** infraestructura AWS/CI y corrección de código fuera de hallazgos.
- **inputs:** SPAs y API implementadas, seed no productivo, credenciales de test generadas fuera del repositorio.
- **implementation_notes:** guest browse/cart, cliente checkout/order/profile/password, admin provisioning/catalog/stock/orders read-only, prohibiciones admin, es-CO/COP.
- **edge_cases:** refresh, 401/403/410/409/422, If-Match, idempotency retry y no exposición de storage.
- **done_criteria:** matriz AC-01..12 con evidencia; no se detectan endpoints inventados ni compra admin.
- **verification:** suite E2E y reporte de cobertura de flujos; revisión manual de riesgos no bloqueantes.
- **dependencies:** MSF-STORE-003, MSF-ADMIN-003, MSF-TEST-002, MSF-TEST-003
- **handoff_context:** solo después de esta tarea puede prepararse la etapa AWS/CI/CD.
- **source_of_truth:** Master Spec AC globales y OpenAPI.
- **stale_terms_guard:** no navegador con carrito local/token persistido, no UI inglesa, no rutas nuevas.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### AWS/CI/CD — etapa posterior

#### MSF-OPS-001 — Preparar AWS/CI/CD solo después de validación funcional
- **agent:** executor
- **spec_refs:** ADR-006/007; Master Spec AC-12, §89-95; Requirements fuera de alcance operacional
- **goal:** empaquetar y desplegar API/SPAs con seguridad operacional, sin cambiar comportamiento contractual.
- **scope:** configuración posterior para ECS Fargate/RDS/CloudWatch/Secrets Manager, S3 privado+CloudFront/OAC, CI/CD y jobs.
- **out_of_scope:** cualquier cambio a OpenAPI, dominio, migraciones, endpoints, reglas de negocio o aprobación de producción.
- **inputs:** MSF-TEST-004 verde; artefactos API/SPAs; decisiones ADR-006/007; secretos suministrados por plataforma.
- **implementation_notes:** buckets privados, OAC, allowlists CSP/CORS, secrets fuera de repo; bootstrap admin recibe referencia externa; pipelines validan migraciones, tests, dependency-cruiser y cobertura.
- **edge_cases:** rollback, migración expand/contract, proveedor caído, DLQ/replay selectivo, no exponer activation token.
- **done_criteria:** despliegue reproducible y observabilidad de métricas/logs requeridos; checklist preproducción legal/contable pendiente explícito.
- **verification:** revisión de infraestructura/config, dry-run de pipeline y pre-flight con evidencia; no ejecutar scripts durante esta descomposición.
- **dependencies:** MSF-TEST-004
- **handoff_context:** producción requiere revisión legal/contable de NC-08 y canal operativo seguro de activación.
- **source_of_truth:** ADR-006/007, Master Spec §89-95.
- **stale_terms_guard:** no secretos versionados, bucket público, Step Functions v1, notificación de compra v1 ni producción sin revisiones pendientes.
- **status:** `blocked`
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

## Riesgos y decisiones no implementables en esta descomposición

1. El canal operacional seguro para entregar tokens de activación sigue abierto; no bloquea el API porque queda fuera del contrato, pero sí la operación productiva.
2. La retención/anonimización NC-08 requiere revisión legal/contable antes de producción.
3. Las rutas y schemas son los del OpenAPI vigente; cualquier necesidad nueva exige volver a Planner y revalidar el conjunto canónico.
4. Los tres repositorios están vacíos salvo README/gitignore; Executor debe crear la estructura prevista dentro de cada repositorio, sin asumir archivos históricos.

## Registro de ejecución

Cada Executor actualiza únicamente en su tarea: `status`, `executor_notes`, `verification_result`, archivos cambiados y `blocker`. Un bloqueo debe incluir `blocked_reason`, `conflicting_artifacts`, `required_owner` y `next_required_decision`.
