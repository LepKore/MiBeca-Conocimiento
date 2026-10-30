---
tags: [mibeca, proceso, agentes]
---

# Agentes y skills

Fuente: `AGENTS.md` de ambos repos, `.agents/skills/*`, `docs/automation.md`.

## Regla número uno: el router primero

En **ambos** repos, la primera acción de cualquier tarea es cargar el **router** de
skills del repo. Él decide qué skills aplican.

| Repo | Router |
|---|---|
| Backend | `.agents/skills/mibeca-sdd-router/SKILL.md` |
| Frontend | `.agents/skills/mibeca-web-sdd-router/SKILL.md` |

Cascada de decisión:

1. Skills **locales** del proyecto → si ninguna encaja…
2. Skills **globales** del entorno → si tampoco…
3. Implementar sin skills y **anunciarlo**: «No se usaron skills para esta tarea: ninguna
   skill aplicaba a [tipo de trabajo]».

Los routers **no nombran** skills globales por nombre cuando una local aplica.

## `.agents/skills` es la fuente única

Las skills de proyecto viven en **`.agents/skills/`** y las consumen tanto Codex como
OpenCode. **No se duplican** en `.codex/` y `.opencode/`.

> [!note] Excepción observada
> En el repo web existen además `.claude/skills/` con copias del pack de Firebase. Son
> artefactos de instalación, no la fuente de verdad.

## Skills locales del backend

| Skill | Cuándo |
|---|---|
| `mibeca-sdd-router` | Siempre, al iniciar |
| `api-contract-validation` | Cambios en API o `openapi.yaml` |
| `database-migration-safety` | Cambios de esquema o migraciones |
| `deploy-checklist` | Despliegue / revisión de CI |
| `testing` | ⚠️ **No aplicable**: exige Grafana Cloud, tokens de k6/Faro y sondas remotas. Solo referencia futura |

## Skills locales del frontend

| Skill | Cuándo |
|---|---|
| `mibeca-web-sdd-router` | Siempre, al iniciar |
| `vercel-react-best-practices` | Cualquier componente React |
| `vitest` | Pruebas |
| `api-integration-contract` | Pantallas o formularios que consumen la API |
| `frontend-accessibility` | Componentes, estilos, responsive |
| `firebase-hosting-basics` | **Solo con tarea expresa** de despliegue |

**Pack de Firebase instalado y NO aplicable al MVP**: `firebase-auth-basics`,
`firebase-firestore`, `firebase-data-connect`, `firebase-ai-logic-basics`,
`firebase-crashlytics`, `firebase-security-rules-auditor`, `firestore-rules-creation`,
`firebase-remote-config-basics`, `firebase-app-hosting-basics`, `firebase-basics`,
`extension-to-functions-codebase`, `xcode-project-setup`.

> El frontend usa la API del backend como fuente de autenticación y datos. **No** se
> monta Firestore, Auth de Firebase ni ningún servicio extra. Esas skills quedan como
> referencia futura.

## Perfiles de agente

| Archivo | Repo | Alcance |
|---|---|---|
| `.codex/agents/backend-data.toml` | backend | API, Docker, migraciones |
| `.codex/agents/reviewer.toml` | ambos | Revisión en lectura |
| `.codex/agents/frontend.toml` | web | Componentes, estilos, adaptador API |
| `.opencode/agent/backend-data.md` | backend | Equivalente OpenCode |
| `.opencode/agent/reviewer.md` | ambos | Equivalente OpenCode |
| `.opencode/agent/frontend.md` | web | Equivalente OpenCode |

Son **instrucciones de alcance, no permisos de sandbox**. Un agente puede leer fuera de
su alcance aunque su perfil diga que no debería.

## Reglas que un agente nunca debe romper

- No ejecutar despliegues ni migraciones remotas por iniciativa propia.
- No borrar volúmenes de datos.
- Datos de prueba solo con comando explícito de desarrollo.
- No autoautorizarse ni editar controles para evitar un rechazo.
- Corregir el **código**, no el **control** (auditor de código).
- Toda ruta de API del frontend va en `src/api/endpoints.js`.

Relacionado: [[SDD con Spec Kit]] · [[Guardas y auditor de código]]
