# Reporte de Pruebas Funcionales - Frontend

**Última Actualización:** 2026-08-21 12:19:00 UTC
**Estado Global:** FAILED
**Ejecutor:** functional-tester-agent
**Alcance:** Checkout E2E controlado en PRODUCCIÓN (https://www.merkee.shop + https://api.merkee.shop). No destructivo. Sin modificaciones a Git/AWS/código. Sin creación de usuario ni de orden/pago real (solo 1 hold de carrito guest, transitorio, expiró por inactividad).

---

## 1. Resumen de Ejecución

- **Total Tests Ejecutados:** 6 (4 escenarios funcionales + 2 controles de mecanismo)
- **Pasados:** 3 (carrito guest, login en checkout, auth-gate 401)
- **Fallados:** 3 (410 persistente en checkout autenticado — RCA abajo; desajuste HTTP 400 vs body 401; DELETE ítem 500)
- **Entorno de Red:** Producción (api.merkee.shop / www.merkee.shop)
- **Herramientas:** Puppeteer MCP (navegador, origen www.merkee.shop) + curl (controles de cookie/status). No se usaron credenciales (sin registro ni login).

---

## 2. Errores Detectados

### [ERR-001] - 410 Gone persistente en checkout por desajuste cookie-guest vs sesión autenticada (ROOT CAUSE)

- **Componente / Ruta:** `POST https://api.merkee.shop/v1/checkouts` (módulo `checkout`) + `POST /auth/login` (módulo `identity`) + storefront `projects/merkee-shop-storefront`.
- **Escenario Asociado:** TS_003 / TS_004 (checkout guest→cliente).
- **Comportamiento Esperado (OpenAPI):** El checkout requiere rol `cliente` (bearer). Un invitado con carrito debe poder loguearse y que su carrito (reservas) se conserven y conviertan en el checkout. El contrato NO declara 410 para `/checkouts` (solo 201/400/401/403/409/422/500). El 410 `Gone` solo está definido para endpoints de carrito ("Sesión o reserva de carrito expirada").
- **Comportamiento Actual:** El 410 NO se reproduce en la ruta NO autenticada (guest/anónimo → 401 `AUTHENTICATION_REQUIRED`, correcto). El 410 solo es alcanzable en la ruta AUTENTICADA, y su mecanismo quedó probado: cualquier sesión de carrito no resoluble → `SESSION_EXPIRED` → **410**. La causa raíz es que el carrito del invitado y la sesión autenticada son **sesiones distintas y nunca reconciliadas**.

- **Evidencia (red, sin secretos):**
  - Guest cart funciona: `GET /v1/cart` (anónimo) → 200; `POST /v1/cart/items` → 201; `GET /v1/cart` → 200 con 1 ítem, `items_subtotal_cop=5000`, `delivery_fee_cop=5000`, `iva_cop=950`, `total_cop=10950`, `reservation_expires_at=2026-08-21T12:21:16Z` (reserva de 10 min guest). Cookie `merkee_cart_session` creada y persistida (HttpOnly; no legible por JS, pero el carrito persiste entre requests → confirmado funcionalmente).
  - Checkout guest (cookie presente, sin bearer) → HTTP 400 / body `status:401, code:AUTHENTICATION_REQUIRED` (puerta de auth correcta, NO 410).
  - Checkout anónimo puro (sin cookie ni bearer) → idéntico 401.
  - **Mecanismo 410 probado:** `GET /v1/cart` con cookie guest inexistente → **HTTP 410, code:SESSION_EXPIRED, message:"La sesión ha expirado."** (cart-reservation `GetCartUseCase` → `CartErrors.sessionExpired()` → mapeado a 410 en `domain-error-mapper.ts` líneas 58-59).

- **Causa Raíz Sospechada (RCA exacto):**
  1. El carrito del invitado se crea bajo una sesión **GUEST** identificada por la cookie `merkee_cart_session` (session id distinto al del JWT).
  2. `CreateCheckoutUseCaseImpl` resuelve el carrito **exclusivamente por `command.sessionId` = `session_id` del JWT del cliente autenticado** (`checkout.controller.ts` → `getActor(req).sessionId` → `cartRepo.findCartWithItems(command.sessionId)`). **NO** usa el resolver de cookie (cuya prioridad cookie>bearer solo aplica a los endpoints de carrito, no a checkout).
  3. El login SÍ tiene un branch de promoción guest→cliente (`login.use-case.ts`: `if (command.guestSessionId) { ... transferGuestCart(guestSession.id, newSessionId); revoke(guestSession.id) }`), **pero ese branch es inalcanzable**: el contrato OpenAPI `LoginRequest = {email, password}` y el tipo del storefront `LoginRequest = {email, password}` **no incluyen `guestSessionId`**, y el storefront nunca lee/envía la cookie `merkee_cart_session` al login.
  4. Resultado: tras login, el JWT apunta a una sesión autenticada **sin carrito** (el carrito sigue huérfano bajo la cookie guest). El checkout busca el carrito por la sesión auth → no lo encuentra → `CheckoutErrors.sessionExpired()` (si el lookup de sesión del módulo cart no resuelve la sesión auth) → **410 Gone**, o `checkoutNotAllowed()` → 422 si la resuelve pero vacía.
  5. Esto es exactamente el "410 persiste (cookie guest vs sesión)": la cookie guest y la sesión autenticada son disjoint; el checkout mira la sesión equivocada (auth) y la reporta como "gone".

- **Estado:** BLOCKED (requiere cambio de contrato OpenAPI + storefront + posible ajuste backend; NO es un fix mecánico de frontend).

### [ERR-002] - Desajuste de status HTTP (400) vs cuerpo ApiErrorResponse (status:401) en checkout no autenticado

- **Componente / Ruta:** `POST /v1/checkouts` sin bearer.
- **Comportamiento Esperado:** status HTTP 401 coherente con el cuerpo `ApiErrorResponse.status=401`.
- **Comportamiento Actual:** línea de estado HTTP **400**, pero el cuerpo declara `status:401, code:AUTHENTICATION_REQUIRED`. Inconsistencia de proyección de error.
- **Evidencia:** `HTTP 400` + body `{"status":401,"error":"Unauthorized","code":"AUTHENTICATION_REQUIRED",...}`.
- **Causa Raíz Sospechada:** El `TransportAuthGuard`/`controller` lanza con status 400 en lugar de 401 al faltar el bearer, o el `result-projector` no propaga el status del `DomainError`. Bug de proyección de error (menor, no bloquea el RCA).
- **Estado:** TODO (backend, no bloqueante).

### [ERR-003] - DELETE /cart/items/{productId} devuelve 500 en producción

- **Componente / Ruta:** `DELETE https://api.merkee.shop/v1/cart/items/{productId}` (con cookie guest válida).
- **Comportamiento Esperado:** 204 (reserva liberada) o 404 si no existe.
- **Comportamiento Actual:** **HTTP 500, code:TECHNICAL_DEPENDENCY_FAILURE**. El ítem de prueba (1 "Arroz") NO se liberó; el hold expiró por inactividad (~12:21:16Z) de forma transitoria.
- **Evidencia:** `{"status":500,"error":"Internal Server Error","code":"TECHNICAL_DEPENDENCY_FAILURE","path":"/v1/cart/items/2c565fbc-...","trace_id":"18cb2b04-..."}`.
- **Causa Raíz Sospechada:** Excepción técnica no clasificada en `remove-cart-item.use-case` / adaptador de stock al liberar el hold. Hallazgo secundario, no central para el RCA del 410.
- **Estado:** TODO (backend).

---

## 3. Plan de Remediación

- **ERR-001 (RAÍZ):** Derivar al `planner` / usuario. Acciones sugeridas (cualquiera de estas cierra el 410):
  1. **Opción A (storefront + contrato):** Añadir `guest_session_id` al `LoginRequest` (OpenAPI + tipo storefront) y enviar el valor de `merkee_cart_session` al login para activar `transferGuestCart` (promoción guest→cliente). Requiere también que el registro lo soporte.
  2. **Opción B (backend checkout):** Que `CreateCheckoutUseCase` resuelva el carrito por la cookie guest (`CartSessionResolverPort`) cuando el actor sea cliente y su sesión auth no tenga carrito, fusionando el carrito guest al cliente.
  3. **Opción C (transitoria/defensiva):** Si no hay carrito para la sesión auth, devolver **422 `CHECKOUT_NOT_ALLOWED`** (no 410 `SESSION_EXPIRED`), para no confundir "sesión expirada" con "carrito vacío/desvinculado".
  - **Asignado a:** planner / usuario (cambio de contrato + backend + storefront; fuera del alcance de fix mecánico de frontend).
- **ERR-002:** Corregir el status HTTP de la respuesta de auth-gate a 401 en `checkout.controller` / `result-projector`. Asignado a: backend.
- **ERR-003:** Investigar `remove-cart-item.use-case` (liberación de hold de stock) tras 500. Asignado a: backend.

---

## 4. Notas de Ejecución No Destructiva

- No se creó usuario, no se logueó con credenciales, no se adivinaron passwords.
- No se creó orden/pago real: el POST `/v1/checkouts` solo se intentó sin bearer (→ 401) y con body válido pero sin auth; no alcanzó 201.
- La única reserva creada (1 ítem "Arroz", guest) fue transitoria y expiró por inactividad (~12:21:16Z); el intento de liberación vía DELETE falló con 500 (ERR-003) pero el hold no es permanente.
- No se expusieron tokens, cookies (HttpOnly) ni secretos en este reporte.
- No se modificó Git, AWS ni el código fuente; este reporte es un artefacto local de prueba.
