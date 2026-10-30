---
tags: [mibeca, investigacion, arquitectura]
---

# Estructura de proyecto Node.js

**Fuentes:** [[enlaces web]] · **Consultado:** 2026-10-29.

Tres referencias sobre cómo organizar un backend Node.js y una sobre convenciones para trabajar con
agentes. La conclusión práctica para MiBeca está al final: **no hay que reorganizar nada**, y esta
nota explica por qué.

## 1. Modular + por capas (devActivity)

Artículo de blog que resume una discusión de GitHub sobre "la mejor estructura de carpetas". Su
recomendación:

```
src/
├── modules/            # una carpeta por funcionalidad
│   ├── auth/
│   │   ├── auth.controller.js
│   │   ├── auth.service.js
│   │   ├── auth.repository.js
│   │   ├── auth.routes.js
│   │   └── auth.validation.js
│   └── blog/...
├── common/             # middleware, utilidades, errores compartidos
├── config/
├── loaders/            # arranque: express, base de datos
├── app.js
└── server.js
```

Reparto de responsabilidades dentro de cada módulo:

| Capa | Qué hace | Qué NO hace |
|---|---|---|
| Controller | Recibe HTTP, delega, responde. Fino | No tiene reglas de negocio |
| Service | Reglas de negocio del dominio | No habla SQL directamente |
| Repository | Acceso a datos (CRUD, queries) | No decide reglas |
| Model / schema | Forma del dato | No ejecuta lógica |
| Validation | Valida entrada antes del service | No formatea respuestas |

Argumentos que da a favor: agregar una funcionalidad = crear una carpeta; el trabajo de dos personas
choca menos porque cada módulo tiene dueño; cambiar de base de datos toca solo el repository.

También menciona **Clean/Hexagonal** (`core/`, `infrastructure/`, `interfaces/`) y lo descarta para
proyectos medianos por costo de arranque y complejidad.

Consejos operativos que da fuera de la estructura: configuración por variables de entorno, logging
real (Winston/Pino), capa de validación dedicada (Joi, Zod, express-validator), manejo centralizado
de errores y módulos lo más independientes posible entre sí.

## 2. Node.js Best Practices (goldbergyoni)

Repositorio de referencia, **102 prácticas** verificadas por la comunidad, actualizado para Node 22
(última actualización del README: enero 2024). Está dividido en secciones: `projectstructre`,
`errorhandling`, `codestylepractices`, `testingandquality`, `production`, `security`, `performance`,
`docker` y `drafts`.

De la sección de estructura, las dos reglas que importan:

1. **Estructurar por componentes de negocio, no por capa técnica global.** Un cambio de una
   funcionalidad debe tocar una carpeta, no cinco.
2. **Poner capas dentro de cada componente** y mantener Express en el borde: la capa web no debe
   contener reglas de negocio ni acceso a datos.

Es la misma tesis que el artículo de arriba, con más años de respaldo.

## 3. Arquitectura, convenciones y agentes (Peter Ciluzzi, 2026)

Experiencia de alguien que construyó 50+ apps full-stack para estudiar patrones en aislamiento.
Lo relevante no es su stack, es el método:

- **Un documento de convenciones vale más que prompts largos.** Lo que empezó como pegar contexto
  en cada mensaje terminó como un conjunto de reglas de 19 secciones, y un `CLAUDE.md` corto en la
  raíz que lo referencia. Cada sesión arranca con el mismo criterio, sin repetirlo.
- **Los planes van en documento aparte, no en el chat.** Al crecer la conversación, el contexto se
  rompía; sacar el plan a un archivo lo hizo más confiable para él *y* para la herramienta.
- **La repetición es lo que enseña.** Rehacer la misma app tres veces (funcional, con clases, con
  TypeScript) fue lo que convirtió "arquitectura por capas" de concepto abstracto en decisiones
  concretas.
- **Todo lo que está en su guía existe porque le dolió:** contratos de respuesta uniformes, manejo
  centralizado de errores con códigos compartidos entre back y front, una sola fuente de verdad para
  la configuración.

## Qué nos sirve

**Mi no necesita migrar a `modules/`.** Su estructura actual, verificada el 2026-10-29:

```
backend/src/  →  lib/  middleware/  routes/  services/
web/src/      →  api/  assets/  auth/  components/  config/  lib/  pages/  styles/
```

Es una estructura **por capa con archivos por funcionalidad**: `routes/trabajos.js`,
`services/trabajos.js`, `middleware/*`, `lib/*`. No es el `modules/` del artículo, pero consigue lo
que el artículo busca —cohesión por funcionalidad— sin una carpeta por entidad ni una migración de
por medio. Mover 30 archivos para cumplir una recomendación de blog es exactamente lo que el
principio 5 de las constituciones (simplicidad, YAGNI) prohíbe.

Lo que **sí** conviene sostener, porque ya se cumple y hay que no perderlo:

| Recomendación | Estado en MiBeca |
|---|---|
| Validación en el borde antes del service | Sí: `zod` en los handlers |
| Manejo centralizado de errores | Sí: `ApiError` + middleware de error |
| Configuración por variables de entorno | Sí: `CORS_ORIGIN`, `DB_*`, `.env` |
| Express en el borde, sin reglas de negocio | Sí: `routes/` delega en `services/` |
| Módulos independientes, poco acoplamiento | Parcial: los services comparten `lib/db.js` |
| Logging estructurado (Pino/Winston) | **No**. No hay logger; se usa `console`/respuestas |
| Un dueño por área de archivos | Sí: ver [[Reparto de tareas]] |

El logging es la única brecha real que señalan las tres fuentes. No es urgente: con tres juniors y
sin alertas, un logger estructurado agrega configuración antes de aportar. Anotado como deuda, no
como tarea.

**Sobre el documento de convenciones:** MiBeca ya tiene el equivalente (`AGENTS.md`,
`docs/automation.md`, `.specify/memory/constitution.md` por repo). La lección aplicable es
mantenerlos cortos y que apunten a las reglas, no duplicarlas.

## Qué NO aplicamos

- **Migrar a `modules/` o a Clean Architecture.** El propio artículo dice que Clean cuesta más al
  arrancar; para tres juniors, el costo no se paga.
- **Repositorios, ORM, modelos por entidad.** MiBeca usa SQL con `pg` a propósito.
- **Monorepo de 50 apps, PM2, asignación de puertos por categoría.** Contexto del autor, no el
  nuestro.

## Límite

- El artículo de devActivity es **divulgación**, no evidencia: cita una discusión de GitHub y
  opiniones, sin mediciones. Su propio negocio es analítica de productividad, así que habla de
  estructura en términos de desempeño del equipo, no de corrección técnica.
- De Node Best Practices **no se verificó práctica por práctica**; se leyó el índice de secciones y
  las reglas de estructura. Antes de citar un número concreto, abrir la sección correspondiente.
- La nota sobre convenciones es experiencia personal de un desarrollador, no un estudio.

Relacionado: [[Arquitectura general]] · [[SOLID en TypeScript]] · [[Estilo de JavaScript (Airbnb)]] ·
[[Ingeniería de prompts]] · [[Reparto de tareas]] · [[00 Índice]]
