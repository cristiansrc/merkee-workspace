# Master Spec — merkee.shop

**Lifecycle status:** `planning`  
**Incremento:** `merkee-shop-foundation` · **Fecha:** 2026-08-14

## 1. Propósito y fuentes

Construir un supermercado digital en español para Colombia, con precios enteros en COP, storefront React SPA y una aplicación React administrativa separada. Esta especificación consolida la fuente funcional `docs/specs/requirements/merkee-shop-foundation-requirements-brief.md`, los requisitos de la prueba PDF y las decisiones confirmadas por la persona usuaria. La plantilla Next.js/Strapi se usará solo como referencia visual y de flujos; no se reutilizan Next.js ni Strapi.

La prueba exige SPA React, Redux, Jest, backend TypeScript/NestJS, separación de lógica de controladores, Hexagonal/ROP, flujo de pago de cinco pasos y sandbox. Sus credenciales visibles en el PDF son material sensible: quedan explícitamente prohibidas en código, documentación, ejemplos, repositorio o logs.

## 2. Alcance y criterios de aceptación

| ID | Requisito verificable |
|---|---|
| AC-01 | Un visitante ve home, categorías, productos populares, búsqueda y detalle con disponibilidad sin autenticarse. |
| AC-02 | Tanto invitado como cliente agregan, modifican y eliminan ítems mediante el carrito servidor. Cada cantidad activa una reserva de inventario; al reducir/eliminar el ítem se ajusta/libera atómicamente. Redux conserva solo la vista y el identificador opaco está en cookie `HttpOnly`, nunca en `localStorage`. |
| AC-03 | Registro crea únicamente rol `cliente`, inicia sesión; login no se solicita si la sesión sigue válida; recuperación no revela si un email existe. |
| AC-04 | Checkout exige autenticación, conserva la reserva del carrito al promover la sesión de invitado a cliente, solicita dirección y muestra `items_subtotal_cop + delivery_fee_cop (5000) + iva_cop` antes de pago. Calcula una sola vez `iva_cop = floor((items_subtotal_cop * 19 + 50) / 100)` (19%, redondeo comercial HALF_UP al peso COP) y `total_cop = items_subtotal_cop + 5000 + iva_cop`. |
| AC-05 | Wompi es la opción preseleccionada y Mercado Pago alternativa. La tarjeta se captura/tokeniza exclusivamente en el proveedor; la API nunca acepta PAN, CVV o fecha de vencimiento. |
| AC-06 | Una orden/pago pendiente se crea solo desde reservas activas del carrito. Mientras el checkout/pago está activo, el hold `CHECKOUT_PENDING` se conserva hasta que el pago alcance un estado terminal; webhook `APPROVED` vigente lo consume y descuenta `stock_on_hand` atómicamente. Si el proveedor aprueba tras expiración o sin hold/stock consumible, no descuenta stock y solicita reembolso automático idempotente. Timeout, logout, eliminación de ítem y fallos liberan reservas según su estado; nunca hay overselling. |
| AC-07 | Cliente consulta solo sus órdenes; admin consulta todas y no existe endpoint de mutación administrativa de órdenes. |
| AC-08 | Admin gestiona categorías, productos (exactamente una categoría), banners e imágenes S3. La edición general de producto no puede mutar `stock_on_hand` ni `stock_reserved`; el ajuste de existencias es exclusivamente `POST /v1/admin/products/{productId}/stock-adjustments`, auditado, idempotente y nunca deja `stock_on_hand < stock_reserved`. No se permite eliminar una categoría con productos activos. |
| AC-09 | API se ajusta a `docs/api/openapi.yaml`; PostgreSQL y Prisma Migrate se usan según el contrato de migración. |
| AC-10 | Storefront y admin son SPAs React móviles (mínimo iPhone SE 2020), usan Redux; Jest falla si cualquier archivo testeable queda bajo 85% de cobertura. |
| AC-11 | El bootstrap idempotente crea/valida el único admin inicial `cristiansrc@gmail.com` solo si obtiene la contraseña desde Secret Manager/entorno no versionado; deja `must_change_password=true`, no expone la contraseña/hash y bloquea operaciones protegidas hasta `POST /auth/initial-password-change`. |
| AC-12 | Todo texto visible para personas usuarias en storefront y admin está en español de Colombia (`es-CO`): navegación, botones, etiquetas, formularios, ayudas, validaciones, errores, confirmaciones, estados vacíos y estados de carrito, pago y órdenes. |
| AC-13 | Los correos de recuperación de contraseña y los mensajes de sistema orientados a personas usuarias se emiten en español de Colombia; el flujo de recuperación mantiene su respuesta neutra y no revela la existencia del correo. |
| AC-14 | La UI no contiene textos en inglés hardcodeados. Los textos visibles se resuelven desde un catálogo de traducciones/localización disponible desde el inicio del incremento; se permiten exclusivamente nombres propios de proveedores, marcas y términos técnicos necesarios. |
| AC-15 | Todo importe visible se presenta con localización colombiana y moneda COP, sin decimales, y conserva los valores enteros COP del contrato API. |

