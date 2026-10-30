---
tags: [mibeca, proyecto, estado]
---

# Estado actual

Última verificación de este vault: **2026-10-29**.
Fuente principal: `docs/AI_CONTEXT.md` de cada repo, `docs/VERIFICACION_FASE_*.md` y el
historial de git.

## Resumen

Las fases 0 a 6 del plan están implementadas. El contrato OpenAPI va por la
**revisión 1.4.0**. Todo corre en local contra PostgreSQL; **no hay despliegue a
producción verificado**.

## Implementado (con evidencia documentada)

| Área | Estado |
|---|---|
| Auth y usuarios (fase 2) | Login/logout, cambio obligatorio, reset, desactivación, último admin protegido. 25 pruebas de API en su momento |
| Trabajos y postulaciones (fase 3) | Publicar, editar antes del inicio, postular, aceptar/rechazar con cupos seguros |
| Control (fase 4) | Cierre transaccional con asistencia exhaustiva, idempotencia, progreso e historiales. Verificado en Chromium: 276/0 minutos para asistente/ausente |
| Dashboards y supervisión (fase 5) | `GET /me/dashboard`, `GET /students/progress`, `GET /students/{id}/history`, paneles por rol |
| Calidad y operación (fase 6) | CI de ambos repos, workflows Firebase, auditor de código, hooks Git/Codex |
| Frontend | 19 archivos de prueba, **135 pruebas** en ~18 s; `lint` y `build` en verde |
| Progreso de beca en postulantes | `GET /jobs/{id}/applications` incluye `semester`, `targetMinutes`, `accreditedMinutes`, `remainingMinutes` (aditivo, contrato 1.4.0) |

## MVP v2: documento en desuso, su contenido pasa a specs

`docs/PLAN_MVP_MIBECA_V2.md` (2026-10-23, v2.0) **está casi deprecado**. Su contenido se
está **migrando a especificaciones** (`specs/` de cada repo, una carpeta por funcionalidad)
y el documento se retirará cuando la migración termine. Por eso **no se ejecuta desde
ahí**: el trabajo nuevo sale de un spec generado con `/speckit.specify` y repartido según
[[Reparto de tareas]].

Hoy `specs/` solo tiene `001-*` en cada repo (`001-becas-upb-management` en backend,
`001-becario-work-management` en web). Las specs 002 en adelante — contextos y
evaluaciones — **todavía no existen**.

Nada de lo que describe el v2 está en el código. Evidencia del 2026-10-29:

| Qué exige el v2 | Estado real |
|---|---|
| Migración `0004_contextos.sql` | **No existe**. `migrations/` solo llega a `0003` |
| Migración `0005_evaluaciones.sql` | **No existe** |
| `openapi.yaml` en revisión **1.5.0** | Sigue en **1.4.0** |
| Módulo `contextos` (taxonomía nueva) | No implementado |
| Módulo `evaluaciones` (4 criterios, promedio) | No implementado |
| UI de evaluaciones | No implementada |

Es decir: el v2 es **intención documentada**, no trabajo entregado. Mientras se migra a
specs se puede leer como referencia de alcance, pero **no como plan de ejecución**. Ver
[[Fuentes en el repositorio]].

## Pendiente o sin verificar

> [!danger] Antes de cualquier despliegue
> 1. **Rotar la contraseña de Supabase** que quedó expuesta en el chat el 2026-10-18.
> 2. Resolver el nombre de los modos SSL: `DB_SSL_MODE=require` se comporta como
>    `verify-ca` del estándar, no como `require`. Ver [[Supabase]].

- **Sin cobertura de pruebas configurada**: el equipo exige cobertura total antes de
  publicar, pero `vitest.config.js` no tiene bloque `coverage`, no hay script `--coverage`,
  la CI no la mide y el backend (`node:test`) tampoco. Es la deuda más urgente porque
  bloquea la regla de relevo de [[Reparto de tareas]].
- **CI remota nunca observada**: las ramas tienen commits locales sin publicar.
- **Cookies entre dominios sin resolver**: si el frontend está en Firebase Hosting y la
  API en otro dominio, las cookies de sesión de terceros pueden ser rechazadas. Hay que
  decidir dominio/ruta antes de declarar producción funcional.
- **CORS del preview de Firebase**: falta una política explícita para el origen dinámico
  que genera Firebase; no se añadió comodín porque el backend valida orígenes exactos.
- **`ApiError.detalles`** se transporta y se prueba, pero ningún formulario lo asigna
  todavía a campos concretos.
- **Verificación de fase 5 parcial**: las pruebas de integración que necesitan
  PostgreSQL de prueba no se ejecutaron en esa sesión; el lint de Redocly tampoco por
  falta de acceso al registry.
- **Confianza de los hooks de Codex** sin verificar en el cliente; su matcher `Bash` no
  cubre comandos lanzados desde PowerShell.
- No se han verificado permisos/protección de rama en GitHub, secretos, ni accesos reales
  a servicios.

## Decisiones funcionales ya cerradas

Código académico de cinco dígitos **como texto**; meta individual de demostración;
contraseña temporal aleatoria entregada una sola vez por el administrador y reset
administrativo; **una actividad y un cierre por trabajo**; vista limitada de
postulantes; **sin correcciones de horas** en la aplicación; exceso de meta informado
por separado.

Relacionado: [[Visión y alcance]] · [[CI y CD]] · [[Reparto de tareas]] · [[Pruebas]]
