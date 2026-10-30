---
tags: [mibeca, indice]
---

# MiBeca · Base de conocimiento

Plataforma institucional de la Universidad Privada Boliviana (UPB) para gestionar y
hacer seguimiento del **trabajo becario**: publicación de trabajos, postulaciones,
cierre con asistencia y acreditación de horas.

> [!info] Origen de este vault
> Construido el 2026-10-29 leyendo los repositorios `gestion-de-trabajo-becario-web`
> y `gestion-de-trabajo-becario-backend`. Todo lo que dice aquí proviene de
> documentación versionada o de código; lo que no se pudo verificar aparece marcado
> como **pendiente** o **no verificado**. Ver [[Fuentes en el repositorio]].

## Empezar aquí
- [[Visión y alcance]] — qué resuelve MiBeca y qué queda explícitamente fuera
- [[Equipo]] — integrantes, ramas y reparto de trabajo
- [[Estado actual]] — qué está implementado, con qué evidencia, qué falta
- [[Glosario]] — vocabulario del dominio

## Reglas de negocio
- [[Reglamento de becas UPB]] — modalidades, porcentajes y condiciones de mantenimiento
- [[Roles y permisos]] — quién puede hacer qué
- [[Usuarios y sesiones]] — altas, contraseñas, sesiones, desactivación
- [[Trabajos y postulaciones]] — publicación, cupos, estados
- [[Cierre, asistencia y horas]] — el corazón del sistema
- [[Modelo de datos]] — tablas, restricciones e índices

## Arquitectura
- [[Arquitectura general]] — repos, stack, flujo de una petición
- [[API v1]] — endpoints del contrato vigente
- [[PostgreSQL, pool y RLS]] — pool de conexiones, SQL y seguridad por filas

## Cómo se trabaja
- [[SDD con Spec Kit]] — el marco de trabajo del equipo
- [[Reparto de tareas]] — bloques de 3, relevo secuencial, backend primero
- [[Ramas y PR]] — ramas, revisión, reglas de integración
- [[Agentes y skills]] — perfiles de agente y skills del proyecto
- [[Guardas y auditor de código]] — hooks y reglas automáticas
- [[CI y CD]] — workflows y despliegue

## Procedimientos
- [[Entorno local]] — levantar API + PostgreSQL + frontend
- [[Migraciones]] — aplicar y validar SQL
- [[Primer administrador]] — arrancar un entorno desde cero
- [[Pruebas]] — comandos y qué cubre cada suite
- [[Supabase]] — base de datos remota y sus guardas

## Referencias
- [[Fuentes en el repositorio]] — qué documento manda sobre qué
- [[Prototipo de rediseño]] — el mock que **no** es la aplicación
- [[Cómo mantener este vault]] — qué se guarda aquí y con qué formato
- [[enlaces web]] — registro de toda fuente de internet consultada
- [[00 Índice]] de `07 Investigación` — hallazgos reutilizables y temas por investigar
- [[Clockify]] — plataforma comercial similar; qué copiamos y qué no
- [[Asistencia biométrica universitaria (Kakepoto 2022)]] — verificar asistencia y sus efectos
- [[Sistema web de becas EGPP]] — antecedente boliviano de gestión y difusión de becas
- [[Ingeniería de prompts]] — cómo pedirle trabajo a un agente
- [[Estructura de proyecto Node.js]] — por qué **no** reorganizar `src/`
- [[Estilo de JavaScript (Airbnb)]] — reglas de estilo con efecto real
- [[Vite - características]] · [[SOLID en TypeScript]]