## 3. Arquitectura bloqueada

### 3.1 Componentes y despliegue

* `storefront`: React SPA + TypeScript, Redux Toolkit/RTK Query, Tailwind; adapta los patrones visuales de la plantilla.
* `admin`: React SPA + TypeScript, Refine, Redux Toolkit para sesión/estado transversal; origen y despliegue independientes del storefront.
 * `api`: NestJS TypeScript, monolito modular con módulos `identity`, `catalog`, `cart-reservation`, `media`, `checkout`, `orders`, `payments` y `admin-query`. PostgreSQL mediante Prisma. API REST `/v1` definida antes de implementación.
* Cada módulo respeta `src/{domain,application,infrastructure}`: controladores/webhooks son adaptadores de entrada; Prisma, S3, email y proveedores de pago son adaptadores de salida. Dominio sin NestJS/Prisma/HTTP. Los casos de uso devuelven `Result<Success, DomainError>` (ROP); un mapper de entrada convierte el resultado a la respuesta OpenAPI. Ningún controlador consulta Prisma directamente.
* Patrón **Strategy**: `PaymentProviderPort` tiene implementaciones `WompiPaymentProviderAdapter` y `MercadoPagoPaymentProviderAdapter`; un selector por `payment_provider` evita `if` de proveedor en casos de uso. Patrón **Adapter** para Prisma/S3/email/pagos. No se adopta Factory, CQRS ni microservicios: no resuelven una variabilidad confirmada adicional.

### 3.4 Idioma y localización de interfaces

El idioma canónico de toda interfaz visible al usuario es español de Colombia (`es-CO`) para las dos SPAs: `storefront` y `admin`. Este requisito incluye navegación, botones, llamadas a la acción, etiquetas, placeholders, formularios, textos de ayuda, validaciones de cliente, confirmaciones, errores mostrados, estados vacíos, disponibilidad, carrito, checkout, pago, órdenes y pantallas de autenticación/perfil.

Los correos de recuperación de contraseña y todo mensaje de sistema destinado a una persona usuaria también deben estar en español de Colombia. Los códigos, enums y campos de transporte del API conservan sus identificadores técnicos canónicos; la UI no los expone como texto de cara al usuario y resuelve el mensaje localizado correspondiente.

Desde el inicio se usará un catálogo de traducciones/localización como fuente de los textos visibles; no se permite texto de UI hardcodeado en inglés. Se pueden conservar sin traducción los nombres propios de proveedores o marcas (por ejemplo, Wompi y Mercado Pago) y términos técnicos necesarios cuando traducirlos perjudique su identificación. Los importes se calculan y transportan como enteros COP según este contrato, y se presentan usando la localización colombiana, símbolo/denominación COP y cero decimales.

**Nota de migración de plantilla.** La plantilla de referencia contiene textos en inglés y solo aporta referencia visual/funcional. Antes de adoptar cada pantalla o flujo de la plantilla, se debe inventariar cada texto visible y sustituirlo por su clave y contenido en español de Colombia en el catálogo canónico; no se copiarán literales ingleses. Esta migración no cambia rutas, DTOs, contratos OpenAPI ni cálculos monetarios.

**Consulta formal a Solution Architect (histórico superseded):** se solicitó validar el monolito modular, Strategy de pagos, outbox y el límite de bounded contexts antes de cualquier handoff. **Resuelta:** el `solution-architect` emitió veredicto final `ready-with-final-verdict` (2026-08-15), documentado en ADR-002/005/007/008/009/010/011/012/013 y en el shared context. El incremento permanece en `planning` a la espera de la revisión del Spec Validator y la aprobación humana.

### 3.2 AWS propuesta

CloudFront distribuye dos SPAs desde buckets S3 privados distintos (`storefront`, `admin`) con OAC; un dominio/origen por aplicación y CSP/CORS de allowlist. El hosting de ambas SPAs es **S3 privado + CloudFront con OAC** (sin acceso público al bucket; solo CloudFront lee mediante Origin Access Control), documentado en ADR-006. API NestJS corre en ECS Fargate detrás de ALB en subredes privadas, con autoscaling por CPU/requests; es preferible a Lambda porque webhooks, polling de reconciliación y conexiones Prisma se benefician de proceso persistente y capacidad predecible. RDS PostgreSQL Multi-AZ queda privado; S3 privado de media se sirve por CloudFront, con carga por URL prefirmada de corta duración. Secrets Manager almacena secretos de sesión, SMTP y proveedores; ECS obtiene secretos por IAM task role. CloudWatch centraliza logs JSON, métricas, alarmas y trazas.

Usar SNS+SQS con DLQ **solo** para publicaciones de outbox no críticas para confirmar el pago (email de recuperación y reconciliación diferida), con consumidores idempotentes. El webhook confirma stock dentro de transacción PostgreSQL y no depende de la cola. No usar Step Functions inicialmente: no hay workflow largo, humano ni multietapa que justifique su estado/coste; revaluar si aparecen fulfillment, reembolsos o conciliación multi-proveedor compleja.

