---
tags: [mibeca, procedimiento, infraestructura]
---

# Supabase

Destino **principal** de la base de datos de producción. La referencia operativa completa
está en `gestion-de-trabajo-becario-backend/README.md`, sección
**"Conectar con Supabase"** — este vault resume lo verificado.

## Por qué el pooler y no la conexión directa

Se usa **Supavisor en modo transacción, puerto 6543**. En ese modo:

- El pool es pequeño: `DB_POOL_MAX` bajo para no agotar el plan.
- **No se admiten sentencias preparadas con nombre.** Por eso `src/db.js` no pasa `name` en
  las consultas. Si alguien lo añade, la conexión falla de forma intermitente.

## Certificado

El pooler de Supabase firma con una CA privada (`Supabase Root 2021 CA`) que **no está en
el almacén de confianza del sistema**. Sin ella:

```
SELF_SIGNED_CERT_IN_CHAIN
```

El certificado raíz público está versionado en **`certs/prod-ca-2021.crt`**. **No contiene
clave privada**: es material público, y omitirlo rompe el despliegue.

```
DB_SSL_CA_FILE=certs/prod-ca-2021.crt
```

## Configuración

```dotenv
DATABASE_URL=postgresql://postgres.<project-ref>:<clave>@aws-<n>-<region>.pooler.supabase.com:6543/postgres
DB_SSL_MODE=require
DB_SSL_CA_FILE=certs/prod-ca-2021.crt
DB_POOL_MAX=5
MIGRATION_TARGET=supabase
MIGRATION_CONFIRM_HOST=aws-<n>-<region>.pooler.supabase.com
```

- Percent-encodear la contraseña en la URL (`@` → `%40`, `:` → `%3A`).
- `MIGRATION_CONFIRM_HOST` debe ser **igual al hostname** de `DATABASE_URL`.
- El ejecutor rechaza hosts que no terminen en `supabase.com` y rechaza TLS desactivado.
- Historial de migraciones en `schema_migrations`. **No se usa el CLI de Supabase.**

## Desviaciones y deudas conocidas

> [!danger] Pendientes antes de usar producción en serio
> 1. **Rotar la contraseña de Supabase** usada en la prueba del 2026-10-18. Quedó expuesta
>    en el chat.
> 2. **Renombrar los modos SSL.** `DB_SSL_MODE=require` activa `rejectUnauthorized: true`
>    sin CA, es decir se comporta como **`verify-ca`** del estándar, no como `require`.
>    Es una desviación de nomenclatura que confunde.

Observación: `pg_stat_ssl` informa `ssl: false` en el tramo pooler→Postgres. Eso es
configuración del lado de Supabase (SSL Enforcement), **no** del backend.

## Checklist de despliegue a Supabase

1. Proyecto Supabase **exclusivo de MiBeca**. La base Render de otro proyecto queda fuera
   de alcance.
2. `DATABASE_URL` del pooler en modo transacción, con el certificado configurado.
3. Migraciones con `MIGRATION_TARGET=supabase` en un paso controlado, sin carreras entre
   instancias.
4. Comprobar `GET /ready`.
5. Publicar la URL pública de la API como variable de build del frontend y recompilar (el
   frontend incrusta `VITE_API_URL` en el bundle).
6. Probar el flujo completo con los tres roles.
7. Preparar respaldo y rollback de código y migraciones.

## Render como alternativa

`MIGRATION_TARGET=render` existe para ese destino. El CD del backend despliega en Render
vía `RENDER_DEPLOY_HOOK_URL`. Ver [[CI y CD]].

## Lo que NO está verificado

- Nada ejecutado contra un proyecto Supabase **recién creado** (la prueba histórica se hizo
  sobre uno existente y luego se expuso su contraseña).
- Cookies entre el dominio de Firebase y el de la API.
- Ningún despliegue de producción observado.

Relacionado: [[Migraciones]] · [[Entorno local]] · [[Estado actual]]
