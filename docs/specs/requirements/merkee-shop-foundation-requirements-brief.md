# Requirements Brief — merkee.shop Foundation

---

## 1. Status

`planning`

El brief conserva el levantamiento funcional histórico, pero las decisiones consolidadas posteriores tienen como fuente canónica la Master Spec, OpenAPI, contrato Prisma, ADR y shared context. No autoriza handoff: quedan pendientes la revisión del Spec Validator y la aprobación humana. *(Histórico superseded: la consulta formal al Solution Architect quedó resuelta con veredicto final `ready-with-final-verdict` el 2026-08-15.)*

---

## 2. Objetivo

Construir **merkee.shop**, un portal de supermercado en línea para el mercado de **Colombia** (moneda **COP**, idioma español) que permita a los clientes navegar productos por categoría, gestionar un carrito de compras (incluido en modo invitado), autenticarse, recuperar contraseña, gestionar su perfil y completar pagos con **Wompi** (proveedor predeterminado) y **Mercado Pago** (segunda opción). Incluye un panel de administración para gestionar productos, categorías y banners, y consultar órdenes. El sistema debe usar PostgreSQL, API Design First, almacenamiento de imágenes en Amazon S3 y mantener una cobertura mínima de pruebas del 85% por archivo testeable.

---

## 3. Contexto

### Antecedentes
- Se parte de una plantilla de referencia (`Online_Glocery_Store_Next.js_Tailwind_Strapi-main/`) que implementa un grocery store con Next.js 14 + Strapi + Tailwind + Shadcn UI.
- La plantilla es un **punto de partida visual/funcional**, no la arquitectura definitiva. Su stack Next.js/Strapi está superseded: la fuente canónica define React SPA y backend NestJS TypeScript con PostgreSQL/Prisma y API Design First.
- El alcance funcional se basa en la plantilla de referencia y en las decisiones confirmadas por el usuario en esta revisión.

### Decisiones canónicas posteriores al levantamiento inicial
Las reglas de esta subsección sustituyen cualquier afirmación histórica incompatible en el resto del brief; el historial se conserva únicamente para trazabilidad.

- Carrito de servidor para invitado y cliente, asociado a una sesión opaca. Agregar o aumentar un ítem crea/actualiza una reserva activa de stock; la UI Redux conserva solo una vista derivada y no persiste carrito ni tokens en `localStorage`.
- El checkout convierte las reservas activas en holds `CHECKOUT_PENDING`, que permanecen hasta un estado terminal de pago o reconciliación; no expiran por inactividad. Solo un pago aprobado con hold consumible descuenta `stock_on_hand` y `stock_reserved`; una aprobación sin hold consumible inicia reembolso automático idempotente sin descontar inventario.
- El IVA es 19% del subtotal de ítems y se calcula una única vez como `floor((items_subtotal_cop * 19 + 50) / 100)` (HALF_UP al peso COP). El total es subtotal + entrega fija de 5000 COP + IVA.
- La edición general de producto no altera existencias. Un administrador realiza aumentos o reducciones mediante un ajuste explícito con delta entero no cero y motivo; no puede modificar reservas activas directamente ni dejar el stock físico por debajo del reservado.
- Todo ajuste administrativo deja una auditoría mínima inmutable con producto, actor administrador, delta, motivo y existencias antes/después. La retención legal definitiva sigue pendiente.
- Las ediciones administrativas de categorías, productos y banners usan **optimistic locking** por `version`: la actualización exige el `version` esperado vía `If-Match` y un desajuste responde `409`. El ajuste de stock queda excluido de este mecanismo (ya tiene lock transaccional e idempotencia).

### Lo que la plantilla ya tiene (referencia funcional)
| Funcionalidad | Estado en plantilla | Observaciones |
|---|---|---|
| Home con sliders, categorías y productos populares | Implementado | Sliders + lista de categorías + top 8 productos + banner promocional |
| Navegación por categoría | Implementado | Ruta `/product-category/[categoryName]` con filtrado |
| Detalle de producto (modal) | Implementado | Dialog con imagen, descripción, precios, selector de cantidad |
| Autenticación (registro/login) | Implementado | Usa Strapi `/auth/local`; JWT en `sessionStorage` |
| Agregar al carrito | Parcial | Requiere login en la plantilla; **en merkee.shop el carrito de invitado estará permitido** |
| Contador de carrito en header | Parcial | `getCartItems` existe pero el contador no se actualiza dinámicamente |
| Búsqueda | No funcional | Input visible en header sin lógica asociada |
| Página de perfil | No implementado | El menú muestra "Profile" pero no hay ruta |
| Página de mis órdenes | No implementado | El menú muestra "My Orders" pero no hay ruta |
| Página/checkout de carrito | No implementado | Solo existe agregar al carrito y contador |
| Pagos | No implementado | Sin integración de pago |
| Panel de administración | No implementado | Sin CRUD de productos/categorías |
| Gestión de imágenes en S3 | No implementado | Imágenes vienen de Unsplash (mock) o Strapi |

