# Graph Report - merkee-workspace  (2026-08-16)

## Corpus Check
- 184 files · ~98,136 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 1245 nodes · 2603 edges · 87 communities (77 shown, 10 thin omitted)
- Extraction: 98% EXTRACTED · 2% INFERRED · 0% AMBIGUOUS · INFERRED: 50 edges (avg confidence: 0.8)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `89bcd155`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- request-validators.ts
- login.use-case.spec.ts
- domain-error-mapper.ts
- login.use-case.ts
- devDependencies
- compilerOptions
- bootstrap-initial-admin.use-case.spec.ts
- domainError
- Result
- result.ts
- identity.module.ts
- IdentityController
- dto.ts
- operation-map.ts
- fail
- prisma-idempotency-purge-repository.adapter.ts
- AdminActivationTokenRepositoryPort
- prometheus-purge-metrics.adapter.ts
- schemas.ts
- dependencies
- prisma.service.ts
- purge-idempotency-records.use-case.ts
- scripts
- InMemoryPurgeMetricsAdapter
- admin-provision-scope-evaluator.adapter.ts
- payment-provider.port.ts
- seed.ts
- provision-admin-user.use-case.spec.ts
- PurgeMetricsPort
- PrismaService
- inventory-reservation.port.ts
- product-repository.port.ts
- list-orders.use-case.ts
- purge-idempotency-records.use-case.spec.ts
- prisma-activate-admin-unit-of-work.adapter.ts
- ScheduledIdempotencyPurgeAdapter
- OrderRepositoryPort
- integration-purge-msf-id-002.ts
- app.module.ts
- schemas.spec.ts
- order-query.port.ts
- activate-admin.use-case.spec.ts
- jest
- media-storage.port.ts
- catalog.module.ts
- package.json
- cart-reservation.module.ts
- checkout.module.ts
- CheckoutReservationPort
- PrismaProvisionUnitOfWorkAdapter
- admin-query.module.ts
- Tareas
- env-initial-admin-secret.adapter.ts
- media.module.ts
- orders.module.ts
- payments.module.ts
- TransportAuthGuard
- nest-cli.json
- Delta Spec — merkee-shop-foundation-stabilization
- cart-repository.port.ts
- logout.use-case.spec.ts
- Argon2PasswordHasherAdapter
- Shared Context — merkee-shop-foundation-stabilization
- category-repository.port.ts
- payment-repository.port.ts
- idempotency-response-rename.spec.ts
- ClockPort
- parameters.ts
- Master Spec — merkee.shop
- Controlled Pre-Ready Remediation: approved_by_user
- Shared Context — merkee-shop-foundation
- Requirements Brief — merkee.shop Foundation
- exclude
- Seed NO PRODUCTIVO — merkee.shop API (MSF-DATA-003)
- Contrato de migración Prisma — merkee.shop
- list-products.use-case.ts
- Decisiones de arquitectura — merkee.shop
- Registro global de deuda técnica
- merkee-shop-admin
- Registro local de deuda técnica — merkee-shop-api
- merkee-shop-api
- merkee-shop-storefront
- ts-loader
- ts-node
- @types/jest

## God Nodes (most connected - your core abstractions)
1. `domainError` - 57 edges
2. `Result` - 38 edges
3. `ok()` - 27 edges
4. `fail()` - 26 edges
5. `UserRepositoryPort` - 25 edges
6. `createContext()` - 24 edges
7. `toResult()` - 24 edges
8. `technicalFailure()` - 24 edges
9. `SessionRepositoryPort` - 23 edges
10. `checkRecord()` - 22 edges

## Surprising Connections (you probably didn't know these)
- `LoginResult` --references--> `SessionDto`  [EXTRACTED]
  projects/merkee-shop-api/src/modules/identity/application/use-cases/login.use-case.ts → projects/merkee-shop-api/src/contract/application/dto.ts
- `RefreshSessionResult` --references--> `SessionDto`  [EXTRACTED]
  projects/merkee-shop-api/src/modules/identity/application/use-cases/refresh-session.use-case.ts → projects/merkee-shop-api/src/contract/application/dto.ts
- `RegisterResult` --references--> `SessionDto`  [EXTRACTED]
  projects/merkee-shop-api/src/modules/identity/application/use-cases/register.use-case.ts → projects/merkee-shop-api/src/contract/application/dto.ts
