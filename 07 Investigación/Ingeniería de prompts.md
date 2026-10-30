---
tags: [mibeca, investigacion, agentes]
---

# Ingeniería de prompts

**Fuentes:** <https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview> ·
<https://developers.openai.com/api/docs/guides/prompt-engineering?api-mode=responses> ·
**Consultado:** 2026-10-29.

Las dos guías oficiales, resumidas hacia un caso concreto: **el equipo usa agentes de IA para
implementar specs**. No es teoría de prompting para productos con LLM; es cómo pedirle trabajo a un
agente sin que invente, sin que toque de más y sin que pierda el hilo entre sesiones.

## Antes de escribir el prompt (las dos coinciden)

Claude lo pone como requisito previo; OpenAI lo pone como control de calidad:

1. **Criterio de éxito explícito.** Si no sabes cómo se ve "bien hecho", el modelo tampoco.
2. **Una forma de comprobarlo**, no una opinión. En MiBeca eso ya existe: los tests.
3. **Un borrador** del prompt, que se mejora midiendo, no reescribiendo de cero.

Y una advertencia que conviene tener presente: **no todo se arregla con prompting.** A veces el
problema es el modelo o la tarea, no las palabras. Si un agente falla tres veces con el mismo
enunciado, cambiar la redacción no es la solución.

## Las técnicas que sí sirven

### Claridad y contexto

| Técnica | Cómo se ve |
|---|---|
| **Claridad directa** | Prueba de Claude: dale el prompt a un compañero con poco contexto. Si se confunde, el modelo también. Los prompts cortos asumen contexto que no está |
| **Decir el porqué** | "Nunca uses…" funciona peor que "…porque la respuesta se lee en voz alta". El modelo generaliza desde la razón, no desde la prohibición |
| **Formato de salida pedido** | Si quieres un diff, pide un diff; si quieres un archivo, pide un archivo. No lo dejes implícito |
| **Pasos numerados** | Solo cuando el orden importa. Si el orden no importa, la lista confunde |

### Estructura

- **XML tags** para separar contenido del instructivo: `<documento>`, `<reglas>`, `<ejemplo>`.
  Nombres descriptivos y consistentes; anidar para jerarquía.
- **Ejemplos**: entre 3 y 5, diversos y representativos, envueltos en `<example>`.
- **Rol**: una frase en el system prompt basta. No hace falta un párrafo de personalidad.
- **Contexto largo** (20k+ tokens): el **material arriba**, la **pregunta al final**. Claude reporta
  hasta 30 % de mejora en sus pruebas con ese orden. Primero citar, después responder.
- **OpenAI ordena el mensaje de sistema** como Identidad → Instrucciones → Ejemplos → Contexto.
  Misma idea: el contexto cerca del final.
- **Caché de prompt**: lo que se reutiliza va al principio, tanto del texto como del cuerpo JSON.
  Si el contenido estable está al final, se paga cada vez.

### Trabajo con herramientas

- **Pedir la acción, no la sugerencia**: "cambia esta función" produce un cambio; "sugiere cambios"
  produce un comentario.
- **Control de impulso**: tanto Claude (`<default_to_action>` / `<do_not_act_before_instructions>`)
  como OpenAI advierten que los modelos nuevos actúan de más. Suavizar "CRITICAL: you MUST" a "usa
  esta herramienta cuando…" reduce el disparo innecesario.
- **Paralelizar** lecturas independientes; no paralelizar escrituras.
- **Prohibir marcadores de posición**: "nunca inventes parámetros faltantes".
- **Verificación real**: OpenAI avisa que una herramienta de edición puede responder "Done" aunque
  haya fallado. La única prueba es correr el test o leer el archivo.

### Tareas largas

- **Decirle la duración**: avisarle que el contexto se va a compactar evita que "termine" antes de
  tiempo.
- **Estado en archivos, no en la cabeza**: lista de pendientes en un archivo, `tests.json` con
  resultados, git como registro. Entre ventanas de contexto, el marco (tests, scripts, estructura)
  va en la primera y luego se itera.
