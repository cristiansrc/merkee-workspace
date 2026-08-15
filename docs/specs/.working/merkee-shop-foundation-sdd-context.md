# SDD Shared Context — merkee-shop-foundation

## Current status
`planning`. Se añadió ADR-017 y reglas ROP transversales el 2026-08-15. El estado objetivo tras una nueva validación sigue siendo `awaiting-human-plan-approval`, pero el cambio invalida el ready histórico y exige Spec Validator review antes de cualquier commit, handoff o transición. No existe task board. No se ejecutó Git, scripts, migraciones ni código. Graphify no está configurado: `graphify-out/GRAPH_REPORT.md` no existe; no se ejecutó `graphify --update` por instrucción explícita de no ejecutar scripts.

## Canonical artifacts
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md`
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/requirements/merkee-shop-foundation-requirements-brief.md`
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md`
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md`
* `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md`

## Artifact evidence
| Ruta absoluta | Campo, endpoint, columna o flujo verificado | Resultado observado | Estado |
|---|---|---|---|
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` | §ROP, catálogo `DomainError`, límites y pruebas | `Result<Success, DomainError>`, proyección a `ApiErrorResponse`, flujos obligatorios y dependency-cruiser definidos. | pass; cambio pendiente de revalidación |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml` | `/auth/password-change`, `/auth/admin-activations`, `/admin/users`, cart 403 | Endpoints/schemas/status/idempotencia definidos; cart endpoints documentan 403 admin. API Governance ejecutó parseo YAML real y `npx @redocly/cli lint docs/api/openapi.yaml`: válido sin errores; advertencia no bloqueante: falta `info.license`. | pass; API Governance lint OK; pendiente revalidación Spec Validator |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md` | migración 002, invariantes y retención | Token hash de activación, consumo atómico, índices y pruebas futuras definidos. | pass; pendiente revalidación Spec Validator |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` | ADR-017 | Decisión ROP, límites y dependency-cruiser registrada. | pass; cambio pendiente de revalidación |
| `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/requirements/merkee-shop-foundation-requirements-brief.md` | requisito 9, CA-09..11 | Requisito y aceptación de ROP/pruebas incorporados. | pass; cambio pendiente de revalidación |

## Spec Validator Approval
verdict: pending
reviewed_at: none
validator_agent: spec-validator
artifact_set_reviewed: /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md, /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/requirements/merkee-shop-foundation-requirements-brief.md, /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml, /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/prisma-migration-contract.md, /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md, /home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-sdd-context.md
summary: El veredicto `ready` histórico del 2026-08-15 quedó invalidado por ADR-017 y las correcciones F1-F5. Pendiente nueva validación de Spec Validator sobre el conjunto canónico antes de cualquier commit.
invalidated_by_changes_since: 2026-08-15: ADR-017 (ROP estricto, catálogo `DomainError`, límites de adapters y dependency-cruiser) y las correcciones F1-F5 invalidan el ready histórico; requiere nueva validación de Spec Validator.

## Decisions locked
IVA 19% HALF_UP; ajuste de stock auditado; If-Match; S3 privado; guest/cliente reservan; ACTIVE 10m y reaper/minuto; CHECKOUT_PENDING hasta terminal; admin no compra; guest→admin libera ACTIVE; no auto-registro admin; provisión solo por admin con contraseña cambiada; activación hashada/única/24h con índice parcial único `WHERE used_at IS NULL` y vigencia validada atómicamente en el canje (reemisión marca/revoca el token no usado expirado); perfil display_name/phone; dirección snapshot; password-change idempotente y revoca otras sesiones; ROP: todo caso de uso retorna `Result<Success, DomainError>`, excepciones técnicas se traducen solo en adapters, controllers/webhooks sin Prisma/reglas y catálogo HTTP/OpenAPI estable; dependency-cruiser bloquea imports hexagonales prohibidos; notificación compra v2; productos soft delete v1; paginación de productos/órdenes; UI y mensajes al usuario en `es-CO` con valores COP enteros; política NC-08 provisional (Ley 1581) y revisión legal/contable preproducción.

## Validator findings
Correcciones F1-F5 aplicadas y verificadas documentalmente (contract-drift/mechanical): snapshots de entrega en `OrderResponse`, mapeo de autoservicio v1, ownership `identity` de provisión/activación, header `Set-Cookie` en password-change y anonimización v1. ADR-017 (ROP estricto, catálogo `DomainError`, límites de adapters y dependency-cruiser) queda pendiente de nueva validación de Spec Validator antes de commit; no es resoluble por este contexto.

