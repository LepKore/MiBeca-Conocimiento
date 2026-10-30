---
tags: [mibeca, arquitectura, api]
---

# API v1

**Contrato canónico:** `gestion-de-trabajo-becario-backend/openapi.yaml` (OpenAPI 3.1,
revisión **1.6.0**, backend `dev` commit `130d2c9`).
`docs/api-v1.md` es un resumen legible **subordinado** al contrato, no una fuente de
verdad. Si hay discrepancia, manda `openapi.yaml`.

## Endpoints

Todos bajo el prefijo `/api/v1`, excepto los de salud.

### Salud

| Método | Ruta | Nota |
|---|---|---|
| GET | `/health` | Liveness: `{ status: "ok" }` |
| GET | `/health_db` | Consulta `SELECT 1`; 503 `database_unavailable` si falla. Documentado en el contrato |
| GET | `/ready` | Igual que `health_db` pero `{ status: "ready" }`. **No está en el contrato**; lo usa la CI y los workflows de Firebase |

### Auth

| Método | Ruta | Quién |
|---|---|---|
| POST | `/auth/login` | Público (con límite de intentos) |
| POST | `/auth/logout` | Autenticado |
| GET | `/auth/me` | Autenticado |
| POST | `/auth/change-password` | Autenticado |

### Usuarios (solo ADMIN)

| Método | Ruta |
|---|---|
| GET | `/users` |
| POST | `/users` |
| GET | `/users/{id}` |
| PATCH | `/users/{id}` |
| POST | `/users/{id}/deactivate` |
| POST | `/users/{id}/reactivate` |
| POST | `/users/{id}/reset-password` |

### Trabajos y postulaciones

| Método | Ruta | Quién |
|---|---|---|
| GET | `/jobs` | Todos (publicados y cerrados) |
| POST | `/jobs` | ENCARGADO |
| GET | `/jobs/{id}` | Todos |
| PATCH | `/jobs/{id}` | ENCARGADO dueño, antes del inicio |
| GET | `/jobs/{id}/applications` | ENCARGADO dueño (incluye progreso de beca del postulante desde 1.4.0) |
| POST | `/jobs/{id}/applications` | BECARIO (postularse) |
| POST | `/applications/{id}/accept` | ENCARGADO dueño |
| POST | `/applications/{id}/reject` | ENCARGADO dueño |
| POST | `/applications/{id}/evaluation` | ENCARGADO dueño, tras cierre con asistencia confirmada |
| GET | `/me/rating` | BECARIO autenticado; promedio agregado |
| GET | `/me/jobs` | ENCARGADO |
| GET | `/me/applications` | BECARIO |

### Control, progreso y supervisión

| Método | Ruta | Quién |
|---|---|---|
| POST | `/jobs/{id}/close` | ENCARGADO dueño (requiere `Idempotency-Key`) |
| GET | `/jobs/{id}/attendance` | ENCARGADO dueño |
| GET | `/me/progress` | BECARIO |
| GET | `/me/dashboard` | ENCARGADO (métricas de sus trabajos) |
| GET | `/me/history` | BECARIO o ENCARGADO, con respuesta propia según rol |
| GET | `/students/progress` | ADMIN (búsqueda, filtro de pendientes, paginación) |
| GET | `/students/compliance` | ADMIN; cumplimiento de todos los becarios en una respuesta, sin evaluaciones |
| GET | `/students/{id}/history` | ADMIN |

## Reglas de consumo

- **Cabeceras obligatorias en escrituras**: `X-CSRF-Token`.
- **`Idempotency-Key`** solo al cerrar un trabajo (8–128 caracteres).
- El cliente HTTP es `src/lib/api.js`; las rutas viven **solo** en
  `src/api/endpoints.js`. Un `test` falla si aparece una ruta literal en otro archivo.
- `CORS` permite `Content-Type`, `X-CSRF-Token`, `Idempotency-Key` y métodos
  `GET, POST, PATCH, OPTIONS`.
- Listados paginados devuelven `meta`; si falta, el frontend debe **fallar
  explícitamente** en lugar de asumir una sola página.

## Errores

Forma única:

```json
{ "error": { "code": "validation_error", "message": "...", "detalles": { } } }
```

- `detalles` es opcional y existe para mapear errores a campos del formulario.
  **Pendiente**: ningún formulario del frontend lo usa todavía.
- Códigos conocidos: `validation_error`, `database_unavailable`, `internal_error`,
  `ultimo_admin_activo` (409).
- Nunca se devuelven trazas ni SQL.

El frontend `dev` en `eb73b00` consume evaluación y rating de OpenAPI 1.6.0; evaluación persistida en asistencia e historial del propietario. ADMIN y becario no reciben evaluaciones individuales.

Relacionado: [[Arquitectura general]] · [[Cierre, asistencia y horas]]
