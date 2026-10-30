---
tags: [mibeca, reglas, seguridad]
---

# Usuarios y sesiones

Fuente: `docs/AI_CONTEXT.md` (backend §2 y §4), `docs/mvp-y-permisos.md`
§Seguridad y aceptación, `migrations/0001_init.sql`.

## Altas y contraseñas

- **Sin registro público.** Solo un `ADMIN` crea cuentas.
- El alta entrega una **contraseña temporal aleatoria mostrada una sola vez**.
- `requiere_cambio_password = true` por defecto: el usuario debe cambiarla **antes** de
  usar el negocio.
- El **reset administrativo** genera una contraseña temporal nueva y **revoca las
  sesiones** del usuario.
- Hash con **argon2id**; el hash nunca se registra ni se devuelve.
- No hay credenciales por defecto en el repositorio. El primer administrador se crea
  con un comando explícito: ver [[Primer administrador]].

## Desactivación

- Es **lógica** (`activo = false`), nunca `DELETE`.
- **Revoca las sesiones** vigentes.
- **Conserva relaciones e historial**: trabajos, postulaciones, cierres y asistencias
  siguen existiendo.
- **Reactivar no revive sesiones antiguas.** El usuario debe volver a entrar.
- **No se puede desactivar al último administrador activo**: la API responde
  `409` con código `ultimo_admin_activo`.

## Sesiones

| Aspecto | Decisión |
|---|---|
| Tipo | Sesiones **opacas en servidor**, tabla `sesiones` |
| Cookie | `HttpOnly`, contiene un token aleatorio; es el **único portador del secreto** |
| Almacenamiento | Solo el **hash SHA-256** del token (`sesiones.token_hash`), índice único |
| CSRF | Cabecera `X-CSRF-Token` obligatoria en **toda** escritura |
| Expiración | `sesiones.expira_en`; `revocado_en` marca revocación explícita |
| Logout | Revoca la sesión en base de datos |

`cookieParser` se monta **sin secreto**: la cookie es un identificador opaco y la
integridad la da el hash almacenado, no una firma.

## Otras defensas del backend

- `helmet()` para cabeceras de seguridad.
- CORS de **origen exacto** con `Access-Control-Allow-Credentials: true` y `Vary: Origin`.
  Nunca comodín: las sesiones viajan en cookie.
- Límite de intentos de login (`src/lib/rateLimit.js`).
- Validación de entrada con **zod**.
- Manejador de errores centralizado: **nunca** devuelve trazas ni SQL en la respuesta,
  solo `{ error: { code, message, detalles? } }`.
- Tabla `auditoria` con actor, acción, recurso y detalle `jsonb`.

> [!note] Mensajes de login
> La regla `AUTH-04` del auditor prohíbe un mensaje que revele si la cuenta existe.

Relacionado: [[Roles y permisos]] · [[API v1]] · [[Modelo de datos]]