- `stubUnitOfWork()` --calls--> `fail()`  [EXTRACTED]
  projects/merkee-shop-api/src/modules/identity/application/use-cases/bootstrap-initial-admin.use-case.spec.ts → projects/merkee-shop-api/src/shared/domain/result.ts
- `stubUnitOfWork()` --calls--> `technicalFailure()`  [EXTRACTED]
  projects/merkee-shop-api/src/modules/identity/application/use-cases/provision-admin-user.use-case.spec.ts → projects/merkee-shop-api/src/modules/identity/domain/identity-errors.ts

## Import Cycles
- None detected.

## Communities (87 total, 10 thin omitted)

### Community 0 - "request-validators.ts"
Cohesion: 0.19
Nodes (40): validateIdempotencyKey(), validateIfMatch(), validateWebhookHeader(), checkNoAdditionalProperties(), validateAdminActivationRequest(), validateBannerWriteRequest(), validateCartItemMutationRequest(), validateCategoryWriteRequest() (+32 more)

### Community 1 - "login.use-case.spec.ts"
Cohesion: 0.06
Nodes (44): createUseCase(), existingAdmin, existingUser, fixedDate, stubCartReservation(), stubClock(), stubCookieToken(), stubJwt() (+36 more)

### Community 2 - "domain-error-mapper.ts"
Cohesion: 0.10
Nodes (23): Catch, ApiErrorDetail, ApiErrorResponse, isApiErrorResponse(), DEFAULT_MESSAGE_BY_CODE, DOMAIN_ERROR_HTTP, HTTP_ERROR_PHRASE, mapDomainError() (+15 more)

### Community 3 - "login.use-case.ts"
Cohesion: 0.11
Nodes (20): SessionDto, ActivateAdminCommand, ActivateAdminUseCase, ActivationOutcome, LoginCommand, LoginResult, LoginUseCase, LogoutUseCase (+12 more)

### Community 4 - "devDependencies"
Cohesion: 0.07
Nodes (29): jest, @nestjs/cli, @nestjs/schematics, @nestjs/testing, prisma, @prisma/client, devDependencies, dependency-cruiser (+21 more)

### Community 5 - "compilerOptions"
Cohesion: 0.07
Nodes (27): compilerOptions, allowSyntheticDefaultImports, baseUrl, declaration, emitDecoratorMetadata, esModuleInterop, experimentalDecorators, forceConsistentCasingInFileNames (+19 more)

### Community 6 - "bootstrap-initial-admin.use-case.spec.ts"
Cohesion: 0.10
Nodes (24): NoContentDto, BootstrapInitialAdminResult, BootstrapInitialAdminUseCase, BootstrapOutcome, INITIAL_ADMIN_DISPLAY_NAME, INITIAL_ADMIN_EMAIL, adminUser(), createUseCase() (+16 more)

### Community 7 - "domainError"
Cohesion: 0.10
Nodes (29): AdminUserProvisionDto, LogoutCommand, canonicalBodyHash(), ProvisionAdminUserCommand, ProvisionOutcome, ProvisionResponseSnapshot, toSnapshot(), ADR-0018 (+21 more)

### Community 8 - "Result"
Cohesion: 0.07
Nodes (25): AdminOrderView, ListAdminOrdersQuery, ListAdminOrdersResult, ListAdminOrdersUseCase, AddCartItemCommand, AddCartItemResult, AddCartItemUseCase, CreateCheckoutCommand (+17 more)

### Community 9 - "result.ts"
Cohesion: 0.26
Nodes (7): main(), prisma, FailureResult, isFailure(), isSuccess(), SuccessResult, ADR-0017

### Community 10 - "identity.module.ts"
Cohesion: 0.14
Nodes (15): ADR-0004, ADR-0015, IdentityModule, ADR-0010, ADR-0014, Module, IDENTITY_TOKENS, CookieTokenAdapter (+7 more)

### Community 11 - "IdentityController"
Cohesion: 0.19
Nodes (14): Body, Headers, HttpCode, Post, IdentityController, Controller, createHost(), MockHost (+6 more)

### Community 12 - "dto.ts"
Cohesion: 0.09
Nodes (21): BannerDto, CartDto, CategoryDto, CheckoutDto, OrderDto, PagedOrdersDto, PagedProductsDto, ProductDto (+13 more)

