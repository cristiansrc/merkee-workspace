# Contrato de migración Prisma — merkee.shop

**Lifecycle status:** `planning`

Contrato, no migración ejecutable. Ruta futura: `apps/api/prisma/schema.prisma` y `apps/api/prisma/migrations/`; Prisma Migrate es el único DDL. Aplicar en PostgreSQL/Testcontainers; nunca `db push`; evolución expand/contract mediante nueva migración.

## Secuencia futura y aceptación

| Orden | Nombre | Objetos | Criterio |
|---|---|---|---|
| 001 | `identity_and_auth` | `USER_ROLE`, `SESSION_KIND`, users, sessions, password_reset_tokens | email activo único CI, Argon2 hash solo, `must_change_password NOT NULL`, guest con `user_id` nullable; índices de expiración/usuario. |
| 002 | `admin_activation_tokens` | `admin_activation_tokens` | campos/índices/FKs de Master Spec; token hash único, `used_at` nullable, índice parcial único `WHERE used_at IS NULL` (sin `expires_at > now()` en el índice) para a lo sumo un token no usado por usuario; vigencia validada atómicamente en el canje; reemisión marca/revoca el token no usado expirado antes de insertar; jamás token claro. |
| 003 | `catalog_and_media` | categorías, productos, imágenes, banners | soft delete; una categoría/producto; checks precios/stock, `version DEFAULT 1`; no hard delete/purga v1. |
| 004 | `cart_reservations` | carts, cart_items, stock_reservations | 1:1 carrito/sesión y reserva/item; `ACTIVE` expirable, terminales retenidos 30 días. |
| 005 | `orders_payments_outbox` | órdenes/snapshots, pagos, refunds, webhooks, outbox | IVA/totales NOT NULL, `delivery_fee_cop=5000`, snapshots de dirección `orders.delivery_recipient_name/delivery_line1/delivery_city/delivery_phone` NOT NULL (copiados de `CreateCheckoutRequest.delivery_address`), sin `base_fee_cop`, idempotencias únicas. |
| 006 | `product_stock_adjustments` | auditoría inmutable | delta no cero, snapshots no negativos y ecuación after=before+delta; FKs restrictivas e índices por producto/admin. |
| 007 | `non_production_seed` | catálogo dummy | sin secreto, contraseña, hash ni admin. |

## Consistencia y retención

El login admin desde guest libera ACTIVE/cierra carrito/revoca guest en transacción y crea sesión admin sin carrito; CHECKOUT_PENDING no se toca. Reaper cada minuto procesa ACTIVE vencidas, lote 500, timeout 5s, 3 reintentos 1/5/15s. Checkout conserva CHECKOUT_PENDING hasta terminal. Webhook/reconciliación consume exactamente una vez o crea refund único sin descuento. PATCH catálogo compara `If-Match` y sube `version`; stock-adjustments usa lock/idempotencia, no `If-Match`.

Purgas técnicas: tokens 30 días tras terminal; carritos/reservas terminales 30 días; sesiones por operación; logs 90 días; órdenes/pagos/refunds/auditoría 5 años provisionales. Anonimización operacional de PII de usuario: `display_name`/`phone` → `null` o neutro no identificable; `password_hash` invalidado con hash aleatorio no reutilizable; `email` → identificador irreversible/no operativo si la política legal lo permite; snapshots de orden (`orders.delivery_*`) preservados solo mientras sean necesarios y luego anonimizados. Revisión legal/contable previa a producción obligatoria.

## Pruebas futuras obligatorias

Aplicación limpia 001→007; índices/checks; consumo concurrente de activación (un éxito); activación expirada/usada rechazada sin exponer token; login guest→admin libera ACTIVE sin carrito admin; reaper y checkout hold; dos reservas último stock; reembolso tardío; If-Match; ajuste idempotente; retenciones/purgas sin borrar snapshots requeridos.
