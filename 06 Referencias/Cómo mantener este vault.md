---
tags: [mibeca, proceso, conocimiento]
---

# Cómo mantener este vault

**Regla del equipo, confirmada el 2026-10-29:** lo que los agentes investigan y descubren
útil **se queda aquí**, no se pierde en la conversación.

## La regla

> Cuando un agente investiga en internet (o deduce algo no obvio del entorno) y encuentra
> información **reutilizable** para este proyecto, **debe añadirla al vault** antes de
> cerrar la tarea. Si tuvo que buscarlo una vez, lo va a tener que buscar de nuevo.

### Qué cuenta como reutilizable

- Cómo funciona una herramienta que usamos (Spec Kit, Vitest, Redocly, Docker, Supabase).
- Un procedimiento que no está en la documentación del repo.
- Una restricción del entorno que costó descubrir (Windows, PowerShell, puertos, Python).
- Una metodología o proceso adoptado, con su fuente.
- Un error conocido y su solución.

### Qué NO cuenta

- Lo que **ya está** en un documento del repo: se enlaza, no se copia. Ver
  [[Fuentes en el repositorio]].
- Lo que **contradice** al código: el código gana. Se corrige la nota, no el código (salvo
  que el bug sea real).
- Notas de una conversación, decisiones personales, o "creo que…".
- Volcados enteros de documentación. Se resume **qué nos sirve y por qué**.

## Guardar la fuente (obligatorio)

Conocimiento sin fuente no se puede verificar ni actualizar. Por eso:

| Tipo de fuente | Dónde se guarda |
|---|---|
| **Enlace web** | Una fila en `docs/enlaces web.md` — ver [[enlaces web]] |
| **PDF, DOCX, XLSX, PPTX, CSV**, o cualquier archivo recibido | El **archivo** en `MiBeca-Conocimiento/docs/`, y su fila en `docs/enlaces web.md` |
| Código o documento de los repos | No se copia: se enlaza la ruta. Ver [[Fuentes en el repositorio]] |

Reglas de esa carpeta:

- El nombre del archivo dice qué es y de cuándo: `2026-10-29 normativa becarios UPB.pdf`. Sin
  nombres como `documento1.pdf`.
- No se guardan copias de lo que ya vive en un repo (planes, migraciones, `openapi.yaml`).
- Un archivo guardado **no** se cita de memoria: se lee y se resume en su nota, con la
  referencia al archivo.
- Si una fuente se cae o cambia, la fila se marca; no se borra. Ver [[enlaces web]].

> [!warning] Por qué esto es regla y no cortesía
> Un agente que investigó algo y no dejó la fuente obliga al siguiente a repetir la búsqueda,
> o peor: a confiar en una nota sin saber de dónde salió. Ver [[00 Índice]].

## Dónde va

| Tipo | Destino |
|---|---|
| Investigación nueva (herramientas, procesos, metodologías) | `07 Investigación/` — ver [[00 Índice]] de esa carpeta |
| Regla de negocio | `02 Reglas de negocio/` |
| Cómo se trabaja en el equipo | `04 Cómo se trabaja/` |
| Paso a paso operativo | `05 Procedimientos/` |
| Fuente y autoridad documental | `06 Referencias/` |

Una nota por tema. Si el tema ya tiene nota, **se amplía**; no se crea `Tema 2.md`.

## Formato mínimo de una entrada de investigación

```markdown
## <Título corto>

**Fuente:** <URL o `docs/archivo.pdf`> · **Consultado:** AAAA-MM-DD
**Por qué nos sirve:** una o dos frases concretas.

<El contenido útil, resumido. Comandos, no párrafos.>

**Límite:** qué NO cubre o cuándo deja de ser cierto.
```

La fecha y la fuente no son burocracia: sin ellas nadie sabe si el dato sigue vigente, y
una nota vieja sin fuente es peor que ninguna.

## Higiene

- **Enlaces, no duplicados.** Si el dato vive en `openapi.yaml`, se enlaza el archivo.
- **Marcar lo no verificado.** Si algo se leyó pero no se probó, decirlo. El vault ya usa
  las convenciones: bloque `> [!warning]` para trampas, `> [!danger]` para deudas que
  bloquean, y la palabra **pendiente** para lo que falta.
- **Tope de tamaño.** Si una nota pasa de ~200 líneas, dividir por tema y enlazar.
- **Actualizar `Estado actual`** cuando cambia algo que el vault afirma como implementado.
  Ver [[Estado actual]].

## Revisión antes de actualizar

1. Leer `git status`, `HEAD` y la referencia remota local de ambos repositorios.
2. Registrar rama y commit usados en la nota. Si el checkout está atrasado, verificar
   primero la revisión que se quiere documentar; ausencia local no demuestra ausencia remota.
3. Distinguir tareas marcadas completas, evidencia ejecutada y despliegue público.
4. Actualizar las notas enlazadas cuando una decisión cambie permisos o contrato.
5. Comprobar enlaces y `git diff --check`; guardar el cambio en Git mediante el flujo del equipo.

El estado de referencia del 2026-10-30 es backend `130d2c9`, frontend `eb73b00` y
OpenAPI 1.6.0. Las fechas históricas se conservan como evidencia de aquella revisión.

## Colores del grafo

El grafo se colorea por **tag de sección**, no a mano. La configuración vive en
`.obsidian/graph.json` (grupos de color) y se corresponde con el segundo tag de cada nota:

| Color | Query | Sección |
|---|---|---|
| Blanco | `tag:#indice` | Índices (`00 Inicio`, este índice de investigación) |
| Azul | `tag:#proyecto` | `01 Proyecto` |
| Verde | `tag:#reglas` | `02 Reglas de negocio` |
| Morado | `tag:#arquitectura` | `03 Arquitectura` |
| Naranja | `tag:#proceso` | `04 Cómo se trabaja` |
| Turquesa | `tag:#procedimiento` | `05 Procedimientos` |
| Rosa | `tag:#investigacion` | `07 Investigación` |
| Gris | `tag:#referencia` | `06 Referencias` |

Por eso **toda nota nueva lleva `mibeca` y el tag de su sección** en el frontmatter. Una nota sin
tag de sección aparece del color por defecto y rompe la lectura del grafo.

Los grupos se evalúan en orden: gana el primero que coincide.

## El vault no es código

No se versiona con los repos de la aplicación (vive en la raíz del workspace, junto a
ellos). Es documentación de trabajo: se mantiene, no se despliega.

Relacionado: [[00 Índice]] · [[Fuentes en el repositorio]]