### Modelo de datos de la plantilla (referencia)
- **Slider**: `name`, `image` (una imagen)
- **Category**: `name`, `icon` (una imagen)
- **Product**: `name`, `description`, `mrp` (precio regular), `sellingPrice` (precio de venta), `itemQuantityType` (unidad de medida), `images` (múltiples imágenes), `categories` (relación a Category)
- **UserCart**: `quantity`, `amount`, `products` (relación), `userId`

---

## 4. Actores y Permisos

El sistema define **exactamente dos roles**: **admin** (panel de administración) y **cliente** (portal). No existen roles intermedios.

| Actor | Descripción | Permisos funcionales |
|---|---|---|
| **Visitante (anónimo)** | Usuario no autenticado que navega el portal | Ver home, categorías, productos, detalle de producto. **Puede agregar productos al carrito (carrito de invitado)**. No puede pagar ni ver perfil/órdenes hasta autenticarse. |
| **Cliente** | Usuario autenticado con rol `cliente` | Todo lo del visitante + al pagar se le exige login (si no lo está), ver/editar carrito, checkout, pagar, ver/editar perfil, ver historial de órdenes, recuperar contraseña. |
| **Administrador** | Usuario autenticado con rol `admin` | Acceso al panel de administración: gestionar productos sin alterar existencias desde la edición general, categorías y banners; aumentar/reducir existencias mediante ajuste explícito con delta y motivo; consultar órdenes solo en lectura. **No puede modificar órdenes** (no cambiar estados, no cancelar, no editar). |

### Reglas de permisos confirmadas
- El carrito de invitado está permitido: un visitante puede armar su carrito sin iniciar sesión.
- El login se exige **al momento de pagar** (checkout/pago), no al agregar al carrito.
- Si el usuario ya tiene una sesión válida, **no se le debe solicitar login otra vez**.
- El admin **no puede modificar órdenes**; solo consultarlas.
- El admin **puede gestionar banners** (sliders) desde el panel.
- El admin ajusta existencias mediante una operación explícita y auditada; no modifica `stock_reserved` ni reservas activas directamente.

### Preguntas de permisos sin responder (no bloqueantes)
- ¿El admin puede comprar como cliente en el portal? (ver sección 13, NC-04)
- ¿Quién provisiona las cuentas de administrador? (ver sección 13, NC-05)

---

## 5. Alcance (Scope)

### 5.1 Portal del Cliente (Storefront)
- **Home**: Carrusel de sliders/banners, lista de categorías con imagen, listado de productos populares, banner promocional.
- **Navegación por categoría**: Página dedicada que muestra productos filtrados por categoría, con selector de categorías en la parte superior.
- **Detalle de producto**: Vista con imagen, descripción, precios (regular y de venta), unidad de medida, selector de cantidad, indicador de disponibilidad/stock y botón de agregar al carrito.
- **Búsqueda de productos**: Input funcional en el header que filtra productos por nombre o descripción. *(La plantilla tiene el input pero no la lógica.)*
- **Autenticación**: Registro de cuenta, inicio de sesión, cierre de sesión y **recuperación de contraseña**.
- **Perfil de usuario**: Visualización y edición de datos del perfil (campos exactos pendiente confirmar — ver NC-01).
- **Carrito de compras (incluido modo invitado)**: Listado de items, edición de cantidades, cálculo de subtotal, IVA y total. Es un carrito de servidor asociado a sesión y cada cantidad activa reserva stock; la vista local no es fuente de verdad. Login/registro promueve y conserva el carrito de invitado y sus reservas activas.
- **Checkout/Pago**: Flujo de pago con **Wompi** (predeterminado) y **Mercado Pago** (segunda opción). El login se exige en este punto si el usuario no está autenticado. El checkout conserva el hold pendiente hasta el resultado terminal del pago o su reconciliación.
- **Historial de órdenes**: Listado de órdenes anteriores del cliente con estado.

### 5.2 Panel de Administración (Admin)
- **Gestión de categorías**: Crear, leer, actualizar y eliminar categorías con imagen asociada.
- **Gestión de productos**: Crear, leer, actualizar y eliminar productos con múltiples imágenes, precios, descripción, unidad de medida y **una sola categoría asociada**. La edición general no modifica stock; el stock físico se incrementa o reduce solo mediante un ajuste administrativo explícito con delta entero no cero, motivo y auditoría mínima.
- **Gestión de banners/sliders**: Crear, leer, actualizar y eliminar banners (imágenes promocionales del home).
- **Consulta de órdenes**: Visualización de órdenes de todos los clientes **en modo solo lectura**. El admin no puede modificar estados, cancelar ni editar órdenes.