### Community 13 - "operation-map.ts"
Cohesion: 0.10
Nodes (20): findOperation(), HttpMethod, OperationContract, OPERATIONS, BannerWriteRequest, CartItemMutationRequest, CategoryWriteRequest, CreateAdminUserRequest (+12 more)

### Community 15 - "prisma-idempotency-purge-repository.adapter.ts"
Cohesion: 0.14
Nodes (12): PurgeBatchResult, PurgeCandidate, PurgeClassification, PurgeEvaluation, ADR-0018, parseActivationExpiresAt(), PrismaIdempotencyPurgeRepositoryAdapter, PURGE_ADVISORY_LOCK_KEY (+4 more)

### Community 16 - "AdminActivationTokenRepositoryPort"
Cohesion: 0.22
Nodes (5): AdminActivationToken, CreateAdminActivationTokenData, AdminActivationTokenRepositoryPort, PrismaAdminActivationTokenRepositoryAdapter, Injectable

### Community 17 - "prometheus-purge-metrics.adapter.ts"
Cohesion: 0.15
Nodes (9): PrometheusPurgeMetricsAdapter, PURGE_METRIC_DELETED, PURGE_METRIC_ERRORS, PURGE_METRIC_LAST_SUCCESS, PURGE_METRIC_RUNS, PURGE_METRIC_SKIPPED, ADR-0018, Injectable (+1 more)

### Community 18 - "schemas.ts"
Cohesion: 0.11
Nodes (17): ApiErrorDetail, ApiErrorResponse, CartItemResponse, CartStatus, CreateUploadUrlRequest, DeliveryAddressRequest, ImageResponse, OrderItemResponse (+9 more)

### Community 19 - "dependencies"
Cohesion: 0.12
Nodes (17): argon2, jsonwebtoken, @nestjs/common, @nestjs/core, @nestjs/platform-express, dependencies, argon2, jsonwebtoken (+9 more)

### Community 20 - "prisma.service.ts"
Cohesion: 0.20
Nodes (5): IdempotencyPort, IdempotencyRecord, PrismaIdempotencyAdapter, ADR-0018, Injectable

### Community 21 - "purge-idempotency-records.use-case.ts"
Cohesion: 0.15
Nodes (8): BATCH_RETRY_DELAYS_MS, SleepFn, ADR-0018, PurgeLoggerPort, ADR-0018, ConsolePurgeLoggerAdapter, ADR-0018, Injectable

### Community 22 - "scripts"
Cohesion: 0.13
Nodes (15): scripts, build, depcruise, depcruise:graph, prisma:generate, prisma:migrate:deploy, prisma:migrate:dev, prisma:seed (+7 more)

### Community 23 - "InMemoryPurgeMetricsAdapter"
Cohesion: 0.22
Nodes (7): PurgeOutcome, PurgeSkipReason, ADR-0018, InMemoryPurgeMetricsAdapter, PurgeMetricsSnapshot, ADR-0018, Injectable

### Community 24 - "admin-provision-scope-evaluator.adapter.ts"
Cohesion: 0.20
Nodes (8): ADMIN_PROVISION_SCOPE_PREFIX, ADR-0018, IdempotencyScopeEvaluatorPort, ADR-0018, AdminProvisionScopeEvaluatorAdapter, isValidAdminProvisionScope(), ADR-0018, Injectable

### Community 25 - "payment-provider.port.ts"
Cohesion: 0.25
Nodes (8): CreatePaymentRequest, CreatePaymentResult, PaymentProviderPort, RefundRequest, RefundResult, ADR-0005, PaymentProviderAdapter, Injectable

### Community 26 - "seed.ts"
Cohesion: 0.15
Nodes (11): BANNER_IDS, banners, BannerSeed, categories, CATEGORY_IDS, CategorySeed, prisma, PRODUCT_IDS (+3 more)

### Community 27 - "provision-admin-user.use-case.spec.ts"
Cohesion: 0.23
Nodes (11): adminActor, adminPending, command, createUseCase(), fixedDate, stubActivationTokenRepo(), stubClock(), stubCookieToken() (+3 more)

### Community 28 - "PurgeMetricsPort"
Cohesion: 0.26
Nodes (3): PurgeIdempotencyRecordsUseCase, IdempotencyPurgeRepositoryPort, PurgeMetricsPort

### Community 29 - "PrismaService"
Cohesion: 0.14
Nodes (6): Inject, Inject, Inject, Inject, PrismaService, Injectable

