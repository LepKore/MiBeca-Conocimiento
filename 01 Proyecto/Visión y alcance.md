---
tags: [mibeca, proyecto, alcance]
---

# Visión y alcance

Fuente: `gestion-de-trabajo-becario-backend/docs/PLAN_MVP_MIBECA.md` (plan canónico) y
`docs/mvp-y-permisos.md` (resumen acordado). Ver [[Fuentes en el repositorio]].

## Problema que resuelve

Una universidad asigna becas de trabajo: el estudiante cumple horas en actividades de
una materia, facultad o área, y debe acumular una meta individual de minutos. Sin
sistema, el control de asistencia y horas vive en hojas de cálculo y confianza.

MiBeca digitaliza ese ciclo completo con tres roles y permisos reales verificados en
el servidor.

## Flujo mínimo del producto

1. Un **administrador inicial** se crea con un comando explícito (no hay registro público).
2. El administrador crea cuentas de estudiantes y encargados, y fija la **meta individual de minutos**.
3. El **encargado** publica un trabajo: inicio, fin, duración acreditable y cupos.
4. El **estudiante** se postula. El encargado acepta o rechaza **antes del inicio**.
5. Solo una aceptación reserva cupo.
6. Tras el fin previsto, el encargado **cierra el trabajo una sola vez** y registra la asistencia de **todos** los aceptados.
7. En una transacción: se acredita la duración completa a quien asistió, cero a quien faltó, y el trabajo queda cerrado.
8. El estudiante consulta su progreso e historial.

## Incluido en el MVP

- Auth: login, logout, cambio de contraseña y revocación de sesiones.
- Usuarios: alta, consulta, edición acotada, activación y desactivación por administrador.
- Trabajos: publicación, edición previa al inicio, consulta, postulación, aceptación y rechazo.
- Control: cierre con asistencia, acreditación única y progreso.
- Consulta administrativa básica de estudiantes y horas, sin módulo avanzado de reportes.
- Historial persistente, permisos reales en backend, Docker, CI, hooks y uso verificable de subagentes.

## Fuera del MVP (no crear tablas, endpoints ni indicadores para esto)

Reputación pública, penalizaciones por retiro o inasistencia, registro público,
notificaciones, correo, recuperación automática de contraseñas,
archivos, exportaciones, reportes avanzados, calendario avanzado, múltiples
instituciones, múltiples roles por cuenta, reasignación de encargados,
retiro/cancelación de postulaciones, cancelación de trabajos y edición posterior de
acreditaciones.

La spec `001-becas-upb-management` añadió evaluaciones de cuatro criterios y rating
calculado al backend. La interfaz de evaluación y rating ya está integrada y verificada;
reputación pública, retiro y penalizaciones siguen fuera del alcance actual.

> [!warning] Trampa frecuente
> El plan v1 describe el flujo original, pero no todo el alcance actual. Para
> evaluaciones, consultar la spec `001-becas-upb-management`, `openapi.yaml` 1.6.0 y
> [[Estado actual]]. El v2 conserva decisiones de alcance que aún no se migraron,
> especialmente contextos; no equivale por sí solo a funcionalidad terminada.

## La beca y la cuota de horas

Confirmado por el equipo el 2026-10-29.

Un **becario de la UPB** debe cumplir una **cuota de horas** para no tener problemas con
su beca. En el sistema esa cuota se materializa como **meta individual en minutos**
(`estudiantes.meta_minutos`), **fijada al inscribir al becario**, en el momento del alta de
la cuenta.

Quién define el valor: el administrador, al crear la cuenta y, si corresponde, al
editar el perfil del estudiante mediante `PATCH /users/{id}`. No es un valor institucional
único que el sistema calcule.

El **reglamento de becas** fija **50 h/semestre** de servicio becario en varias modalidades
(comunitaria, colegio) y remite al contrato en otras (equidad, traspaso). MiBeca guarda ese
valor como meta individual y **no valida** que coincida con la beca del estudiante, porque no
sabe qué beca tiene. Ver [[Reglamento de becas UPB]].

También existe la **beca convenio**: la otorga una institución externa, con un porcentaje de
hasta 100%, y exige las mismas reglas. Cambia quién paga, no qué se exige.

### Cuándo puede cambiar

La cuota puede cambiar en **casos excepcionales**, cuando el becario incumple una regla de
su compromiso: reprobar una materia, no alcanzar el promedio requerido, no cumplir las
horas becarias correspondientes.

> [!important] Ese cambio se acuerda **fuera** de la plataforma
> El monto nuevo se negocia y decide fuera del sistema. La aplicación solo registra el
> resultado.

### Edición de la meta

El backend ya permite al administrador editar `metaMinutos` con `PATCH /users/{id}`.
No hace falta SQL directo para esa operación. Si el equipo quiere prohibir cambios de
meta en una etapa futura, debe decidirlo y modificar contrato, API y frontend de forma
coordinada; una nota del vault no puede anular la ruta existente.

## Criterio de terminado

El MVP termina cuando el flujo completo funciona contra PostgreSQL, sobrevive a
recrear el contenedor sin borrar el volumen y supera las pruebas de permisos, cupos
concurrentes, cierre atómico e idempotencia. **Mocks, documentación, perfiles de
agente y archivos de workflow no acreditan ese resultado.**

Relacionado: [[Roles y permisos]] · [[Estado actual]]