### 5.3 Infraestructura y Calidad (requisitos no funcionales del usuario)
- Base de datos: PostgreSQL.
- API Design First (contrato OpenAPI antes de implementación).
- Almacenamiento de imágenes en Amazon S3.
- Cobertura mínima de pruebas: 85% por archivo testeable.
- Mercado: Colombia. Moneda: COP. Idioma: español.

---

## 6. No Objetivos (Out of Scope)

- **No** se diseñará en este brief: endpoints OpenAPI, esquemas de DB, migraciones, modelos de datos técnicos, payloads, ni arquitectura de despliegue (labor de Planner).
- **No** se incluye (salvo confirmación del usuario): logística de envío/entrega con tracking, multi-moneda, multi-idioma, cupones/descuentos, reviews/calificaciones de productos, wishlist, notificaciones push, programa de fidelización, ventas cruzadas (cross-sell), soporte de chat en vivo.
- **No** se incluye modificación de órdenes desde el panel admin (el admin solo consulta).
- **Histórico superseded:** la exclusión de persistencia para carrito local queda sustituida por el carrito de servidor asociado a sesión y la promoción de sesión guest al autenticar. La UI no persiste el carrito ni tokens en navegador.
- **No** se incluye migración de datos desde la plantilla Strapi (es un proyecto nuevo).
- **No** se incluye reutilización literal del código de la plantilla; esta sirve solo como referencia funcional y visual.

---

## 7. Flujos de Usuario

### 7.1 Flujo: Visitante navega y arma carrito (carrito de invitado)
1. El visitante accede al home.
2. Ve carrusel de banners, lista de categorías y productos populares.
3. Puede hacer clic en una categoría para ver sus productos.
4. Puede hacer clic en un producto para ver su detalle (modal o página), incluyendo disponibilidad/stock.
5. **Puede agregar productos al carrito sin iniciar sesión**; el servidor crea/mantiene la sesión guest y reserva el stock solicitado.
6. El contador del carrito se actualiza en el header.

### 7.2 Flujo: Registro, login y recuperación de contraseña
1. El visitante accede a la página de registro.
2. Ingresa nombre de usuario, email y contraseña.
3. Se crea la cuenta (rol `cliente`) y se inicia sesión automáticamente.
4. Alternativamente, un visitante existente inicia sesión con email y contraseña.
5. La sesión se persiste para mantenerse activa; **una sesión válida no debe solicitar login otra vez**.
6. **Recuperación de contraseña**: un usuario que olvidó su contraseña puede solicitar recuperación (típicamente vía email con enlace/token). El flujo exacto de entrega es decisión de Planner.

### 7.3 Flujo: Compra (carrito → checkout → pago)
1. El cliente (o invitado) navega productos y agrega items al carrito (con cantidad). El stock físico **no** se descuenta en este punto, pero cada cantidad crea una reserva activa que reduce disponibilidad.
2. El contador del carrito se actualiza en el header.
3. Accede a la página del carrito.
4. Revisa items, ajusta cantidades o elimina productos.
5. Procede al checkout.
6. **Si no está autenticado, se le exige login en este punto.** Si ya tiene sesión válida, continúa sin volver a pedir login.
7. Selecciona método de pago: **Wompi** (predeterminado) o **Mercado Pago**.
8. El sistema calcula una única vez el IVA de 19% con HALF_UP al peso COP sobre el subtotal de ítems, agrega entrega fija de 5000 COP y muestra el total antes del pago.
9. Completa el pago a través del proveedor seleccionado; el hold de checkout permanece hasta el estado terminal o reconciliación.
10. **El stock físico se descuenta solo después de que el pago es aprobado** y el hold es consumible; una aprobación sin hold consumible no descuenta stock y solicita reembolso automático.
11. Recibe confirmación de orden.
12. La orden queda registrada en el historial.

### 7.4 Flujo: Gestión de perfil
1. El cliente accede a su perfil desde el menú de usuario en el header.
2. Visualiza sus datos.
3. Puede editar campos permitidos (pendiente definir cuáles — ver NC-01).
4. Puede cambiar su contraseña.

### 7.5 Flujo: Cierre de sesión
1. El cliente cierra sesión desde el menú de usuario.
2. La sesión se invalida.
3. Se revoca la sesión y se liberan sus reservas `ACTIVE`; la UI elimina su vista derivada. Los holds `CHECKOUT_PENDING` no se liberan por logout.

### 7.6 Flujo: Administrador gestiona categorías
1. El administrador accede al panel de administración.
2. Navega a la sección de categorías.
3. Puede crear una nueva categoría con nombre e imagen.
4. Puede editar una categoría existente (nombre, imagen).
5. Puede eliminar una categoría solo si no tiene productos activos asociados; en caso contrario el sistema bloquea la eliminación.

### 7.7 Flujo: Administrador gestiona productos
1. El administrador accede al panel de administración.
2. Navega a la sección de productos.
3. Puede crear un nuevo producto con nombre, descripción, precios, unidad, stock, imágenes y **una sola categoría**.
4. Puede editar los datos generales de un producto existente, sin alterar existencias ni reservas.
5. Puede eliminar un producto mediante borrado lógico; su retención definitiva sigue pendiente.