### 3.3 Ownership y dependencias modulares

**Ownership de escrituras administrativas (C1).** El módulo `admin-query` es **solo lectura cross-cutting**: expone únicamente la consulta de órdenes de todos los clientes (`GET /v1/admin/orders` y `GET /v1/admin/orders/{orderId}`) y no posee ninguna mutación. Las escrituras administrativas pertenecen a sus módulos de origen: categorías, productos y ajustes de stock a `catalog`; banners e imágenes/media a `media`. No existe endpoint de mutación administrativa de órdenes.

**Grafo dirigido de dependencias modulares (C2).** Las dependencias son unidireccionales y forman un DAG sin ciclos (orden topológico: `identity` → `media` → `catalog` → `cart-reservation` → `orders` → `payments` → `checkout`; `admin-query` depende solo de lectura de `identity`, `catalog` y `orders`). Ningún módulo depende de `checkout` ni de `admin-query`.

| Módulo | Depende de (dirigido) | Notas |
|---|---|---|
| `identity` | — | Sesión, registro, login, reset, bootstrap admin. Sin dependencias internas. |
| `media` | `identity` | Upload/validación S3, claves de imagen. |
| `catalog` | `identity`, `media` | Categorías, productos, banners, ajuste auditado de stock. |
| `cart-reservation` | `identity`, `catalog` | Carrito de servidor, reservas, reaper. |
| `orders` | `identity`, `catalog`, `cart-reservation` | Órdenes y snapshots. |
| `payments` | `identity`, `catalog`, `cart-reservation`, `orders` | Pagos, reembolsos, webhooks y job de reconciliación. |
| `checkout` | `identity`, `catalog`, `cart-reservation`, `orders`, `payments` | Orquesta checkout y hold `CHECKOUT_PENDING`. |
| `admin-query` | `identity`, `catalog`, `orders` (solo lectura) | Consulta cross-cutting de órdenes; sin mutaciones. |

**Ownership de inventario (C3).** `products.stock_on_hand` y `products.stock_reserved` tienen ownership explícito por módulo y operación:

| Columna | Módulo propietario | Operaciones que la escriben |
|---|---|---|
| `products.stock_on_hand` | `catalog` (creación y ajuste auditado) y `payments` (consumo) | `catalog`: valor inicial en `POST /v1/admin/products` y delta en `POST /v1/admin/products/{productId}/stock-adjustments`. `payments`: decremento atómico al consumir un hold `APPROVED`. Ninguna otra operación la escribe. |
| `products.stock_reserved` | `cart-reservation` y `payments` | `cart-reservation`: incremento/decremento al crear/liberar/vencer reservas `ACTIVE` (agregar, reducir, eliminar, logout, reaper). `payments`: decremento al consumir un hold `APPROVED` y liberación de holds `CHECKOUT_PENDING` en estados terminales no aprobados. `checkout` solo convierte `ACTIVE`→`CHECKOUT_PENDING` sin cambiar el contador. Ningún endpoint, DTO ni ajuste administrativo escribe `stock_reserved`. |

**Job de reconciliación (O4).** El job periódico que consulta pagos pendientes y cierra/reconcilia holds pertenece a `payments` como **scheduled driving adapter** (adaptador de entrada programado), no a un módulo separado. Usa la misma transición idempotente del webhook y no depende de la cola.

## 4. Modelo funcional y persistencia

Todos los UUID son públicos; cantidades son `integer > 0`; dinero COP es `bigint` de pesos sin decimales; tiempos son `timestamptz`. Tablas de negocio tienen `created_at`, `updated_at` y borrado lógico donde se indica. Retención definitiva y Ley 1581 requieren confirmación legal.

