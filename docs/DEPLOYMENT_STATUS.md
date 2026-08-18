# Estado de Despliegue — merkee.shop (AWS)

> **ADVERTENCIA DE ESTADO:** Este documento describe el estado operativo del
> despliegue AWS al momento de la redacción. **No se afirma que el servicio esté
> sano ni que el despliegue productivo esté terminado.** El estado final requiere
> verificación en AWS en vivo (ECS running/desired, health check, logs). Cualquier
> afirmación de "producción estable" debe quedar pendiente hasta contar con esa
> evidencia.

- **Fecha de captura:** 2026-08-18
- **Región:** `us-east-1`
- **Ambientes:** un único ambiente (cuenta de aprendizaje; ID `275201671637` según `AUDITORIA_AWS_MERKEE.md`, lectura read-only, sin mutación).
- **Fuente de verdad del sistema:** `docs/specs/master_spec.md` (`validated-not-executed` / `revision-needed` según trazabilidad) y `docs/api/openapi.yaml`.
- **Documentos relacionados:** `README.md` (workspace), `projects/merkee-shop-api/README.md` (API), `AUDITORIA_AWS_MERKEE.md` (auditoría read-only), `docs/specs/technical_debt.md` (TD-AWS-*).

---

## 1. Avances del proyecto (estado local confirmado en disco)

Estos avances corresponden a la **implementación local** (repositorios en `projects/`), no al entorno AWS. No constituyen evidencia de despliegue productivo.

- **API modular NestJS + ROP (Programación Orientada a Resultados):** monolito modular hexagonal con 8 módulos — `identity`, `media`, `catalog`, `cart-reservation`, `orders`, `payments`, `checkout`, `admin-query`. Todo caso de uso devuelve `Result<Success, DomainError>`; errores de negocio por el rail `Failure`; excepciones solo como fallo técnico traducido a `TECHNICAL_DEPENDENCY_FAILURE` (ADR-017).
- **Persistencia Prisma / migraciones 001–014:** PostgreSQL gestionado por Prisma Migrate (nunca `db push`). Secuencia 001–014 aplicada localmente: identidad/auth, tokens de activación, catálogo/media, carrito/reservas, órdenes/pagos/outbox, ajustes de stock auditados, `idempotency_records` (007–013) y `password_reset_tokens_active_unique_index` (014). Seed no productivo (catálogo dummy, idempotente, sin PII/credenciales).
- **Catálogo:** categorías, productos (soft delete), ajustes de stock auditados con `If-Match` / `Idempotency-Key`.
- **Carrito / reservas:** carrito de servidor con reserva por ítem, reaper cada 1 min, `stock_reserved` agregado.
- **Checkout / órdenes:** cálculo IVA 19% HALF_UP, entrega 5000 COP, transición `ACTIVE→CHECKOUT_PENDING`; órdenes con snapshot de dirección.
- **Pagos / webhooks / reconciliación:** estrategia Wompi / Mercado Pago, webhook firmado validado sobre raw body, reembolsos idempotentes, reconciliación. (En dev: `FakePaymentProviderAdapter`; proveedores externos no configurados.)
- **Media S3:** URLs prefirmadas S3 (en dev: `FakeS3MediaStorageAdapter`; adapter S3 real existe, AWS no configurado).
- **Storefront / Admin (React):** portal storefront (SPA, flujo de compra) y panel admin (React + Refine, RBAC solo lectura). Ambos con código en disco (no se afirman porcentajes de cobertura para ellos).
- **Tests y cobertura del API (local):** última medición local confirmada (`npm run test:cov`, exit code 0, según `projects/merkee-shop-api/README.md`):
  - **125 suites / 1232 tests PASS**
  - **Statements:** 93.36%
  - **Branches:** 84.43%
  - **Functions:** 93.01%
  - **Lines:** 93.57%
  - `build` OK, `depcruise` sin violaciones de arquitectura.
  - *Nota:* esta medición es local; no es reporte de CI externo ni de producción.

**Pendientes de producción (gates abiertos, no bloquean desarrollo local):** email `NoopEmailAdapter`, `FakeS3MediaStorageAdapter` / `FakePaymentProviderAdapter` en dev, `CartReservationPort` noop en identity (TD-MSF-API-001), protecciones HTTP de borde (TD-NEW-HTTP-SEC), revalidación focalizada de Spec Validator tras MSF-ID-003.

