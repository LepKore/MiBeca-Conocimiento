---
tags: [mibeca, procedimiento]
---

# Primer administrador

Sin administrador no hay forma de crear cuentas: **no existe registro público**. Hay dos
scripts y una diferencia importante entre ellos.

## Opción A — Bootstrap idempotente (recomendado)

```bash
npm run db:bootstrap-admin
```

- Crea el administrador **solo si no existe ningún ADMIN activo**. Segunda ejecución: no
  duplica.
- Lee `BOOTSTRAP_ADMIN_NOMBRE`, `BOOTSTRAP_ADMIN_CORREO` y opcionalmente
  `BOOTSTRAP_ADMIN_PASSWORD` del `.env`.
- Si `BOOTSTRAP_ADMIN_PASSWORD` está **vacío**, genera una contraseña temporal y la
  **muestra una sola vez** por consola.
- **Nunca sobrescribe** una cuenta existente. Si hay que cambiarla, se hace después con
  SQL sobre `usuarios`.

## Opción B — Alta manual interactiva

```bash
npm run db:create-admin
```

Pregunta por consola. Ojo: el entorno de agente es **no interactivo**, así que este
comando se ejecuta a mano en una terminal real.

## Reglas

- **No hay credenciales por defecto en el repositorio.** Define una propia y fuerte.
- La cuenta nace con `requiere_cambio_password = true`: el administrador debe cambiarla
  antes de operar. Ver [[Usuarios y sesiones]].
- La contraseña del primer admin **se muestra una sola vez**. Si se pierde, hay que
  rehacer el alta con SQL.

## Secuencia completa desde cero

```bash
docker compose up -d
npm run db:migrate
npm run db:bootstrap-admin     # apuntar la salida
npm run dev
# abrir http://localhost:5173, entrar, cambiar la contraseña obligatoria
```

Después, desde la interfaz: crear encargados y estudiantes, fijar metas individuales.

> [!tip] Verificación de que quedó bien
> `GET /ready` responde `{"status":"ready"}` y el login con el correo del bootstrap
> devuelve la pantalla de cambio obligatorio, no el panel.

Relacionado: [[Entorno local]] · [[Migraciones]] · [[Roles y permisos]]
