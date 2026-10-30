---
tags: [mibeca, reglas, horas]
---

# Cierre, asistencia y horas

Es la parte más delicada del sistema: de aquí salen las horas que cuentan para la beca.
Fuente: `docs/PLAN_MVP_MIBECA.md` §4, `migrations/0003_cierre_asistencia.sql`,
`docs/VERIFICACION_FASE_4.md`.

## El cierre

Un **cierre único por trabajo**, ejecutado por el encargado dueño, en **una
transacción**:

1. Se registra la asistencia de **todos** los aceptados.
2. Se acredita la **duración completa** del trabajo a quienes asistieron.
3. Se acredita **cero** a quienes faltaron.
4. Se marca el trabajo como `CERRADO`.
5. Se guardan auditoría y la respuesta para idempotencia.

### Condiciones de rechazo (sin escrituras parciales)

- Cierre **antes** del fin previsto.
- Lista de asistencia **incompleta** (falta algún aceptado).
- Asistencia registrada para alguien **no aceptado**.

Todo esto se rechaza **antes** de escribir nada. Nunca queda un cierre a medias.

## Cómo se calculan los minutos

| Situación | Minutos acreditados |
|---|---|
| Asistió | `duracion_minutos` del trabajo (entero > 0, múltiplo de 6) |
| No asistió | **0** exacto |
| No hay asistencia registrada | No existe fila; no acredita |

Restricción en base de datos, no solo en código:

```sql
CHECK ((asistio AND minutos_acreditados > 0 AND minutos_acreditados % 6 = 0)
    OR (NOT asistio AND minutos_acreditados = 0))
```

**Inmutabilidad:** tras el cierre no se edita la asistencia ni los minutos. No hay
endpoint de corrección; la regla aplica incluso al administrador.

## Idempotencia del cierre

- El cliente envía la cabecera **`Idempotency-Key`** (8 a 128 caracteres).
- Se guarda en `cierres`: `idempotency_key`, `request_hash` (SHA-256 hex de 64
  caracteres) y `respuesta` `jsonb`. Restricción `UNIQUE (encargado_id, idempotency_key)`.
- **Mismo contenido → misma clave.** Reintentar devuelve la respuesta almacenada, sin
  duplicar la acreditación.
- **Si cambió la asistencia → clave nueva.** Reutilizar la clave con otro contenido debe
  fallar, no "actualizar".

## Progreso e historial

- **Progreso** = suma de `asistencias.minutos_acreditados` de todas las postulaciones del
  estudiante. Se consulta con `GET /me/progress`; el administrador usa
  `GET /students/progress`.
- **Excedente**: si el estudiante supera su `meta_minutos`, los *faltantes* se muestran
  como **cero** y el exceso se informa **aparte**. No hay errores de precisión porque
  todo es entero.
- **Historial del becario**: incluye las **ausencias** (cero horas). Un ausente aparece,
  no desaparece.
- **Historial del encargado**: muestra los asistentes de **sus** trabajos cerrados.

## Zona horaria

- Las fechas se muestran y se ingresan en `VITE_TIME_ZONE` (por defecto
  **`America/La_Paz`**); la API recibe **UTC**.
- El formulario convierte local→UTC al enviar y UTC→local al editar.
- Se validan fechas imposibles y horas inexistentes o ambiguas en cambios de horario.
- Los **minutos acreditados provienen exclusivamente del backend**: el frontend nunca
  los calcula ni los envía.

Relacionado: [[Trabajos y postulaciones]] · [[Modelo de datos]] · [[API v1]] · [[Asistencia biométrica universitaria (Kakepoto 2022)]]
