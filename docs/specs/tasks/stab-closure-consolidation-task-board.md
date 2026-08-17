# Task Board — stab-closure-consolidation

**Estado:** `done`
**Incremento:** `stab-closure-consolidation`
**Owner esperado:** Planner/documentation
**Alcance:** exclusivamente documental; no ejecutar Git ni modificar código.
**Ejecutado:** 2026-08-16 (SCC-A1–A7 completadas; verificación en disco registrada abajo).

## Contexto fijado

- **Spec aprobada:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/stab-closure-consolidation-delta-spec.md`
- **Shared context:** `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/stab-closure-consolidation-sdd-context.md`
- **Estado verificado:** `validated-not-executed`.
- **Aprobación:** `## Spec Validator Approval` con `verdict: ready` y `## Human Plan Approval: approved_by_user`.
- **Decomposition Contract:** delta spec, sección `## Decomposition Contract`.

## Restricciones globales

- Archivos permitidos únicamente: `docs/specs/**` y `projects/merkee-shop-api/docs/specs/**`, limitados a los listados en cada tarea.
- Prohibido modificar código, OpenAPI, Prisma, `package.json`, `.env*`, runtime config, tests, scripts, UI, handlers, queries o automatización.
- No crear `docs/specs/archive/` ni mover artefactos; mantener los artefactos de estabilización in-place como `closed`/históricos.
- No marcar `TD-NEW-ROP-SIGN`, `TD-NEW-HTTP-SEC` ni `TD-NEW-COV` como resueltas.
- Marcar resuelta únicamente `TD-NEW-STAB-LIFECYCLE`; mantener activos los gates legal/AWS.
- No fijar `iss`, `aud`, `typ` ni proveedor JWT. No reeditar ADR-017.
- No ejecutar operaciones de Git; la frescura de Graphify permanece `blocked`.

## Tareas

### SCC-A1 — Cerrar lifecycle del incremento de estabilización

- **id:** SCC-A1
- **title:** Alinear delta spec y shared context de estabilización a `closed`
- **agent:** Planner/documentation
- **spec_refs:** SCC-A1; SCC-DEC-01; AC-SCC-01; AC-SCC-02
- **goal:** Corregir la inconsistencia `awaiting-human-plan-approval` frente a la implementación verificada, distinguiendo `implemented` (B1-B5) de `closed` (cierre/consolidación).
- **scope:** Cambiar los encabezados de lifecycle/status del delta spec y del shared context predecesor a `closed`; actualizar `Current status` del shared context predecesor para reflejar cierre; conservar todas las aprobaciones y trazabilidad históricas.
- **out_of_scope:** Código, contratos, OpenAPI, Prisma, deuda distinta de lifecycle, eliminación de aprobaciones o cambio del task board predecesor.
- **inputs:** Delta spec predecesora; shared context predecesor; evidencia del board predecesor `done`; decisiones SCC-DEC-01.
- **implementation_notes:** Editar solo `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` y `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md`. Mantener `verdict: ready`, `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user`.
- **edge_cases:** No usar `superseded`; el incremento fue implementado, no reemplazado. No eliminar la aprobación histórica ni declarar listo para producción.
- **done_criteria:** Ambos artefactos declaran lifecycle `closed`; el shared context refleja cierre; las aprobaciones históricas permanecen intactas.
- **verification:** Leer ambos archivos y comprobar encabezados, `Current status`, aprobaciones y ausencia de cambios fuera de las dos rutas permitidas; registrar pass/fail con rutas absolutas.
- **dependencies:** Ninguna.
- **handoff_context:** SCC-A2 puede citar el estado cerrado; SCC-A3 debe confirmar que los artefactos continúan in-place.
- **source_of_truth:** Delta spec activa, SCC-DEC-01 y shared context activo.
- **stale_terms_guard:** Prohibidos `awaiting-human-plan-approval` como lifecycle vigente del predecesor, `superseded` para el predecesor y eliminación de aprobaciones históricas.
- **status:** `done`
- **executor_notes:** Editados `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md` (líneas 3-4 → `## Status: closed` y `**Lifecycle status:** closed`; añadida nota de cierre línea 9) y `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` (línea 3 → `**Lifecycle status:** closed`; `Current status` línea 7 → `closed` reflejando implemented→closed). Se conservaron `**Spec Validator:** ready`, `## Spec Validator Approval` (verdict: ready), `## Human Plan Approval: approved_by_user` y `## Controlled Pre-Ready Remediation: approved_by_user` como trazabilidad histórica. No se usó `superseded`.
- **verification_result:** PASS — Leídos ambos archivos en disco: delta spec línea 3 `## Status: closed`, línea 4 `**Lifecycle status:** \`closed\``; shared context línea 3 `**Lifecycle status:** \`closed\``, línea 7 `Current status: closed`. Aprobaciones históricas intactas (no editadas). Cambios limitados a las dos rutas permitidas.
- **blocker:** `none`