---

## 2. AWS configurado (un ambiente, us-east-1)

Estado consolidado de la auditoría read-only (`AUDITORIA_AWS_MERKEE.md`, 2026-08-18) y la documentación de specs. **No se afirma despliegue terminado.**

| Componente AWS | Propósito | Estado (requiere verificación en vivo) |
|---|---|---|
| **ECS Fargate** | Contenedor de la API | Task definition `merkee-backend-task` (revision 2 según specs; revision 1 según auditoría — discrepancia de revisiones pendiente de reconciliar). Servicio `merkee-backend-service` con `Desired: 1` pero `Running: 0` en la auditoría. Task role `merkee-backend-task-role` creado. **En despliegue / pendiente de verificación.** |
| **ECR** | Registro de imágenes | Repositorio `merkee-backend-api` existe. Según auditoría: **0 imágenes** (bloqueante). Dockerfile multi-stage no-root con build local validado; push pendiente de verificación. |
| **RDS PostgreSQL** | Base de datos | `merkee-db` (PostgreSQL 18.3, disponible). Riesgo: auditoría indicó `PubliclyAccessible=True` (TD-AWS-RDS-PUBLIC, no se afirma corrección). Endpoint/SG en auditoría: `sg-0ba7750721b6422a0`, puerto 5432. |
| **S3 + CloudFront/OAC** | Hosting SPAs y media | Buckets `merkee-frontend-client` (CloudFront `E32P11SX9DFU82` → `merkee.shop`) y `merkee-frontend-admin` (CloudFront `E119IKP00L5RU` → `admin.merkee.shop`) desplegados. Bucket de media `merkee-media` referenciado en el ecosistema (nombre canónico del proyecto; verificar existencia/estado en vivo). `aws-s3-tickets-images` queda excluido (no pertenece al proyecto). |
| **Secrets Manager** | Secretos de app | Secreto `merkee/app` creado y referenciado por la task definition vía mapeo `secrets` JSON. **No se exponen valores.** Variables inyectadas: las del cuadro de `Variables de entorno` del API README (solo nombres). |
| **CloudWatch** | Logs/métricas | Log group `/ecs/merkee-backend-task` creado (0 bytes en auditoría). Alarms/observabilidad no configuradas (TD-AWS-OBSERVABILITY). Local: bridge Prometheus (`prom-client`). |
| **Route53 / ACM** | DNS / TLS | DNS gestionado en Spaceship (no Route53). `api.merkee.shop`, `merkee.shop`, `admin.merkee.shop` existen. `swagger.merkee.shop` pendiente de distribución/origen (TD-AWS-SWAGGER-DNS). ACM wildcard `*.merkee.shop` válido. |
| **IAM / OIDC** | Identidad de carga de trabajo | Role OIDC de GitHub `merkee-github-actions-deploy` con trust ajustado a subjects GitHub reales (con IDs). Task role `merkee-backend-task-role` creado. Perfil local AWS CLI `merkee`. |
| **GitHub Actions** | CI/CD | Workflows de API/storefront/admin migrados a OIDC; validación CI antes de deploy. CI anterior falló por OIDC/permissions y secreto CloudFront vacío; fixes aplicados, pero **no se afirma** que el deploy final terminó (TD-AWS-ECS-VALIDATION). |
| **SNS/SQS/DLQ / KMS** | Efectos desacoplados / cifrado | Pendientes de configuración AWS. Lógica de reintentos/idempotencia es local; sin Step Functions (ADR-007). |

**Implementado localmente (no requiere AWS):** scheduler diario de purga de `idempotency_records` (driving adapter, `setTimeout`, UTC default `02:00`); métricas Prometheus locales; validación de firma de webhooks sobre raw body; migraciones Prisma reproducibles; Dockerfile multi-stage no-root con engines Prisma y OpenSSL (ver §3).

---

## 3. Acciones realizadas (configuración infraestructura)

Acciones de configuración reportadas como ejecutadas. **Requieren verificación contra AWS en vivo** (la auditoría read-only previa no las confirma todas, p. ej. ECR sin imágenes, RDS público, Secrets Manager vacío en la auditoría — posiblemente anterior a estas acciones).