| Tabla / entidad | Campos principales y restricciones | Relaciones, índices y retención |
|---|---|---|
| `users` | `id`, `email varchar(254)` único case-insensitive, `display_name varchar(100)`, `password_hash`, `role USER_ROLE` (`admin`,`cliente`), `must_change_password boolean NOT NULL`, `phone nullable`, `deleted_at nullable` | único parcial por email activo; no se auto-registra `admin`. El admin inicial es `cristiansrc@gmail.com`; su contraseña viene exclusivamente de Secret Manager/variable de entorno no versionada, exige cambio inicial y jamás se registra en logs/documentación pública/seed. PII: retención pendiente. |
| `sessions` | `id`, `user_id nullable`, `kind SESSION_KIND` (`GUEST`,`AUTHENTICATED`), `token_hash` único, `last_activity_at`, `inactivity_expires_at`, `revoked_at`, `ip_hash`, `user_agent_hash` | cookie opaca HttpOnly; índice `(inactivity_expires_at, revoked_at)` y `(user_id, inactivity_expires_at)`. Toda petición autenticada o de carrito válida renueva a 10 min; al vencer/revocar se liberan reservas. La promoción login/registro conserva el `id` de sesión guest y su carrito. |
| `password_reset_tokens` | `id`, `user_id`, `token_hash` único, `expires_at`, `used_at` | índice token y expiración; un token activo por usuario; purgar tras 30 días es decisión técnica provisional. |
| `categories` | `id`, `name varchar(100)` único activo, `image_key`, `version integer` | productos 1:N; eliminación bloqueada si existen productos activos. Soft delete. `version` es optimistic locking (ver §4.1). |
| `products` | `id`, `category_id` NOT NULL, `name varchar(160)`, `description text`, `regular_price_cop bigint`, `sale_price_cop bigint`, `unit varchar(40)`, `stock_on_hand integer`, `stock_reserved integer NOT NULL DEFAULT 0`, `version integer`, `deleted_at` | `sale_price_cop <= regular_price_cop`, `stock_on_hand >= 0`, `0 <= stock_reserved <= stock_on_hand`; disponible público=`stock_on_hand-stock_reserved`. Índices categoría+activo, búsqueda full text nombre/descripción; una categoría; soft delete. Reducir stock bajo `stock_reserved` responde conflicto. `version` es optimistic locking (ver §4.1). |
| `carts` | `id`, `session_id` único NOT NULL, `status CART_STATUS` (`ACTIVE`,`CHECKOUT_PENDING`,`CLOSED`,`EXPIRED`), `last_activity_at`, `expires_at` | 1:1 con sesión, FK restrictiva; índice `(status, expires_at)`. Invitados reservan stock usando la misma sesión opaca; no existe carrito persistido en navegador. Retener 30 días tras cierre/expiración para auditoría técnica, sujeto a política legal. |
| `cart_items` | `id`, `cart_id`, `product_id`, `quantity integer`, `created_at`, `updated_at` | único `(cart_id, product_id)`; `quantity > 0`; FK a carrito/producto. Se elimina al checkout consumido o por limpieza de carrito. |
| `stock_reservations` | `id`, `cart_item_id` único, `product_id`, `quantity integer`, `status RESERVATION_STATUS` (`ACTIVE`,`CHECKOUT_PENDING`,`CONSUMED`,`RELEASED`,`EXPIRED`), `expires_at`, `released_at nullable`, `release_reason nullable` | `quantity > 0`; índices `(status, expires_at)` y `(product_id,status)`. Retención 30 días de filas terminales. La suma ACTIVE/CHECKOUT_PENDING debe coincidir con `products.stock_reserved`, comprobada transaccionalmente. |
| `product_images` | `id`, `product_id`, `s3_key` único, `alt_text`, `position` | único `(product_id, position)`; el objeto S3 se elimina mediante outbox después del commit. |
| `banners` | `id`, `name`, `image_key`, `target_path nullable`, `display_order`, `active`, `version`, `deleted_at` | índice `(active, display_order)`; soft delete. `version` es optimistic locking (ver §4.1). |
| `orders` | `id`, `order_number` único, `customer_id`, `cart_id` único, `items_subtotal_cop`, `delivery_fee_cop=5000`, `iva_cop`, `tax_rate_basis_points=1900`, `total_cop`, `status ORDER_STATUS`, `delivery_*` snapshot | `PENDING_PAYMENT`,`PAID`,`PAYMENT_FAILED`,`PAYMENT_EXPIRED`,`RESERVATION_EXPIRED`,`PAYMENT_REFUND_PENDING`,`PAYMENT_REFUNDED`,`PAYMENT_REFUND_FAILED`; IVA/tasa/total son NOT NULL. `iva_cop = floor((items_subtotal_cop * 19 + 50) / 100)` se calcula una sola vez y `total_cop = items_subtotal_cop + 5000 + iva_cop`. No logística. Índices cliente+creación, estado+creación. |
| `order_items` | `id`, `order_id`, `product_id nullable`, nombre/unidad/precio/cantidad/subtotal snapshot | FK orden; producto nullable para preservar historia tras borrado. |
| `payments` | `id`, `order_id`, `provider PAYMENT_PROVIDER`, `status PAYMENT_STATUS`, `amount_cop`, `provider_reference` único nullable, `idempotency_key` único, `provider_payload jsonb` minimizado | una orden puede tener intentos; índice orden+creación. Nunca datos de tarjeta. |
| `payment_refunds` | `id`, `payment_id` único, `provider`, `amount_cop`, `status REFUND_STATUS`, `idempotency_key` único, `provider_refund_reference` único nullable, `attempts`, `last_error_code nullable`, `completed_at nullable` | Un reembolso por pago tardíamente aprobado; índice `(status, updated_at)`. Retención igual a pagos, pendiente de política legal. |
| `payment_webhook_events` | `id`, `provider`, `provider_event_id` único, `payload jsonb`, `signature_valid`, `processed_at`, `processing_error` | deduplicación y auditoría técnica; retención pendiente. |
| `outbox_events` | `id`, `aggregate_type`, `aggregate_id`, `event_type`, `payload jsonb`, `idempotency_key` único, `published_at`, `attempts` | índice no-publicados; DLQ tras política indicada abajo. |
| `product_stock_adjustments` | `id`, `product_id` NOT NULL, `admin_user_id` NOT NULL, `idempotency_key uuid` único, `quantity_delta integer NOT NULL`, `reason varchar(500) NOT NULL`, `stock_on_hand_before integer NOT NULL`, `stock_on_hand_after integer NOT NULL`, `created_at` | Registro inmutable de auditoría mínima para todo ajuste manual; FKs restrictivas a `products` y `users`; checks `quantity_delta <> 0`, ambos snapshots `>= 0` y `stock_on_hand_after = stock_on_hand_before + quantity_delta`; índice `(product_id, created_at DESC)` y `(admin_user_id, created_at DESC)`. Retención: conservar mientras exista el producto, sujeta a política legal. `stock_reserved` no es mutable por este flujo. |