### SCC-A2 — Consolidar decisiones durables B1-B5

- **id:** SCC-A2
- **title:** Consolidar B1, B3 y B5 en la Master Spec
- **agent:** Planner/documentation
- **spec_refs:** SCC-A2; SCC-DEC-02; SCC-DEC-04; SCC-DEC-07; AC-SCC-03; AC-SCC-11
- **goal:** Hacer durable y concisa en la Master Spec la aplicación ROP y el fail-fast JWT, sin duplicar historial STAB.
- **scope:** Integrar en secciones existentes `§ROP`/`§Identidad` que `activate-admin` y `jwt.verify` usan `Result` con traducción en adapter y sin `try/catch` técnico en application/controller; documentar fail-fast de `JWT_SECRET` en producción (ausente o <32 bytes detiene arranque; default solo desarrollo con advertencia; `.env.example` documenta); verificar AC-05, 404/DUPLICATE y NC-08 ya consolidados.
- **out_of_scope:** Reeditar ADR-017, crear sección histórica, modificar OpenAPI/código/config, o fijar `iss`/`aud`/`typ`/proveedor JWT.
- **inputs:** `docs/specs/master_spec.md`; delta spec §B y decisiones SCC-DEC-02/04/07; ADR-017 solo como referencia.
- **implementation_notes:** Editar exclusivamente `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/master_spec.md`; conservar la decisión pendiente STAB-DEC-12 y no repetir etiquetas STAB-A/B/DEC como reglas nuevas.
- **edge_cases:** B3 solo cubre secreto JWT, no claims ni proveedor; B5 solo cubre traducción ROP de `jwt.verify`; no reescribir lo que ya está consolidado.
- **done_criteria:** Master Spec contiene las tres reglas durables en secciones existentes y mantiene explícitamente pendiente STAB-DEC-12.
- **verification:** Buscar y leer las secciones `§ROP`, `§Identidad`, AC-05 y `§NC-08`; comprobar las tres reglas, ausencia de historial duplicado y que `architecture-decisions.md` no fue modificado.
- **dependencies:** SCC-A1.
- **handoff_context:** SCC-A4/A5 deben usar la Master Spec actualizada como referencia de deuda y evidencia.
- **source_of_truth:** Delta spec §§B, decisiones SCC-DEC-02/04/07 y Master Spec actual.
- **stale_terms_guard:** Prohibido fijar claims/proveedor JWT, presentar `try/catch` técnico en application como patrón vigente o reeditar ADR-017.
- **status:** `done`
- **executor_notes:** Editado únicamente `docs/specs/master_spec.md`: en §ROP (línea 44) se consolidó B1 (activate-admin ROP) y B5 (jwt.verify ROP) como regla durable; en §Identidad (línea 83) se consolidó B3 (fail-fast del secreto JWT en producción) antes de la decisión pendiente. No se reeditó ADR-017 (`architecture-decisions.md` sin modificar); no se duplicaron etiquetas STAB-*; AC-05, §ROP 404/DUPLICATE_WEBHOOK_EVENT y §NC-08 no se reeditaron. STAB-DEC-12 (`iss`/`aud`/`typ`/proveedor) permanece explícitamente pendiente. Se dejó constancia de que `jwt.sign`/register-login-refresh-session try/catch es deuda activa (TD-NEW-ROP-SIGN), no patrón vigente.
- **verification_result:** PASS — Leídas §ROP (línea 44) y §Identidad (línea 83) en disco: presentes las tres reglas durables (B1/B3/B5); STAB-DEC-12 pendiente; `architecture-decisions.md` no modificado (sin edición en esta tarea); sin historial STAB-* duplicado como reglas nuevas.
- **blocker:** `none`

