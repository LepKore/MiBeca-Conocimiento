---
tags: [mibeca, investigacion, estilo]
---

# Estilo de JavaScript (Airbnb)

**Fuente:** <https://github.com/airbnb/javascript> · **Consultado:** 2026-10-29.

La guía de estilo de JavaScript más citada del ecosistema. **MiBeca no la usa** y esta nota explica
por qué no hace falta adoptarla entera, qué partes ya se cumplen y cuáles serían un cambio real.

## Qué es

Guía de estilo, no librería. Se consume como configuración de ESLint: `eslint-config-airbnb` (con
React) o `eslint-config-airbnb-base` (sin React). Supone Babel con `babel-preset-airbnb` y polyfills
vía `airbnb-browser-shims`.

Contenido por secciones: tipos, referencias, objetos, arreglos, desestructuración, strings, funciones,
arrow functions, clases y constructores, módulos, iteradores y generadores, propiedades, variables,
hoisting, comparación e igualdad, bloques, control de flujo, comentarios, espacios, comas, punto y
coma, coerción de tipos, convenciones de nombres, accesores, eventos, jQuery, compatibilidad ES5,
estilos ES2015+, librería estándar, testing y performance.

Existe también para React, CSS-in-JavaScript y CSS/Sass. La guía de ES5 está **deprecada**.

## Las reglas que de verdad cambian código

Son pocas. El resto es formato, y el formato lo resuelve un formateador.

| Regla | Qué pide | Impacto en MiBeca |
|---|---|---|
| **`const` por defecto, `let` solo si se reasigna, nunca `var`** | Inmutabilidad por defecto | Ya se cumple. Cero trabajo |
| **Funciones flecha para callbacks anónimos** | Evita `this` dinámico | Ya se cumple de forma natural en el frontend |
| **`import`/`export`, no `require`** | ESM | Ya se cumple en ambos repos (ESM). Cero trabajo |
| **Desestructuración** en vez de acceso encadenado | Legibilidad | Cosmético |
| **`===` siempre** | Igualdad estricta | Ya se cumple |
| **Nombres: `camelCase` variables, `PascalCase` clases/componentes, `UPPER_SNAKE` constantes** | Consistencia | Ya se cumple en el frontend; el backend usa `snake_case` **en la base de datos**, correcto |
| **Una clase por archivo** | Legibilidad | Irrelevante: no se usan clases |
| **Nunca mutar parámetros** | Evita efectos colaterales | Ya se cumple |
| **Módulos: un export por archivo, preferido** | Claridad | MiBeca exporta varios nombres por archivo. **No se cambia** |
| **Iteradores: preferir `map`/`filter`/`reduce` sobre `for...of`** | Declarativo | Recomendación blanda; usamos `for` cuando hay que cortar antes |

Ninguna de estas es una decisión de arquitectura. Cuando aparecen, ya aparecían.

## Qué nos sirve

1. **Vocabulario común.** Si alguien dice "esto no pasa Airbnb", se entiende: nombres, `const`,
   igualdad estricta, funciones puras. No hace falta instalar nada para hablar el idioma.
2. **Argumento para no pelear por formato.** Las discusiones de estilo las gana la guía, no la
   persona con más antigüedad. Eso ahorra tiempo en revisión.
3. **Una referencia cuando no hay configuración.** El backend **no tiene ESLint** (verificado el
   2026-10-29: no hay `.eslintrc*` ni `eslint.config.js`). Su única validación automática es
   `npm run check` (`node scripts/check-syntax.js`) más los tests. Airbnb es el criterio informal
   que cubre ese hueco.
4. **El frontend ya tiene flat config** con `@eslint/js`, `react-hooks` y `react-refresh`. Ahí sí hay
   reglas reales, y son las que importan (hooks, no variables sin usar).

## Qué NO aplicamos

| Regla / decisión | Por qué no |
|---|---|
| **Instalar `eslint-config-airbnb`** | Arrastra Babel, `babel-preset-airbnb` y decenas de reglas. Tres juniors no necesitan esa configuración para escribir `const` |
| **Un export por archivo** | MiBeca agrupa por módulo (`lib/db.js`, `lib/errors.js` exportan varios nombres). Dividirlos fragmenta más de lo que aclara |
| **Preferir `forEach`/`for...of` sin `break`** | Los bucles con corte temprano son más legibles donde se usan |
| **Guía de React y CSS de Airbnb** | El frontend ya tiene Tailwind y su propio estilo; no hay conflicto que resolver |
| **Espacio, comas, punto y coma** | Lo resuelve el formateador del editor. No es tema de revisión |

Migrar al estilo Airbnb entero es una reescritura con cero funcionalidad nueva. Va contra el
principio 5 de la constitución (simplicidad) y contra la regla del equipo: el diff más corto que
resuelve el problema.

## Lo que sí vale la pena, aquí y ahora

El backend ya incorporó ESLint y `npm run lint`; la verificación local del bloque
T016–T030 pasó en `130d2c9`. La guía de Airbnb sigue siendo una referencia opcional,
no una configuración adoptada ni una razón para reescribir el proyecto.

## Límite

- Es una guía de **estilo**, no de arquitectura ni de corrección: cumplirla no hace el código
  correcto, solo consistente.
- No se leyó el README completo regla por regla; se revisaron el índice de secciones y las reglas
  con efecto real en este proyecto.
- **No se verificó qué reglas de Airbnb viola MiBeca hoy.** Para eso hace falta correr
  `eslint-config-airbnb` sobre el código, que es justo lo que esta nota recomienda no instalar.

Relacionado: [[Estructura de proyecto Node.js]] · [[SOLID en TypeScript]] · [[Agentes y skills]] ·
[[Pruebas]] · [[00 Índice]]
