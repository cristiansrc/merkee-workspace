# Contrato de migración Prisma — merkee.shop

**Lifecycle status:** `planning`

Este artefacto define el resultado esperado de Prisma Migrate; no contiene ni autoriza una migración SQL/Prisma ejecutable. La fuente de tablas, campos, índices, relaciones, retención y consistencia es la sección 4 de `master_spec.md`.

## Secuencia y aceptación

| Orden | Nombre futuro | Objetos | Criterio de aceptación |
|---|---|---|---|
| 001 | `identity_and_auth` | enums `USER_ROLE`, `SESSION_KIND`; `users`, `sessions`, `password_reset_tokens` | email activo único case-insensitive; `sessions.user_id` nullable para guest; `kind`, `last_activity_at`, `inactivity_expires_at` no nulos; índices de reaper; hashes nunca retornados. `users.must_change_password` no nulo. No hay contraseña ni seed para admin. |
| 002 | `catalog_and_media` | `categories`, `products`, `product_images`, `banners` | `products.stock_reserved integer NOT NULL DEFAULT 0`; checks `stock_on_hand >= 0`, `stock_reserved >= 0`, `stock_reserved <= stock_on_hand`, precio venta no mayor a regular; una FK `category_id` no nula; índices declarados. `categories.version`, `products.version` y `banners.version` son `integer NOT NULL DEFAULT 1` y se usan como optimistic locking (incremento atómico por la aplicación en cada actualización; ver §4.1 de la Master Spec). |
| 003 | `cart_reservations` | enums `CART_STATUS`, `RESERVATION_STATUS`; `carts`, `cart_items`, `stock_reservations` | `carts.session_id` único/FK; `cart_items(cart_id,product_id)` único; `stock_reservations.cart_item_id` único; cantidad positiva; índices `(status,expires_at)` y `(product_id,status)`; retención terminal 30 días queda documentada para job, no como borrado en DDL. |
| 004 | `orders_payments_refunds_outbox` | enums `ORDER_STATUS`, `PAYMENT_STATUS`, `REFUND_STATUS`; `orders`, `order_items`, `payments`, `payment_refunds`, `payment_webhook_events`, `outbox_events` | `orders.cart_id` único; `delivery_fee_cop NOT NULL DEFAULT 5000`; `tax_rate_basis_points NOT NULL DEFAULT 1900`; `iva_cop` y `total_cop NOT NULL`; sin `base_fee_cop`. `payment_refunds.payment_id` e `idempotency_key` son únicos, `provider_refund_reference` único nullable, e índice `(status,updated_at)` permite replay operativo. Referencias/idempotencia únicas y snapshots inmutables. |
| 005 | `product_stock_adjustments` | `product_stock_adjustments` | Registro inmutable con `id UUID`, `product_id` y `admin_user_id` NOT NULL/FK restrictivas, `idempotency_key UUID` único, `quantity_delta integer NOT NULL`, `reason varchar(500) NOT NULL`, snapshots `stock_on_hand_before/after integer NOT NULL` y `created_at timestamptz NOT NULL`. Checks: delta no cero, snapshots no negativos y `after = before + delta`; índices `(product_id, created_at DESC)` y `(admin_user_id, created_at DESC)`. No columna para modificar `stock_reserved`. |
| 006 | `non_production_seed` | categorías/productos dummy | Datos solo no productivos, no secretos, permite probar stock/reservas. Prohibido crear el admin inicial, contraseña, hash derivado o referencia de secreto en este seed. |

## Reglas de ejecución futura

* Ubicaciones: `apps/api/prisma/schema.prisma` y `apps/api/prisma/migrations/`; Prisma Migrate es el único mecanismo de DDL.
* Aplicar en PostgreSQL efímero de Testcontainers antes de ambiente compartido. Prohibido `db push` como despliegue.
* Toda evolución posterior es expand/contract y una nueva migración; no editar migraciones aplicadas. Índices grandes se crean de forma compatible con producción y se documenta bloqueo esperado.
* 004 fija IVA al 19% (`tax_rate_basis_points=1900`): una sola vez por orden, `iva_cop = floor((items_subtotal_cop * 19 + 50) / 100)` (HALF_UP al peso COP), y `total_cop = items_subtotal_cop + delivery_fee_cop + iva_cop`. La aplicación calcula los snapshots antes de insertar; la base de datos los mantiene no nulos.
* El admin inicial se aprovisiona por bootstrap de runtime idempotente después de 001, con secreto inyectado fuera de Prisma Migrate. Si falta, bootstrap falla sin cuenta por defecto.

