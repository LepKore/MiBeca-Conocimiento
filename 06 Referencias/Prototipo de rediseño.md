---
tags: [mibeca, referencia, prototipo]
---

# Prototipo de rediseño

Carpeta: `base nuevo diseno frontend/`

> [!danger] Esto NO es la aplicación
> Es un prototipo visual generado con **Google AI Studio**. No está conectado a la API, no
> tiene autenticación real y sus datos son inventados. **No confundirlo con
> `gestion-de-trabajo-becario-web/`**, que sí es el producto.

## Qué es

Una maqueta en **TypeScript + React** que explora un rediseño completo de la interfaz:

| Señal | Valor |
|---|---|
| `package.json` name | `react-example` |
| `metadata.json` name | "UPB MiBeca - Gestión de Trabajo Becario" |
| Origen | AI Studio app `d5e16206-…` |
| Dependencias extra | `@google/genai` 2.4, `motion` 12, `lucide-react`, `express` 4 |
| Datos | `src/data/mockData.ts` (30 KB de datos falsos) |
| Componentes | `student/`, `supervisor/`, `admin/`, `common/`, `modals/` |
| Scripts | `dev` en **puerto 3000**, `build`, `lint` = `tsc --noEmit` |

## Diferencias con el producto real

| Prototipo | Producto (`gestion-de-trabajo-becario-web`) |
|---|---|
| TypeScript | JavaScript |
| Datos mock en el cliente | API real con sesión y cookies |
| Sin roles ni permisos reales | `ProtectedRoute` + autorización en backend |
| Puerto 3000 (choca con la API) | 5173 con proxy a la API |
| `@google/genai` | ninguna dependencia de IA |

## Cómo usarlo (si hay que revisarlo)

```bash
cd "base nuevo diseno frontend"
npm install
npm run dev        # http://localhost:3000
```

Requiere `GEMINI_API_KEY` en `.env.local` según su README, aunque las pantallas de mock no
la necesiten. **Ojo**: el puerto 3000 es el mismo que usa la API del backend; no correr
ambos a la vez sin cambiarlo.

## Regla

> Lo que se vea en el prototipo **no es una regla de negocio**. Las reglas viven en
> [[Roles y permisos]], [[Trabajos y postulaciones]] y [[Cierre, asistencia y horas]], y se
> validan contra el backend.

Relacionado: [[Arquitectura general]] · [[Visión y alcance]]
