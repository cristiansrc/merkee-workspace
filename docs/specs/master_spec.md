# Master Spec — merkee.shop

**Lifecycle status:** `planning`  
**Incremento activo:** `merkee-shop-foundation` · **Actualizado:** 2026-08-15

## Propósito, límites y fuentes canónicas

Supermercado digital colombiano: storefront React SPA para visitantes/clientes y SPA React administrativa separada. API NestJS TypeScript, monolito modular hexagonal/ROP, PostgreSQL y Prisma Migrate; el contrato público es `docs/api/openapi.yaml`. La plantilla Next.js/Strapi es solo referencia visual histórica y queda **superseded** como arquitectura.

Precedencia: esta Master Spec, OpenAPI, contrato Prisma, ADRs y shared context. Todos los importes son COP enteros. UI y comunicaciones al usuario: `es-CO`, catálogo de traducción desde el inicio, sin literales ingleses salvo marcas/términos técnicos necesarios.

## Criterios de aceptación globales

| ID | Criterio verificable |
|---|---|
| AC-01 | Visitante consulta categorías, banners, productos paginados, búsqueda y detalle; `GET /v1/products` acepta `page` (≥1, defecto 1), `size` (1..100, defecto 20), `category_id` y `q` (2..100), y devuelve `items,page{page,size,total}`. |
| AC-02 | Guest y cliente pueden operar un carrito servidor con reserva; Redux contiene solo vista derivada y ningún token/carrito se persiste en navegador. |
| AC-03 | Un admin recibe `403 ADMIN_STOREFRONT_PURCHASE_FORBIDDEN` en carrito, mutaciones de carrito, checkout y órdenes propias; no se crea ni conserva carrito de compra para su sesión. |
| AC-04 | Al autenticar como admin una sesión guest, una única transacción libera sus reservas `ACTIVE`, cierra/expira su carrito, revoca la sesión guest y crea sesión admin sin carrito; `CHECKOUT_PENDING` no se libera por este flujo. |
| AC-05 | Registro público crea exclusivamente `cliente`. No existe auto-registro admin. Solo admin autenticado con `must_change_password=false` puede provisionar otro admin por `POST /v1/admin/users`; la respuesta nunca contiene contraseña ni token. |
| AC-06 | La activación pública de admin consume una sola vez un token opaco, hasheado, no registrado y con expiración; establece primera contraseña y deja `must_change_password=false`. |
| AC-07 | `PATCH /v1/me` solo modifica `display_name` y `phone`; email y rol son inmutables por API. La dirección solo se recibe en checkout y se persiste como snapshot de orden. `POST /v1/auth/password-change` exige contraseña actual, `Idempotency-Key` y revoca las demás sesiones. |
| AC-08 | IVA = `floor((items_subtotal_cop * 19 + 50) / 100)` (19%, HALF_UP COP), una vez; entrega=5000; total=subtotal+entrega+IVA. |
| AC-09 | Stock adjustment es auditado, idempotente, no escribe `stock_reserved`, respeta `stock_on_hand >= stock_reserved`; PATCH admin usa `If-Match`. |
| AC-10 | Productos solo soft delete en v1; purga/hard delete se planifica para v2. |
| AC-11 | El reaper cada minuto expira solo reservas `ACTIVE` tras 10 minutos de inactividad y libera `stock_reserved`; `CHECKOUT_PENDING` permanece hasta terminal de pago/reconciliación. |
| AC-12 | S3 y buckets SPA son privados; CloudFront/OAC sirve lectura. No se aceptan ni almacenan PAN/CVV/fecha de tarjeta. |

## Arquitectura y ownership bloqueados

Módulos: `identity`, `media`, `catalog`, `cart-reservation`, `orders`, `payments`, `checkout`, `admin-query`. DAG: identity→media→catalog→cart-reservation→orders→payments→checkout, con dependencia directa adicional `checkout`→`cart-reservation` (checkout usa los puertos de `cart-reservation` para convertir ACTIVE→CHECKOUT_PENDING, sin depender transitivamente vía `orders`); `admin-query` solo lee identity/catalog/orders. `POST /v1/admin/users` y `POST /v1/auth/admin-activations` pertenecen al módulo `identity`; el tag `Admin` de OpenAPI solo agrupa superficie HTTP y no implica ownership de `admin-query`, que permanece solo lectura de órdenes. Strategy para Wompi/Mercado Pago; Adapter para Prisma/S3/email/pagos. Sin CQRS, Factory, microservicios ni Step Functions en v1.

