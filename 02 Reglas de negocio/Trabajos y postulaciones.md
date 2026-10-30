---
tags: [mibeca, reglas, trabajos]
---

# Trabajos y postulaciones

Fuente: `docs/PLAN_MVP_MIBECA.md` §4, `migrations/0002_trabajos_postulaciones.sql`,
`docs/mvp-y-permisos.md`.

## Trabajo

Un **trabajo es una actividad** becaria con **un solo cierre**. No existe el concepto de
"completion" ni de tareas hijas.

### Campos obligatorios y límites

| Campo | Regla |
|---|---|
| `titulo` | 3 a 120 caracteres, sin espacios sobrantes |
| `descripcion` | 3 a 4000 caracteres |
| `ubicacion` | 2 a 200 caracteres |
| `contexto_tipo` | `MATERIA`, `FACULTAD` o `AREA` |
| `contexto_nombre` | 2 a 120 caracteres |
| `inicio_previsto` / `fin_previsto` | `timestamptz` en **UTC**; `fin > inicio` |
| `duracion_minutos` | entero > 0 y **múltiplo de 6** |
| `cupos` | entero > 0 |

### Estados

Solo dos: `PUBLICADO` y `CERRADO`. No hay borrador publicado ni cancelación.

### Edición

- Se puede editar **antes del inicio previsto**.
- Después del inicio, **las condiciones quedan fijas** (fecha, duración, cupos). Un
  intento de cambio tardío se rechaza.
- El trabajo **no se cancela** desde la aplicación.

## Postulación

### Reglas

- **Una postulación por pareja estudiante/trabajo** (`UNIQUE (trabajo_id, estudiante_id)`).
  Reintentar no crea registros extra.
- Estados: `PENDIENTE`, `ACEPTADA`, `RECHAZADA`.
- **Las pendientes NO reservan cupo.** Solo una `ACEPTADA` reserva.
- **Aceptaciones concurrentes del último cupo: solo una gana.** Debe estar garantizado
  a nivel de base de datos/transacción, no solo en la interfaz.
- **No hay retiro ni cancelación** de postulaciones en este MVP.

### Rechazo automático al cerrar

Cuando el encargado cierra el trabajo, las postulaciones que quedaron `PENDIENTE` pasan
a `RECHAZADA` con `motivo_rechazo = 'CIERRE_TRABAJO'`. Es el **único** valor permitido, y
solo es válido si el estado es `RECHAZADA` (restricción `CHECK` en base de datos).

## Error típico a evitar

> [!warning] No confundir reserva con postulación
> Mostrar "quedan 2 cupos" contando pendientes es un bug de negocio. El cupo consumido
> es exactamente el número de postulaciones `ACEPTADA`. La UI puede mostrar
> `aceptados de cupos totales` (por ejemplo `0 de 1`) sin descontar pendientes.

Relacionado: [[Cierre, asistencia y horas]] · [[Roles y permisos]] · [[Modelo de datos]]
