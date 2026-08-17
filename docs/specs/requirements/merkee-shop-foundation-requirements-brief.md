# Requirements Brief — merkee.shop Foundation

## Status
`revision-needed`. Brief funcional consolidado; aprobación humana del plan otorgada el 2026-08-15 (**histórica/superseded**: invalidada por los cambios materiales posteriores ADR-018, migraciones 010–013, correcciones ROP y purga MSF-ID-002, más la remediación documental de estabilización 2026-08-16). Master Spec y OpenAPI son la fuente contractual. Se requiere revalidación focalizada de Spec Validator y, tras `ready`, nueva aprobación humana aplicable antes de cualquier handoff.

## Objetivo y alcance

Supermercado online para Colombia (COP, `es-CO`), con React storefront y panel admin, catálogo, carrito servidor guest/cliente, checkout Wompi/Mercado Pago, órdenes, perfil, recuperación de contraseña y administración de catálogo/media.

### Actores y permisos

| Actor | Permitido | Prohibido |
|---|---|---|
| Visitante | navegar y crear carrito guest con reserva | checkout, perfil, órdenes |
| Cliente | carrito, checkout, pago, perfil, órdenes propias | admin |
| Admin | catálogo, banners, ajustes auditados, lectura de órdenes, provisionar admin si ya cambió contraseña | carrito, checkout, compra, órdenes propias, mutar órdenes, auto-registro |

## Requisitos funcionales aprobados

1. Producto: una categoría, solo soft delete v1; hard delete/purga v2. Productos públicos y administrativos se paginan; órdenes conservan paginación; banners no requieren paginación v1.
2. Carrito guest/cliente reserva inventario. ACTIVE expira tras 10 min de inactividad; reaper cada minuto libera stock. CHECKOUT_PENDING no se libera por abandono. Guest autenticado como cliente conserva carrito; guest autenticado como admin libera ACTIVE, cierra carrito y no crea/conserva carrito admin.
3. Admin recibe 403 en carrito, checkout y órdenes. La API bloquea también mutaciones y lecturas de carrito para admin.
4. Registro público solo crea `cliente`. Un admin autenticado con contraseña inicial cambiada provisiona admins de un solo uso; no devuelve contraseña. El admin activado canjea token público de un uso y define primera contraseña.
5. Perfil v1: editar `display_name`, `phone`; email/rol no editables; dirección solo snapshot de orden (`orders.delivery_recipient_name`, `delivery_line1`, `delivery_city`, `delivery_phone`, copiados de `CreateCheckoutRequest.delivery_address`). Cambio de contraseña autenticado con contraseña actual, idempotencia y revocación de otras sesiones. Recuperación continúa v1. Notificación de compra: v2.
6. Checkout calcula IVA 19% HALF_UP COP una vez, entrega 5000 COP; la aprobación sin hold consumible reembolsa sin descontar stock. Ajustes de stock son auditados e idempotentes; edición usa `If-Match`.
7. S3 privado/CloudFront OAC; no tarjeta. Texto visible y mensajes a usuario en `es-CO`; valores COP sin decimales.
8. Retención técnica provisional NC-08: minimización, logs sin PII, tokens/carritos terminales 30 días, sesiones purga operativa, logs 90 días, órdenes/pagos/auditoría 5 años, anonimización operacional de PII y credenciales; revisión legal/contable previa a producción. Acceso/rectificación por autoservicio v1 con endpoints existentes: acceso vía `GET /me` y `GET /orders`/detalle; rectificación vía `PATCH /me` solo de `display_name`/`phone`. Email, rol y dirección de perfil no editables en v1. **Anonimización v1:** `display_name`/`phone` → `null` o neutro no identificable; `password_hash` invalidado con hash aleatorio no reutilizable; `email` → identificador irreversible/no operativo si la política legal lo permite; snapshots de orden preservados solo mientras sean necesarios y luego anonimizados. Supuestos legales provisionales, no asesoría.
9. Codificación futura: cada caso de uso de dominio/aplicación retorna `Result<Success, DomainError>`; los errores de negocio esperados no lanzan excepciones. Controllers/webhooks validan transporte/firma, llaman un caso de uso y mapean su Result al `ApiErrorResponse` OpenAPI; no acceden a Prisma ni contienen reglas de negocio. Dominio sin NestJS, Prisma, HTTP o SDKs externos; dependency-cruiser bloquea dependencias `domain→application|infrastructure` y `application→infrastructure`. El catálogo de códigos HTTP/OpenAPI estable y los flujos obligatorios están en Master Spec §ROP.

## Criterios de aceptación

| ID | Criterio |
|---|---|
| CA-01 | `GET /products` y `GET /admin/products` devuelven páginas tipadas y respetan límites de página. |
| CA-02 | Admin en cualquier operación carrito/checkout/orden recibe 403 y no cambia reservas ni crea orden. |
| CA-03 | Login guest→admin libera exactamente una vez ACTIVE y deja CHECKOUT_PENDING intacto. |
| CA-04 | Solo admin elegible puede crear admin; reintento idempotente no crea duplicado ni expone secreto. |
| CA-05 | Token de activación usado/expirado falla; éxito define contraseña, marca token usado y habilita admin. |
| CA-06 | PATCH perfil rechaza email, role y dirección; cambio de contraseña revoca otras sesiones. |
| CA-07 | Soft delete oculta producto público; no existe hard delete en v1. |
| CA-08 | Retenciones y anonimización se aplican según política provisional y dejan evidencia sin PII en logs. |
| CA-09 | Cada caso de uso afectado prueba `Success` y sus `Failure` de negocio; adapters traducen fallos técnicos y HTTP devuelve el `ApiErrorResponse.code` estable. |
| CA-10 | Identidad/provisión/activación, perfil/password-change, ajuste de stock, cart-reservation/reaper, checkout y pagos/webhooks/refunds tienen pruebas de idempotencia y concurrencia cuando mutan estado. |
| CA-11 | Dependency-cruiser falla ante `domain→application|infrastructure` o `application→infrastructure`; controllers y webhooks no acceden a Prisma. |

## Fuera de alcance y pendientes reales

Notificación de compra, exportación/autoservicio completo de datos, hard-delete/purga de producto y política legal definitiva son v2 o revisión preproducción. Pendiente no bloqueante: canal operativo seguro para entregar el token de activación.

## Historial superseded

Admin comprador, provisión admin indefinida, perfil/dirección pendiente, paginación de productos pendiente, retención sin decisión, hard delete v1, y notificación de compra v1 son superseded. Next.js/Strapi y carrito local son referencias históricas no implementables.