`catalog` escribe stock físico al crear/ajustar; `payments` lo descuenta al consumir pago aprobado. Solo `cart-reservation` y `payments` escriben `products.stock_reserved`. `admin-query` nunca escribe. Controladores no consultan Prisma; dominio no depende de NestJS/Prisma/HTTP.

## Identidad, autorización y sesiones

Roles exclusivos: `admin`, `cliente`. JWT de acceso ≤10 min solo en memoria; token opaco hashado en cookie `HttpOnly; Secure; SameSite=Lax`, rotado al refresh. Argon2id. CSRF Origin/double-submit, CORS allowlist, CSP/HSTS/nosniff, rate limits de login/registro/reset/activación.

Admin con `must_change_password=true` solo puede `GET /me`, refresh, logout y `POST /v1/auth/password-change`; los demás recursos protegidos responden `403 INITIAL_PASSWORD_CHANGE_REQUIRED`. Bootstrap histórico del único admin inicial se conserva: secreto externo, no seed/log/documentación pública, no-op si ya cambió contraseña.

Provisión admin: creador debe ser admin con contraseña cambiada; crea usuario `admin`, `must_change_password=true`, hash de token de activación y expiración de 24 horas. Repetición con igual `Idempotency-Key`, actor y cuerpo devuelve el mismo `201`; reutilización divergente da `409 IDEMPOTENCY_KEY_REUSED`. La entrega segura del token queda fuera del API/logs: se proporciona únicamente al canal administrativo aprobado; v1 no envía notificación de compra. Recuperación de contraseña permanece v1, token de un uso 30 minutos y respuesta neutra.

## Datos, migraciones y retención v1 provisional

UUID públicos; `timestamptz`; dinero `bigint`; soft delete donde se indica. Tablas: `users`, `sessions`, `password_reset_tokens`, `admin_activation_tokens`, catálogo/media, `carts`, `cart_items`, `stock_reservations`, órdenes/snapshots, pagos/reembolsos/webhooks/outbox y `product_stock_adjustments`.

`admin_activation_tokens`: `id uuid`, `user_id uuid NOT NULL` FK restrictiva a users, `token_hash varchar(255) NOT NULL UNIQUE`, `expires_at timestamptz NOT NULL`, `used_at timestamptz NULL`, `created_by_user_id uuid NOT NULL` FK restrictiva, timestamps. Índices `(expires_at,used_at)`, `(user_id,used_at)`. Un índice parcial único `CREATE UNIQUE INDEX ... ON admin_activation_tokens(user_id) WHERE used_at IS NULL` garantiza a lo sumo un token no usado por admin; la vigencia `expires_at > now()` NO forma parte del índice (es no determinista) y se valida atómicamente en la transacción de canje con `used_at IS NULL AND expires_at > now()`. Para permitir reemisión si el token no usado expiró, la provisión/canje marca o revoca el token no usado expirado (p. ej. `UPDATE ... SET used_at = now() WHERE user_id = :id AND used_at IS NULL AND expires_at <= now()`) antes de insertar uno nuevo, de modo que nunca existan dos tokens no usados para el mismo admin. Nunca se persiste token en claro ni se incluye en logs, métricas, trazas, errores, OpenAPI examples o outbox.

Invariantes: email activo único case-insensitive; carrito 1:1 sesión; item único carrito/producto; reserva 1:1 item; `0<=stock_reserved<=stock_on_hand`; suma ACTIVE+CHECKOUT_PENDING igual a reservado; orden única por carrito; idempotencias únicas por intención. Dirección (`recipient_name,line1,city,phone`) es snapshot `orders.delivery_*`, no perfil.

