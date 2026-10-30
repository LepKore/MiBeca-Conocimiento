---
tags: [mibeca, proyecto, equipo]
---

# Equipo

Fuente: `docs/plan-colaboracion.md`, `docs/PLAN_DE_TRABAJO_MIBECA.md` §"Orden de entrega",
y confirmación directa del equipo (2026-10-29).

## Quiénes somos

**Tres personas**, las tres **desarrolladoras junior** y todavía **estudiando en la
universidad**:

| Persona | Nota |
|---|---|
| Ivan Zabala | Ramas/experimentos: `Ivan` (trabajo de conexión con Supabase) |
| Erick Magueño | Reparto inicial sugerido: Auth/Usuarios backend |
| Sergio Ramos | Reparto inicial sugerido: frontend contra contrato; rama `sergio` |

Los roles **rotan por bloque de tareas**: uno implementa, otro revisa al integrar, el
tercero valida. Ver [[Reparto de tareas]].

> El reparto "sugerido" de arriba viene del plan y no registra lo que cada uno hizo
> realmente. El reparto que manda es el de cada spec.

## Restricción número uno: mantenibilidad

> [!important] Esto gobierna todas las decisiones técnicas
> El proyecto lo mantienen **tres juniors que aún estudian**. Debe seguir siendo
> **fácilmente mantenible** y **no volverse demasiado complejo**.

Consecuencias prácticas, que ya están alineadas con la constitución de ambos repos
(simplicidad, YAGNI, entrega incremental):

- **Nada de abstracciones nuevas** que no resuelvan un problema presente y concreto.
- **Nada de dependencias nuevas** si el lenguaje, el navegador o lo ya instalado lo
  resuelve.
- **Nada de migrar a TypeScript/ORM** ni de patrones que solo entiende quien los escribió.
  Lo prohíben el principio 5 de las constituciones (simplicidad, YAGNI) y el §8 del plan v2
  (en desuso, pero la decisión sigue viva).
- **El código más corto que funcione**, en la estructura que ya existe: `routes` →
  `services` → `lib`, páginas → `endpoints.js` → `api.js`.
- Si una tarea exige explicar el mecanismo dos veces, el diseño está mal.
- Preferir **repetir tres líneas** antes que inventar un helper compartido que hay que
  entender en otro archivo.

La regla práctica: si dentro de seis meses no lo entiendes en una lectura, no entra.

## Ramas locales (verificado 2026-10-30)

Los dos clones actuales tienen ramas locales `main` y `dev`, y están en **`dev`**.
Las ramas personales descritas en documentación histórica no se presuponen presentes.
Backend: `130d2c9`; frontend: `eb73b00`. Comprobar `git status` y referencias antes de cada relevo.

Patrón de rama por tarea: `feat/...` y `fix/...`.

Ver [[Ramas y PR]].

## Coordinación

- Un **coordinador** (rol que rota) asigna tareas, **congela decisiones de contrato**, evita
  que dos personas editen los mismos archivos e integra el PR.
- `openapi.yaml` y las migraciones tienen **un único responsable por cambio**.
- Cada integrante mantiene **su propio PostgreSQL local y su volumen**. Git comparte
  Compose, configuración de ejemplo, migraciones y seeds explícitos; **no** datos ni
  secretos.
- Al integrar, los tres aplican las migraciones nuevas en su base local.
- Los cambios de contrato se revisan en **ambos** repos.
- Las decisiones duraderas se escriben en `docs/`, en el PR/issue o en `AGENTS.md`.
  **Nunca** se presupone memoria compartida entre sesiones.

## Subagentes de IA

No equivalen a los tres integrantes y no hay obligación de tenerlos activos.

| Subagente | Alcance |
|---|---|
| `frontend` | Solo repo web: componentes, estilos, adaptador API |
| `backend-data` | Solo repo backend: API, Docker, migraciones |
| `reviewer` | Inspección en lectura, reporta defectos reproducibles |

Sus archivos TOML son **instrucciones de alcance, no permisos de sandbox**.

Relacionado: [[Reparto de tareas]] · [[Visión y alcance]]
