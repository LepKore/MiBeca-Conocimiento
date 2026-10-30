---
tags: [mibeca, referencia]
---

# Fuentes en el repositorio

Qué documento manda sobre qué. Cuando dos se contradicen, gana el de mayor autoridad en
esta tabla.

## Autoridad documental

| Documento | Repo | Autoridad |
|---|---|---|
| **`openapi.yaml`** (rev. 1.4.0) | backend | **Contrato técnico canónico**. Nada lo sustituye |
| **`docs/PLAN_MVP_MIBECA.md`** | ambos (misma versión) | **Plan funcional canónico** |
| **`docs/PLAN_DE_TRABAJO_MIBECA.md`** | backend | Numeración operativa de fases |
| `.specify/memory/constitution.md` | cada repo | Principios de trabajo. **No iguales entre repos** |
| `docs/AI_CONTEXT.md` | cada repo | Contexto de agentes; el más actualizado sobre estado |
| `docs/automation.md` | cada repo | Hooks, auditor, límites |
| `docs/plan-colaboracion.md` | backend | Procedimiento de colaboración e inspección histórica |
| `docs/mvp-y-permisos.md` | backend | Resumen acordado del alcance (no es lo implementado) |
| `docs/api-v1.md` | backend | **Subordinado** a `openapi.yaml` |
| `docs/VERIFICACION_FASE_*.md` | ambos | Evidencia de lo que se probó realmente |
| `docs/PLAN_MVP_MIBECA_V2.md`, `PLAN_CORRECCION_AUDITORIA.md` | ambos | **En desuso**. El v2 se está migrando a `specs/`; no es plan de ejecución |
| `AGENTS.md` | cada repo | Reglas para agentes |
| `specs/NNN-*/` | cada repo | **Vehículo del trabajo nuevo.** Hoy solo `001-*` en cada repo |

> [!warning] Los documentos históricos engañan
> `PLAN_MVP_MIBECA_V2.md` y el plan del compañero describen cosas que **no** están
> implementadas (calificaciones, reputación, penalizaciones, Prisma, JWT). No implementar
> nada de ahí sin pasar por `/speckit.specify`.
>
> El v2 no está deprecado porque sea incorrecto, sino porque **su contenido se está
> moviendo a specs**. Lo que queda en el plan es referencia de alcance; lo que se ejecuta
> es la spec.

## Archivos por repo

### `gestion-de-trabajo-becario-backend/`

```
migrations/0001_init.sql              usuarios, estudiantes, encargados, sesiones, auditoria
migrations/0002_trabajos_postulaciones.sql
migrations/0003_cierre_asistencia.sql cierres y asistencias
src/app.js                            createApp: CORS, helmet, rutas, errores
src/config.js                         appConfig(env)
src/db.js                             pool, databaseConfig, isSupabaseHost
src/lib/{errors,validation,passwords,sessions,rateLimit,control-validation,jobs-validation}.js
src/middleware/auth.js                sesión y rol
src/routes/{auth,users,jobs,control}.js
src/services/{users,jobs,control,sessions,student-progress}.js
scripts/{migrate,create-admin,bootstrap-admin,create-test-database,check-syntax}.js
test/*.test.js                        node:test
openapi.yaml                          contrato
compose.yaml                          PostgreSQL 17.11
certs/prod-ca-2021.crt                CA pública de Supabase (material público)
```

### `gestion-de-trabajo-becario-web/`

```
src/App.jsx, src/main.jsx
src/auth/{AuthProvider.jsx,useAuth.js,auth-context.js}
src/lib/{api.js,trabajos.js,useRecurso.js}
src/api/endpoints.js                  ÚNICA fuente de rutas de API
src/components/{AppShell,ProtectedRoute,ui}.jsx
src/config/navegacion.js
src/pages/{Login,ChangePassword,Panel,Trabajos,Postulaciones,Control,AdminUsuarios,AdminSupervision,NoEncontrado}.jsx
src/styles/theme.css                  tokens de identidad visual
test/                                 19 archivos Vitest
firebase.json, .firebaserc            proyecto "mibeca"
scripts/{start.js,validate-deploy-env.js}
```

### Raíz del workspace

```
opencode.json                    MCP de ClickUp (proyecto)
mibeca-becario.png               referencia visual
conocimiento MiBeca/             este vault
```

## Documentos que conviene leer en este orden

Para entender el producto: `PLAN_MVP_MIBECA.md` → `mvp-y-permisos.md` →
`openapi.yaml` → las migraciones.

Para entender el estado real: `docs/AI_CONTEXT.md` de cada repo → los
`VERIFICACION_FASE_*.md` → el historial de git.

Relacionado: [[Estado actual]] · [[SDD con Spec Kit]]