- **"Es inaceptable borrar o editar tests"** — textual en la guía de Claude. Es la misma regla que
  el equipo ya tiene por proceso ([[Pruebas]]).
- **Confirmar antes de lo irreversible**: borrar archivos o ramas, `git push --force`,
  `git reset --hard`, reescribir commits publicados. Las dos guías piden un paso de confirmación.

## Lo que cambió y rompe prompts viejos

- **Prefill eliminado**: rellenar el turno del asistente devuelve **error 400** en Claude 4.6+.
  Reemplazo: salidas estructuradas o mover el texto al mensaje del usuario.
- **`budget_tokens` de thinking**: deprecado, error 400 en 4.7+. Se controla con `effort`.
- **Objetos de prompt reutilizables de OpenAI**: en retirada (dejar de recomendarse desde 2026-06-03,
  apagado de `v1/prompts` el 2026-11-30). La guía ahora dice: **versionar los prompts en el código**,
  en un módulo pequeño junto a la funcionalidad, con argumentos tipados, fixtures, tests y feature
  flags.
- **No asumir la forma de la respuesta**: en la API de OpenAI `output` trae varios ítems (llamadas a
  herramientas, metadatos de razonamiento). `output[0].content[0].text` no es seguro; usar
  `output_text`.

## Qué nos sirve

1. **El vault ya hace lo correcto sin saberlo.** Los prompts efectivos de este equipo no son
   mensajes largos: son `AGENTS.md`, `docs/automation.md`, la constitución y la spec. Ambas guías
   llegan a la misma conclusión por otro camino: **reglas versionadas > contexto pegado en el chat**.
   Ver [[Estructura de proyecto Node.js]].
2. **La spec es el prompt.** Un spec con criterios de aceptación verificables es exactamente el
   "criterio de éxito + forma de comprobarlo" que las dos guías piden antes de empezar. Cuando el
   agente falla, revisar la spec antes que la redacción del mensaje.
3. **Pedir la acción, no la sugerencia.** En este equipo el agente implementa; si se le pide "revisa
   y sugiere", devuelve prosa y el trabajo queda a medias.
4. **Prohibir tocar los tests** ya está en la constitución y ahora tiene respaldo externo explícito.
5. **Avisar de la compactación** cuando la tarea es larga evita que un bloque de 7 tareas se "termine"
   a la tercera.
6. **Confirmar lo destructivo** coincide con las reglas de ramas y PR del equipo.
7. **Si algún día el proyecto llama a un LLM desde el código**, el prompt va en un módulo del
   proyecto, versionado y con tests — no incrustado en un handler.

## Qué NO aplicamos

- **Elegir modelo por benchmark**: irrelevante mientras el agente sea una herramienta de desarrollo,
  no parte del producto.
- **Frontend generado** (Tailwind, shadcn/ui, Radix, Lucide, Motion como recomendación de OpenAI):
  MiBeca ya tiene su stack y su diseño. No se adopta nada por esta guía.
- **Evals automatizadas**: útiles cuando hay un producto con LLM. Aquí la "eval" son los tests del
  repo.

## Límite

- Ambas guías están escritas para **quien construye un producto con la API**, no para quien usa un
  agente de código. Las traduje al segundo caso; esa traducción es mía, no de las fuentes.
- La página de Claude cambia con cada modelo (cubre de Fable 5.1 a Haiku 4.5). Lo que aquí está
  puede quedar obsoleto en semanas: la fuente viva es `claude-prompting-best-practices`.
- De la guía de Claude solo se leyó hasta la línea ~888 de 1172.
- No se probó ninguna técnica; no hay medición propia.

Relacionado: [[SDD con Spec Kit]] · [[Agentes y skills]] · [[Guardas y auditor de código]] ·
[[Estructura de proyecto Node.js]] · [[Pruebas]] · [[00 Índice]]
