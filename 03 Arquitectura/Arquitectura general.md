---
tags: [mibeca, arquitectura]
---

# Arquitectura general

Fuente: `docs/AI_CONTEXT.md` de ambos repos, `src/app.js`, `package.json`.

## Forma del sistema

Dos repositorios hermanos y un coordinador en la raíz:

```
ProyectoMiBeca/
├── gestion-de-trabajo-becario-web/      React 19 + Vite 8  → Firebase Hosting
├── gestion-de-trabajo-becario-backend/  Express 5 + PostgreSQL → Render / Supabase
├── base nuevo diseno frontend/          prototipo mock (NO es la app)
└── MiBeca-Conocimiento/                 este vault
```

**El frontend nunca habla con la base de datos.** Consume una API HTTP; la autorización
real vive en el backend.

## Stack verificado

### Frontend

| Pieza | Versión / nota |
|---|---|
| React + React DOM | 19.2.8 |
| Router | `react-router-dom` 7.18.4 |
| Build | Vite 8.3.0 |
| Estilos | Tailwind 4.3.3 (`@tailwindcss/vite`) + `src/styles/theme.css` con tokens |
| Calidad | ESLint 10, Vitest 5, Testing Library, jsdom |
| Hosting | Firebase Hosting, proyecto `mibeca`, publica `dist` |

### Backend

| Pieza | Versión / nota |
|---|---|
| Runtime | Node **>= 22** (CI usa Node 24), ESM (`"type": "module"`) |
| Framework | Express 5.3 |
| Base de datos | PostgreSQL **17.11**, cliente `pg` 8.16, sin ORM |
| Validación | zod 4.6 |
| Contraseñas | `argon2` 0.45 (argon2id) |
| Seguridad HTTP | helmet 8.3, cookie-parser |
| Migraciones | SQL propio en `migrations/`, ejecutado por `scripts/migrate.js` |

## Flujo de una petición

```
Navegador
  └─ React (src/pages/*)
       └─ src/lib/api.js        ← único cliente HTTP (fetch con credentials)
            └─ src/api/endpoints.js  ← única fuente de verdad de rutas
                 └─ /api/v1/...      ← proxy de Vite en desarrollo
                      └─ Express createApp({ pool, env })
                           ├─ cors (origen exacto) → helmet → json (64 kb)
                           ├─ /health /health_db /ready
                           ├─ cookieParser
                           ├─ /api/v1/auth     → authRouter
                           ├─ /api/v1/users    → usersRouter
                           ├─ /api/v1          → jobsRouter
                           ├─ /api/v1          → controlRouter
                           └─ notFound → manejador de errores centralizado
                                └─ PostgreSQL (pool)
```

## Detalles que explican decisiones raras

- **`createApp({ pool, env })`** está separado de `src/server.js` para montar la API en
  pruebas sin abrir un puerto fijo.
- **CORS se aplica antes de helmet** para que sus cabeceras no pisen las de la respuesta.
- **Proxy de Vite**: en desarrollo `/api` se reenvía a `http://127.0.0.1:3000` para que
  la cookie sea del **mismo origen**. Sin eso, el navegador no envía la sesión.
- **`DB_POOL_MAX` por defecto 5** y sin sentencias preparadas con nombre: Supavisor en
  modo transacción no las admite.
- **`app.set('trust proxy', false)`** y `x-powered-by` desactivado.
- **Límite de cuerpo 64 kb**: no se suben archivos (fuera del MVP).

## Configuración por variables

- Frontend: `VITE_API_URL` (debe ser HTTPS con ruta `/api/v1`) y `VITE_TIME_ZONE`
  (por defecto `America/La_Paz`).
- Backend: `DATABASE_URL`, `DB_SSL_MODE`, `DB_SSL_CA_FILE`, `DB_POOL_MAX`,
  `DB_POOL_IDLE_TIMEOUT_MS`, `HOST`, `PORT`, `CORS_ORIGIN`, `MIGRATION_TARGET`,
  `MIGRATION_CONFIRM_HOST`, `BOOTSTRAP_ADMIN_*`. Ver `.env.example`; el proceso Node
  carga `.env` **solo si existe** y lo ya presente en el entorno tiene prioridad (así en
  producción mandan las variables del proveedor).

## Frontend: estructura que importa

| Archivo | Rol |
|---|---|
| `src/App.jsx` | Monta las rutas |
| `src/auth/AuthProvider.jsx` | Mantiene la sesión con `GET /auth/me`; resuelve el 401 |
| `src/lib/api.js` | Cliente HTTP único; envía CSRF e `Idempotency-Key` |
| `src/api/endpoints.js` | **Toda** ruta de la API; `test/endpoints.test.js` falla si aparece una ruta literal fuera de ahí |
| `src/components/ProtectedRoute.jsx` | Restricción por rol (comodidad, no seguridad) |
| `src/components/AppShell.jsx` | Navegación, enlace "Saltar al contenido", `Dialog` con foco atrapado |
| `src/styles/theme.css` | Tokens de identidad visual |

Páginas: `Login`, `ChangePassword`, `Panel`, `Trabajos`, `Postulaciones`, `Control`,
`AdminUsuarios`, `AdminSupervision`, `NoEncontrado`.

Relacionado: [[API v1]] · [[Usuarios y sesiones]] · [[Entorno local]]
