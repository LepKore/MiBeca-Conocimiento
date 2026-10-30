---
tags: [mibeca, investigacion, arquitectura]
---

# Sistema web de gestión y difusión de becas EGPP

**Fuente:** `docs/2026-10-29 Sistema web gestion y difusion becas EGPP (2014).pdf` · **Consultado:** 2026-10-29.

Proyecto de grado de Richar Ángel Márquez Zacari (UMSA, Informática, 2014) para la Escuela de
Gestión Pública Plurinacional (EGPP). Describe un sistema para administrar y difundir ofertas de
becas de cooperación internacional canalizadas por convenios con el Gobierno boliviano.

> [!warning] Es una referencia histórica
> No describe MiBeca, ni sus reglas actuales, ni una arquitectura que debamos copiar. El trabajo
> usa tecnologías de 2014 y sirve como comparación de dominio y de decisiones de diseño.

## Qué problema resolvía

El módulo anterior estaba integrado en un sistema GeneXus. El documento identifica lentitud,
duplicación de datos personales, convocatorias e instituciones, acceso de personal no autorizado,
alteración de información, duplicación de archivos, ausencia de historial de descargas y
dificultad de mantenimiento por depender de una herramienta propietaria.

El sistema propuesto separaba la gestión y difusión de becas del sistema institucional. Su alcance
era limitado a **convocatorias** y **postulantes**, sin conexión directa con el sistema web EGPP.

## Modelo funcional que nos sirve

| Concepto | Cómo aparece en EGPP | Comparación con MiBeca |
|---|---|---|
| Convocatoria | Oferta con requisitos, cooperación/institución patrocinante y fechas | MiBeca publica trabajos becarios, no ofertas académicas externas |
| Postulante | Persona que busca y se registra para una convocatoria | Equivalente parcial a estudiante + postulación |
| Institución patrocinante | Organismo que ofrece o respalda la beca | Útil para modelar una futura **institución de beca convenio** |
| Difusión | Web, folletos, correo, prensa, cartas y atención presencial | MiBeca necesita principalmente publicación y consulta dentro de la plataforma |
| Reporte | Informes generados para funcionarios | Refuerza separar reportes de la operación diaria |
| Historial de descargas | Registra quién descargó archivos de convocatorias | Idea útil si MiBeca algún día distribuye contratos o comprobantes |

## Decisiones técnicas del trabajo

- Metodología: **Programación Extrema (XP)** y WebML.
- Modelo de datos: normalización para reducir duplicidad y mejorar integridad.
- Funciones centrales: módulo de convocatorias, ingreso de usuarios y postulantes.
- Herramientas de la época: Spring MVC, iBATIS, PostgreSQL 9.3, Tomcat, jQuery/Ajax, HTML5/CSS3,
  iReport/JasperReports y NetBeans.
- Propone acceso autenticado para limitar el módulo a personal autorizado.

## Prácticas de proceso reutilizables

El texto resume XP como planeación, diseño, codificación y pruebas, con historias de usuario,
plan de entregas, tareas, velocidad y entregas pequeñas. También destaca:

- diseñar la solución más simple que funcione;
- eliminar complejidad y código innecesario;
- refactorizar sin cambiar el comportamiento;
- escribir pruebas antes del código cuando sea viable;
- integración continua ejecutando todas las pruebas;
- estándares de programación para que el código sea mantenible.

Esto coincide parcialmente con el proceso actual de MiBeca: specs, bloques pequeños, CI y pruebas.
No reemplaza las reglas vigentes de [[SDD con Spec Kit]] ni [[Reparto de tareas]].

## Qué aporta a la beca convenio

El documento ofrece un antecedente cercano para distinguir tres actores que hoy no están en el
modelo de MiBeca: **institución patrocinante**, **convocatoria externa** y **postulante**. Si la
beca convenio entra al producto, una spec podría estudiar una relación entre el estudiante, la
institución externa, el porcentaje aportado y la evidencia del examen aprobado.

Pero no se debe inferir que MiBeca necesite todo el flujo EGPP: el usuario confirmó que la beca
convenio ya se adquiere externamente y que la plataforma solo debe respetar las mismas exigencias
de no reprobar, promedio mínimo y horas becarias.

## Qué no copiamos

- GeneXus, Spring MVC, iBATIS, Tomcat o WebML: son elecciones históricas del trabajo, no requisitos.
- Un portal de difusión de becas internacionales: MiBeca gestiona trabajo becario y acreditación.
- Copiar postulantes e instituciones sin una necesidad confirmada: sería complejidad prematura.
- MD5: el documento lo menciona dentro de software seguro, pero es obsoleto para contraseñas.

## Límites de la evidencia

Es un proyecto académico de 2014, centrado en EGPP y no en UPB. Describe objetivos y diseño del
sistema, pero no demuestra que todas sus funciones hayan sido operadas en producción. La
comparación sirve para generar preguntas de producto, no para afirmar que esas reglas aplican a
MiBeca.

Relacionado: [[Reglamento de becas UPB]] · [[Visión y alcance]] · [[Modelo de datos]] ·
[[SDD con Spec Kit]] · [[Reparto de tareas]] · [[00 Índice]] · [[enlaces web]]