## Consistencia crítica y concurrencia

1. Agregar/aumentar reserva bloquea producto y reserva por `product_id` ascendente, valida `stock_on_hand - stock_reserved >= delta`, e incrementa ambos contador/reserva en una transacción.
2. Reducir, borrar, logout y reaper actúan únicamente sobre reservas `ACTIVE`, hacen transición condicional y decrementan `stock_reserved` en la misma transacción; repetir una operación terminal no cambia contador. `CHECKOUT_PENDING` solo termina por evento terminal de pago/reconciliación.
3. Toda acción válida del usuario asociada a sesión reinicia `sessions.inactivity_expires_at`, `carts.expires_at` y `stock_reservations.expires_at` con estado `ACTIVE` a `now()+10m`. El reaper selecciona únicamente `ACTIVE` por `(status,expires_at)` en lotes de 500 cada minuto. Su transacción tiene timeout 5 s y reintenta 3 veces con 1/5/15 s; fallo final deja alerta, no una liberación parcial.
4. Checkout cambia reservas activas a `CHECKOUT_PENDING`. Ese estado no vence por inactividad y se mantiene hasta que pago/reconciliación llegue a terminal. Webhook aprobado solo consume holds `CHECKOUT_PENDING` equivalentes a los ítems: decrementa `stock_on_hand` y `stock_reserved`, marca reserva `CONSUMED`, pago/orden y outbox en una transacción.
5. `payment_webhook_events.provider_event_id`, `payments.idempotency_key`, `payment_refunds.payment_id`, `payment_refunds.idempotency_key` y `orders.cart_id` son únicos. Una aprobación sin hold consumible no descuenta: inserta atómicamente el único reembolso `PENDING`, marca la orden `PAYMENT_REFUND_PENDING` y agenda la llamada de reembolso. Éxito marca `REFUNDED`/`PAYMENT_REFUNDED`; cinco fallos máximos (1m/5m/15m/1h/6h, solo red/5xx) marcan `REFUND_FAILED`/`PAYMENT_REFUND_FAILED` y activan alerta/replay idempotente.
6. Ajuste de stock: una transacción bloquea `products`, valida `quantity_delta <> 0` y `stock_on_hand + quantity_delta >= stock_reserved`, actualiza solo `stock_on_hand` e inserta el único registro de auditoría asociado a la clave de idempotencia. Misma clave, admin y cuerpo canónico devuelve el registro existente; cuerpo divergente responde `409 IDEMPOTENCY_KEY_REUSED`. No hay DDL, repositorio ni DTO de este flujo que permita mutar `stock_reserved`. El ajuste de stock **no** usa optimistic locking de `version` (ya tiene lock transaccional e idempotencia).
7. Optimistic locking de `version`: las actualizaciones de categoría/producto/banner comparan el `version` esperado (vía `If-Match`) contra el persistido en la misma transacción; un desajuste responde `409` sin aplicar cambios y un acierto incrementa `version` en 1. La creación inicializa `version=1`.

## Pruebas de migración requeridas

* Aplicación limpia 001→006 y nueva base con schema Prisma equivalente.
* Checks e índices rechazan `stock_reserved > stock_on_hand`, reserva duplicada por cart-item, doble carrito por sesión y orden duplicada por carrito.
* Prueba concurrente PostgreSQL de dos reservas del último ítem: una confirma y una falla sin desbalance de `stock_reserved`.
* Pruebas de renovación por acción válida, reaper/logout repetidos, hold persistente durante checkout, consumo de hold y aprobación tardía confirman que `stock_on_hand` nunca queda negativo ni se libera/consume dos veces.
* Prueba de aprobación tardía/sin hold crea un único `payment_refunds` por pago, no descuenta stock, y cubre éxito, cinco reintentos recuperables y fallo final del reembolso.
* Ajuste concurrente/repetido: un delta que dejaría `stock_on_hand < stock_reserved` falla con `STOCK_ADJUSTMENT_WOULD_VIOLATE_RESERVED_STOCK`; un replay idempotente no vuelve a cambiar stock y deja un solo `product_stock_adjustments`; no existe vía para mutar `stock_reserved` fuera de reservas/pago.
