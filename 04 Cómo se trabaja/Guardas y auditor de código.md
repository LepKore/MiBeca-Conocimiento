---
tags: [mibeca, proceso, seguridad, automatizacion]
---

# Guardas y auditor de código

Fuente: `docs/automation.md` (ambos repos), `.opencode/lib/code-audit-rules.js`,
`.codex/hooks/*`, `.githooks/pre-commit`.

Hay **cuatro** capas de control local. Ninguna sustituye a CI, a los permisos del
proveedor ni a la revisión humana: solo adelantan la señal.

## 1. Hook de Codex (`PreToolUse` / `PostToolUse`)

- `python .codex/hooks/pre_tool_use.py` sobre comandos Bash.
- **Bloquea destrucción de volúmenes** de Compose cuando el directorio, el archivo
  Compose o el nombre de proyecto identifican al backend MiBeca (`down -v`, `rm -v`).
- Bloquea `docker volume rm` solo si el nombre termina en `_mibeca_pgdata`.
- Bloquea `volume prune` y `system prune --volumes` sin filtro de proyecto ajeno (alcance
  ambiguo).
- **Permite** `up`, `stop`, `start` y `down` sin `-v`.
- Requiere **Python 3 accesible como `python`** (en este equipo, 3.11.9). `python3` no
  resuelve en Windows.
- La confianza del hook en el cliente **no está verificada**, y su matcher `Bash` no
  cubre comandos lanzados desde PowerShell.

## 2. Git `pre-commit`

- Script idéntico en ambos repos: `.githooks/pre-commit`.
- Instalación por clon: `git config core.hooksPath .githooks`.
- Rechaza nombres típicos de archivos de secretos y marcadores de claves en los blobs
  preparados. **Muestra rutas, nunca valores.**
- Se puede omitir en otros clones; no substituye controles del servidor.

## 3. Auditor de código `mibeca.code-audit`

El corazón del control. **Un solo motor**, dos clientes:

| Pieza | Archivo |
|---|---|
| Reglas puras | `.opencode/lib/code-audit-rules.js` (backend) / `.opencode/lib/*` (web) |
| Plugin OpenCode | `.opencode/plugin/code-audit.js` — expone `audit_code` y `/auditar` |
| Puente Codex | `.codex/hooks/audit_bridge.mjs` sobre `apply_patch` |
| Verificación | `.opencode/plugin/code-audit.verify.mjs` |
| Pruebas | `test/code-audit.test.js`, `test/codex-audit.test.js` |

- **26 reglas, 5 duras (`deny`)**: `SEC-01`, `SEC-02`, `SEC-03`, `DB-01`, `AUTH-01`.
  Solo las `deny` bloquean la escritura; `warn` e `info` informan.
- En OpenCode lanza `throw` y acumula avisos para el turno siguiente (máx. 20).
- En Codex, `PreToolUse` deniega solo las 5 duras; `PostToolUse` relee los archivos y
  devuelve avisos como contexto.
- Comandos: `npm run audit:changed`, `npm run audit:all`, `npm run audit:verify`.
- Calibración sobre el árbol: **0 bloqueos, 0 avisos, 4 informativos**, sin rutas fuera
  de `openapi.yaml`.

### Las 5 reglas que bloquean

| Regla | Detecta |
|---|---|
| `SEC-01` | Secreto incrustado: clave PEM, `ghp_`, `github_pat_`, `AKIA`, `AIza` |
| `SEC-02` | Ruta de credenciales: `.env` (salvo `.env.example`), `firebase-adminsdk`, `*service-account*`, `credentials.json`, `.pem`, `.p12`, `.pfx`, `.key` |
| `SEC-03` | `postgres://usuario:contraseña@` con contraseña literal en `src/` |
| `DB-01` | SQL que interpola `${req.}`, `${body.}`, `${params.}`, `${headers.}` |
| `AUTH-01` | `router.post/patch/put/delete` sin `requireCsrf` en `src/routes/` |

### Avisos que conviene conocer

`SEC-04` log de sesión/credenciales · `SEC-05` cookie con `httpOnly`/`secure` en `false` ·
`SEC-06` CORS comodín · `SEC-07` hash `md5`/`sha1` · `API-01` `throw new Error` en vez de
`src/lib/errors.js` · `API-02` `request.body` sin `parse(` · `API-03` `response.send` ·
`API-05` `error.stack` en respuesta · `DB-02` `BEGIN` sin `ROLLBACK` · `DB-03`
`UPDATE`/`DELETE` sin `WHERE` · `DB-04` `process.env.DATABASE_URL` fuera del borde de
entorno · `AUTH-02` ruta sin middleware de sesión/rol · `AUTH-03` comparación de token con
`===` en vez de `safeEqual` · `AUTH-04` mensaje de login que revela si la cuenta existe ·
`MIG-01` `DROP`/`TRUNCATE` sin justificación · `MIG-02` nombre de migración incorrecto ·
`HYG-01` `console.log`/`debugger` · `HYG-02` `TODO`/`FIXME` (info) · `HYG-03` `.only` ·
`SUP-01` `eslint-disable`/`@ts-ignore` sin justificar · `DB-05` `SELECT *` (info).

### Qué NO hace

- No ejecuta el código, no sigue imports, no entiende el flujo real.
- Un aviso puede ser **falso positivo**; código ofuscado o dinámico puede eludirlo.
- No cubre archivos fuera de las extensiones y carpetas declaradas (`.js`, `.mjs`, `.sql`
  en `src`, `scripts`, `test`, `migrations`), ni datos en tiempo de ejecución.

## 4. Plugin `docker-guard`

`.opencode/plugin/docker-guard.js`, con lógica pura en
`.opencode/lib/docker-guard-rules.js`. Migrado al mismo contrato V2 de plugins. Su
verificación: **34/34**.

## Cómo comprobar que todo sigue vivo

```bash
python -m py_compile .codex/hooks/pre_tool_use.py .githooks/pre-commit
node .opencode/plugin/docker-guard.verify.mjs     # 34/34
node .opencode/plugin/code-audit.verify.mjs       # 40/40
node --test test/code-audit.test.js test/codex-audit.test.js
node .codex/hooks/audit_bridge.mjs --all --info
```

Los `.verify.mjs` imprimen `OK` por caso y salen en 0 si todo pasa. En Windows, sustituir
`printf` por `Write-Output`.

> [!warning] Si te bloquea
> Registra la orden, la capa (hook Codex, Git, permisos del entorno, GitHub o proveedor),
> la condición y la vía permitida. La salida correcta es **corregir el código**. La
> excepción legítima la ejecuta una persona, manualmente, fuera de la guardia.

Relacionado: [[Agentes y skills]] · [[CI y CD]] · [[Supabase]]
