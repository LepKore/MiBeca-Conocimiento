---
tags: [mibeca, reglas, datos]
---

# Modelo de datos

Fuente: `gestion-de-trabajo-becario-backend/migrations/0001` a `0003` y `0005_evaluaciones.sql` en `dev` (`130d2c9`). No existe `0004_contextos.sql`.
Convenciones globales: **UUID**, **UTC** (`timestamptz`), **duraciones en minutos
enteros**. No se usa Prisma ni ORM; el historial de migraciones vive en la tabla
`schema_migrations`.

## Tablas

### `usuarios` (0001)

`id` uuid PK · `nombre` (3–120) · `correo` · `password_hash` · `rol` (`ADMIN|ENCARGADO|BECARIO`) ·
`activo` bool · `requiere_cambio_password` bool · `creado_en` · `actualizado_en`

- `correo` se guarda **normalizado** (`= lower(btrim(correo))`) y con formato válido.
- Índice único sobre `lower(correo)` → unicidad insensible a mayúsculas.
- Índice parcial por rol donde `activo`.

### `estudiantes` (0001)

`usuario_id` PK → `usuarios.id` (`ON DELETE RESTRICT`) · `codigo` (`^[0-9]{5}$`) ·
`carrera` (2–120) · `semestre` (1–12) · `meta_minutos` (> 0 y **% 6 = 0**)

- `codigo` es **texto** con índice único: conserva ceros iniciales.
- `ON DELETE RESTRICT` en todas las FK a `usuarios`: el historial no se puede borrar.

### `encargados` (0001)

`usuario_id` PK · `contexto_tipo` (`MATERIA|FACULTAD|AREA`) · `contexto_nombre` (2–120)

### `sesiones` (0001)

`id` · `usuario_id` · `token_hash` (único) · `csrf_token` · `user_agent` · `creado_en` ·
`expira_en` · `revocado_en`

### `auditoria` (0001)

`id` bigserial · `actor_id` (FK, `ON DELETE SET NULL`) · `accion` · `recurso` ·
`detalle` jsonb · `creado_en`

### `trabajos` (0002)

`id` · `encargado_id` → `encargados` · `titulo` (3–120) · `descripcion` (3–4000) ·
`ubicacion` (2–200) · `contexto_tipo` · `contexto_nombre` · `inicio_previsto` ·
`fin_previsto` (**> inicio**) · `duracion_minutos` (> 0, **% 6 = 0**) · `cupos` (> 0) ·
`estado` (`PUBLICADO|CERRADO`) · timestamps

Índices: `(encargado_id, inicio_previsto, id)` y `(estado, inicio_previsto, id)` — pensados
para el listado paginado estable.

### `postulaciones` (0002 + 0003)

`id` · `trabajo_id` · `estudiante_id` · `estado` (`PENDIENTE|ACEPTADA|RECHAZADA`) ·
`motivo_rechazo` · timestamps

- `UNIQUE (trabajo_id, estudiante_id)` → una postulación por pareja.
- `motivo_rechazo` solo puede ser `'CIERRE_TRABAJO'` y solo con estado `RECHAZADA`.

### `cierres` (0003)

`trabajo_id` **PK** (un cierre por trabajo) · `encargado_id` · `idempotency_key` (8–128) ·
`request_hash` (`^[0-9a-f]{64}$`) · `respuesta` jsonb · `cerrado_en`

- `UNIQUE (encargado_id, idempotency_key)` → idempotencia por encargado.

### `asistencias` (0003)

`postulacion_id` **PK** (una asistencia por postulación) · `trabajo_id` → `cierres` ·
`asistio` · `minutos_acreditados` · `registrado_por` · `registrado_en`

- El `CHECK` de minutos es la regla de negocio más importante del esquema: ver
  [[Cierre, asistencia y horas]].

### `evaluaciones` (0005)

`id` uuid PK · `postulacion_id` único → `postulaciones` · `evaluador_id` → `encargados` ·
`responsabilidad`, `puntualidad`, `proactividad`, `comunicacion` (enteros 1–5) ·
`promedio` generado · `creado_en`. La API solo permite crear una evaluación por
postulación aceptada con asistencia confirmada tras el cierre; no expone edición ni borrado.

## Diagrama de dependencias

```
usuarios ──1:1──> estudiantes ──┐
    └────1:1──> encargados ──┐  │
             │               │  │
             ├──> trabajos <─┘  │
             │        │         │
             │        └──> postulaciones <──┘
             │                  │
             └──> cierres <─────┤ (trabajo_id)
                        │       │
                        └──> asistencias (postulacion_id PK)
             postulaciones ──> evaluaciones (postulacion_id único)
```

## Reglas de esquema que no se deben romper

1. Las migraciones son **aditivas** y numeradas `NNNN_descripcion.sql` (regla `MIG-02`).
2. `DROP`/`TRUNCATE` exigen justificación en la línea o la anterior (regla `MIG-01`).
3. Nada de `ON DELETE CASCADE` sobre historial: se usa `RESTRICT`.
4. Sin `SELECT *` (regla `DB-05`, informativa).
5. SQL nunca interpola datos de petición (`DB-01`, bloqueante).

Ver [[Migraciones]] para el procedimiento.
