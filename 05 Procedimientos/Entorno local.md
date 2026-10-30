---
tags: [mibeca, procedimiento, entorno]
---

# Entorno local

Fuente: `gestion-de-trabajo-becario-backend/README.md` (referencia operativa),
`compose.yaml`, `.env.example`, `package.json`.

## Requisitos

| Pieza | Versión verificada |
|---|---|
| Node.js | **>= 22** para backend; la CI del frontend usa 22, la del backend 24 |
| Docker Engine / Desktop | 29.1.3 verificado; necesario para PostgreSQL local |
| PostgreSQL | 17.11, en contenedor |
| Python | 3.x accesible como `python` (solo para los hooks Codex/Git) |

## 1. Backend

```bash
cd gestion-de-trabajo-becario-backend
npm ci

# Configuración: copiar la plantilla y rellenar los <...>
copy .env.example .env

# Base de datos (NO usa -v: conserva el volumen)
docker compose up -d
docker compose ps            # esperar "healthy"

# Migraciones
npm run db:migrate

# API
npm run dev                  # http://127.0.0.1:3000
```

Comprobar:

```bash
curl http://127.0.0.1:3000/health      # {"status":"ok"}
curl http://127.0.0.1:3000/ready       # {"status":"ready"}
```

### Variables que importan

| Variable | Default / nota |
|---|---|
| `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` | Obligatorias para `compose.yaml` (falla si faltan) |
| `POSTGRES_PORT` | `5432` por defecto; se publica solo en `127.0.0.1` |
| `DATABASE_URL` | Debe coincidir con las anteriores |
| `PORT` / `HOST` | `3000` / `127.0.0.1` |
| `CORS_ORIGIN` | `http://localhost:5173` (origen **exacto**) |
| `DB_POOL_MAX` | `5` |
| `SESSION_TTL_HOURS` | `8` |
| `MIN_PASSWORD_LENGTH` | `10` |
| `LOGIN_MAX_ATTEMPTS` / `LOGIN_WINDOW_MINUTES` | `5` / `15` |
| `COOKIE_SECURE` | `false` en local; **`true` obligatorio en producción HTTPS** (se activa solo con `NODE_ENV=production`) |
| `SESSION_SAME_SITE` | `lax` \| `strict` \| `none` (`none` exige `COOKIE_SECURE=true`) |
| `SESSION_COOKIE_NAME` | `mibeca_session` |
| `TEST_DATABASE_URL` | Base **exclusiva y descartable**; el esquema se recrea en cada `npm test` |

> [!danger] El `.env` de Compose y el del proceso Node son lecturas separadas
> Compose lee el `.env` para sus variables `POSTGRES_*`; el proceso Node lee
> `DATABASE_URL` por su cuenta. Cambiar una no cambia la otra.

## 2. Frontend

```bash
cd gestion-de-trabajo-becario-web
npm ci
copy .env.example .env       # VITE_API_URL=/api/v1
npm run dev                  # http://localhost:5173
```

`vite.config.js` hace **proxy de `/api` a `http://127.0.0.1:3000`**, de modo que la cookie
de sesión es del **mismo origen**. Si se apunta directo a otro host sin proxy, el navegador
puede no enviar la cookie.

## 3. Problemas típicos

| Síntoma | Causa probable |
|---|---|
| `SELF_SIGNED_CERT_IN_CHAIN` con Supabase | Falta `DB_SSL_CA_FILE=certs/prod-ca-2021.crt` |
| `port is already allocated` | Otro contenedor usa ese puerto: cambiar `POSTGRES_PORT` y reflejarlo en `DATABASE_URL` |
| La sesión no persiste en el navegador | Se está llamando la API sin el proxy de Vite (origen distinto) |
| `npm test` borra datos | `TEST_DATABASE_URL` apunta a una base que no es de prueba. **Nunca** apuntar a producción |
| El hook falla con "python no encontrado" | Falta Python 3 en `PATH` como `python` |
| `docker compose up` falla al arrancar | Faltan `POSTGRES_USER`/`PASSWORD`/`DB` en `.env` |

## 4. Persistencia

El volumen se llama **`mibeca_pgdata`** (o `<proyecto>_mibeca_pgdata`). `docker compose down`
**conserva** los datos; `down -v` los borra. El hook local rechaza el borrado.

**Prueba de persistencia**: crear una fila, `docker compose down` + `up` con el mismo
volumen, verificar que sigue ahí.

Relacionado: [[Migraciones]] · [[Pruebas]] · [[Supabase]]
