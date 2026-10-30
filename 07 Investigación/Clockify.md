---
tags: [mibeca, investigacion, referencia-externa]
---

# Clockify

**Fuente:** `https://dev-docs.marketplace.cake.com/clockify/learn/introduction.html` (enlace dado por el
equipo) · espejo accesible: `https://docs.clockify.me` — misma documentación de la API v1.
**Consultado:** 2026-10-29.
**Por qué nos sirve:** es una plataforma de registro de horas con aprobación y reportes, o sea el
mismo problema de fondo que MiBeca, pero resuelto como producto comercial multi-empresa. Sirve
para comparar decisiones, no para copiar código.

> [!warning] El enlace original no respondió
> El dominio `dev-docs.marketplace.cake.com` devolvió contenido vacío en dos intentos
> (2026-10-29). El contenido de abajo viene del espejo oficial `docs.clockify.me`, que es la
> misma especificación. Registrado en [[enlaces web]].

## Qué es

SaaS de seguimiento de tiempo. La unidad de todo es el **workspace** (una empresa). Dentro del
workspace viven usuarios, clientes, proyectos, tareas y **time entries**. La API es REST, versión
`v1`, y todo se factura por planes (`FREE`, `TRIAL`, `BASIC`, `STANDARD`, `PRO`, `ENTERPRISE`).

## Modelo de datos (lo que expone la API)

| Entidad | Para qué |
|---|---|
| **Workspace** | Contenedor raíz. Tiene tarifas, moneda, features según plan |
| **User** | Miembro. Estado `PENDING`/`ACTIVE`/`DECLINED`/`INACTIVE`. Roles `OWNER`, `WORKSPACE_ADMIN`, `TEAM_MANAGER`, `PROJECT_MANAGER` |
| **Client / Project / Task / Tag** | Jerarquía de a qué se imputan las horas |
| **Time entry** | El registro de horas: inicio, fin, duración, proyecto, tarea, tags, facturable |
| **Approval** | Solicitud de aprobación de entradas. Estados: pendiente, aprobada, rechazada, retirada |
| **Time off / Policy / Balance** | Vacaciones y saldos por política |
| **Custom fields** | Campos propios sobre usuarios y proyectos |
| **Expense / Invoice / Holiday** | Gastos, facturación, feriados |
| **Reports** | Generación de reportes: detallado, resumen, semanal, asistencia, gastos |
| **Audit log** | Reporte de auditoría, con su propia API |
| **Webhooks** | Notificaciones de eventos (timer iniciado, entrada borrada…) |

## Cómo resuelven lo operativo

- **Autenticación**: cabecera `X-Api-Key` (usuario) o `X-Addon-Token` (integración publicada).
- **Límite de peticiones** por workspace: `FREE` 30/hora; `TRIAL`, `BASIC`, `STANDARD`, `PRO`
  20/segundo; `ENTERPRISE` 50/segundo. Add-ons: 50/segundo. Al pasarse responde "Too many requests".
- **Paginación** en los `GET` de listas: `page` (empieza en 1) y `pageSize`; la respuesta trae la
  cabecera **`Last-Page`** con `true`/`false`. No hay total de páginas ni de ítems.
- **URLs por región**: prefijos `euc1` (UE/Alemania), `use2` (EEUU), `euw2` (Reino Unido),
  `apse2` (Australia). La región cambia el host, no la ruta.
- **Webhooks limitados por plan**: 3 en `FREE`; 10 por usuario / 100 por workspace en planes
  medios; 100 por usuario / 300 por workspace en `ENTERPRISE`.
- **Duraciones en formato ISO-8601**: la capacidad de trabajo de un miembro se expresa como
  `PT7H`, y los días laborables como lista de enums `MONDAY`…`SUNDAY`.
- **APIs experimentales**: publicadas y "listas para producción", pero pueden cambiar sin el
  proceso de cambios incompatibles.

## Qué nos sirve

1. **`Last-Page` como convención de paginación.** Si alguna lista de MiBeca crece (historial,
   auditoría, postulantes), copiar la forma —`page`/`pageSize` + cabecera `Last-Page`— es más
   simple que devolver un objeto con `total`, `pages`, `hasNext`.
2. **Separar "lo que se registra" de "lo que se aprueba".** En Clockify la entrada de tiempo y la
   solicitud de aprobación son entidades distintas con estados propios. MiBeca ya separa
   aceptación de postulación y cierre; esto confirma el camino y descarta meter estados de
   aprobación dentro de la asistencia.
3. **Un reporte de asistencia existe como tal** (`Team Report › attendance`), separado del reporte
   de horas. MiBeca tiene progreso por estudiante; si hace falta un reporte de asistencia, es una
   vista distinta, no un filtro del panel.
4. **Auditoría en su propia API.** Clockify expone el log de auditoría aparte del resto. MiBeca lo
   tiene como tabla `auditoria` sin endpoint; si se expone, conviene que sea su propio recurso.
5. **Capacidad y días laborables como atributos del usuario.** El equivalente de nuestra meta
   individual y nuestra disponibilidad, pero declarado por usuario, no por institución. Refuerza
   que la meta individual en `estudiantes.meta_minutos` es la decisión correcta.

## Qué NO copiamos

| Clockify | MiBeca | Por qué |
|---|---|---|
| Timer en marcha por usuario, muchas entradas al día | Un **cierre por trabajo**, minutos por asistencia | Menos estados, menos casos raros. Se decidió y está probado |
| Workspace multi-empresa con subdominio y región | Una sola institución (UPB) | No somos SaaS; el multi-tenant es complejidad sin cliente |
| Tarifas, moneda, facturas, gastos | Nada de dinero | El becario no cobra por hora aquí |
| Aprobación con 4 estados y reenvío | Aceptación + cierre único, sin correcciones | La corrección de horas está fuera del alcance a propósito |
| Duraciones ISO-8601 (`PT7H`) | Enteros de **minutos**, múltiplo de 6 | Un entero es más simple de validar y de sumar en SQL |
| Webhooks | Sin webhooks | Nadie consume eventos todavía |

La lección general: Clockify puede permitirse todo eso porque es un producto comercial con
clientes que lo piden. Nosotros mantenemos tres juniors; cada concepto extra se paga en cada
cambio futuro.

## Límite de esta nota

Describe **la API v1**, no la interfaz web ni precios. Que algo exista en la API no significa que
el plan gratuito lo permita. No se probó ninguna cuenta real: todo sale de la documentación.

Relacionado: [[Visión y alcance]] · [[Cierre, asistencia y horas]] · [[API v1]] · [[00 Índice]]