1. **Security Group ECS → RDS (`TCP 5432`):** regla de salida/entrada para que la tarea ECS alcance `merkee-db` por 5432. *Verificar SG de la tarea y del RDS en vivo; la auditoría listó SG `sg-0ba7750721b6422a0` en RDS.*
2. **Mapeo de Secrets:** task definition `merkee-backend-task` (revision 2) con `secrets` JSON apuntando al secreto `merkee/app` de Secrets Manager; las variables de entorno de la app se inyectan desde ese secreto (no de `.env` local). *No se documentan ni exponen valores.*
3. **Task role:** IAM role `merkee-backend-task-role` creado y asociado a la task definition (permite acceso a Secrets Manager / S3 según políticas aplicadas).
4. **OIDC trust:** role IAM `merkee-github-actions-deploy` con trust policy ajustada a los subjects (`sub`) reales de GitHub (con IDs de repos/branches), para federación sin claves de larga duración.
5. **Dockerfile Prisma engines / OpenSSL:** evidencia en `projects/merkee-shop-api/Dockerfile`:
   - Stage build: `npx prisma generate` con `binaryTargets` multi-plataforma (native + `debian-openssl-1.1.x` + `debian-openssl-3.0.x`) definidos en `schema.prisma` (líneas 53–58).
   - Stage runtime: instalación de `libargon2-1` y `openssl`; el paquete `openssl` provee `libssl3` + symlinks no versionados (`libssl.so`, `libcrypto.so`) para que Prisma cargue el engine `debian-openssl-3.0.x` (líneas 82–90).
   - Imagen no-root (`USER node`), `ENV PORT=3000`, `EXPOSE 3000` (líneas 96, 112–116).

---

## 4. Incidente actual (ECS)

> Estado del incidente basado en el reporte operativo. **Requiere verificación en AWS en vivo**; no se afirma resolución.

- **Síntoma:** ECS arranca el contenedor NestJS, pero el servicio entra en **rollback / circuit breaker** (la tarea no alcanza estado estable; `Running` no igual a `Desired`).
- **Fase 1 — P1001 (conectividad RDS):** error Prisma `P1001` ("Can't reach database server") por conectividad a `merkee-db`. **Reportado como ya corregido** (acción §3.1 SG ECS→RDS 5432). *Verificar conectividad real desde el task role antes de cerrar.*
- **Fase 2 — health check / puertos desalineados:** tras superar la conectividad, el despliegue falla por desalineación de puertos y health check:
  - App escucha en **3000** (`main.ts`: `PORT ?? 3000`; Dockerfile `EXPOSE 3000`).
  - Task definition mapea el contenedor en **80** (según reporte del incidente).
  - Target group del ALB apunta a **8080** (según reporte del incidente).
  - El health check del target group no coincide con la ruta/estado del contenedor.
- **Endpoint `/health` — hallazgo de discrepancia (importante):** el reporte del incidente indica "falta desplegar endpoint `/health`". Sin embargo, **el código fuente SÍ contiene el endpoint**:
  - `projects/merkee-shop-api/src/shared/http/health.controller.ts` → `@Controller()` + `@Get('health')` (línea 23), devuelve `{ status: 'ok', timestamp }` sin auth ni BD.
  - `projects/merkee-shop-api/src/shared/http/health.module.ts` lo registra; `app.module.ts` lo importa (evidencia: `import { HealthModule }`).
  - **Pero** el `Dockerfile` (líneas 118–121) tiene un comentario desactualizado que afirma "el main.ts no expone endpoint /health real" y deja el `HEALTHCHECK` comentado. Este comentario es **stale** y contradictorio con el código actual.
  - **Conclusión documental:** el endpoint existe en la imagen construida desde la fuente actual; lo que falta/requiere verificación es (a) que la **imagen desplegada en ECR** corresponda a esa fuente, y (b) que la **ruta de health check del target group** apunte a `/health` y al puerto correcto. No se afirma que el endpoint esté ausente en la imagen vigente.
- **ALB / target group / ECS:** pendiente alinear ALB→ECS (SG, AZ/subnets) y el health check con `/health` en el puerto real del contenedor.

**No se afirma servicio sano.** El estado final (running = desired, health check OK, logs sin errores) está pendiente de verificación.

---

## 5. Próximos pasos exactos (para cerrar el incidente)

