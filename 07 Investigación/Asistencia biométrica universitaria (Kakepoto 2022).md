---
tags: [mibeca, investigacion, asistencia]
---

# Asistencia biométrica universitaria (Kakepoto 2022)

**Fuente:** `docs/2026-10-29 Kakepoto 2022 - Biometric attendance (GSSR).pdf` (archivo guardado en
el vault) · DOI: <https://doi.org/10.31703/gssr.2022(VII-II).31> · **Consultado:** 2026-10-29.

Kakepoto, I., Bhutto, N. A., & Umrani, A. I. (2022). *Biometric Attendance Management System: A
University Perspective*. Global Social Sciences Review, VII(II), 309–318.

**Por qué nos sirve:** estudia qué pasa en una universidad cuando se empieza a **verificar
asistencia de forma automática**. MiBeca hace algo parecido con horas becarias, así que sus
ventajas y, sobre todo, sus efectos secundarios son directamente relevantes.

## Qué es y cómo se hizo

Estudio **cualitativo exploratorio** en una universidad pública de Pakistán, recién instalado el
sistema biométrico. Observación directa durante **unos dos meses**, más conversaciones con
colegas para validar. No hay encuestas, ni cifras, ni grupo de control: son observaciones
interpretadas.

## Hallazgos que reporta

**Ventajas**

| Ventaja | Qué significa |
|---|---|
| Seguimiento de entrada/salida en tiempo real | El registro es automático y con hora exacta |
| Mantenimiento del registro sin esfuerzo | Un administrativo gestiona todo; desaparecen los cálculos a mano |
| Mejora del desempeño | La gente llega a tiempo y permanece en el puesto |
| Control del ausentismo | Ya no se puede firmar por otro ni regalar permisos |

**Desventajas**

| Desventaja | Qué significa |
|---|---|
| **Estrés laboral** | Miedo al descuento de salario, sensación de vigilancia, desconfianza. Reporta que empleados perciben favoritismo y castigo selectivo |
| **Dependencia de la electricidad** | Con cortes de 10–18 h/día el sistema no registra; la universidad termina manteniendo **biométrico y registro manual en paralelo** |
| **Miedo al descuento** | El registro se usa como prueba en contra del empleado, sin posibilidad de réplica |
| **Retraso a clases** | El docente debe marcar en su departamento antes de ir a otra facultad: el punto de registro está lejos del lugar donde ocurre el trabajo |

La conclusión de los autores va más allá del hardware: para el personal académico, un sistema que
mide **presencia** es el instrumento equivocado; debería medirse por **resultado** (cursos
dictados, trabajos publicados, tesis dirigidas).

## Qué nos sirve

1. **Verificar la asistencia tiene un costo social.** El paper lo documenta como desventaja
   principal, no como efecto secundario menor. MiBeca ya eligió una forma menos invasiva: no hay
   reloj ni biométrico, hay **un cierre por trabajo** que el encargado hace después, con los
   minutos de cada asistente. Eso es una decisión de producto defendible y conviene no
   "mejorarla" con timbres o marcas de entrada.
2. **El punto de registro debe estar donde ocurre el trabajo.** La desventaja de "retraso a
   clases" es exactamente lo que evitamos: el encargado cierra desde la misma actividad, no desde
   una oficina distinta.
3. **Nunca un único camino para registrar.** Si el dispositivo o el sistema cae, esa jornada no
   se registra y la persona pierde horas. En MiBeca el equivalente es que la API o PostgreSQL
   estén caídos al momento del cierre. No hay procedimiento de contingencia documentado: es una
   decisión pendiente real, no un detalle.
4. **Un registro que solo sirve para castigar se rechaza.** Si las horas acreditadas se usan
   únicamente para sancionar al becario, aparece el mismo estrés. MiBeca muestra progreso y saldo
   al propio estudiante, y la cuota se negocia fuera de la plataforma. Mantener esa transparencia
   es parte del diseño, no un adorno del panel.
5. **Medir presencia no es medir aporte.** El paper lo dice para el profesorado y no aplica
   entero a la beca: la UPB exige horas, así que aquí la presencia **es** el requisito. Pero
   conviene tenerlo presente al diseñar reportes: horas cumplidas no es lo mismo que trabajo bien
   hecho.

## Qué NO aplica

- **Biometría, hardware, tornos, lectores**: fuera de alcance y sin relación con el producto.
- **Nómina y descuentos de salario**: MiBeca no maneja dinero.
- **Cortes de electricidad**: contexto de Pakistán; aquí el riesgo es de red y de base de datos,
  no de suministro eléctrico.

## Límites de la evidencia

Es un estudio **cualitativo por conveniencia**, de **una sola institución**, basado en observación
y opinión, sin mediciones. Sirve para **generar criterio sobre cómo se siente la gente vigilada**,
no como evidencia técnica ni como cifra de referencia. Varias de sus desventajas son del contexto
paquistaní (crisis energética, injerencia política) y no se trasladan a la UPB.

Relacionado: [[Cierre, asistencia y horas]] · [[Clockify]] · [[Visión y alcance]] · [[00 Índice]] ·
[[enlaces web]]
