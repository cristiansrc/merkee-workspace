# Task Board — merkee-shop-foundation-stabilization

## Control

- **Estado:** `done`
- **Spec canónica:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md`
- **Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md`
- **Contrato API de referencia:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`
- **Regla de ejecución:** alcance exclusivo STAB-B1, STAB-B2, STAB-B3, STAB-B4 y STAB-B5. No modificar OpenAPI, Prisma, migraciones, `package.json`, endpoints ni reglas de negocio.
- **Owner de implementación:** `executor`

## Tareas

### STAB-B1 — Corregir ROP de activate-admin

- **id:** `STAB-B1`
- **title:** Corregir la frontera ROP de la unidad de trabajo de activación de admin
- **agent:** `executor`
- **spec_refs:** Delta Spec §Hallazgos B/STAB-B1, §STAB-DEC-04, §Criterios de aceptación AC-STAB-04 y AC-STAB-07; Master Spec §ROP; ADR-017.
- **goal:** hacer que el adapter de persistencia traduzca fallos técnicos a `Result` y eliminar el `try/catch` técnico de application.
- **scope:** cambiar la firma `ActivateAdminUnitOfWorkPort.run` a `Result<T, DomainError>`; capturar excepciones en `PrismaActivateAdminUnitOfWorkAdapter`, registrar solo el código técnico permitido y devolver `fail(technicalFailure())`; adaptar `ActivateAdminUseCase` para consumir `Result`; actualizar el wiring necesario en `identity.module.ts`; actualizar pruebas afectadas.
- **out_of_scope:** nuevos endpoints, migraciones, cambios Prisma, cambios OpenAPI, nuevas reglas de negocio, cambios en otros casos de uso y cambios de autenticación JWT.
- **inputs:** delta spec §STAB-B1/§STAB-DEC-04; Master Spec §ROP; ADR-017; patrón vigente de `PrismaProvisionUnitOfWorkAdapter`; archivos existentes listados en `source_of_truth`.
- **implementation_notes:** preservar la transacción y el comportamiento funcional de activación; la causa de la excepción no debe entrar en `DomainError`, logs ni respuestas; no usar excepciones como rail de negocio en application.
- **edge_cases:** fallo de `$transaction`; éxito de activación; `Failure` de dominio ya producido por el adapter; ausencia de causa/PII en el resultado y registro.
- **done_criteria:** el puerto devuelve `Result`; el adapter devuelve `fail(technicalFailure())` ante excepción técnica; `activate-admin.use-case.ts` no contiene `try/catch` técnico; DI y tipos compilan; pruebas cubren éxito y fallo técnico sin causa/PII.
- **verification:** ejecutar primero las pruebas unitarias actualizadas; después `npm run build`, `npm test` y `npm run depcruise` desde `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api`; comprobar que no hay dependencia prohibida ni `try/catch` técnico en application.
- **dependencies:** `none`
- **blocking_criteria:** bloquear si el contrato del puerto, el catálogo `DomainError` o el patrón de traducción del adapter no coincide con las fuentes canónicas; bloquear si se requiere editar OpenAPI, Prisma o `package.json`.
- **handoff_context:** al terminar, dejar documentado el resultado de pruebas y archivos cambiados para B5, que comparte `identity.module.ts`.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` §196-202; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` §81-87 y §106-113; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/domain/ports/activate-admin-unit-of-work.port.ts`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/prisma-activate-admin-unit-of-work.adapter.ts`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/application/use-cases/activate-admin.use-case.ts`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/identity.module.ts`.
- **allowed_files:** los cuatro archivos de implementación de `source_of_truth` y sus pruebas afectadas: `activate-admin.use-case.spec.ts`, `prisma-activate-admin-unit-of-work.adapter.spec.ts`.
- **stale_terms_guard:** no reintroducir `Promise<T>` en el puerto, `try/catch` técnico en application, propagación de excepciones del adapter, `HttpException` en application ni causas/PII en `DomainError`.
- **status:** `done`
- **executor_notes:** Implementado ROP en el puerto y adapter de activate-admin; pruebas unitarias actualizadas.
- **verification_result:** Suite focalizada PASS; build PASS.
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### STAB-B2 — Fijar `must_change_password` del replay de provisión

- **id:** `STAB-B2`
- **title:** Alinear el replay de provisión admin con `must_change_password: true`
- **agent:** `executor`
- **spec_refs:** Delta Spec §Hallazgos B/STAB-B2, §STAB-DEC-01, §Criterios de aceptación AC-STAB-05; Master Spec AC-05; OpenAPI `AdminUserProvisionResponse`.
- **goal:** garantizar que el replay de la provisión devuelve la constante contractual `true`, incluso después de la activación del admin.
- **scope:** forzar `must_change_password: true` en `replayFromResource` y tipar el campo como literal `true` en `schemas.ts`; actualizar la prueba de replay afectada.
- **out_of_scope:** modificar OpenAPI, cambiar la semántica de activación, modificar endpoints, cambiar persistencia o introducir DTOs nuevos.
- **inputs:** delta spec §STAB-B2/§STAB-DEC-01; OpenAPI real `AdminUserProvisionResponse.must_change_password.const`; Master Spec AC-05; implementación y prueba existentes.
- **implementation_notes:** conservar la reconstrucción del replay desde el recurso vigente y cambiar únicamente el valor contractual del campo; el contrato OpenAPI permanece sin cambios.
- **edge_cases:** replay después de activación, replay de un admin aún pendiente y tipado que impida asignar un booleano dinámico al campo contractual.
- **done_criteria:** replay devuelve `must_change_password: true`; `schemas.ts` declara el literal `true`; prueba de replay después de activación pasa; OpenAPI no cambia.
- **verification:** ejecutar la suite de `provision-admin-user`; comparar el tipo con OpenAPI; ejecutar `npm run build` y `npm test` desde `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api`.
- **dependencies:** `none`
- **blocking_criteria:** bloquear si para cumplir se necesita cambiar OpenAPI, alterar el recurso persistido o elegir una semántica distinta de la Opción A aprobada.
- **handoff_context:** dejar evidencia del test de replay y del tipo literal para la validación final del incremento.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` §56-63 y §196-202; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` §81-87 y §106-113; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/application/use-cases/provision-admin-user.use-case.ts`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/contract/schemas.ts`.
- **allowed_files:** `projects/merkee-shop-api/src/modules/identity/application/use-cases/provision-admin-user.use-case.ts`, `projects/merkee-shop-api/src/contract/schemas.ts` y `projects/merkee-shop-api/src/modules/identity/application/use-cases/provision-admin-user.use-case.spec.ts`.
- **stale_terms_guard:** no usar el estado actual del recurso para el valor contractual del replay, no cambiar `const: true`, no añadir endpoints ni alterar activación.
- **status:** `done`
- **executor_notes:** Replay contractual y schema alineados a literal true; test tras activación actualizado.
- **verification_result:** Suite focalizada PASS; build PASS.
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### STAB-B3 — Aplicar fail-fast al secreto JWT en producción

- **id:** `STAB-B3`
- **title:** Eliminar el secreto JWT inseguro en arranque de producción
- **agent:** `executor`
- **spec_refs:** Delta Spec §Hallazgos B/STAB-B3, §STAB-DEC-05, §Criterios de aceptación AC-STAB-06 y AC-STAB-07.
- **goal:** impedir que producción arranque con `JWT_SECRET` ausente o menor de 32 bytes, manteniendo un valor de desarrollo explícitamente advertido.
- **scope:** implementar la validación de arranque en `jwt.adapter.ts`; conservar el default únicamente para desarrollo con advertencia explícita; documentar `JWT_SECRET` y la prohibición de versionar su valor real en `.env.example`; añadir o actualizar pruebas de fail-fast.
- **out_of_scope:** cambios en la firma/verificación JWT, cambios de duración, endpoints, OpenAPI, runtime adicional o `package.json`.
- **inputs:** delta spec §STAB-B3/§STAB-DEC-05; Master Spec §Identidad; adapter y configuración de entorno existentes.
- **implementation_notes:** evaluar `NODE_ENV=production` antes de aceptar el secreto; considerar longitud en bytes; el error de arranque no debe exponer el secreto; la advertencia de desarrollo no debe contener secretos.
- **edge_cases:** secreto ausente en producción, secreto de 31 bytes, secreto de exactamente 32 bytes, secreto válido, desarrollo sin secreto y restauración de variables en pruebas.
- **done_criteria:** producción falla al arrancar si falta el secreto o tiene menos de 32 bytes; producción acepta uno válido; desarrollo permite el default solo con advertencia; `.env.example` documenta `JWT_SECRET`; pruebas cubren los límites.
- **verification:** ejecutar las pruebas del adapter; ejecutar `npm run build` y `npm test` desde `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api`; revisar que ningún output de prueba contiene el secreto.
- **dependencies:** `none`
- **blocking_criteria:** bloquear si el cambio requiere modificar `package.json`, `.env` real, OpenAPI o una decisión de contrato no presente en la delta; bloquear si el test no puede aislar/restaurar el entorno.
- **handoff_context:** dejar evidencia de los casos de producción/desarrollo y de los archivos modificados para coordinar con B5, que comparte `jwt.adapter.ts`.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` §80-82 y §196-202; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` §81-87 y §106-113; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/.env.example`.
- **allowed_files:** `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts`, `projects/merkee-shop-api/.env.example` y la prueba existente o nueva del adapter JWT.
- **stale_terms_guard:** no permitir el default en producción, no registrar secretos, no cambiar el contrato JWT fuera del fail-fast del secreto.
- **status:** `done`
- **executor_notes:** Fail-fast de JWT en producción, default advertido solo en desarrollo y documentación de JWT_SECRET.
- **verification_result:** Tests del adapter PASS; build PASS.
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### STAB-B4 — Sincronizar `operation-map` con el 404 de provisión

