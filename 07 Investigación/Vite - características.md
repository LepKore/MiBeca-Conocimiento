---
tags: [mibeca, investigacion, frontend]
---

# Vite — características

**Fuente:** <https://vite.dev/guide/features.html> · **Consultado:** 2026-10-29.
Documentación de **Vite 8.3.4**, la misma versión que tiene el proyecto
(`gestion-de-trabajo-becario-web` → `vite@8.3.4`, `vitest@5.0.5`).

Qué hace Vite por nosotros y qué hay que configurar aparte. Sirve para no activar cosas que ya son
automáticas y para saber dónde se rompe la ilusión.

## Lo que ya viene sin configurar

| Capacidad | Detalle |
|---|---|
| **Pre-bundling de dependencias** | Convierte CommonJS/UMD a ESM y reescribe `import 'my-dep'` a URLs reales. Lo hace con Rolldown. Cachea fuerte por HTTP |
| **HMR** | API sobre ESM nativo. Con `@vitejs/plugin-react` hay Fast Refresh; el estado no se pierde al editar |
| **TypeScript** | Importa `.ts`/`.tsx` directo, **solo transpila, no verifica tipos** |
| **JSX** | `.jsx`/`.tsx` listos, vía Oxc Transformer |
| **CSS** | `import './x.css'` inyecta con HMR; `@import` se resuelve con `postcss-import`; `url()` se rebasa solo |
| **CSS Modules** | Todo archivo `*.module.css` devuelve un objeto de clases |
| **Preprocesadores** | `.scss`, `.sass`, `.less`, `.styl` funcionan si instalas el preprocesador (`sass-embedded`, `less`, `stylus`). No hay plugin de Vite |
| **JSON** | Importa el objeto entero o campos con nombre (mejor tree-shaking) |
| **Assets** | `?url` (URL), `?raw` (string), `?worker` / `?worker&inline`, `?inline` (CSS sin inyectar) |
| **HTML como entrada** | `index.html`, `about.html`, `blog/index.html` se sirven por ruta. SPA y MPA sin config extra |
| **Web Workers / WASM** | `new Worker(new URL('./w.js', import.meta.url))` es la forma recomendada; `.wasm` directo o con `?init` |
| **Optimizaciones de build** | Code splitting de CSS, `<link rel="modulepreload">` automático, precarga de chunks comunes para eliminar roundtrips |

## Lo que NO hace (y hay que hacer aparte)

**Verificación de tipos.** Es la trampa principal: `vite build` puede pasar con errores de TypeScript.
La doc lo dice explícito y propone:

```bash
tsc --noEmit          # en el build de producción
tsc --noEmit --watch  # durante desarrollo, en otro proceso
```

Motivo declarado: transpilar es por archivo; verificar tipos necesita el grafo completo, y meterlo en
el pipeline destruiría la velocidad. Por eso también recomienda sacar ESLint del transform.

Otras cosas que exigen decisión:

- **`"isolatedModules": true`** obligatorio en `tsconfig.json` si hay TS: el transformador no tiene
  información de tipos y no soporta `const enum` ni imports implícitos de solo-tipo.
- **`import type` / `export type`** para evitar que un tipo termine empaquetado como valor.
- **`paths` del tsconfig** no se respetan salvo que actives `resolve.tsconfigPaths: true`, con costo
  de rendimiento.
- **`build.target`** ignora el `target` del tsconfig.
- **Rutas dinámicas** (`import(\`./dir/${file}.js\`)`) solo funcionan un nivel, con extensión y sin
  variables en el nombre. Para más, `import.meta.glob`.
- **CSP:** si se despliega con política estricta hay que usar `html.cspNonce` y decidir sobre
  `data:` en `img-src`/`font-src` (nunca en `script-src`).

## Qué nos sirve

1. **El proyecto ya está bien.** Vite 8.3.4 sin configuración especial es exactamente el caso de uso
   que la doc describe: SPA React, CSS y assets. No hay nada que migrar ni activar.
2. **`tsc --noEmit` en CI es lo que falta, si se adopta TS.** Hoy el frontend es `.jsx`/`.js` con
   `typescript` instalado. Si en algún momento se pasan archivos a `.ts`, el build **no** los
   valida: hay que añadir el paso a mano o los errores pasan.
3. **`import.meta.glob`** es la forma barata de cargar un conjunto de módulos si alguna vez hay
   páginas o rutas por carpeta, sin escribir un registro manual.
4. **`?raw` y `?url`** evitan tener que montar un servidor de estáticos para un archivo suelto
   (por ejemplo, una plantilla o un CSV de ejemplo).

## Límite

- La página describe **la herramienta**, no a MiBeca: nada de esto se probó contra el proyecto. El
  número de versión sí se verificó en `package-lock.json` (`8.3.4`).
- Las notas de TypeScript solo aplican si el proyecto migra; hoy `tsconfig` existe pero el código es
  JavaScript.
- El proyecto usa además `vitest@5.0.5`, que no se documenta en esta página.

Relacionado: [[Arquitectura general]] · [[Entorno local]] · [[Pruebas]] · [[00 Índice]]
