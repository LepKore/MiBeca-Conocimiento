---
tags: [mibeca, reglas, evaluacion]
---

# Evaluaciones y calificación

Verificado el 2026-10-30 contra backend `130d2c9`, frontend `eb73b00` y
`openapi.yaml` 1.6.0. Fuentes: ambas specs `001-*`, servicios de trabajos/control,
`src/api/endpoints.js` y quickstarts de verificación.

## Evaluación individual

El encargado propietario evalúa una postulación ACEPTADA de su trabajo CERRADO
solo si tiene asistencia confirmada. Envía cuatro enteros de 1 a 5:
Responsabilidad, Puntualidad, Proactividad y Comunicación.

`POST /applications/{id}/evaluation` requiere sesión activa y CSRF; devuelve 201.
La base calcula el promedio y limita a una evaluación por postulación. El duplicado
devuelve 409. No existen operaciones de edición ni borrado en la API.

La evaluación no crea ni modifica minutos. Las horas siguen procediendo solo del
cierre con asistencia. Ver [[Cierre, asistencia y horas]].

## Lectura y privacidad

Desde 1.6.0, la asistencia y el historial del encargado propietario devuelven
`evaluation: Evaluacion | null`. La interfaz ofrece Evaluar a asistentes sin evaluación
y muestra Evaluado y su promedio al guardarla, incluso después de recargar.
El formulario está en la asistencia del trabajo cerrado y el historial del encargado.

ADMIN y becario no reciben criterios ni evaluaciones individuales en sus historiales.
El becario consulta únicamente su rating agregado mediante `GET /me/rating`.

## Rating agregado

Trabajo realizado significa cerrado con asistencia confirmada. Se eligen los diez
más recientes según cierre descendente y UUID de postulación como desempate;
se promedian solo las evaluaciones disponibles, sin convertir faltantes en cero.
Con menos de diez trabajos realizados se muestra `faltan realizar trabajos`.
Con suficientes trabajos sin evaluaciones disponibles, rating es null y no debe
inventarse una puntuación. El contrato define `trabajosRealizados`,
`evaluacionesConsideradas`, `rating` y `message`.

## Supervisión administrativa

La pantalla ADMIN usa progreso e historial paginados para mostrar metas, horas,
faltantes, excedentes y cumplimiento. No muestra calificaciones. El endpoint adicional
`GET /students/compliance` entrega el cumplimiento completo solo para ADMIN.

Relacionado: [[Roles y permisos]] · [[Modelo de datos]] · [[API v1]] · [[Estado actual]]