### 7.7.1 Flujo: Administrador ajusta stock
1. El administrador autorizado abre la operación explícita de ajuste de stock de un producto.
2. Indica una cantidad delta entera distinta de cero y un motivo obligatorio.
3. El sistema rechaza el ajuste si dejaría el stock físico por debajo de las reservas activas.
4. Si se acepta, actualiza solo el stock físico y registra una auditoría mínima inmutable con administrador, producto, delta, motivo y stock antes/después.
5. Reintentar la misma solicitud no debe duplicar el ajuste; no existe operación administrativa para editar `stock_reserved`.

### 7.8 Flujo: Administrador gestiona banners
1. El administrador accede al panel de administración.
2. Navega a la sección de banners/sliders.
3. Puede crear, editar y eliminar banners (imagen promocional del home).

### 7.9 Flujo: Administrador consulta órdenes
1. El administrador accede al panel de administración.
2. Navega a la sección de órdenes.
3. Visualiza el listado de órdenes de todos los clientes.
4. Puede ver el detalle de cada orden (items, total, estado, método de pago).
5. **No puede modificar la orden** (no cambiar estados, no cancelar, no editar).

### 7.10 Flujo: Cliente consulta historial de órdenes
1. El cliente accede a "Mis Órdenes" desde el menú de usuario.
2. Visualiza el listado de órdenes anteriores.
3. Puede ver el detalle de cada orden (items, total, estado, método de pago).

---

## 8. Entidades Funcionales

> **Nota:** Estas son entidades de negocio, no esquemas de DB. Planner definirá el modelo de persistencia.

| Entidad | Datos funcionales clave | Observaciones |
|---|---|---|
| **Usuario** | ID, nombre de usuario, email, contraseña (hash), rol (`admin` o `cliente`), datos de perfil (pendiente definir campos), dirección de envío (pendiente confirmar) | Exactamente dos roles. |
| **Categoría** | ID, nombre, imagen (una), productos asociados | La plantilla usa `icon`; el usuario pidió "categorías con imagen". |
| **Producto** | ID, nombre, descripción, precio regular, precio de venta, unidad de medida, imágenes, **una sola categoría asociada**, stock físico, stock reservado y disponibilidad | El stock reservado pertenece exclusivamente al ciclo de carrito/reserva/pago; la disponibilidad es stock físico menos reservado. |
| **Carrito** | Sesión de servidor, items (producto + cantidad), subtotal, IVA, entrega, total y estado de reserva | Funciona en modo invitado y autenticado; no es un carrito local persistido. |
| **Reserva de stock** | Producto, cantidad, estado, vencimiento o hold de checkout | Al agregar se activa; en checkout queda pendiente hasta resultado terminal; evita overselling. |
| **Ajuste administrativo de stock** | Producto, administrador, delta no cero, motivo, stock antes/después, fecha | Auditoría mínima inmutable; no modifica stock reservado ni reservas activas directamente. |
| **Orden** | ID, usuario, items, subtotal, IVA, entrega, total, estado, método de pago, fecha, dirección de envío | IVA 19% calculado una vez con HALF_UP COP. Estados canónicos en Master Spec/OpenAPI. |
| **Pago** | ID, orden, proveedor de pago (Wompi o Mercado Pago), estado del pago, referencia externa del proveedor, monto | Dos proveedores confirmados. |
| **Banner/Slider** | ID, nombre, imagen | Gestión confirmada en scope del admin. |
| **Imagen** | Referencia a objeto en S3, tipo (producto/categoría/banner) | Almacenamiento en Amazon S3. *(Histórico superseded: "URL pública" queda sustituido por S3 privado servido mediante CloudFront + OAC; la lectura se hace vía URL prefirmada de corta duración, sin acceso público al bucket.)* |

---

## 9. Integraciones

| Sistema externo | Propósito | Dirección del flujo | Criticidad |
|---|---|---|---|
| **Wompi** (proveedor predeterminado) | Procesar pagos de clientes | Bidireccional (crear transacción → recibir webhook de confirmación) | Crítica — sin esto no hay checkout |
| **Mercado Pago** (segunda opción de pago) | Procesar pagos alternativos | Bidireccional (crear transacción → recibir webhook de confirmación) | Crítica — fallback de pago |
| **Amazon S3** | Almacenamiento de imágenes de productos, categorías y banners | Salida (upload) + Entrada (lectura vía URL) | Alta — afecta visualización de catálogo |
| **PostgreSQL** | Persistencia de datos | Bidireccional | Crítica — base del sistema |
| **Servicio de email** (para recuperación de contraseña) | Envío de enlace/token de recuperación de contraseña | Salida | Alta — la recuperación de contraseña está en scope. Uso adicional (ej. confirmación de orden) pendiente confirmar (ver NC-03). |