### 4.1 Optimistic locking de `version` (C4/O5)

`categories.version`, `products.version` y `banners.version` son **optimistic locking** para ediciones administrativas de datos generales. Mecánica:

* Toda respuesta de lectura/escritura de categoría, producto o banner expone el `version` vigente.
* Toda **actualización** (`PATCH /v1/admin/categories/{categoryId}`, `PATCH /v1/admin/products/{productId}`, `PATCH /v1/admin/banners/{bannerId}`) debe enviar el `version` esperado mediante la cabecera `If-Match` con el valor de `version`. El servidor compara contra el valor persistido en la misma transacción.
* Si el `version` esperado no coincide con el vigente, la actualización se rechaza con **`409`** (conflicto de modificación concurrente) y no aplica ningún cambio.
* Al aplicar la actualización, `version` se incrementa en 1 de forma atómica.
* **No aplica a ajustes de stock**: `POST /v1/admin/products/{productId}/stock-adjustments` ya tiene su propio lock transaccional sobre el producto e idempotencia por `Idempotency-Key`; no exige `version` ni `If-Match`. El diseño no justifica acoplarlo al optimistic locking de edición general.
* La creación (`POST`) no requiere `version`; el servidor lo inicializa en 1.

### Concurrencia, estado y consistencia

**Reserva viable en este alcance:** se adopta reserva para **carritos guest y autenticados** usando una sesión de servidor opaca; es viable porque PostgreSQL conserva el contador agregado `stock_reserved` y las reservas por ítem en la misma transacción. No se aceptará una alternativa que deje invitado sin reserva.

Al agregar/aumentar un ítem, una transacción bloquea producto y reserva existente en orden ascendente de `product_id`, valida `stock_on_hand - stock_reserved >= delta`, actualiza `stock_reserved`, la cantidad y `expires_at=now()+10m`. **Toda acción válida del usuario asociada a la sesión** (incluidos accesos válidos al carrito, mutaciones, login, refresh y operaciones protegidas) reinicia `sessions.inactivity_expires_at`, `carts.expires_at` y todas las reservas `ACTIVE` a `now()+10m`; una solicitud inválida, expirada o de webhook no los renueva. Reducir/eliminar invierte el delta y marca `RELEASED` si queda en cero. Dos mutaciones concurrentes sobre el último disponible producen una sola aceptación; la otra responde `409 STOCK_RESERVATION_UNAVAILABLE`. El reaper y logout liberan solo `ACTIVE`; nunca vencen un `CHECKOUT_PENDING` por inactividad.

Checkout bloquea todas las reservas `ACTIVE` del carrito, recalcula precio, aplica una sola vez **IVA 19%** (`tax_rate_basis_points=1900`) sobre `items_subtotal_cop` con `iva_cop = floor((items_subtotal_cop * 19 + 50) / 100)` (HALF_UP al peso COP), entrega fija de **5000 COP** y `total_cop = items_subtotal_cop + 5000 + iva_cop`, antes de convertir reservas o crear orden/pago. Convierte las reservas a `CHECKOUT_PENDING`. El hold no tiene timeout de inactividad: dura hasta `APPROVED`, `DECLINED`, `ERROR`, `EXPIRED` o el cierre de la reconciliación. En `APPROVED`, una única transacción bloquea productos, exige holds `CHECKOUT_PENDING` equivalentes a los ítems, decrementa `stock_on_hand` y `stock_reserved`, marca reservas `CONSUMED`, pago/orden `PAID` y escribe outbox. Si faltan holds consumibles (por expiración previa, liberación o stock inconsistente), no descuenta; crea exactamente un `payment_refunds` `PENDING`, orden `PAYMENT_REFUND_PENDING` y una orden de reembolso idempotente. Éxito marca pago/orden `REFUNDED`/`PAYMENT_REFUNDED`; fallo final marca `REFUND_FAILED`/`PAYMENT_REFUND_FAILED`, conserva evidencia y alerta para operación. Reintentos no duplican por `provider_event_id`, transiciones condicionales, `payments.idempotency_key` y `payment_refunds.payment_id` único.