### SCC-A3 — Marcar artefactos in-place como cerrados/históricos

- **id:** SCC-A3
- **title:** Confirmar el lifecycle histórico in-place sin archive
- **agent:** Planner/documentation
- **spec_refs:** SCC-A3; SCC-DEC-06; AC-SCC-12
- **goal:** Dejar explícito que los artefactos del incremento de estabilización permanecen en sus rutas y no son superseded ni archivados.
- **scope:** Verificar que el delta y shared context predecesores quedaron `closed` por SCC-A1 y que el task board predecesor conserva estado superior `done`; añadir solo la señal documental histórica mínima si falta, sin alterar tareas done.
- **out_of_scope:** Crear/mover archivos, crear `docs/specs/archive/`, cambiar estados de tareas o modificar el task board predecesor innecesariamente.
- **inputs:** Tres artefactos predecesores; SCC-DEC-06; resultado SCC-A1.
- **implementation_notes:** Archivos permitidos: los dos predecesores de SCC-A1 y, solo si es indispensable para la señal histórica, `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/tasks/merkee-shop-foundation-stabilization-task-board.md`.
- **edge_cases:** `closed`/histórico no equivale a `superseded`; el task board debe permanecer `done`.
- **done_criteria:** Los tres artefactos siguen in-place, sin archive, con lifecycle/estado conforme al contrato.
- **verification:** Listar directorios `docs/specs/increments`, `docs/specs/.working` y `docs/specs/tasks`; leer los tres archivos y verificar estados/rutas; confirmar que no existe creación o movimiento a archive.
- **dependencies:** SCC-A1.
- **handoff_context:** Las tareas posteriores deben referenciar estos artefactos por sus rutas actuales.
- **source_of_truth:** SCC-DEC-06, delta spec §C y AC-SCC-12.
- **stale_terms_guard:** Prohibidos `superseded` para estos artefactos, archive, mover archivos o cambiar el board predecesor de `done`.
- **status:** `done`
- **executor_notes:** Sin edición de archivos (la señal histórica ya quedó establecida por SCC-A1: delta spec y shared context predecesores `closed`; task board predecesor `done`). Verificación de directorios: `docs/specs/increments/`, `docs/specs/.working/` y `docs/specs/tasks/` existen con los artefactos in-place; glob `docs/specs/archive/**` sin resultados y el listado de `docs/specs/` no contiene `archive/`. Task board predecesor `merkee-shop-foundation-stabilization-task-board.md` línea 5 permanece `Estado: done` (sin alterar).
- **verification_result:** PASS — Tres artefactos in-place en sus rutas (`increments/`, `.working/`, `tasks/`); delta spec y shared context predecesores `closed` (verificado en SCC-A1); task board predecesor `done` (línea 5); no existe `docs/specs/archive/` ni se movieron archivos.
- **blocker:** `none`

### SCC-A4 — Registrar deudas nuevas activas

