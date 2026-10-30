---
tags: [mibeca, proceso, reparto]
---

# Reparto de tareas

**Regla del equipo, confirmada el 2026-10-29.** Gobierna cómo se convierte un spec en
trabajo para tres personas.

## Principio

Un spec se divide en **tres bloques consecutivos**. **Nadie trabaja en paralelo**: un
bloque termina, se publica, y el siguiente empieza. Así nunca hay dos personas tocando los
mismos archivos.

## Cómo se calculan los bloques

Con **N** tareas y **3** integrantes:

```
tamaño de bloque = ⌈N / 3⌉
bloque 1 = tamaño
bloque 2 = tamaño
bloque 3 = N − 2 × tamaño      ← absorbe el sobrante
```

| Tareas | Reparto |
|---|---|
| 21 | 7 · 7 · 7 |
| 20 | 7 · 7 · **6** |
| 19 | 7 · 7 · **5** |
| 10 | 4 · 4 · **2** |
| 3 | 1 · 1 · 1 |

El último bloque es el que puede quedar más corto. **Nunca** se compensa dejando a alguien
sin bloque.

## Protocolo de relevo

```
Integrante N                          Integrante N+1
────────────────────────────────────────────────────────────────
1. Implementa sus tareas
2. Verifica cobertura total  ──►
3. CI en verde (lint+tests+build)
4. Push a la rama
                                    5. Pull del repositorio
                                    6. Revisa QUÉ tareas están
                                       hechas y cuáles faltan
                                    7. Recién entonces empieza su bloque
```

Puntos que no se saltan:

- **La CI debe estar en verde antes del push.** No se publica rojo.
- **La cobertura total del proyecto es obligatoria** (ver más abajo).
- El integrante N+1 **no confía en el mensaje de relevo**: hace `pull` y **verifica en el
  código** qué tareas quedaron cumplidas y cuáles pendientes. Si algo no coincide con lo
  que dijo el anterior, para y avisa.

> [!warning] Qué NO es este protocolo
> No es "cada uno hace su parte y se juntan al final". Es una **cadena**: mientras un
> bloque no está publicado, el siguiente no existe. Eso es lo que evita el solapamiento.

## Numeración de tareas

Las tareas se numeran **globalmente y de forma estable** dentro del spec: `T01`, `T02`, …
`T21`. El número no cambia al repartir. Así una nota de dependencia puede decir
"`T12`" y todos saben de qué se habla.

Cada bloque declara su rango: **bloque 1 = `T01–T07`**, bloque 2 = `T08–T14`, bloque 3 =
`T15–T21`.

## Orden entre repos: backend primero

Cuando el spec toca **frontend y backend**, el orden es:

1. **Backend**: contrato (`openapi.yaml`), migración, servicio, endpoint, pruebas.
2. **Frontend**: consume el contrato ya existente.

Nadie programa interfaz contra un contrato que todavía no existe.

### Excepción: cuando el bloqueo va al revés

Si el backend **no puede avanzar** hasta que cierta tarea de frontend esté terminada (o al
revés), **la tarea debe llevar la nota explícita**:

```markdown
> [!warning] Dependencia cruzada
> No avanzar hasta que `T09` de frontend (repo `gestion-de-trabajo-becario-web`) esté
> terminada y publicada.
```

Reglas de esa nota:

- Va **en la propia tarea**, no en un documento aparte.
- Nombra **la tarea exacta** y **el repo**.
- Aplica en **ambas direcciones**: backend→frontend y frontend→backend.
- **No elimina** las dependencias internas del área: una tarea de frontend puede depender
  de otra tarea de frontend, y eso se sigue anotando igual.

Nadie "avanza así por así" sobre un contrato que aún no existe.

## Cobertura: la regla que todavía no se cumple

El equipo exige **cobertura de pruebas total** antes de publicar un bloque. **Hoy no está
implementado y no se puede verificar.**

Estado real verificado el 2026-10-29:

| Qué | Estado |
|---|---|
| `vitest.config.js` | **Sin** bloque `coverage` |
| `scripts` de `package.json` (web) | **Sin** `--coverage` |
| CI del frontend | **Sin** paso de cobertura |
| Backend (`node:test`) | **Sin** medición de cobertura |

`@vitest/coverage-v8` aparece en `package-lock.json` solo como peer opcional de Vitest, no
como dependencia instalada ni configurada.

> [!danger] Consecuencia
> Mientras esto siga así, "cobertura total" es una intención, no un gate. **No se puede
> declarar cumplida.** Configurarla es una tarea pendiente y debe ser una de las primeras
> del próximo spec: sin eso, la regla de relevo tiene un agujero.

## De dónde salen las tareas

De un **spec**, no de un plan. `PLAN_MVP_MIBECA_V2.md` está en desuso y su contenido se
está migrando a `specs/`; el plan v1 describe lo ya implementado pero no genera trabajo
nuevo.

Eso resuelve la contradicción que tenía este documento: el §7 del v2 repartía por
**propiedad de módulo A/B/C en paralelo**, y eso **ya no aplica**. La regla es la de esta
nota: bloques consecutivos por spec.

Lo que **sí** se conserva de aquella idea y sigue siendo obligatorio: **propiedad
exclusiva de archivos** por bloque, y **un único dueño** de `openapi.yaml` y de cada
migración. Un bloque no edita lo que otro declaró suyo.

Relacionado: [[Ramas y PR]] · [[SDD con Spec Kit]] · [[Pruebas]] · [[Equipo]]
