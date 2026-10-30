---
tags: [mibeca, investigacion, diseno]
---

# SOLID en TypeScript

**Fuente:** <https://khalilstemmler.com/articles/solid-principles/solid-typescript/> ·
**Consultado:** 2026-10-29. Última actualización del artículo: 2019-07-16.

Artículo de referencia sobre los cinco principios de diseño, explicados con TypeScript y Node.
Sirve para **nombrar** decisiones que MiBeca ya toma y para detectar cuándo una abstracción sobra.

## Los cinco principios, en corto

| Principio | Frase del artículo | Traducción a código |
|---|---|---|
| **S** — Single Responsibility | "Una clase o función debe tener una sola razón para cambiar" | Un archivo hace una cosa. El criterio de separación es **quién pide el cambio**, no el tamaño |
| **O** — Open/Closed | "Abierto a extensión, cerrado a modificación" | Agregar un caso no debería editar el código de los casos existentes |
| **L** — Liskov Substitution | Se puede intercambiar una implementación por otra sin romper | Si dos servicios cumplen el mismo contrato, se elige en el arranque, no dentro de la lógica |
| **I** — Interface Segregation | Un cliente no debe depender de lo que no usa | Interfaces pequeñas; nada de un `Operations` con 6 métodos para usar uno |
| **D** — Dependency Inversion | "Las abstracciones no dependen de los detalles; los detalles dependen de las abstracciones" | Recibir la dependencia por parámetro, no instanciarla dentro |

El ejemplo que usa para todo el artículo es un servicio de correo:

```ts
interface IMailService {
  sendMail(email: IMail): Promise<IEmailTransmissionResult>
}
// SendGrid, MailChimp, MailGun implementan lo mismo...
class CreateUserController {
  private emailService: IMailService          // abstracción
  constructor (emailService: IMailService) {  // inyectada
    this.emailService = emailService
  }
}
```

Cambiar de proveedor es cambiar una línea en el "main component" (el archivo de rutas que arma los
objetos concretos). Nada más se toca.

## Las dos advertencias del propio autor

1. **"Interfaces no existen en JavaScript."** Sin compilador que verifique el contrato, el
   cumplimiento es disciplina, no garantía.
2. **"Existe el ser *demasiado* SOLID."** Escribir abstracciones para cada cosa multiplica archivos.
   Si estás seguro de que nunca vas a necesitar la abstracción, no la escribas.

También distingue los **main components**: el único lugar donde está bien referenciar clases
concretas (`new MailGunEmailService()`), porque ahí se arma el sistema. El resto del código depende
de abstracciones.

## Qué nos sirve

MiBeca está en **JavaScript ESM**, sin TypeScript en el backend (`typescript` figura como
devDependency solo para detectar el entorno; el código es `.js`). Eso cambia el cálculo:

| Principio | Cómo aplica hoy |
|---|---|
| S | Ya se cumple por archivo: `services/trabajos.js` tiene las reglas de trabajos y nada más. Mantenerlo |
| O | Aplica de forma imperfecta. Agregar un estado nuevo de trabajo toca el service, no un archivo nuevo. **Es aceptable**: con 3 personas, un `switch` sobre un enum de estados es más legible que 5 clases |
| L | Relevante solo si algún día hay dos implementaciones de algo (por ejemplo, dos formas de calcular horas). Hoy no las hay |
| I | Ninguna interfaz que segregar: no hay interfaces |
| D | Aquí está lo aplicable de verdad: los services reciben `req`/datos, no instancian conexiones. `lib/db.js` es un singleton importado. Eso es **suficiente** para tres juniors |

La lección que sí conviene adoptar, sin reescribir nada:

> **No instanciar dependencias dentro de la lógica de negocio.** Si mañana un service necesita enviar
> correo o leer un archivo, recibe la función por parámetro; no hace `import` del cliente concreto y
> lo llama adentro.

Eso deja el código testeable sin contenedores de inyección ni frameworks.

## Qué NO aplicamos

- **Contenedores de inyección de dependencias** (Inversify y similares). El autor los menciona como
  opción para cientos de dependencias. MiBeca tiene seis.
- **Migrar a TypeScript u OOP con clases y jerarquías.** Ya está descartado por el §8 del plan v2 (en
  desuso, decisión viva) y por el principio de simplicidad.
- **Interfaces y clases abstractas** como capa ceremonial donde no hay segunda implementación.
- **Prefijo `I`** en nombres: no hay interfaces que nombrar.

## Límite

El artículo es de **2019** y está armado alrededor de TypeScript + orientación a objetos. Varias de
sus recomendaciones (interfaces, clases abstractas, DI) suponen un compilador que verifique
contratos, cosa que MiBeca no tiene. Tomar el criterio, no la sintaxis.

Relacionado: [[Estructura de proyecto Node.js]] · [[Arquitectura general]] · [[Pruebas]] · [[00 Índice]]
