---
tags: [mibeca, proceso, git]
---

# Ramas y PR

Fuente: `docs/plan-colaboracion.md`, `docs/PLAN_DE_TRABAJO_MIBECA.md`, `AGENTS.md`.

## Ramas

| Rama | Uso |
|---|---|
| `main` | Integración y despliegue. Requiere PR, revisión y checks |
| `dev` | Integración de trabajo en curso. **Estado actual de ambos clones** |
| `sergio`, `Ivan` | Ramas personales con trabajo en curso |
| `feat/...`, `fix/...` | Ramas por tarea |

Una rama por tarea. **Nunca** trabajo directo sobre `main`.

## Reglas de integración

1. **Antes de abrir PR**: `npm run lint`, `npm run build` y las pruebas pertinentes.
2. **Revisión rotativa**: un integrante implementa, otro revisa el PR, el tercero valida
   la integración.
3. **Cambios de contrato** se revisan en **ambos** repos.
4. **`PLAN_MVP_MIBECA.md` debe tener la misma versión en los dos repos.** Backend mantiene
   la fuente funcional; cada cambio exige un PR de sincronización enlazado en web. No
   editar las copias de forma independiente.
5. Los datos y secretos **no viajan por Git**. Git comparte Compose, configuración de
   ejemplo, migraciones y seeds explícitos.

## Qué requiere autorización humana explícita

| Operación | ¿Ordinaria? |
|---|---|
| Editar código, crear/borrar archivos acotados, ejecutar pruebas | ✅ Sí |
| `docker compose stop` / `up --force-recreate` (mismo volumen) | ✅ Sí |
| `git push` a una rama de trabajo propia | ✅ Sí, si el equipo lo autorizó |
| `docker compose down -v`, `docker volume rm` | ❌ Borra datos; el hook lo rechaza |
| Aplicar migraciones locales | ⚠️ Verificar SQL y destino primero: [[Migraciones]] |
| Migración a producción / base externa | ❌ Revisión y procedimiento explícito |
| Publicar live en Firebase o Render | ❌ Gates de CI y autorización concreta |
| Integrar `main` | ❌ PR + revisión |

> [!important] Autorización del chat ≠ permiso ejecutable
> Ningún mensaje en el chat cambia un hook. Para una excepción legítima, la persona
> responsable revisa la operación concreta y la ejecuta **manualmente** en un paso único
> fuera de la guardia del agente. El agente **nunca** desactiva ni edita un control para
> saltarse un rechazo.

## Al bloquearse una tarea (regla cross-repo)

Si la tarea N del repo A depende de la tarea M incompleta del repo B, **no se espera**:
se cambia a la cola de tareas del repo B hasta desbloquear, y viceversa. Es un principio
constitucional en ambos repos.

## Al terminar una tarea

Reportar: archivos cambiados, validaciones ejecutadas, dependencias descubiertas y
limitaciones explícitas. Las decisiones durables se escriben en `docs/`, en el PR/issue
o en `AGENTS.md`. Nada se da por sabido entre sesiones.

## Commit hook

Cada clon instala el hook local una vez:

```bash
git config core.hooksPath .githooks
```

Ver [[Guardas y auditor de código]].