### Community 30 - "inventory-reservation.port.ts"
Cohesion: 0.23
Nodes (6): InventoryReservationPort, ReservationResult, ADR-0008, ADR-0013, InventoryReservationAdapter, Injectable

### Community 31 - "product-repository.port.ts"
Cohesion: 0.26
Nodes (6): ProductPage, ProductRecord, ProductRepositoryPort, ADR-0012, ProductRepositoryAdapter, Injectable

### Community 32 - "list-orders.use-case.ts"
Cohesion: 0.33
Nodes (4): ListOrdersQuery, ListOrdersResult, ListOrdersUseCase, OrderView

### Community 33 - "purge-idempotency-records.use-case.spec.ts"
Cohesion: 0.29
Nodes (11): candidate(), captureEvaluate(), createUseCase(), daysAgo(), emptySkipped(), now, stubClock(), stubLogger() (+3 more)

### Community 34 - "prisma-activate-admin-unit-of-work.adapter.ts"
Cohesion: 0.24
Nodes (4): ActivateAdminTransaction, ActivateAdminUnitOfWorkPort, PrismaActivateAdminUnitOfWorkAdapter, Injectable

### Community 35 - "ScheduledIdempotencyPurgeAdapter"
Cohesion: 0.25
Nodes (4): ScheduledIdempotencyPurgeAdapter, Inject, Injectable, Optional

### Community 36 - "OrderRepositoryPort"
Cohesion: 0.29
Nodes (5): OrderPage, OrderRecord, OrderRepositoryPort, OrderRepositoryAdapter, Injectable

### Community 37 - "integration-purge-msf-id-002.ts"
Cohesion: 0.28
Nodes (6): expectRollback(), insertRecord(), main(), prisma, SystemClockAdapter, Injectable

### Community 38 - "app.module.ts"
Cohesion: 0.27
Nodes (5): AppModule, ADR-0013, Module, HttpModule, Module

### Community 39 - "schemas.spec.ts"
Cohesion: 0.20
Nodes (9): AdminActivationRequest, AdminUserProvisionResponse, CartResponse, OrderResponse, PaymentProvider, ReservationStatus, Role, KeysOf (+1 more)

### Community 40 - "order-query.port.ts"
Cohesion: 0.29
Nodes (6): AdminOrderPage, AdminOrderRecord, OrderQueryPort, ADR-0013, OrderQueryAdapter, Injectable

### Community 41 - "activate-admin.use-case.spec.ts"
Cohesion: 0.31
Nodes (9): activeToken, command, createUseCase(), fixedDate, stubActivationTokenRepo(), stubClock(), stubCookieToken(), stubPasswordHasher() (+1 more)

### Community 42 - "jest"
Cohesion: 0.15
Nodes (13): jest, collectCoverageFrom, coverageDirectory, moduleFileExtensions, rootDir, testEnvironment, testRegex, transform (+5 more)

### Community 43 - "media-storage.port.ts"
Cohesion: 0.33
Nodes (5): MediaStoragePort, ADR-0006, UploadUrl, MediaStorageAdapter, Injectable

### Community 44 - "catalog.module.ts"
Cohesion: 0.29
Nodes (6): ADR-0011, CatalogController, Controller, CatalogModule, ADR-0012, Module

### Community 45 - "package.json"
Cohesion: 0.25
Nodes (7): description, license, name, prisma, seed, private, version

### Community 46 - "cart-reservation.module.ts"
Cohesion: 0.29
Nodes (6): CartReservationController, Controller, CartReservationModule, ADR-0008, ADR-0014, Module

### Community 47 - "checkout.module.ts"
Cohesion: 0.29
Nodes (6): CheckoutController, Controller, CheckoutModule, ADR-0009, ADR-0013, Module

### Community 48 - "CheckoutReservationPort"
Cohesion: 0.32
Nodes (4): CheckoutReservationPort, ADR-0013, CheckoutReservationAdapter, Injectable

### Community 50 - "admin-query.module.ts"
Cohesion: 0.33
Nodes (5): AdminQueryController, Controller, AdminQueryModule, ADR-0013, Module