### Notas sobre integraciones
- **Wompi** es el proveedor de pago predeterminado (gateway colombiano), confirmado por el usuario.
- **Mercado Pago** es la segunda opción de pago, confirmada por el usuario.
- Flujo canónico de pago: creación desde reservas activas → hold `CHECKOUT_PENDING` → callback/webhook firmado y reconciliación → consumo atómico del hold y descuento de stock solo con aprobación consumible. Una aprobación no consumible inicia reembolso automático idempotente.

---

## 10. Seguridad y Restricciones

### Acceso
- El carrito de invitado está permitido; **el login se exige al pagar**, no al agregar al carrito.
- Una sesión válida no debe solicitar login otra vez.
- Solo usuarios autenticados pueden pagar, ver perfil y ver historial de órdenes.
- Solo administradores pueden acceder al panel de administración.
- El admin **solo puede consultar órdenes**; no puede modificarlas.
- Las contraseñas deben almacenarse hasheadas (no en texto plano).
- La sesión/JWT no debe exponerse en URLs ni logs.

### Datos sensibles
- Datos de pago: **no** deben almacenarse en el sistema (tokenización vía proveedor). Solo referencias externas y estados.
- Datos personales del cliente: email, nombre, dirección de envío. Pendiente definir política de retención y cumplimiento de la Ley 1581 (protección de datos personales de Colombia) — ver NC-08.

### Auditoría
- Confirmada para ajustes de stock: se registra administrador, producto, delta, motivo y stock antes/después. La auditoría de otras acciones administrativas sigue fuera de alcance hasta nueva decisión.

### Abuso esperado
- Un usuario no puede ver el carrito ni las órdenes de otro usuario.
- Un cliente no puede acceder a rutas de administración.
- Un admin no puede modificar órdenes (solo lectura).
- Validación de inputs en formularios de registro, perfil, recuperación de contraseña y admin.
- Rate limiting en endpoints de autenticación y recuperación de contraseña (prevención de fuerza bruta y abuso de recuperación).
- Protección contra CSRF, XSS, SQL injection (responsabilidad de Planner, pero se registra como requisito funcional).

---

## 11. Edge Cases

| Escenario | Comportamiento esperado |
|---|---|
| Visitante agrega al carrito sin sesión | Permitido. El servidor crea sesión guest, carrito servidor y reserva activa; la UI no persiste el carrito localmente. |
| Visitante intenta pagar sin sesión | Redirigir a login en el punto de checkout; preservar el carrito para continuar tras autenticarse. |
| Usuario con sesión válida intenta pagar | No se le solicita login otra vez; continúa directo al pago. |
| Cierre de sesión | Revoca sesión y libera reservas `ACTIVE`; no libera holds `CHECKOUT_PENDING`. |
| Producto en el carrito queda sin stock antes del pago | No ocurre overselling si la reserva activa fue creada; aumentos o checkout con reserva expirada/no válida se rechazan y el cliente refresca el carrito. |
| Concurrencia: dos clientes agregan/pagan el último item | Una sola reserva/consumo puede confirmar; el otro flujo recibe conflicto de disponibilidad sin desbalancear reservas. |
| Pago aprobado: descuento de stock | El stock se descuenta únicamente después de confirmación aprobada y consumo de hold equivalente. |
| Pago aprobado sin hold consumible | No descuenta inventario; inicia un reembolso automático idempotente y deja evidencia operativa. |
| Pago fallido o rechazado por el proveedor | Mostrar error, permitir reintentar con mismo o el otro proveedor. El stock no se descuenta. |
| Webhook de pago llega antes de que la orden exista en BD | Se deduplica y se reconcilia idempotentemente contra la referencia del proveedor; no modifica inventario sin hold consumible. |
| Webhook de pago no llega (timeout del proveedor) | La orden sigue pendiente y el job de reconciliación consulta al proveedor; tras su ventana máxima libera hold o procesa aprobación tardía con reembolso si no es consumible. |
| Categoría eliminada que tiene productos asociados | Se bloquea la eliminación mientras existan productos activos asociados. |
| Imagen de producto/categoría/banner falla al subir a S3 | Mostrar error al admin, no crear/actualizar el registro sin imagen (o permitir placeholder — pendiente confirmar). |
| Carrito vacío en checkout | Bloquear checkout, mostrar mensaje. |
| Cantidades inválidas (cero, negativo, excesivo) | Validar en frontend y backend. |
| Doble precio (mrp y sellingPrice) cuando sellingPrice > mrp | Validar que sellingPrice <= mrp o manejar como precio sin oferta. |
| Usuario edita perfil con email ya existente | Rechazar con mensaje claro. |
| Recuperación de contraseña para email inexistente | No revelar si el email existe o no (seguridad); comportamiento de no-confirmación. |
| Concurrencia: dos admins editan el mismo producto/categoría/banner | **Resuelta/canónica:** optimistic locking por `version`; la actualización exige el `version` esperado vía `If-Match` y un desajuste responde `409`. No aplica al ajuste de stock, que ya tiene lock transaccional e idempotencia. |
| Sesión expira durante checkout | Las reservas `ACTIVE` vencen/liberan; los holds `CHECKOUT_PENDING` sobreviven a la inactividad hasta terminal/reconciliación. |
| Ajuste administrativo reduciría stock bajo reservado | Rechazar el ajuste, conservar stock y reservas, y no crear una auditoría de ajuste exitoso. |
| Reintento de ajuste administrativo | No duplica el cambio de stock ni la auditoría para la misma intención idempotente. |