El ajuste administrativo bloquea el producto en una transacción, rechaza `quantity_delta = 0` y todo resultado `stock_on_hand + quantity_delta < stock_reserved` con `409 STOCK_ADJUSTMENT_WOULD_VIOLATE_RESERVED_STOCK`, actualiza exclusivamente `stock_on_hand` e inserta un único `product_stock_adjustments` con snapshots antes/después, actor, razón e idempotency key. La misma clave, actor y cuerpo canónico devuelve el ajuste original; una reutilización divergente devuelve `409 IDEMPOTENCY_KEY_REUSED`. Ningún endpoint, DTO ni ajuste administrativo permite escribir `stock_reserved`.

| Agregado | Evento/guarda | Transición obligatoria |
|---|---|---|
| Reserva `ACTIVE` | acción válida del usuario | continúa `ACTIVE`; reinicia vencimiento a `now()+10m`. |
| Reserva `ACTIVE` | eliminación, logout o reaper tras inactividad | `RELEASED` o `EXPIRED`; decrementa una vez `products.stock_reserved`. |
| Reserva `ACTIVE` | checkout válido | `CHECKOUT_PENDING`; deja de estar sujeta al timeout de inactividad. |
| Hold `CHECKOUT_PENDING` | pago `DECLINED`, `ERROR` o conciliación `EXPIRED` sin aprobación | `RELEASED`; decrementa una vez `products.stock_reserved`; pago/orden pasan a `DECLINED|ERROR|EXPIRED` y `PAYMENT_FAILED|PAYMENT_EXPIRED`. |
| Hold `CHECKOUT_PENDING` | `APPROVED` con holds equivalentes consumibles | `CONSUMED`; decrementa una vez `stock_on_hand` y `stock_reserved`; pago/orden `APPROVED`/`PAID`. |
| Pago `APPROVED` | holds no consumibles | orden `PAYMENT_REFUND_PENDING`, refund `PENDING`; no cambia inventario. |
| Refund `PENDING` | proveedor confirma reembolso / agota 5 reintentos recuperables | `REFUNDED`/`PAYMENT_REFUNDED` o `REFUND_FAILED`/`PAYMENT_REFUND_FAILED`; este último alerta y requiere replay operativo idempotente. |

## 5. Migraciones Prisma (contrato; no es una migración ejecutable)

Ruta futura autoritativa: `apps/api/prisma/schema.prisma`; migraciones futuras: `apps/api/prisma/migrations/<timestamp>_foundation/`. Prisma Migrate debe generar, revisar y aplicar en orden:

1. extensiones `pgcrypto`, enums (`USER_ROLE`, `SESSION_KIND`, `CART_STATUS`, `RESERVATION_STATUS`, `ORDER_STATUS`, `PAYMENT_PROVIDER`, `PAYMENT_STATUS`, `REFUND_STATUS`) y tablas identity/sesión/reset;
2. catálogo, `stock_reserved`, constraints precio/stock, índices parciales y FTS;
3. carrito, ítems y reservas;
4. media, banners, órdenes, snapshots, pagos, reembolsos, webhooks y outbox;
5. `product_stock_adjustments`, sus FKs, checks e índices de auditoría;
6. datos semilla no productivos de categorías/productos requeridos por la prueba, sin credenciales ni datos reales. La creación del admin inicial se hace en bootstrap seguro idempotente, no seed versionado.

La migración debe ser expand/contract, reversible solo mediante nueva migración compensatoria, y validar en PostgreSQL Testcontainers. `updated_at` se mantiene por aplicación/Prisma; no se usará SQLite como sustituto de comportamiento PostgreSQL.

## 6. Seguridad, sesión y recuperación

Acceso: JWT de acceso máximo 10 minutos solo en memoria Redux; refresh/cart token opaco aleatorio, hasheado en `sessions`, cookie `HttpOnly; Secure; SameSite=Lax` con Path restringido, rotado en refresh y revocado en logout. Una sesión vence tras 10 minutos sin petición válida asociada; su cierre revoca cookie, cierra carrito y libera reservas. CSRF: double-submit token/validación de `Origin` para endpoints por cookie; access token Bearer para API. No hay token de sesión ni carrito en URL/localStorage. Contraseñas usan Argon2id. CORS permite exclusivamente orígenes configurados de ambos SPAs; CSP, HSTS, `nosniff`, frame ancestors y rate limits: login 5/15 min por IP+email hash, registro 3/h por IP, reset 3/h por IP+email hash.

El bootstrap idempotente del admin usa correo fijo `cristiansrc@gmail.com`, rol `admin` y `must_change_password=true`; falla de forma segura si el secreto requerido no está presente. Si el admin ya existe con `must_change_password=false`, el bootstrap es un **no-op**: no reescribe la contraseña ni el flag ni expone credenciales. El secreto solo se obtiene por referencia de Secret Manager/entorno de despliegue, se excluye de OpenAPI, migraciones, seed, ejemplos, logs, métricas y mensajes de error. La **rotación del secreto** se realiza fuera del bootstrap (actualización del valor en Secret Manager y reinicio del contenedor para que ECS lo relea por IAM task role); el bootstrap nunca imprime, devuelve ni registra la contraseña/hash, y una rotación no cambia la contraseña ya establecida de un admin con `must_change_password=false`.