- **id:** `STAB-B4`
- **title:** Reflejar el status 404 de `provisionAdminUser` en el mapa de operaciones
- **agent:** `executor`
- **spec_refs:** Delta Spec §STAB-DEC-10, §Hallazgos B/STAB-B4, §Criterios de aceptación AC-STAB-07; OpenAPI `provisionAdminUser`.
- **goal:** eliminar el drift de trazabilidad entre OpenAPI y `operation-map.ts` para la operación de provisión.
- **scope:** añadir `404` al conjunto de statuses de `provisionAdminUser`, conservando exactamente `201,400,401,403,404,409,429,500`; actualizar la prueba de trazabilidad si es necesario.
- **out_of_scope:** editar OpenAPI, cambiar handlers, modificar status de webhooks, crear códigos de error o alterar cualquier otra operación.
- **inputs:** delta spec §88-96 y §141; shared context §47 y §86; OpenAPI real línea 205; `operation-map.ts` y su prueba.
- **implementation_notes:** realizar únicamente el cambio de metadata de la operación existente; mantener método, path, schemas, headers e idempotencia sin cambios.
- **edge_cases:** conservar el orden o representación canónica usada por el mapa; no añadir 404 a operaciones distintas; detectar discrepancia con OpenAPI antes de cerrar.
- **done_criteria:** el mapa declara los ocho statuses exactos de `provisionAdminUser`; la prueba de trazabilidad pasa; OpenAPI permanece intacto.
- **verification:** ejecutar `operation-map.spec.ts`; comparar campo por campo con OpenAPI; ejecutar `npm run build` y `npm run depcruise` desde `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api`.
- **dependencies:** `none`
- **blocking_criteria:** bloquear si OpenAPI no contiene el 404 canónico o si la corrección exige modificar otra operación, OpenAPI o un handler.
- **handoff_context:** dejar evidencia de la lista exacta de statuses y de la comparación operation-map↔OpenAPI.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` §88-96 y §196-202; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` §47, §86 y §106-113; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/contract/operation-map.ts`.
- **allowed_files:** `projects/merkee-shop-api/src/contract/operation-map.ts` y `projects/merkee-shop-api/src/contract/operation-map.spec.ts`.
- **stale_terms_guard:** no eliminar el 404 del contrato, no añadir 409 a webhooks, no modificar statuses de operaciones no relacionadas.
- **status:** `done`
- **executor_notes:** operation-map de provisionAdminUser alineado con los ocho statuses canónicos.
- **verification_result:** Test de trazabilidad PASS.
- **executor_notes:**
- **verification_result:**
- **blocker:** `none`

### STAB-B5 — Traducir ROP de `JwtAdapter.verify`

- **id:** `STAB-B5`
- **title:** Convertir la verificación JWT a `Result` en puerto, adapter y controller
- **agent:** `executor`
- **spec_refs:** Delta Spec §STAB-DEC-13, §Hallazgos B/STAB-B5, §Criterios de aceptación AC-STAB-07; Master Spec §ROP; ADR-017.
- **goal:** impedir que las excepciones de `jwt.verify` atraviesen el adapter y hacer que el controller consuma el puerto ROP.
- **scope:** cambiar `JwtPort.verify` a `Result<JwtPayload, DomainError>`; capturar en `JwtAdapter.verify` los fallos de verificación y traducirlos al catálogo existente; traducir errores técnicos inesperados a `TECHNICAL_DEPENDENCY_FAILURE` sin causa/PII; cambiar `identity.controller.ts` para inyectar `JwtPort`, consumir `Success/Failure` y eliminar el `try/catch`; actualizar DI y pruebas afectadas.
- **out_of_scope:** cambios en emisión JWT, secretos de arranque, endpoints, OpenAPI, Prisma o reglas de sesión.
- **inputs:** delta spec §117-124 y §196-202; shared context §132 y §106-113; Master Spec §ROP; ADR-017; catálogo existente `identity-errors.ts`.
- **implementation_notes:** conservar el comportamiento del controller de devolver `undefined` cuando el token no puede aportar identidad; no inyectar el adapter concreto; no inventar códigos de error; no filtrar causa técnica.
- **edge_cases:** token expirado, token inválido, token no válido aún, error inesperado de la biblioteca, `Success` con payload sin claims requeridos y `Failure` consumido por `extractSessionId`/`extractUserId`.
- **done_criteria:** el puerto devuelve `Result`; el adapter traduce fallos esperados al error de autenticación y fallos inesperados al error técnico; el controller solo usa `JwtPort` y `Result`; DI compila; pruebas cubren traducciones y extracción de identidad sin excepciones.
- **verification:** ejecutar las pruebas del adapter y controller; ejecutar `npm run build`, `npm test` y `npm run depcruise` desde `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api`; comprobar que no se filtran causas/PII y que el controller no captura excepciones de verificación.
- **dependencies:** `STAB-B1`, `STAB-B3`
- **blocking_criteria:** bloquear si se necesita modificar otro contrato, el fail-fast del secreto, OpenAPI, `package.json` o cualquier código fuera del puerto, adapter, controller, DI y pruebas autorizadas; bloquear si no puede preservarse el comportamiento `undefined` para `Failure` en extracción.
- **handoff_context:** ejecutar después de B1 y B3 para evitar conflictos en `identity.module.ts` y `jwt.adapter.ts`; dejar evidencia del mapeo de errores y de la inyección por puerto.
- **source_of_truth:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` §117-124 y §196-202; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` §37-39, §81-87, §106-113 y §130-132; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md` §36-42; `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/architecture-decisions.md` ADR-017.
- **allowed_files:** `projects/merkee-shop-api/src/modules/identity/domain/ports/jwt.port.ts`, `projects/merkee-shop-api/src/modules/identity/infrastructure/adapters/jwt.adapter.ts`, `projects/merkee-shop-api/src/modules/identity/identity.controller.ts`, `projects/merkee-shop-api/src/modules/identity/identity.module.ts` y las pruebas existentes o nuevas de adapter/controller.
- **stale_terms_guard:** no propagar excepciones de `jwt.verify`, no inyectar `JwtAdapter` concreto, no inventar códigos, no convertir esta tarea en cambios de emisión o configuración JWT.
- **status:** `done`
- **executor_notes:** JwtPort, adapter, controller y DI alineados al rail ROP; errores JWT sanitizados y extracción consume el puerto.
- **verification_result:** Tests JWT adapter/controller PASS; suite completa, build y dependency-cruiser PASS.
- **blocker:** `none`
