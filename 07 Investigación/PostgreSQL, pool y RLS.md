---
tags: [mibeca, investigacion, arquitectura]
---

# PostgreSQL, pool y seguridad por filas

**Fuentes:** [Postgres pooling in Node](https://medium.com/@1nick1patel1/postgres-pooling-in-node-the-surprise-free-guide-7158d8d7cd62) · [Supabase RLS](https://supabase.com/docs/guides/database/postgres/row-level-security) · [PostgreSQL SQL](https://www.postgresql.org/docs/current/sql.html) · **Consultado:** 2026-10-29.

## Estado de las fuentes

La guía de Medium devolvió **403** y no fue posible extraerla. Se registra como inaccesible y no
se atribuyen recomendaciones a ese artículo. El contenido de esta nota proviene de la guía oficial
de Supabase y de PostgreSQL.

## Pool de conexiones en MiBeca

El backend usa `pg` y un `Pool`, no una conexión nueva por cada petición. La configuración actual
usa `DB_POOL_MAX` y, en Supavisor modo transacción, el pooler está en el puerto 6543. Ver
[[Supabase]] y [[Arquitectura general]].

Reglas prácticas que quedan confirmadas:

- reutilizar el pool compartido;
- liberar el cliente obtenido con `pool.connect()` en un `finally`;
- usar consultas parametrizadas;
- no usar sentencias preparadas con nombre detrás de Supavisor en modo transacción;
- mantener el tamaño del pool pequeño y medido, no igualarlo automáticamente al número de usuarios.

No se cambia `src/db.js`: estas reglas ya están reflejadas en el código y la guía de Medium no fue
accesible para contrastar detalles adicionales.

## RLS en Supabase

PostgreSQL Row Level Security (RLS) añade autorización por fila dentro de la base de datos. No
reemplaza los permisos SQL: primero el rol necesita `GRANT` y después la policy decide qué filas y
qué valores puede tocar.

Para cada tabla expuesta por la API:

1. activar RLS;
2. revocar privilegios de `anon` y `authenticated`;
3. conceder solo `SELECT`, `INSERT`, `UPDATE` o `DELETE` necesarios;
4. crear una policy separada por operación;
5. escribir una prueba pgTAP de permitir y denegar;
6. ejecutar `supabase test db`.

En Supabase, `anon` representa una solicitud no autenticada y `authenticated` una autenticada.
`service_role` puede saltarse RLS y debe permanecer solo en el servidor; nunca debe exponerse al
navegador.

### Forma de las policies

- `USING` filtra las filas existentes para `SELECT`, `UPDATE` y `DELETE`.
- `WITH CHECK` valida la fila nueva o resultante en `INSERT` y `UPDATE`.
- Un `UPDATE` necesita también policy de `SELECT`.
- Especificar siempre `TO authenticated` o `TO anon`; no dejar la policy abierta a todos los roles.
- `auth.uid()` devuelve el usuario actual; si no hay sesión devuelve `null`.
- Indexar las columnas usadas por las policies (`user_id`, por ejemplo).
- Las vistas pueden saltarse RLS por defecto; en PostgreSQL 15+ usar `security_invoker = true` o
  protegerlas mediante schema y permisos.
- Las funciones `security definer` deben fijar `search_path = ''`, calificar nombres con schema y
  no vivir en un schema expuesto.

Una policy no probada no es una policy confiable. En tests, una denegación por privilegio o
`WITH CHECK` debe comprobar el error `42501`; una fila filtrada por `USING` puede devolver cero
filas sin error, por lo que conviene usar `RETURNING` y verificar que la fila original no cambió.

## Qué significa para MiBeca

MiBeca actualmente protege el acceso en Express y usa PostgreSQL detrás de la API; no se debe
activar RLS automáticamente sin diseñar antes la identidad que llegará a la base de datos y probar
las migraciones. La referencia sí deja una regla para cualquier futura exposición directa de
Supabase: **grants mínimos + RLS + tests por tabla**, no solo policies.

El backend actual tiene tres roles de aplicación (`ADMIN`, `ENCARGADO`, `BECARIO`), pero eso no
demuestra que sean roles PostgreSQL ni que `auth.uid()` exista en sus conexiones. No convertirlos
en roles RLS sin una spec y una prueba de integración.

## SQL que conviene ubicar en la documentación oficial

La documentación actual de PostgreSQL 18.6 organiza SQL en sintaxis, DDL, DML, consultas, tipos,
funciones, índices, control de concurrencia y rendimiento. Para MiBeca son especialmente útiles:

- `CREATE TABLE`, restricciones y claves foráneas;
- `ALTER TABLE`, privilegios y policies de seguridad por filas;
- `INSERT`, `UPDATE`, `DELETE` y `RETURNING`;
- transacciones y control de concurrencia;
- índices y `EXPLAIN`.

No es necesario copiar el manual al vault: se conserva el enlace y solo se registran aquí los
criterios que afectan al proyecto.

## Límites

- La fuente de Medium no fue leída: 403.
- RLS es una referencia para un posible diseño futuro, no una afirmación de que MiBeca ya lo use.
- La documentación de PostgreSQL es amplia; esta nota cubre solo pool, permisos, RLS, pruebas e
  índices relevantes para el proyecto.

Relacionado: [[Supabase]] · [[Arquitectura general]] · [[Modelo de datos]] · [[Pruebas]] · [[00 Índice]] · [[enlaces web]]