### Community 51 - "Tareas"
Cohesion: 0.04
Nodes (44): Admin React/Refine/Redux es-CO, API — cart-reservation, API — catalog/media, API — checkout/orders/payments/refunds/webhooks/reconciliation, API — identity, API — Prisma schema/migraciones PostgreSQL y seeds sin secretos, API — scaffolding NestJS modular hexagonal/ROP + contrato/schema, AWS/CI/CD — etapa posterior (+36 more)

### Community 52 - "env-initial-admin-secret.adapter.ts"
Cohesion: 0.38
Nodes (4): EnvInitialAdminSecretAdapter, INITIAL_ADMIN_PASSWORD_ENV, ADR-0010, Injectable

### Community 53 - "media.module.ts"
Cohesion: 0.33
Nodes (5): MediaController, Controller, MediaModule, ADR-0006, Module

### Community 54 - "orders.module.ts"
Cohesion: 0.33
Nodes (5): OrdersController, Controller, OrdersModule, ADR-0009, Module

### Community 55 - "payments.module.ts"
Cohesion: 0.33
Nodes (5): PaymentsController, Controller, PaymentsModule, ADR-0005, Module

### Community 56 - "TransportAuthGuard"
Cohesion: 0.33
Nodes (3): TRANSPORT_CODE_AUTH_REQUIRED, TransportAuthGuard, Injectable

### Community 57 - "nest-cli.json"
Cohesion: 0.33
Nodes (5): collection, compilerOptions, deleteOutDir, $schema, sourceRoot

### Community 58 - "Delta Spec — merkee-shop-foundation-stabilization"
Cohesion: 0.06
Nodes (32): (A) Remediación documental imprescindible, Alcance exacto (exhaustivo y exclusivo), Artefactos a actualizar, (B) Correcciones de código bloqueantes, (C) Deuda técnica posterior (no bloqueante para desarrollo local), Controlled Pre-Ready Remediation: approved_by_user, Criterios de aceptación, (D) Drifts resueltos por decisión del usuario (Opción A, 2026-08-16) (+24 more)

### Community 59 - "cart-repository.port.ts"
Cohesion: 0.33
Nodes (3): CartRecord, CartRepositoryPort, ADR-0008

### Community 60 - "logout.use-case.spec.ts"
Cohesion: 0.20
Nodes (8): activeSession, createUseCase(), fixedDate, stubCartReservation(), stubSessionRepo(), CartReservationPort, NoopCartReservationAdapter, Injectable

### Community 62 - "Shared Context — merkee-shop-foundation-stabilization"
Cohesion: 0.11
Nodes (18): Alcance exacto (exhaustivo y exclusivo), Artifact evidence, Canonical artifacts, Controlled Pre-Ready Remediation: approved_by_user, Current status, Decisions locked, Evidencia futura requerida (para cerrar B1-B5 tras aprobación; no habilita ni es precondición para la revalidación de Spec Validator), Lo que esta autorización NO autoriza (+10 more)

### Community 69 - "parameters.ts"
Cohesion: 0.12
Nodes (18): AdminListOrdersParams, BannerIdPath, CategoryIdPath, CategoryIdQuery, IdempotencyKeyHeader, IfMatchHeader, ListProductsParams, MercadoPagoRequestIdHeader (+10 more)

### Community 70 - "Master Spec — merkee.shop"
Cohesion: 0.13
Nodes (14): Aplicación por flujo y pruebas exigidas, Arquitectura y ownership bloqueados, Carrito, checkout y pagos, Catálogo estable `DomainError` y proyección OpenAPI, Corrección documental MSF-ID-002 — purga y operación, Criterios de aceptación globales, Datos, migraciones y retención v1 provisional, Decomposition Contract (+6 more)

### Community 71 - "Controlled Pre-Ready Remediation: approved_by_user"
Cohesion: 0.14
Nodes (13): Alcance exacto (exhaustivo y exclusivo), Cambio descendente vigente, Cambios completados con evidencia — no son deuda activa, Controlled Pre-Ready Remediation: approved_by_user, Coordinación y gate, Deuda técnica activa sincronizada, Evidencia requerida (para cerrar B1-B5 y habilitar revalidación), Lo que esta autorización NO autoriza (+5 more)

### Community 72 - "Shared Context — merkee-shop-foundation"
Cohesion: 0.15
Nodes (12): Artifact evidence, Canonical artifacts, Current status, Decisions locked, Human Plan Approval — HISTÓRICO / SUPERSEDED / INVALIDADO, Next action, Open questions, Resolved findings (+4 more)