- **id:** SCC-A4
- **title:** Registrar TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC y TD-NEW-COV como `active`
- **agent:** Planner/documentation
- **spec_refs:** SCC-A4; SCC-DEC-03; AC-SCC-04
- **goal:** Sincronizar formalmente las tres deudas nuevas activas en los registros global y local.
- **scope:** Añadir las tres filas con descripción, impacto, responsable, condición de cierre, evidencia requerida y estado idénticos en ambos registros.
- **out_of_scope:** Corregir deuda, editar código, `package.json`, configuración HTTP/Jest o cerrar cualquiera de las tres entradas.
- **inputs:** Contenido exacto de delta spec §§D y “Nuevas deudas técnicas”; evidencia TD-NEW-* del shared context; ambos `technical_debt.md` actuales.
- **implementation_notes:** Editar únicamente `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md` y `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md`; conservar formato y sincronía.
- **edge_cases:** TD-NEW-ROP-SIGN es solo register/login/refresh-session; TD-NEW-HTTP-SEC sigue gate de producción; TD-NEW-COV no autoriza tocar `package.json`.
- **done_criteria:** Las tres filas existen en ambos registros, son idénticas en contenido y `active`, y no se modifican otras deudas por esta tarea.
- **verification:** Leer ambas tablas, comparar las tres filas y comprobar que no contienen `resolved`; verificar diff documental limitado a las dos rutas permitidas.
- **dependencies:** SCC-A2.
- **handoff_context:** SCC-A5 añadirá únicamente la deuda de lifecycle resuelta; SCC-A7 verificará los gates existentes.
- **source_of_truth:** Delta spec §§D y “Nuevas deudas técnicas” filas TD-NEW-ROP-SIGN/HTTP-SEC/COV.
- **stale_terms_guard:** Prohibido declarar estas tres deudas resueltas, implementar sus correcciones o tocar `package.json`/código.
- **status:** `done`
- **executor_notes:** Editados únicamente `docs/specs/technical_debt.md` (global) y `projects/merkee-shop-api/docs/specs/technical_debt.md` (espejo local): añadidas TD-NEW-ROP-SIGN, TD-NEW-HTTP-SEC y TD-NEW-COV a la tabla "Deuda activa" (tras TD-MSF-API-007), con contenido exacto de la delta spec §§D, responsables, condición de cierre, evidencia requerida y estado `active` idénticos en ambos registros. Ninguna contiene `resolved`. No se modificaron otras deudas existentes.
- **verification_result:** PASS — Leídas ambas tablas en disco (global líneas 21-23, local líneas 21-23): las tres filas existen, idénticas, `active`, sin `resolved`. Diff documental limitado a las dos rutas permitidas.
- **blocker:** `none`

### SCC-A5 — Registrar y resolver solo la deuda de lifecycle