Mientras `must_change_password=true`, el admin autenticado recibe `403 INITIAL_PASSWORD_CHANGE_REQUIRED` en toda operación salvo `GET /me`, logout, refresh y `POST /auth/initial-password-change`. Ese endpoint exige contraseña actual, `Idempotency-Key` y nueva contraseña conforme a política; cambia el hash y el flag en una transacción, rota/revoca las demás sesiones y no devuelve ni registra credenciales.

`POST /auth/password-reset-requests` responde siempre 202. Genera token aleatorio de un uso, hash en DB, expiración 30 minutos y email con enlace al storefront; no expone token en logs. `POST /auth/password-resets` consume atómicamente el token, actualiza hash y revoca todas las sesiones. La entrega email se registra en outbox, máximo 5 intentos con backoff exponencial 1m/5m/15m/1h/6h y DLQ/alarma después. Compensación: si no se envía email, token permanece válido hasta expirar; no se modifica contraseña.

## 7. Pagos, webhooks e integraciones

`CreateCheckout` usa exclusivamente el carrito de sesión, recalcula precios/IVA desde servidor, persiste orden `PENDING_PAYMENT` y pago `PENDING` usando `Idempotency-Key` por intento, y llama Strategy. No acepta ítems del cliente. La respuesta entrega URL/token de checkout del proveedor, nunca datos de tarjeta. Tras retorno del navegador, se consulta el estado; la fuente autoritativa es webhook firmado y/o consulta de reconciliación al proveedor, no el redirect del cliente.

| Integración | Idempotencia, retry/timeout y compensación | Observabilidad |
|---|---|---|
| Wompi (sandbox en evaluación; producción configurable) | creación: `payments.idempotency_key` y `provider_event_id`; timeout HTTP 10s, 3 intentos solo red/5xx (0.5/2/8s). **La firma se valida sobre el raw body recibido antes de persistir `payment_webhook_events`**; solo se persiste el evento tras verificar la firma y se registra `signature_valid`. Final failure deja pago pendiente y crea reconciliación. Compensación de aprobación no consumible: `payment_refunds.idempotency_key`, un refund por pago; timeout 10s, 5 intentos 1m/5m/15m/1h/6h solo red/5xx; tras fallo final `REFUND_FAILED`, alerta y replay operativo idempotente. Ante certeza desconocida, consultar por referencia antes de crear pago o reembolso. | logs `payment.provider_result` y `payment.refund_result`; métricas `payment_webhook_processed_total{provider,outcome}`, `payment_refund_total{provider,outcome}`; alarmas `payment_reconciliation_pending`, `payment_refund_failed`. |
| Mercado Pago | Mismas claves, límites, reintentos, compensación y final failure que Wompi; referencia externa única. | mismas métricas y logs con `provider=mercado_pago`. |
| S3 | upload directo con URL prefirmada PUT de 5 min, key generada por servidor; completar registro solo tras `HEAD` validado (tipo/tamaño). No retries automáticos del cliente fuera de una URL nueva. Si DB falla tras carga, outbox borra objeto huérfano. | `media_upload_validated_total`, `media_orphan_cleanup_total`, log `s3.object_validation`. |
| SQS/DLQ outbox | clave `outbox_events.idempotency_key`; 5 intentos 1m/5m/15m/1h/6h, visibilidad > timeout consumidor; fallo final a DLQ, alarma y runbook de replay idempotente. | `outbox_publish_total`, `outbox_dlq_visible_messages`, traza por `event_id`. |
| Reaper de sesiones y reservas | trigger: job cada 1 min y transición síncrona en logout. Clave: `stock_reservations.id` con actualización condicional de estado. Un lote toma hasta 500 expirados, timeout de transacción 5 s, máximo 3 reintentos con 1/5/15 s; tras fallo registra alerta y el siguiente ciclo reintenta. Compensación: si falla después de marcar una reserva, la transacción revierte contador y estado completos. | `reservation_reaper_processed_total{outcome,reason}`, gauge `reservation_active_total`, gauge `reservation_expired_lag_seconds`, log `inventory.reservation_released` con `reservation_id`, `cart_id` y `trace_id`, sin PII. |

Un job cada 15 min consulta pagos pendientes con antigüedad 5 min–24h, timeout 10s y la misma transición idempotente del webhook. Después de 24h marca `PAYMENT_EXPIRED`, libera holds `CHECKOUT_PENDING` y cierra checkout solo tras consulta que no confirme aprobación. Si consulta/webhook confirma aprobación tras ese cierre, dispara el reembolso automático descrito arriba. El job registra `payment_reconciliation_total{outcome}`. Este job pertenece a `payments` como **scheduled driving adapter** (ver §3.3).

## 8. Pruebas y operación

Jest con cobertura Istanbul/v8 configurada para fallar bajo 85% de líneas, ramas, funciones y statements **por archivo testeable** en `api`, `storefront` y `admin`; se excluyen tipos, DTOs pasivos, wiring y código generado, listados explícitamente en configuración. Dominio/aplicación objetivo 100%. TDD Red-Green-Refactor es obligatorio.

