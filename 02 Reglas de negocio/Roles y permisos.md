---
tags: [mibeca, reglas, permisos]
---

# Roles y permisos

Fuente: `docs/mvp-y-permisos.md` §Permisos y `docs/PLAN_MVP_MIBECA.md` §5.
Los roles del código son los que valida el backend; ocultar un botón en el frontend
**no concede ni quita seguridad**.

## Matriz

| Acción | Becario | Encargado | Administrador |
|---|---|---|---|
| Login, logout y cambio de contraseña propia | Sí | Sí | Sí |
| Consultar trabajos publicados o cerrados | Sí | Sí | Sí |
| Postularse y consultar postulaciones | Solo propias | Postulantes de trabajos propios | Supervisión |
| Crear, editar y cerrar trabajos | No | Solo propios, según estado | No |
| Aceptar o rechazar postulaciones | No | Solo de trabajos propios | No |
| Consultar asistencia y horas | Propias | De sus trabajos | Progreso global básico |
| Crear y gestionar cuentas y metas | No | No | Sí |
| Crear y ver evaluación individual | No | Solo de asistentes de sus trabajos cerrados | No |
| Consultar rating agregado propio | Sí | No | No |
| Alterar horas ya cerradas | No | No | **No** |

## Reglas duras tras la matriz

- **No hay registro público.** El administrador crea cada cuenta.
- **El rol es fijo tras el alta.** No se cambia después.
- **El administrador no crea ni edita trabajos.** Esa es tarea del encargado.
- **Nadie corrige horas cerradas**, ni el administrador. No existe endpoint de ajuste.
- El listado del propietario incluye nombre, código, carrera, semestre, estado y progreso de beca; no expone credenciales. Asistencia e historial del encargado incluyen `evaluation` desde OpenAPI 1.6.0. ADMIN y becario no reciben evaluaciones individuales. Ver [[Evaluaciones y calificación]].
- La API comprueba **rol + cuenta activa + propiedad** en cada operación.

## Rol y contexto no son lo mismo

`ADMIN`, `ENCARGADO` y `BECARIO` son los únicos roles. "Docente" no es un rol: un
docente usa `ENCARGADO` y se distingue por su **contexto** (`MATERIA`, `FACULTAD`,
`AREA`). No duplicar roles por esa diferencia.

## Implementación

- Middleware de sesión y de rol en `gestion-de-trabajo-becario-backend/src/middleware/auth.js`.
- Rutas con `requireCsrf` en toda escritura (`AUTH-01` del auditor lo verifica).
- Frontend: `src/components/ProtectedRoute.jsx` restringe por rol, pero solo como
  comodidad de navegación.

Relacionado: [[Usuarios y sesiones]] · [[Trabajos y postulaciones]]