---

## 12. Criterios de Aceptación

### Portal del Cliente
- **CA-01**: Un visitante puede ver el home con sliders, categorías e imágenes, y productos populares sin necesidad de iniciar sesión.
- **CA-02**: Un visitante puede hacer clic en una categoría y ver solo los productos de esa categoría.
- **CA-03**: Un visitante puede ver el detalle de un producto (imagen, descripción, precios, unidad, stock/disponibilidad, selector de cantidad).
- **CA-04**: Un visitante puede agregar productos al carrito sin iniciar sesión (carrito de invitado) y el contador del header se actualiza.
- **CA-05**: Un usuario puede registrarse con nombre, email y contraseña y quedar autenticado con rol `cliente`.
- **CA-06**: Un usuario puede iniciar sesión y cerrar sesión.
- **CA-07**: Un usuario puede recuperar su contraseña mediante el flujo de recuperación (sin necesidad de iniciar sesión previa).
- **CA-08**: Un cliente autenticado puede ver su carrito, cambiar cantidades y eliminar items, con recálculo del subtotal.
- **CA-09**: Un visitante con carrito de invitado, al iniciar checkout, es redirigido a login; tras autenticarse, el carrito se preserva y continúa el checkout.
- **CA-10**: Un cliente con sesión válida puede iniciar checkout sin que se le solicite login otra vez.
- **CA-11**: Un cliente puede seleccionar entre Wompi (predeterminado) y Mercado Pago como proveedores de pago.
- **CA-12**: Tras un pago exitoso, la orden queda registrada con estado apropiado, visible en el historial, y **el stock de los productos se descuenta**.
- **CA-13**: Tras un pago fallido, la orden queda en estado apropiado, el cliente puede reintentar, y **el stock no se descuenta**.
- **CA-14**: Un cliente puede ver y editar su perfil.
- **CA-15**: Un cliente puede ver su historial de órdenes con detalle de cada una.
- **CA-16**: La búsqueda en el header filtra productos por nombre o descripción y muestra resultados.
- **CA-17**: Al cerrar sesión se liberan solo reservas `ACTIVE` del carrito servidor y la UI elimina su vista derivada; los holds `CHECKOUT_PENDING` permanecen hasta un estado terminal.
- **CA-17a**: El carrito guest/autenticado y sus reservas se mantienen en servidor; ni carrito ni tokens se persisten en `localStorage`.
- **CA-17b**: Antes del pago, el cliente ve IVA 19% calculado una vez como `floor((items_subtotal_cop * 19 + 50) / 100)`, entrega fija 5000 COP y total exacto.

### Panel de Administración
- **CA-18**: Un administrador puede crear, editar y eliminar categorías con imagen.
- **CA-19**: Un administrador puede crear, editar y eliminar productos con múltiples imágenes, precios, descripción, unidad y **una sola categoría**; la edición general no modifica existencias ni reservas.
- **CA-19a**: Un administrador puede aumentar/reducir stock mediante delta entero no cero y motivo obligatorio; el resultado nunca es inferior al stock reservado y las reservas activas no se modifican directamente.
- **CA-19b**: Cada ajuste exitoso deja una auditoría mínima inmutable con administrador, producto, delta, motivo y stock antes/después; un reintento de la misma intención no duplica el ajuste.
- **CA-20**: Un administrador puede crear, editar y eliminar banners/sliders del home.
- **CA-21**: Un administrador puede consultar (ver listado y detalle) las órdenes de todos los clientes, pero **no puede modificarlas** (sin cambiar estados, cancelar ni editar).
- **CA-22**: Un cliente que intenta acceder al panel de administración es bloqueado y redirigido.
- **CA-23**: Las imágenes de productos, categorías y banners se almacenan en Amazon S3 privado y se sirven mediante CloudFront con OAC (sin acceso público al bucket; lectura vía URL prefirmada de corta duración). *(Histórico superseded: "se sirven vía URL pública" queda sustituido por S3 privado + CloudFront/OAC.)*

### No Funcionales
- **CA-24**: El sistema usa PostgreSQL como base de datos.
- **CA-25**: El contrato API se define con OpenAPI antes de la implementación (API Design First).
- **CA-26**: La cobertura de pruebas es mínima 85% por archivo testeable.
- **CA-27**: Los precios se muestran en COP (pesos colombianos) y el idioma de la interfaz es español.