1. **Publicar / verificar `/health`:** confirmar que la imagen en ECR incluye `GET /health` (reconstruir y pushear si la imagen desplegada es anterior a la adición del `HealthController`); corregir el comentario stale del `Dockerfile` (fuera de alcance de este doc, pero requerido).
2. **Alinear puerto del contenedor a 3000:** la app escucha en 3000; la task definition debe mapear el container port a `3000` (no 80) o el target group debe apuntar a 3000.
3. **Alinear target group → 3000:** el target group del ALB debe registrar el contenedor en el puerto 3000 (no 8080), coincidiendo con `EXPOSE 3000` / `PORT=3000`.
4. **SG ALB → ECS:** regla de seguridad que permita al ALB llegar al puerto 3000 de la tarea ECS (y ECS → RDS 5432 ya aplicada, verificar).
5. **AZ / subnets:** verificar que la tarea ECS se despliega en las AZ/subnets con ruta a RDS y a Internet (NAT/IGW según arquitectura) y que el target group las cubre.
6. **Health check `/health`:** configurar el health check del target group con path `/health`, protocol HTTP, puerto 3000, umbrales acordes (p. ej. interval 30s, timeout 5s, healthy 2, unhealthy 3, start period ≥10s).
7. **Verificar `running = desired`:** tras el despliegue, confirmar en ECS que `Running: 1` == `Desired: 1` y que no hay eventos de rollback/circuit breaker; revisar CloudWatch `/ecs/merkee-backend-task`.
8. **Swagger DNS:** habilitar `swagger.merkee.shop` (TD-AWS-SWAGGER-DNS) apuntando al ALB/origen correspondiente una vez el servicio esté estable.

---

## 6. Seguridad

- **Sin secretos en este documento:** no se incluyen tokens, credenciales, valores de Secrets Manager ni de `.env`. Solo se mencionan **nombres** de recursos (`merkee/app`, `merkee-backend-task-role`, `merkee-github-actions-deploy`, etc.).
- **Pagos tokenizados:** PAN/CVV/fecha nunca se aceptan ni almacenan; el webhook es autoritativo y su firma se valida sobre raw body.
- **Sesiones:** JWT acceso ≤10 min en memoria; refresh/cart token opaco `HttpOnly; Secure; SameSite=Lax`; Argon2id.
- **RBAC:** `admin`/`cliente`; el admin no accede a carrito/checkout/órdenes propias.
- **RDS:** riesgo `PubliclyAccessible=True` pendiente de corregir a `False` antes de producción (TD-AWS-RDS-PUBLIC).
- **HTTP edge:** protecciones helmet/CSP/HSTS/nosniff, CORS allowlist, CSRF, rate limiting de login/registro/reset/activación aún pendientes (TD-NEW-HTTP-SEC).

---

## 7. Enlaces de repositorios y ramas

| Repositorio | Propósito | Enlace |
|---|---|---|
| `merkee-workspace` | Specs, contratos y documentación canónica (este doc) | https://github.com/cristiansrc/merkee-workspace |
| `merkee-shop-api` | API backend (NestJS, hexagonal/ROP) | https://github.com/cristiansrc/merkee-shop-api |
| `merkee-shop-storefront` | Portal de tienda (React SPA) | https://github.com/cristiansrc/merkee-shop-storefront |
| `merkee-shop-admin` | Panel administrativo (React + Refine SPA) | https://github.com/cristiansrc/merkee-shop-admin |

**Estructura de ramas** (cada repositorio):

- `main` — integración / posible despliegue a producción.
- `qa` — control de calidad y validación.
- `developer` — desarrollo activo (target de las tareas del task board).

---

## 8. Pendientes de verificación (no bloqueantes para este doc, sí para producción)

- ECR con imagen vigente que incluya `GET /health` (TD-AWS-ECS-VALIDATION).
- ECS `running == desired` y sin rollback/circuit breaker.
- RDS `PubliclyAccessible=False` y conectividad ECS→RDS verificada.
- Secrets Manager poblado (`merkee/app`) y mapeado en task definition.
- CloudWatch alarms/observabilidad (TD-AWS-OBSERVABILITY).
- `swagger.merkee.shop` resuelto (TD-AWS-SWAGGER-DNS).
- Revalidación focalizada de Spec Validator tras MSF-ID-003.