Pruebas: unitarias ROP/estados/precios/IVA 19% con `iva_cop = floor((items_subtotal_cop * 19 + 50) / 100)` y total exacto, contratos OpenAPI, integración Prisma+PostgreSQL Testcontainers, adaptadores de proveedores con HTTP mock, concurrencia de dos reservas sobre último stock y de reaper/logout contra mutación, idempotencia de PUT/DELETE/checkout/reembolso, ajuste de stock repetido/concurrente y rechazo bajo `stock_reserved`, renovación por toda acción válida y expiración a 10 min, promoción de carrito guest a login, persistencia del hold durante checkout activo, consumo del hold por webhook, aprobación tardía sin descuento y reembolso automático deduplicado, verificación firma/dedupe webhook, Redux sin persistir ítems ni tokens, rutas RBAC y E2E de cinco pasos con navegador. `dependency-cruiser` bloquea imports domain→application/infrastructure y application→infrastructure. Métricas mínimas: error/latencia HTTP, `stock_reservation_conflict`, `stock_adjustment_total{outcome}`, sesión expirada, reaper lag, pagos y reembolsos por estado, webhooks inválidos, DLQ; logs estructurados con `trace_id`, nunca PII, token ni payload de tarjeta.

## 9. Decisiones pendientes y supuestos explícitos

* Confirmado: IVA 19% aplicado una sola vez al subtotal de ítems con `floor((items_subtotal_cop * 19 + 50) / 100)` (HALF_UP al peso COP); entrega fija 5000 COP; `total_cop = items_subtotal_cop + 5000 + iva_cop`; no existe `base_fee_cop`; precios de producto excluyen IVA.
* Confirmado: carrito invitado y autenticado reserva al agregar; toda acción válida del usuario renueva sesión y reservas `ACTIVE` 10 min; checkout activo conserva `CHECKOUT_PENDING` hasta finalización del pago; aprobación no consumible exige reembolso automático.
* Confirmado: el ajuste administrativo de stock es un subrecurso explícito y auditado; `stock_reserved` solo cambia por el ciclo de reservas/pago y no por edición ni ajuste manual de producto.
* Pendientes no resueltos: campos finales perfil/dirección, retención Ley 1581, emails adicionales, provisión/admin puede comprar, paginación y política definitiva de eliminación.

## 10. Decomposition Contract

Artefactos autoritativos: esta Master Spec, `docs/api/openapi.yaml`, `docs/specs/prisma-migration-contract.md`, `docs/specs/architecture-decisions.md` y shared context. Rutas canónicas API empiezan `/v1`; el ajuste de inventario es exclusivamente `POST /v1/admin/products/{productId}/stock-adjustments`. DTOs/enums canónicos incluyen `CartItemMutationRequest`, `CartResponse`, `CreateCheckoutRequest`, `PaymentRefundResponse`, `StockAdjustmentRequest`, `StockAdjustmentResponse`, `ProductUpdateRequest`, `CategoryWriteRequest`, `BannerWriteRequest`, `SESSION_KIND`, `CART_STATUS`, `RESERVATION_STATUS`, `ORDER_STATUS`, `PAYMENT_STATUS` y `REFUND_STATUS`. Las actualizaciones de categoría/producto/banner usan optimistic locking: el request de actualización exige la cabecera `If-Match` con el `version` esperado, y un desajuste responde `409`; el ajuste de stock queda excluido de este mecanismo (ver §4.1). Tablas canónicas: `sessions`, `carts`, `cart_items`, `stock_reservations`, `products.stock_reserved`, `product_stock_adjustments`, `orders.delivery_fee_cop`, `orders.iva_cop`, `orders.tax_rate_basis_points=1900`, `payments`, `payment_refunds`. El contrato de localización canónico es `es-CO`: todo texto visible de storefront/admin y toda comunicación orientada al usuario se obtiene del catálogo de traducciones; importes visibles usan COP colombiano sin decimales. Orden permitido futuro: contrato+schema → identidad/catálogo → ajuste auditado de stock → carrito/reservas/reaper → checkout/pagos/reembolsos/webhooks → SPAs → infraestructura/observabilidad → pruebas integradas. Términos prohibidos: `Strapi`, `Next.js runtime`, `UserCart` solo-local, `base_fee_cop`, IVA distinto de 19% o redondeo COP distinto de `floor((items_subtotal_cop * 19 + 50) / 100)`, hold de checkout de 10 minutos, revisión manual como sustituto del reembolso automático, roles distintos de `admin`/`cliente`, descuento de stock únicamente tras pago sin reserva, invitado sin reserva, endpoint admin para mutar órdenes, mutar `stock_reserved` directamente o desde edición/ajuste administrativo de producto, almacenamiento de tarjeta, texto visible de UI hardcodeado en inglés (salvo nombres propios de proveedores/marcas o términos técnicos necesarios).
