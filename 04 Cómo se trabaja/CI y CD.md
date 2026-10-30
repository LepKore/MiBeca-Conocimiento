---
tags: [mibeca, proceso, ci, cd]
---

# CI y CD

Fuente: `.github/workflows/*.yml` de ambos repos, `docs/plan-colaboracion.md`.

## Backend

### CI — `.github/workflows/ci.yml`

Dispara en **pull_request** y en **push** a `main` y `dev`. Concurrency con
`cancel-in-progress: true` — el mismo grupo cancela la ejecución anterior.

Servicio `postgres:17.11` efímero con healthcheck. Variables:
`MIGRATION_TARGET=local`, `DB_SSL_MODE=disable` — **nunca** credenciales de Supabase.

Pasos:

1. `npm ci --no-audit --no-fund`
2. `npm run check` (chequeo de sintaxis de `src`, `scripts`, `test`, `.opencode`, `.codex`)
3. `npm run db:migrate:env`
4. **Lint del contrato**: `npx @redocly/cli@1.34.2 lint openapi.yaml`
5. Base de prueba efímera (`scripts/create-test-database.js` + `TEST_DATABASE_URL`) y
   `node --test --test-concurrency=1`
6. Arranca `src/server.js` y espera `GET /ready` (10 intentos, 1 s)

### CD — `.github/workflows/cd.yml`

Se dispara con `workflow_run` de *Backend CI* completado en `main`, y **solo si la CI fue
exitosa**. Exige el secreto `RENDER_DEPLOY_HOOK_URL` y hace `POST` a ese hook.

> Es decir: **Render** es el destino de despliegue del backend, aunque la base de datos
> sea Supabase. `concurrency` con `cancel-in-progress: false` para no cortar un deploy a
> medias.

## Frontend

### CI — `.github/workflows/ci.yml`

Dispara en **pull_request** y en **push** a `main`, `dev` y `sergio`. Node 22 con caché de
npm.

1. `npm ci --no-audit --no-fund`
2. `node .codex/hooks/audit_bridge.mjs --all` (auditor; **sale en 1 si hay hallazgos
   `deny`**)
3. `npm run lint`
4. `npm test`
5. `npm run build`

### Firebase Hosting

- **Preview en PR** (`firebase-hosting-pull-request.yml`) y **live desde `main`**
  (`firebase-hosting-merge.yml`).
- Ambos workflows: validan que `VITE_API_URL` sea **HTTPS con ruta `/api/v1`**, esperan un
  `/ready` exitoso, y aplican lint/pruebas/build **antes** de publicar.
- Una variable ausente o una API caída **detiene** la publicación en vez de publicar una
  interfaz desconectada.
- Solo se publica desde PRs del mismo repositorio.

## Qué NO está verificado

- **Ninguna ejecución remota de GitHub Actions** ha sido observada. Las ramas tienen
  commits locales sin publicar.
- Protección de ramas, revisores requeridos y secretos de GitHub: sin comprobar.
- CORS del dominio de preview de Firebase: pendiente (el backend valida orígenes exactos
  y no se añadió comodín).
- Cookies de sesión entre dominios distintos: sin resolver. `SameSite=None; Secure` y CORS
  correcto **no garantizan** que el navegador acepte cookies de terceros.

## Regla mental

> La existencia de un workflow o de un secreto configurado **no prueba** un despliegue
> exitoso. Preview ≠ live. Compilar ≠ estar en producción.

Relacionado: [[Guardas y auditor de código]] · [[Estado actual]] · [[Supabase]]
