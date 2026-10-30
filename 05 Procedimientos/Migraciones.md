---
tags: [mibeca, procedimiento, base-de-datos]
---

# Migraciones

Fuente: `scripts/migrate.js`, `migrations/*.sql`, `docs/AI_CONTEXT.md` (backend §4).

## Regla base

**Las migraciones no corren al arrancar la API.** Siempre son un paso explícito y
separado. `npm start` no migra nada.

## Comandos

```bash
# Desarrollo: lee .env
npm run db:migrate

# Despliegue: toma las variables del entorno (no de .env)
npm run db:migrate:env
```

El historial vive en la tabla **`schema_migrations`**. **No se usa el CLI de Supabase.**

## `MIGRATION_TARGET`

| Valor | Requisitos |
|---|---|
| `local` | Ninguno especial |
| `supabase` | Host **terminado en `supabase.com`**, `MIGRATION_CONFIRM_HOST` igual al hostname, `DB_SSL_MODE` distinto de `disable` |
| `render` | Destino alternativo |

El ejecutor imprime **una línea con el destino, sin contraseña**. Si alguna guarda falla,
rechaza antes de conectar.

Cinco guardas verificadas: Supabase con `localhost`, host que no es Supabase,
`MIGRATION_CONFIRM_HOST` distinto, `DB_SSL_MODE=disable`, target inválido.

## Convenciones de archivo

- Nombre: **`NNNN_descripcion.sql`** (regla `MIG-02`).
- **Aditivas** por defecto: añadir, no reescribir.
- `DROP TABLE`/`DROP COLUMN`/`TRUNCATE` requieren **justificación en la misma línea o la
  anterior** (regla `MIG-01`).
- Sin `ON DELETE CASCADE` sobre historial: se usa `RESTRICT`.
- Sin interpolar datos de petición en SQL (`DB-01`, bloqueante).

## Antes de aplicar (obligatorio)

1. **Verificar el destino**: ¿a qué `DATABASE_URL` apunta? ¿Es local, Supabase, Render?
2. **Leer el SQL** completo.
3. Si es remoto: `MIGRATION_CONFIRM_HOST` correcto y `DB_SSL_MODE` no deshabilitado.
4. Si es de producción: revisión de otra persona y procedimiento explícito.

## Después de aplicar

```bash
npm test                          # contra TEST_DATABASE_URL
curl http://127.0.0.1:3000/ready
```

Cada integrante ejecuta las migraciones nuevas en **su** base local al integrar cambios.

> [!warning] "Cambiar `DATABASE_URL`" no migra nada
> Cambiar la URL no copia datos ni crea tablas. La migración es un paso aparte y la
> transferencia de datos es una decisión todavía separada.

## Rollback

**No hay rollback automático** de migraciones destructivas. Antes de una migración
arriesgada: respaldo y plan de recuperación escrito.

Relacionado: [[Modelo de datos]] · [[Supabase]] · [[Guardas y auditor de código]]