---

## 13. Preguntas Abiertas

### Preguntas CRÍTICAS (bloquean handoff a Planner)

| # | Pregunta | Impacto |
|---|---|---|
| — | **No hay preguntas críticas abiertas.** Todas las preguntas críticas funcionales fueron resueltas por el usuario en esta revisión. | — |

### Preguntas NO críticas (no bloquean Planner, pero deben resolverse durante el ciclo)

| # | Pregunta | Impacto |
|---|---|---|
| NC-01 | ¿Qué campos exactos tiene el perfil de usuario? (¿teléfono, dirección, ciudad, código postal?) | Detalle de entidad Usuario |
| NC-02 | **Resuelta/canónica:** los estados de orden y pago están definidos en Master Spec y OpenAPI; no introducir estados de logística en este incremento. | Trazabilidad histórica |
| NC-03 | ¿Se requiere envío de emails además de la recuperación de contraseña? (ej. confirmación de orden) | Integración de email |
| NC-04 | ¿El admin puede comprar como cliente en el portal? | Permisos |
| NC-05 | Provisionamiento posterior de administradores adicionales. El único admin inicial ya está definido por bootstrap seguro; no hay auto-registro admin. | Permisos futuros |
| NC-06 | **Resuelta/canónica:** se bloquea la eliminación de categoría con productos activos. | Trazabilidad histórica |
| NC-07 | **Parcialmente resuelta:** productos/categorías/banners usan borrado lógico; la política definitiva de eliminación y retención sigue pendiente. | Cumplimiento |
| NC-08 | IVA resuelto: 19% HALF_UP al peso COP, aplicado una vez; retención Ley 1581 sigue pendiente. | Cumplimiento legal |
| NC-09 | Auditoría resuelta para ajustes de stock; la auditoría de otras acciones administrativas permanece pendiente. | Trazabilidad |
| NC-10 | ¿Se requiere paginación en listados de productos? | UX y performance |

---

## 14. Supuestos

> **Histórico superseded:** estos supuestos reflejan el levantamiento inicial. Las decisiones ya consolidadas en la sección 3 y los artefactos canónicos prevalecen; no deben reutilizarse para implementación.

| # | Supuesto | Riesgo si es falso |
|---|---|---|
| S-01 | **Superseded:** backend a definir. | El backend canónico es NestJS TypeScript, monolito modular hexagonal con PostgreSQL/Prisma. |
| S-02 | **Superseded:** frontend Next.js 14+. | Storefront y admin son React SPA; Next.js es solo referencia visual histórica. |
| S-03 | **Superseded:** carga de imágenes a través del backend. | El flujo canónico usa URL prefirmada directa a S3, validada antes de registrar el media. |
| S-04 | La cobertura del 85% aplica a archivos testeables (lógica de negocio, endpoints, utilidades), no a archivos de configuración o puramente visuales. | Si aplica a todo, el esfuerzo de testing aumenta. |
| S-05 | **Superseded:** carrito guest local sin backend. | El carrito guest es de servidor y reserva stock desde la adición. |
| S-06 | Recuperación por email con enlace/token de un solo uso y expiración. | Confirmada canónicamente; la política de retención sigue pendiente. |

---

## 15. Handoff para Planner

### Resumen funcional
merkee.shop es un portal de supermercado online para **Colombia (COP, español)** con storefront y panel administrativo. Invitados y clientes usan un carrito de servidor que reserva stock desde la adición; al checkout se exige autenticación y se conserva el carrito guest. El IVA es 19% HALF_UP al peso COP aplicado una vez, más entrega fija de 5000 COP. El pago usa **Wompi** o **Mercado Pago**; el stock físico se descuenta solo cuando una aprobación consume su hold pendiente. El administrador gestiona datos generales de producto, categorías y banners, ajusta stock mediante delta/motivo auditado, y **consulta órdenes en modo solo lectura**. El sistema usa PostgreSQL, API Design First, S3 para imágenes y exige 85% de cobertura de pruebas.

### Decisiones confirmadas por el usuario (no requieren acción de Planner)
1. Mercado: Colombia. Moneda: COP. Idioma: español.
2. Stock se descuenta **solo después de pago aprobado**.
3. Admin puede gestionar banners/sliders.
4. Admin **solo consulta órdenes** (no las modifica).
5. Cada producto pertenece a **exactamente una categoría**; una categoría puede tener múltiples productos (cardinalidad 1:N desde categoría hacia productos). *(Histórico superseded: la cardinalidad "1:1" queda sustituida por 1:N categoría→productos.)*
6. Carrito de servidor para guest y cliente; logout libera reservas `ACTIVE`, no holds `CHECKOUT_PENDING`, y la UI no persiste carrito/tokens localmente.
7. Se requiere creación de usuarios, **recuperación de contraseña** y exactamente **dos roles**: `admin` (panel) y `cliente` (portal).
8. **Wompi** es proveedor predeterminado; **Mercado Pago** es la segunda opción.
9. Carrito de invitado permitido; login requerido al pagar; sesión válida no debe solicitar login otra vez.
10. IVA 19%: `floor((items_subtotal_cop * 19 + 50) / 100)` una vez sobre subtotal; entrega fija 5000 COP.
11. Ajuste administrativo explícito de stock con delta no cero, motivo y auditoría mínima; no cambia reservas directamente.