- **id:** SCC-A5
- **title:** Registrar TD-NEW-STAB-LIFECYCLE como resuelta por implementación verificada
- **agent:** Planner/documentation
- **spec_refs:** SCC-A5; SCC-DEC-03; AC-SCC-05
- **goal:** Documentar la deuda de lifecycle y su resolución exclusivamente mediante evidencia ya verificada y SCC-A1.
- **scope:** Añadir TD-NEW-STAB-LIFECYCLE en “Cambios completados — no son deuda activa” de ambos registros con estado `resolved-by-verified-implementation` y evidencia del board `done`, B1-B5, TD-MSF-STAB-001/002/003/004 y SCC-A1.
- **out_of_scope:** Resolver TD-NEW-ROP-SIGN/HTTP-SEC/COV o TD-MSF-ID-002-02/03; cambiar evidencia de implementación.
- **inputs:** Ambos registros de deuda; delta spec fila TD-NEW-STAB-LIFECYCLE; Artifact evidence del shared context de estabilización y SCC-A1.
- **implementation_notes:** Editar solo los dos `technical_debt.md` canónicos; mantener idéntico el texto y la clasificación en ambos.
- **edge_cases:** `implemented` describe B1-B5; `closed` describe el incremento; no mezclar ambos como un único estado ambiguo.
- **done_criteria:** La entrada existe idéntica en ambos registros, en cambios completados, con estado exacto `resolved-by-verified-implementation` y evidencia completa.
- **verification:** Leer y comparar ambas entradas; comprobar referencias a board `done`, B1-B5, cuatro deudas STAB resueltas y SCC-A1; confirmar que solo esta deuda fue marcada resuelta.
- **dependencies:** SCC-A1, SCC-A4.
- **handoff_context:** SCC-A6 documentará el resumen; SCC-A7 comprobará que los gates permanecen activos.
- **source_of_truth:** Delta spec §§D/E y fila TD-NEW-STAB-LIFECYCLE.
- **stale_terms_guard:** Prohibido marcar cualquier deuda nueva distinta de lifecycle como resuelta o declarar cerrados los gates legal/AWS.
- **status:** `done`
- **executor_notes:** Editados únicamente los dos `technical_debt.md` canónicos: añadida TD-NEW-STAB-LIFECYCLE a "Cambios completados — no son deuda activa" (tras TD-MSF-STAB-004), con estado exacto `resolved-by-verified-implementation` y evidencia (task board de estabilización `done` línea 5, B1-B5 verificados en disco, TD-MSF-STAB-001/002/003/004 `resolved-by-verified-implementation`, SCC-A1 aplicado). Texto y clasificación idénticos en ambos registros. Solo esta deuda fue marcada resuelta.
- **verification_result:** PASS — Leídas y comparadas ambas entradas en disco (global línea 37, local línea 37): idénticas, en "Cambios completados", estado `resolved-by-verified-implementation`, con referencias a board `done`, B1-B5, cuatro deudas STAB resueltas y SCC-A1. Confirmado que ninguna otra deuda nueva fue marcada resuelta.
- **blocker:** `none`

### SCC-A6 — Actualizar workspace_changes

- **id:** SCC-A6
- **title:** Registrar cierre, consolidación, deudas y gates en workspace_changes
- **agent:** Planner/documentation
- **spec_refs:** SCC-A6; AC-SCC-06; SCC-DEC-04
- **goal:** Mantener trazabilidad descendente del cierre documental sin inventar cambios de API, rutas o dependencias.
- **scope:** Añadir una sección de cambio descendente que registre lifecycle `closed`, B1-B5 verificados y consolidados, estados TD-NEW-*, gates legal/AWS activos y decisión JWT pendiente.
- **out_of_scope:** Cambiar OpenAPI, rutas, endpoints, schemas, dependencias cross-service, estados de producción o decisiones JWT pendientes.
- **inputs:** `docs/specs/workspace_changes.md`; resultados SCC-A1, SCC-A2, SCC-A4, SCC-A5 y decisión SCC-DEC-04.
- **implementation_notes:** Editar exclusivamente `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md`; conservar las secciones históricas como trazabilidad y no reescribir contratos.
- **edge_cases:** Registrar que Graphify sigue blocked para frescura y que no hubo Git; no afirmar `ready` de Master Spec ni producción.
- **done_criteria:** La sección documenta todos los puntos SCC-A6 y no introduce cambios de contrato ni dependencias.
- **verification:** Leer la sección nueva y las tablas de deuda/gates; comprobar presencia de cada punto y ausencia de rutas/endpoints/schemas nuevos.
- **dependencies:** SCC-A2, SCC-A4, SCC-A5.
- **handoff_context:** SCC-A7 realiza la comprobación final de gates; el registro sirve de evidencia documental de cierre.
- **source_of_truth:** Delta spec §E, AC-SCC-06 y SCC-DEC-04.
- **stale_terms_guard:** Prohibido afirmar gates cerrados, JWT canónicamente decidido, Graphify fresco o cambios cross-service.
- **status:** `done`
- **executor_notes:** Editado únicamente `docs/specs/workspace_changes.md`: añadida sección "## Cierre y consolidación — stab-closure-consolidation (SCC-A1–A7, 2026-08-16)" (tras el encabezado) con los cinco puntos: cierre (lifecycle `closed`, B1-B5 verificados), consolidación B1-B5 en Master Spec, registro TD-NEW-* (tres `active` + lifecycle `resolved-by-verified-implementation`), gates legal/AWS abiertos, decisión JWT pendiente. Se conservaron las secciones históricas como trazabilidad. Se registró que Graphify permanece blocked para frescura y que no hubo Git. No se afirmó `ready` ni producción.
- **verification_result:** PASS — Leída la sección nueva en disco (líneas 7-15): presentes los cinco puntos; sin rutas/endpoints/schemas/dependencias cross-service nuevos; sin afirmar gates cerrados, JWT decidido, Graphify fresco ni producción lista.
- **blocker:** `none`

