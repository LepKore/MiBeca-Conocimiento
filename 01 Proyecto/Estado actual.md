---
tags: [mibeca, proyecto, estado]
---

# Estado actual

Última verificación: **2026-10-30**. Backend `dev` y `origin/dev`: `130d2c9`.
Frontend `dev` y `origin/dev`: `eb73b00`. Ambos checkouts tienen cambios
documentales locales pendientes de commit; no se consideran cambios de aplicación.
Fuentes: `openapi.yaml`, `specs/*/tasks.md`, `specs/*/quickstart.md` y código
de cada repositorio. Publicar código en Git no demuestra un despliegue público.

## Implementado y verificado

| Área | Estado |
|---|---|
| Contrato | OpenAPI **1.6.0**, canónico en backend |
| Backend | Spec `001-becas-upb-management`: T001–T051 marcadas completas; evidencia de convergencia: 86 pruebas, lint, build y auditoría |
| Frontend | Spec `001-becario-work-management`: T001–T035 marcadas completas; 158 pruebas en 19 archivos, lint, build y auditoría aprobados |
| Auth y usuarios | Sesiones opacas, contraseñas temporales, roles, administración y metas individuales |
| Trabajos y postulaciones | Publicación, edición limitada, aceptación/rechazo y control de cupos |
| Cierre y horas | Asistencia exhaustiva, transacción única, idempotencia y progreso basado exclusivamente en asistencia |
| Evaluación | Cuatro criterios 1–5 tras cierre y asistencia confirmada; estado persistido visible solo al encargado propietario |
| Rating del becario | Promedio agregado; con menos de diez trabajos realizados muestra `faltan realizar trabajos` |
| Supervisión ADMIN | Progreso e historial paginados, estados pendiente/completada/excedida; sin evaluaciones |
| Cumplimiento API | `/students/compliance` devuelve una respuesta completa solo para ADMIN; la pantalla usa `/students/progress` e historial |

## Verificación real del frontend

El recorrido Chromium usó Vite temporal y PostgreSQL local descartable, con API
real y sin mocks. Se verificaron login de tres roles, evaluación desde la asistencia
del trabajo cerrado, persistencia después de recargar, historial del encargado,
rating e historial del becario y supervisión ADMIN sin evaluaciones. A 390 px no
hubo desbordamiento de página; la tabla es enfocable y el diálogo cierra con Escape.

La publicación, postulación, aceptación y cierre de la fixture se hicieron con la API
real. Carga, vacío, error, reintento y paginación se verificaron con Vitest; no se
forzaron errores del servidor real. El recorrido reproducible y sus límites están
en `web/scripts/verify-integration.mjs`, `verify-integration.py` y el quickstart web.

## Pendiente o sin verificar

- Taxonomía de contextos del plan v2: no existe `0004_contextos.sql`; siguen
  `MATERIA/FACULTAD/AREA`. No confundir este alcance futuro con tareas abiertas de las specs cerradas.
- CI remota, protección de ramas y despliegues públicos: no verificados en esta revisión.
- Cobertura de código total: no equivale a tener todas las pruebas aprobadas; revisar configuración y política antes del siguiente relevo.
- Cookies y CORS entre dominios de producción y previews: pendientes de validación real.
- Antes de desplegar, confirmar la rotación de la contraseña de Supabase expuesta
  históricamente y revisar la nomenclatura de los modos SSL. Ver [[Supabase]].

## Fuentes y vigencia

El plan v1 conserva el flujo original; el plan v2 aporta decisiones de alcance.
Las specs y el contrato concretan lo implementado. Para el estado actual, comprobar
siempre commit, código y evidencia; `AI_CONTEXT.md` puede conservar entradas históricas.

Relacionado: [[API v1]] · [[Modelo de datos]] · [[Pruebas]] · [[Fuentes en el repositorio]]