### Decisiones históricas superseded por planificación canónica
Stack, sesión, persistencia de carrito, concurrencia/reservas, webhooks/reconciliación, eliminación de categoría, recuperación de contraseña y auditoría de ajustes fueron decididos en los artefactos canónicos. Solo siguen abiertas retención Ley 1581, auditoría administrativa fuera de ajustes de stock, campos finales de perfil/dirección, emails adicionales, si el admin puede comprar y paginación.

### Riesgos funcionales para Planner
- **Histórico superseded:** el riesgo de overselling sin reserva queda mitigado por reservas de servidor y holds de checkout; debe validarse contra la Master Spec, no rediseñarse desde este brief.
- **Riesgo medio**: La dualidad mrp/sellingPrice sugiere lógica de ofertas que no está definida funcionalmente (¿descuentos automáticos? ¿cupones?). Por ahora se trata como precio regular vs precio de venta.
- **Riesgo medio**: El flujo de pago asíncrono (webhooks) con dos proveedores (Wompi y Mercado Pago) requiere diseño cuidadoso de idempotencia y reconciliación.
- **Histórico superseded:** el carrito guest ya es de servidor y se promueve al autenticar; no se sincroniza un carrito local.

### Contradicciones detectadas entre la plantilla y los requisitos del usuario
| # | Contradicción | Detalle |
|---|---|---|
| **X-01** | Backend: Strapi vs PostgreSQL + API Design First | La plantilla usa Strapi (headless CMS) con su API auto-generada. El usuario pide PostgreSQL y API Design First, lo que implica un backend custom con contrato OpenAPI diseñado manualmente. **No se puede mantener Strapi y cumplir API Design First de forma natural.** |
| **X-02** | Imágenes: Strapi media library vs Amazon S3 | La plantilla sirve imágenes desde Strapi o Unsplash. El usuario pide S3. |
| **X-03** | Auth: Strapi auth vs auth custom | La plantilla usa `/auth/local/register` y `/auth/local` de Strapi. Un backend custom requiere implementar autenticación propia, incluyendo recuperación de contraseña. |
| **X-04** | Roles: sin roles vs admin/cliente | La plantilla no tiene concepto de roles ni panel de administración. merkee.shop define exactamente dos roles. |
| **X-05** | Cobertura: sin tests vs 85% | La plantilla no tiene tests. El requisito de 85% es nuevo. |
| **X-06** | Carrito: solo add-to-cart con login vs carrito completo + invitado + checkout | La plantilla requiere login para agregar al carrito. merkee.shop permite carrito de invitado y exige login solo al pagar. |
| **X-07** | Órdenes: sin gestión vs consulta solo lectura | La plantilla no tiene gestión de órdenes. merkee.shop permite al admin consultar órdenes en modo solo lectura. |

### Estado de handoff
`planning/pending`. Este brief no habilita handoff: prevalecen la Master Spec y contratos canónicos, y faltan la revisión de Spec Validator y la aprobación humana. *(Histórico superseded: el dictamen formal de Solution Architect quedó resuelto con veredicto final `ready-with-final-verdict` el 2026-08-15.)*

---

## Historial de revisiones

| Fecha | Autor | Acción |
|---|---|---|
| 2026-08-14 | Requirements Analyst | Creación inicial del brief basada en plantilla y requisitos verbales del usuario. Estado: `requirements-blocked`. |
| 2026-08-14 | Requirements Analyst | Cierre del brief con decisiones confirmadas por el usuario: mercado Colombia/COP, stock post-pago, admin gestiona banners, admin solo consulta órdenes, producto una sola categoría, carrito se pierde al cerrar sesión (superseded: logout libera solo reservas `ACTIVE`; los holds `CHECKOUT_PENDING` sobreviven hasta estado terminal de pago/reconciliación), dos roles (admin/cliente), recuperación de contraseña, Wompi predeterminado + Mercado Pago secundario, carrito de invitado con login al pagar. Eliminada la advertencia incorrecta sobre el PDF. Decisiones técnicas (stack, sesión, concurrencia de stock) delegadas a Planner. Estado: `ready-for-planner`. |
| 2026-08-15 | Planner | Sincronización con decisiones canónicas: IVA 19% HALF_UP COP aplicado una vez, carrito/reservas de servidor guest/autenticado, hold `CHECKOUT_PENDING`, reembolso automático y ajuste administrativo de stock con auditoría mínima. Afirmaciones incompatibles se marcan `superseded`; estado pasa a `planning`. |
