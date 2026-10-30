---
tags: [mibeca, glosario]
---

# Glosario

| Término | Significado preciso en MiBeca |
|---|---|
| **Becario / Estudiante** | Rol `BECARIO`. Etiqueta visible: "Estudiante becario". Tiene perfil en `estudiantes` |
| **Encargado** | Rol `ENCARGADO`. Docente o responsable de una actividad. Etiqueta visible: "Encargado". Perfil en `encargados` |
| **Administrador** | Rol `ADMIN`. Único que crea cuentas y fija metas |
| **Contexto** | Ámbito de un trabajo o encargado: `MATERIA`, `FACULTAD` o `AREA`. Docente y encargado son el mismo rol; se distinguen por contexto, **no** por roles duplicados |
| **Trabajo** | Una actividad becaria publicada. Tiene inicio, fin, duración acreditable, cupos y estado `PUBLICADO`/`CERRADO`. **Un trabajo = una actividad = un cierre** |
| **Postulación** | Vínculo estudiante↔trabajo. Estado `PENDIENTE`, `ACEPTADA` o `RECHAZADA`. Única por pareja |
| **Cupo** | Plaza de un trabajo. Solo lo **reserva una aceptación**; las postulaciones pendientes no reservan nada |
| **Cierre** | Acto único e irreversible del encargado al terminar el trabajo. Registra la asistencia de **todos** los aceptados en una transacción |
| **Asistencia** | Registro por postulación aceptada: `asistio` + `minutos_acreditados`. Inmutable tras el cierre |
| **Minutos acreditados** | Minutos que suman al progreso. Asistente: la duración completa del trabajo. Ausente: cero |
| **Meta individual** | `meta_minutos` del estudiante. Entero positivo **múltiplo de 6**. Se fija al inscribir al becario y equivale a su **cuota de horas semestrales**. Cambiarla exige SQL; editar desde la app está fuera del alcance por ahora. Ver [[Visión y alcance]] |
| **Excedente** | Lo que el estudiante acumuló por encima de su meta. Se informa **por separado**; nunca se resta a la meta ni se corrige |
| **Progreso** | Suma de `asistencias.minutos_acreditados` de todas las postulaciones del estudiante |
| **Código académico** | Exactamente **5 dígitos**, almacenado como **texto** para conservar ceros iniciales (`00123`) |
| **Desactivación** | Baja **lógica** de una cuenta: revoca sesiones, conserva relaciones e historial. No es borrado |
| **Cierre único / idempotencia** | Reintentar el mismo cierre con la misma `Idempotency-Key` devuelve la respuesta guardada en vez de duplicar la acreditación |
| **`motivo_rechazo`** | Único valor posible: `CIERRE_TRABAJO`. Marca postulaciones pendientes rechazadas automáticamente al cerrar |
| **Servicio becario** | Las horas que el becario debe cumplir para conservar la beca. El reglamento fija **50 h/semestre** en varias modalidades; en MiBeca se materializa como **meta individual**. Ver [[Reglamento de becas UPB]] |
| **Beca convenio** | Beca otorgada por una **institución externa** (no la UPB): el becario aprueba su examen y la institución aporta un porcentaje, hasta 100%. Las reglas de mantenimiento son las mismas: no reprobar, promedio mínimo y horas becarias. **No existe en el modelo de datos** |

## Por qué los minutos son múltiplos de 6

Las horas se ingresan con **como máximo un decimal** (0,1 h). 0,1 h = **6 minutos**, así
que todo múltiplo de 6 es representable sin pérdida de precisión. De ahí los `CHECK`
`minutos % 6 = 0` en `estudiantes.meta_minutos`, `trabajos.duracion_minutos` y
`asistencias.minutos_acreditados`. Ver [[Modelo de datos]].
