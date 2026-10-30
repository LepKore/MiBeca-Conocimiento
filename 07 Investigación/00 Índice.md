---
tags: [mibeca, investigacion, indice]
---

# Investigación

Aquí van los hallazgos **reutilizables** de internet o del entorno: cómo funciona una
herramienta, un procedimiento que no está en los repos, una restricción que costó
descubrir.

La regla completa está en [[Cómo mantener este vault]].

**Toda fuente se archiva:** un enlace va a [[enlaces web]]; un PDF, DOCX, XLSX, CSV o similar va
como archivo en `conocimiento MiBeca/docs/` con su fila en ese mismo registro.

## Plantilla

```markdown
## <Título corto>

**Fuente:** <URL o `docs/archivo.pdf`> · **Consultado:** AAAA-MM-DD
**Por qué nos sirve:** una o dos frases concretas.

<Contenido útil. Comandos, no párrafos.>

**Límite:** qué NO cubre o cuándo deja de ser cierto.
```

## Entradas

| Tema | Dónde | Consultado | Para qué |
|---|---|---|---|
| **Ingeniería de prompts** (Claude + OpenAI) | [[Ingeniería de prompts]] | 2026-10-29 | Pedirle trabajo a un agente: acción no sugerencia, tests intocables, confirmar lo irreversible |
| **Estructura de proyecto Node.js** | [[Estructura de proyecto Node.js]] | 2026-10-29 | Por qué **no** migrar a `modules/`; qué reglas ya se cumplen; logging como deuda |
| **Vite** | [[Vite - características]] | 2026-10-29 | Qué viene sin configurar y qué no (`tsc --noEmit` no está incluido) |
| **SOLID en TypeScript** | [[SOLID en TypeScript]] | 2026-10-29 | Nombrar el diseño sin adoptar DI ni TypeScript |
| **Estilo de JavaScript (Airbnb)** | [[Estilo de JavaScript (Airbnb)]] | 2026-10-29 | Reglas con efecto real; el backend no tiene linter |
| **PostgreSQL, pool y RLS** | [[PostgreSQL, pool y RLS]] | 2026-10-29 | Pool compartido, grants mínimos, policies RLS y pruebas por tabla |
| **Asistencia biométrica en universidades** (Kakepoto 2022) | [[Asistencia biométrica universitaria (Kakepoto 2022)]] | 2026-10-29 | Qué efectos tiene verificar asistencia; respalda el diseño de cierre sin vigilancia. PDF en `docs/` |
| **Sistema web de becas EGPP** (UMSA, 2014) | [[Sistema web de becas EGPP]] | 2026-10-29 | Comparar convocatorias, postulantes, instituciones patrocinantes, reportes e historial. PDF en `docs/` |
| **Clockify** (plataforma similar, comercial) | [[Clockify]] | 2026-10-29 | Comparar decisiones de registro de horas, aprobación y reportes. Fuente en [[enlaces web]] |
| Spec Kit (SDD) | `docs/AI_CONTEXT.md` de ambos repos | 2026-10-26 | Ya documentado en [[SDD con Spec Kit]]: se enlaza, no se duplica |
| Vitest: workers y timeout en Windows | `vitest.config.js` (comentario) | — | `pool: 'threads'` + `maxWorkers: 4` evitan el timeout del pool |
| Certificado CA de Supabase | `certs/prod-ca-2021.crt` | 2026-10-18 | Sin `DB_SSL_CA_FILE` la conexión falla con `SELF_SIGNED_CERT_IN_CHAIN`. Ver [[Supabase]] |
| Supavisor no admite sentencias preparadas con nombre | `src/db.js` | — | Por eso ninguna consulta pasa `name` |

> Las filas de arriba son **conocimiento ya absorbido** en otra nota. Se dejan como índice
> para no volver a investigarlas; el detalle vive en la nota enlazada.

## Pendiente de investigar

- Cómo configurar **cobertura de pruebas** en Vitest y en `node:test` sin añadir
  complejidad, y qué umbral es realista para tres juniors. Bloquea la regla de
  [[Reparto de tareas]].
- Estrategia de **cookies entre dominios** (Firebase Hosting + API en otro dominio).
- Alternativa a Redocly si el registry de npm vuelve a ser inaccesible en CI.
