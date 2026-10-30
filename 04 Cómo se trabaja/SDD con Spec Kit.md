---
tags: [mibeca, proceso, sdd]
---

# SDD con Spec Kit

**Spec-Driven Development** (desarrollo guiado por especificación) es el marco de trabajo
declarado del equipo. Se implementa con **Spec Kit**, el toolkit de `github/spec-kit`.

- Versión instalada: **v1.2.4.dev0**, el **2026-10-26**.
- **Reemplaza** a la skill global `spec-driven-development`, que fue eliminada.
- Está instalado en **ambos** repositorios (`.specify/` + comandos en `.opencode/commands/`).

## Qué es y qué no es

**Es** un CLI (`specify`) más un directorio `.specify/` con plantillas y comandos slash.

**No es** un agente. Los agentes leen los archivos de `.specify/` y los siguen.

## El ciclo

```
constitution → specify → clarify → plan → tasks → analyze → implement
                                                    ↘ checklist (opcional)
                                                    ↘ converge (al final)
```

**Cada fase se guarda en `.specify/` y requiere aprobación humana antes de la
siguiente.** No se salta.

| Comando | Para qué |
|---|---|
| `/speckit.constitution` | Principios que gobiernan el proyecto |
| `/speckit.specify` | Crear la especificación (qué y por qué) |
| `/speckit.clarify` | Resolver ambigüedades **antes** de planificar (opcional) |
| `/speckit.plan` | Plan técnico (cómo) |
| `/speckit.tasks` | Lista de tareas ejecutables |
| `/speckit.analyze` | Coherencia entre artefactos (opcional) |
| `/speckit.checklist` | Checklist de calidad (opcional) |
| `/speckit.implement` | Ejecutar la implementación |
| `/speckit.converge` | Evaluar el codebase y añadir tareas pendientes |

## Estructura de `.specify/`

```
.specify/
├── templates/          spec, plan, tasks, constitution, checklist
├── scripts/powershell/ check-prerequisites, create-new-feature, setup-plan, setup-tasks, common
├── memory/constitution.md
├── workflows/speckit/workflow.yml
├── integrations/       opencode.manifest.json
├── integration.json
└── feature.json
```

## Las constituciones vigentes

Son dos y **no son iguales**. Esto es deliberado:

| Repo | Documento | Versión |
|---|---|---|
| Web | *MiBeca Dual-Repository Constitution* | **1.1.0** |
| Backend | *MiBeca Backend API Constitution* | **1.0.0** |

Ambas ratificadas el 2026-03-30. Principios compartidos:

1. **Contrato unificado**: la API es canónica y compartida; ningún repo define contratos
   mock paralelos una vez iniciada la integración.
2. **Progresión sincronizada entre repos**: si una tarea está bloqueada por la otra,
   se cambia de contexto a la que bloquea en vez de quedarse esperando.
3. **Test-first y puertas de calidad (NO NEGOCIABLE)**: `npm run lint`, `npm run build` y
   pruebas. Sin atajos, sin suprimir lint, sin stubs sin prueba.
4. **Alineación con API y tema canónicos** (frontend): endpoints declarados y tokens de
   `theme.css`.
5. **Simplicidad, YAGNI y entrega incremental**: slices finos y verificables; nada que no
   sirva al MVP.

El backend añade: pruebas de integración obligatorias para endpoints nuevos, cambios de
contrato, comunicación entre servicios y esquemas compartidos.

## Obligatorio: todo spec sale repartido en 3 bloques

`/speckit.tasks` produce una lista plana de tareas. **No se usa así.** Antes de ejecutar
nada, esa lista se divide en **tres bloques consecutivos** con el cálculo de
[[Reparto de tareas]]: tamaño `⌈N/3⌉`, el último bloque absorbe el sobrante, numeración
global estable (`T01…TN`), y backend antes que frontend.

Un spec que no trae el reparto escrito **no está listo** para `/speckit.implement`.

## Relación con los planes del proyecto

Las specs **son ahora el vehículo del trabajo**. El orden de autoridad para trabajo nuevo:

1. **La spec** en `specs/NNN-*/` — qué se construye y cómo se verifica.
2. `openapi.yaml` — el contrato técnico que la spec debe respetar.
3. `.specify/memory/constitution.md` — los principios que no se negocian.

Los planes quedan como historia, no como fuente de tareas:

- `PLAN_MVP_MIBECA.md` — describe **lo ya implementado** (fases 0–6). Sirve para entender
  el producto, no para repartir trabajo.
- `PLAN_MVP_MIBECA_V2.md` — **en desuso**. Su contenido se está migrando a specs y el
  documento se retirará al terminar; no se ejecuta desde ahí.

Hoy `specs/` solo tiene `001-*` en cada repo. Las specs de `contextos` y `evaluaciones`
(lo que el v2 describía) están pendientes de crear.

## Instalación (si hay que reinstalar)

```powershell
# 1. uv
Invoke-WebRequest -Uri https://astral.sh/uv/install.ps1 -OutFile $env:TEMP\uv.ps1; . $env:TEMP\uv.ps1
# 2. specify
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
# 3. comprobar
specify --version
specify check
# 4. inicializar en un repo
specify init . --here --integration opencode --force --non-interactive
```

> [!tip] Uso práctico
> El primer paso de cualquier tarea no trivial es cargar el **router** del repo
> ([[Agentes y skills]]), y este decide si hace falta `/speckit.specify` antes de tocar
> código.

Relacionado: [[Ramas y PR]] · [[Agentes y skills]]