## Resolved findings
NC-01, NC-03, NC-04, NC-05, NC-06/07, NC-08 y NC-10 resueltos por decisión explícita de usuario. Solo quedan riesgos no bloqueantes reales.

## Remediation trace (Spec Remediator, 2026-08-15)
Hallazgos del Solution Architect remediados documentalmente (sin Git, scripts ni código):
- **F-01** (`contract-drift`): `OrderResponse` en OpenAPI ahora incluye snapshots `delivery_recipient_name`, `delivery_line1`, `delivery_city`, `delivery_phone` (required, coherentes con `CreateCheckoutRequest.delivery_address`). Reflejado en contrato Prisma (migración 005, columnas NOT NULL) y requirements brief (req. 5).
- **F-02** (`contract-drift`): mapeo de autoservicio v1 documentado en Master Spec (NC-08), requirements brief (req. 8), ADR-016 y este contexto: acceso vía `GET /me` + `GET /orders`/detalle; rectificación vía `PATCH /me` solo `display_name`/`phone`; email, rol y dirección de perfil no editables en v1. Sin endpoint nuevo.
- **F-03** (`contract-drift`): `POST /v1/admin/users` y `POST /v1/auth/admin-activations` declarados del módulo `identity` en OpenAPI (descriptions), Master Spec y ADR-013; el tag `Admin` solo agrupa HTTP; `admin-query` sigue solo lectura de órdenes.
- **H-01** (`contract-drift`): header `Set-Cookie` declarado en la respuesta 204 de `/auth/password-change` explicando rotación de sesión (conserva solo la sesión actual rotada; revoca las demás).
- **H-02** (`design-decision` técnica): mecanismo de anonimización v1 definido en Master Spec (NC-08), ADR-016, requirements brief y contrato Prisma: `display_name`/`phone` → null/neutro; `password_hash` invalidado con hash aleatorio no reutilizable; `email` → identificador irreversible/no operativo si la política legal lo permite; snapshots de orden preservados solo mientras sean necesarios. Supuestos legales marcados como provisionales, sin asesoría.
- **H-03** (`contract-drift`): DAG corregido en Master Spec y ADR-013 para declarar dependencia directa `checkout`→`cart-reservation` (convierte ACTIVE→CHECKOUT_PENDING usando sus puertos, sin depender transitivamente vía `orders`).
- **H-04** (`contract-drift`): código `INITIAL_PASSWORD_CHANGE_REQUIRED` documentado en la respuesta `Forbidden`/`ApiErrorResponse` y en las operaciones protegidas relevantes (perfil PATCH, carrito, checkout, órdenes, admin, media) sin romper el esquema estándar.
- **Revalidación SA 2026-08-15**: Solution Architect emitió veredicto `ready` sobre ADR-013..016 y los hallazgos F-01/F-02/F-03/H-01/H-02/H-03/H-04 remediados.
- **Revalidación Spec Validator 2026-08-15 (histórica/superseded)**: Spec Validator emitió veredicto `ready` sobre el conjunto canónico (seis rutas). Quedó invalidado por ADR-017 y las correcciones F1-F5; requiere nueva validación. El incremento permanece en `planning` hasta el gate humano; siguiente estado `awaiting-human-plan-approval`.

Decisiones previas preservadas: admin no compra, activación por token, perfil mínimo, retención NC-08, IVA 19% HALF_UP, stock auditado, S3 privado, paginación. El incremento permanece en `planning` hasta la aprobación humana del plan.

## Open questions
1. Canal operacional seguro de entrega de token de activación (no debe ser log/API).
2. Revisión legal/contable de retenciones, anonimización y snapshots antes de producción.
3. Contenido/infraestructura de notificaciones de compra v2.

## Stale terms guard
Prohibidos: admin comprador, auto-registro admin, token/contraseña admin en claro, dirección persistida en perfil, hard delete/purga producto v1, notificación de compra v1, carrito admin, `POST /auth/initial-password-change`, hold checkout de 10 min, `base_fee_cop`, IVA distinto, mutar `stock_reserved`, Strapi, Next.js runtime, carrito local, excepción para error de negocio, Prisma en controller/webhook, import `domain→application|infrastructure`, import `application→infrastructure`.

## Next action
Solicitar revalidación de Spec Validator del conjunto canónico, incluyendo ADR-017 y Master Spec §ROP, antes de cualquier commit. Tras un nuevo veredicto `ready`, el estado requerido será `awaiting-human-plan-approval`; no escribir `## Human Plan Approval: approved_by_user` hasta aprobación explícita de la persona usuaria. No hacer handoff.