### Community 73 - "Requirements Brief — merkee.shop Foundation"
Cohesion: 0.22
Nodes (8): Actores y permisos, Criterios de aceptación, Fuera de alcance y pendientes reales, Historial superseded, Objetivo y alcance, Requirements Brief — merkee.shop Foundation, Requisitos funcionales aprobados, Status

### Community 74 - "exclude"
Cohesion: 0.22
Nodes (8): exclude, extends, coverage, dist, node_modules, **/*spec.ts, test, ./tsconfig.json

### Community 75 - "Seed NO PRODUCTIVO — merkee.shop API (MSF-DATA-003)"
Cohesion: 0.25
Nodes (7): Ejecución, Idempotencia, Propósito, Qué NO hace, Requisitos previos, Seed NO PRODUCTIVO — merkee.shop API (MSF-DATA-003), Verificación

### Community 76 - "Contrato de migración Prisma — merkee.shop"
Cohesion: 0.29
Nodes (6): 007 — `idempotency_records`, 013 — `idempotency_records_response_json_strict_validate`, Consistencia y retención, Contrato de migración Prisma — merkee.shop, Pruebas futuras obligatorias, Secuencia futura y aceptación

### Community 77 - "list-products.use-case.ts"
Cohesion: 0.33
Nodes (4): ListProductsQuery, ListProductsResult, ListProductsUseCase, ProductView

### Community 78 - "Decisiones de arquitectura — merkee.shop"
Cohesion: 0.40
Nodes (4): Addendum ADR-018 — corrección vinculante MSF-ID-002, ADR-018 pre-addendum — HISTÓRICO/SUPERSEDED, Consulta de arquitecto requerida, Decisiones de arquitectura — merkee.shop

### Community 79 - "Registro global de deuda técnica"
Cohesion: 0.50
Nodes (3): Cambios completados — no son deuda activa, Deuda activa — sincronizada con `merkee-shop-api`, Registro global de deuda técnica

### Community 80 - "merkee-shop-admin"
Cohesion: 0.50
Nodes (3): merkee-shop-admin, Propósito, Ramas

### Community 81 - "Registro local de deuda técnica — merkee-shop-api"
Cohesion: 0.50
Nodes (3): Cambios completados — no son deuda activa, Deuda activa — espejo del registro global, Registro local de deuda técnica — merkee-shop-api

### Community 82 - "merkee-shop-api"
Cohesion: 0.50
Nodes (3): merkee-shop-api, Propósito, Ramas

### Community 83 - "merkee-shop-storefront"
Cohesion: 0.50
Nodes (3): merkee-shop-storefront, Propósito, Ramas

## Knowledge Gaps
- **411 isolated node(s):** `$schema`, `collection`, `sourceRoot`, `deleteOutDir`, `name` (+406 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **10 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `ScheduledIdempotencyPurgeAdapter` connect `ScheduledIdempotencyPurgeAdapter` to `identity.module.ts`?**
  _High betweenness centrality (0.020) - this node is a cross-community bridge._
- **Why does `domainError` connect `domainError` to `list-orders.use-case.ts`, `domain-error-mapper.ts`, `login.use-case.ts`, `bootstrap-initial-admin.use-case.spec.ts`, `Result`, `result.ts`, `IdentityController`, `list-products.use-case.ts`, `fail`, `PrismaProvisionUnitOfWorkAdapter`?**
  _High betweenness centrality (0.020) - this node is a cross-community bridge._
- **Why does `PrismaSessionRepositoryAdapter` connect `login.use-case.spec.ts` to `identity.module.ts`, `login.use-case.ts`, `prisma-activate-admin-unit-of-work.adapter.ts`, `PrismaService`?**
  _High betweenness centrality (0.013) - this node is a cross-community bridge._
- **What connects `$schema`, `collection`, `sourceRoot` to the rest of the system?**
  _411 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `login.use-case.spec.ts` be split into smaller, more focused modules?**
  _Cohesion score 0.05894736842105263 - nodes in this community are weakly interconnected._
- **Should `domain-error-mapper.ts` be split into smaller, more focused modules?**
  _Cohesion score 0.10227272727272728 - nodes in this community are weakly interconnected._
- **Should `login.use-case.ts` be split into smaller, more focused modules?**
  _Cohesion score 0.11382113821138211 - nodes in this community are weakly interconnected._