### SCC-A7 — Confirmar gates de producción abiertos

- **id:** SCC-A7
- **title:** Verificar y conservar activos los gates legal/AWS
- **agent:** Planner/documentation
- **spec_refs:** SCC-A7; AC-SCC-07; SCC-DEC-03
- **goal:** Asegurar que TD-MSF-ID-002-02 y TD-MSF-ID-002-03 siguen `active` y no se declara producción lista.
- **scope:** Revisar ambos registros de deuda y `workspace_changes.md`; corregir únicamente una discrepancia documental explícita para mantener ambos gates activos y el resumen sincronizado.
- **out_of_scope:** Obtener evidencia legal/contable, configurar AWS, cambiar scheduler, desplegar o resolver gates.
- **inputs:** Dos `technical_debt.md`; `docs/specs/workspace_changes.md`; delta spec §F y evidencia actual.
- **implementation_notes:** Archivos permitidos únicamente `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/technical_debt.md`, `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/docs/specs/technical_debt.md` y `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/specs/workspace_changes.md`.
- **edge_cases:** Si ya están `active`, no editar por ruido; no añadir evidencia de cierre ni cambiar lifecycle global a `closed`.
- **done_criteria:** Ambos gates constan como `active`, están descritos como gates antes de producción y no hay afirmación de readiness productiva.
- **verification:** Leer las tres rutas y registrar campo, texto observado y estado `pass`; confirmar que no se modificaron código/OpenAPI/Prisma/package/config ni se ejecutó Git.
- **dependencies:** SCC-A6.
- **handoff_context:** Entrega evidencia final al cierre documental y a la revalidación focalizada posterior.
- **source_of_truth:** Delta spec §F, AC-SCC-07 y registros de deuda canónicos.
- **stale_terms_guard:** Prohibido declarar TD-MSF-ID-002-02/03 resueltas o afirmar listo para producción/AWS configurado.
- **status:** `done`
- **executor_notes:** Verificación sin edición (los gates ya estaban `active`; no se editó por ruido, per edge case). Revisados `docs/specs/technical_debt.md` (líneas 12-13), `projects/merkee-shop-api/docs/specs/technical_debt.md` (líneas 12-13) y `docs/specs/workspace_changes.md` (sección nueva punto 4, línea 14; tabla existente líneas 26-27): TD-MSF-ID-002-02 y TD-MSF-ID-002-03 constan `active`, descritos como gates antes de producción, sin afirmación de readiness productiva. No se modificaron código/OpenAPI/Prisma/package/config ni se ejecutó Git.
- **verification_result:** PASS — Tres rutas leídas en disco: ambos gates `active` en global (líneas 12-13) y local (líneas 12-13); workspace_changes los describe `active` como gates antes de producción (línea 14 y líneas 26-27). Sin afirmación de producción lista. Estado `pass` registrado.
- **blocker:** `none`

## Dependencias y orden de ejecución

`SCC-A1 → SCC-A2 → SCC-A4 → SCC-A5 → SCC-A6 → SCC-A7`; `SCC-A3` depende de `SCC-A1` y puede ejecutarse antes de SCC-A2. Todas las tareas son documentales y deben ejecutarse exclusivamente después de este board.

## Archivos prohibidos explícitos

- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/src/**`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/projects/merkee-shop-api/prisma/**`
- `/home/cristiansrc/Documentos/Proyectos/merkee-workspace/docs/api/openapi.yaml`
- `/home/cristiansrc/Documentos/Proyectos/merkee-shop-api/package.json`
- cualquier `.env*`, runtime config, tests, scripts, UI, handlers, queries o automatización
- cualquier operación Git y cualquier creación de `docs/specs/archive/`

## Blockers globales

- **Blocker:** ninguno para crear este board; las tareas quedan `todo`.
- La frescura de `graphify-out/GRAPH_REPORT.md` respecto a `HEAD` permanece `blocked` por la prohibición de Git; no es una tarea SCC.
- Los gates legal/AWS permanecen abiertos y no bloquean la ejecución documental, pero bloquean producción.

## Registro de ejecución

| Tarea | Estado | Archivos cambiados | Notas de ejecución | Verification result | Blocker |
|---|---|---|---|---|---|
| SCC-A1 | `done` | `docs/specs/increments/merkee-shop-foundation-stabilization-delta-spec.md`; `docs/specs/.working/merkee-shop-foundation-stabilization-sdd-context.md` | Lifecycle predecesor alineado a `closed` en delta spec (líneas 3-4) y shared context (línea 3 + `Current status` línea 7); aprobaciones históricas conservadas. | PASS (lectura en disco de encabezados, `Current status` y aprobaciones) | `none` |
| SCC-A2 | `done` | `docs/specs/master_spec.md` | B1/B5 consolidados en §ROP (línea 44); B3 consolidado en §Identidad (línea 83); STAB-DEC-12 pendiente; ADR-017 no reeditado; AC-05/§ROP/§NC-08 no reeditados. | PASS (lectura en disco de §ROP/§Identidad; `architecture-decisions.md` sin modificar) | `none` |
| SCC-A3 | `done` | (sin edición) | Verificación in-place: tres artefactos en sus rutas; delta/context predecesores `closed`; board predecesor `done`; sin `archive/`. | PASS (listado de directorios + glob `archive/**` sin resultados + board predecesor línea 5 `done`) | `none` |
| SCC-A4 | `done` | `docs/specs/technical_debt.md`; `projects/merkee-shop-api/docs/specs/technical_debt.md` | TD-NEW-ROP-SIGN/HTTP-SEC/COV añadidas como `active` (idénticas en ambos registros); sin `resolved`. | PASS (lectura en disco global líneas 21-23 y local líneas 21-23) | `none` |
| SCC-A5 | `done` | `docs/specs/technical_debt.md`; `projects/merkee-shop-api/docs/specs/technical_debt.md` | TD-NEW-STAB-LIFECYCLE añadida a "Cambios completados" como `resolved-by-verified-implementation` con evidencia (idéntica en ambos); única deuda marcada resuelta. | PASS (lectura en disco global línea 37 y local línea 37) | `none` |
| SCC-A6 | `done` | `docs/specs/workspace_changes.md` | Sección de cierre/consolidación añadida (líneas 7-15): cierre, consolidación B1-B5, deudas TD-NEW-*, gates abiertos, JWT pendiente, Graphify blocked, sin Git. | PASS (lectura en disco; sin rutas/endpoints/schemas/dependencias nuevos) | `none` |
| SCC-A7 | `done` | (sin edición) | Verificación: TD-MSF-ID-002-02/03 `active` como gates antes de producción en ambos registros y workspace_changes; sin afirmar producción. | PASS (lectura en disco global líneas 12-13, local líneas 12-13, workspace_changes línea 14 y 26-27) | `none` |

## Criterio de cierre del board

El board solo puede pasar a `done` cuando SCC-A1–SCC-A7 tengan estado `done`, evidencia documental real en disco, `verification_result` registrado y ningún archivo prohibido modificado. Si surge una decisión no presente en la delta o una contradicción autoritativa, la tarea afectada debe pasar a `blocked` con `blocked_reason`, `conflicting_artifacts`, `required_owner` y `next_required_decision`.
