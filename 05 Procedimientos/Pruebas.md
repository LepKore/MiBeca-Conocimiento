---
tags: [mibeca, procedimiento, pruebas]
---

# Pruebas

## Backend

```bash
npm run check      # sintaxis de src, scripts, test, .opencode, .codex
npm test           # node:test, secuencial, esquema recreado
npm run audit:verify
```

- Runner: **`node:test` nativo**. Comando exacto:
  `node --env-file-if-exists=.env --test --test-concurrency=1 "test/*.test.js"`.
- **Concurrencia 1**: las suites comparten base de datos y no pueden correr en paralelo.
- Requiere `TEST_DATABASE_URL` con una base **exclusiva y descartable**: el esquema se
  recrea en cada ejecución.
- Los helpers (`test/helpers.js`) **rechazan hosts remotos** salvo
  `TEST_ALLOW_REMOTE=true`, y ese permiso es solo para una base de pruebas.

### Archivos

`api.test.js` · `jobs.test.js` · `control.test.js` · `phase5.unit.test.js` ·
`database-safety.test.js` · `code-audit.test.js` · `codex-audit.test.js`

## Frontend

```bash
npm run lint       # eslint .
npm test           # vitest run
npm run build      # vite build
```

- Runner: **Vitest 5** con Testing Library y jsdom.
- `vitest.config.js` usa `pool: 'threads'` y `maxWorkers: 4`. Los archivos que no tocan
  DOM declaran `@vitest-environment node`.
- Última cifra documentada: **19 archivos, 135 pruebas, ~18 s**, cinco pasadas
  consecutivas sin el timeout de worker.

### Archivos (16)

`sesion` · `postulaciones` · `postulaciones-paginacion` · `panel` · `jobs` ·
`form-field` · `endpoints` · `docker-guard` · `dialog-accessibility` · `deploy-env` ·
`control` · `config` · `codex-audit` · `code-audit` · `auth-provider` · `app-shell` ·
`api` · `admin-usuarios` · `admin-supervision`

## Pruebas obligatorias del flujo (lista del plan)

Esto es lo que **debe** cubrirse antes de considerar el MVP terminado:

- Login válido/inválido, cambio obligatorio y sesión revocada tras logout, desactivación
  o reset.
- Estudiante no puede crear cuentas, cambiar meta ni otorgar horas; encargado no
  administra trabajos ajenos.
- Código con ceros iniciales; longitudes inválidas rechazadas; correo duplicado; horas
  con más de un decimal fallan.
- Postulación duplicada no crea registros extra; **pendientes no reservan cupos**.
- **Aceptaciones concurrentes del último cupo: solo una gana.**
- Rechazo de aceptación tardía y de edición de condiciones ya fijadas.
- Cierre con asistente no aceptado, lista incompleta o antes del fin: rechazo **sin
  escrituras parciales**.
- Cierre concurrente o repetido: una sola acreditación por aceptado; idempotencia
  coherente.
- Asistente recibe la duración completa, ausente cero; ambos aparecen en el historial.
- Meta menor que lo realizado: faltantes en cero, excedente correcto, sin errores de
  precisión.
- Desactivación conserva relaciones e historial; no se desactiva al último admin.
- Los datos sobreviven a recrear el contenedor con el mismo volumen.
- Frontend contra API real, sesión expirada, móvil y teclado.

## Pruebas que NO están disponibles

- La skill `testing` (Grafana Cloud, k6, Faro, sondas remotas) **no aplica**: exige tokens
  y servicios que el proyecto no tiene. Es adaptación pendiente, no respaldada.
- Las pruebas de integración del backend que necesitan PostgreSQL **no se ejecutaron** en
  la verificación de fase 5 (no había credenciales locales en esa sesión).

## Prueba end-to-end manual (lo que se ha hecho)

Se ha verificado en **Chromium** y con **Playwright** contra API + PostgreSQL locales:

- Cierre con asistencia: **276 minutos** al asistente, **0** al ausente.
- Cupo completo: `0 de 1` y postulación deshabilitada.
- Móvil a **320 px** y escritorio a **1280 px** sin desbordamiento de página.
- Sesión expirada, error HTTP 503 y recuperación al reintentar.
- Las cuentas, trabajos y cierres temporales se eliminaron después de cada recorrido.

Relacionado: [[CI y CD]] · [[Entorno local]]