**NC-08, política técnica provisional, no asesoría legal:** minimización; no tarjeta; logs sin PII; tokens (reset/activación) se purgan 30 días tras terminal; carritos/reservas terminales 30 días; sesiones revocadas/expiradas se purgan operativamente; órdenes, pagos, reembolsos y auditoría de stock 5 años; logs 90 días; producto soft delete sin purga v1. **Mecanismo técnico de anonimización v1 (supresión operacional):** `display_name` y `phone` se reemplazan por `null` o valores neutros no identificables; `password_hash` se invalida con un hash aleatorio no reutilizable (imposible de autenticar); `email` se transforma a un identificador irreversible/no operativo (p. ej. hash con sal aleatoria no reversible) solo si la política legal lo permite; los snapshots de orden (`orders.delivery_*`) se preservan únicamente mientras sean necesarios para obligaciones legales/contables y luego se anonimizan. Supuestos legales marcados como provisionales y sujetos a revisión legal/contable/Ley 1581 antes de producción; esto no constituye asesoría legal. Acceso/rectificación por autoservicio v1 usando endpoints existentes, sin inventar rutas nuevas: acceso a datos personales vía `GET /v1/me` (perfil) y `GET /v1/orders` + `GET /v1/orders/{orderId}` (órdenes propias con snapshots de dirección); rectificación vía `PATCH /v1/me` limitada a `display_name` y `phone`. Email, rol y dirección de perfil no son editables en v1 (la dirección solo se rectifica creando una nueva orden). Supresión operacional; exportación y autoservicio completo v2. Revisión legal/contable/Ley 1581 obligatoria antes de producción.

## Carrito, checkout y pagos

Mutación o lectura válida renueva sesión, carrito y reservas ACTIVE a `now()+10m`. Reaper cada 1 min toma hasta 500 filas, timeout transacción 5 s, 3 reintentos 1/5/15 s; transición condicional evita doble liberación. Fallo revierte transacción, alerta y deja reintento en siguiente ciclo. Métricas: `reservation_reaper_processed_total`, `reservation_active_total`, `reservation_expired_lag_seconds`; log sin PII `inventory.reservation_released`.

Checkout cliente bloquea reservas, recalcula snapshots y convierte ACTIVE→CHECKOUT_PENDING. Webhook firmado sobre raw body y reconciliación cada 15 min son autoritativos. APPROVED consume hold equivalente en una transacción y decrementa físico/reservado; sin hold consumible no descuenta y crea un reembolso idempotente. Proveedores: timeout 10 s, 3 reintentos 0.5/2/8 s solo red/5xx; refunds 5 intentos 1m/5m/15m/1h/6h y luego alerta/DLQ/replay. Métricas `payment_webhook_processed_total`, `payment_refund_total`, `payment_reconciliation_total`.

## Decomposition Contract

Fuentes autoritativas: esta spec, OpenAPI, contrato Prisma, ADRs y shared context. Rutas nuevas canónicas: `POST /v1/admin/users`, `POST /v1/auth/admin-activations`, `POST /v1/auth/password-change`; rutas prohibidas: auto-registro admin y `POST /auth/initial-password-change` (histórica **superseded**). DTOs: `CreateAdminUserRequest`, `AdminUserProvisionResponse`, `AdminActivationRequest`, `PasswordChangeRequest`, `UpdateProfileRequest`. Tabla nueva `admin_activation_tokens`. Orden permitido: contrato OpenAPI→migración/esquema→identity→catálogo/carrito→checkout/pagos→SPAs→observabilidad/pruebas. Términos stale prohibidos: admin comprador, perfil con dirección persistida, hard delete de producto v1, token de activación en claro/log, hold checkout 10 min, `base_fee_cop`, IVA no HALF_UP 19%.

## Estado de revisión y riesgos restantes

Las decisiones NC-01, NC-03, NC-04, NC-05, NC-06/07, NC-08 y NC-10 están resueltas. Pendientes reales no bloqueantes: canal seguro operativo para entregar el token de activación, contenido de emails no transaccionales v2 y definición legal definitiva de retención/supresión antes de producción. La revalidación formal de Solution Architect del 2026-08-15 (nuevo flujo identity/provisión) tuvo veredicto `ready` tras remediar F-01/F-02/F-03/H-01/H-02/H-03/H-04. La revalidación de Spec Validator sigue pendiente; no hay aprobación `ready` de Spec Validator ni handoff.

## Historial superseded

Auto-registro/provisión admin no definida, admin comprador, perfil con dirección persistida, eliminación física de producto v1, paginación de productos pendiente, notificación de compra v1 y `POST /auth/initial-password-change` son históricos superseded por esta versión.
