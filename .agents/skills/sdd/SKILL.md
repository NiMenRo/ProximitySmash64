---
name: sdd
description: Úsala siempre que trabajes con Spec-Driven Development en este proyecto (docs/constitution.md o cualquier archivo dentro de docs/specs/) — redactar, revisar o cambiar specs, planes y tareas, o implementar y validar tareas de una spec.
---

# Spec-Driven Development (SDD)

## Flujo

Constitución → Spec → Clarificación → Plan → Tareas → Implementación → Validación → Cambio.

- Nunca pases a la siguiente fase sin la aprobación explícita del usuario.
- La spec manda: si algo no está en la spec, no se implementa. Si falta una decisión, para y pregunta.
- Un cambio de requisitos se hace primero en la spec, luego en el plan y las tareas, y por último en el código.
- Cada spec vive en su carpeta: `docs/specs/features/HU-XX-nombre/` con `spec.md`, `plan.md` y `tasks.md`. Nunca escribas esos archivos sueltos en `features/`. Al crear la carpeta, incluye siempre los tres; rellena solo el que corresponda a la fase actual. Usa `docs/specs/features/HU-01-base-local/` como ejemplo de estructura.
- Al terminar cada fase, actualiza `MEMORY.md`.
- Explora el código con CodeGraph antes que con grep/Read, y ejecuta `codegraph sync` después de cada cambio en el código.
- Usa Context7 cuando necesites información sobre una librería.

## Plantilla de spec (spec.md)

```
# Spec HU-XX — <Nombre>
Estado: borrador | aprobada | implementada

## Contexto y objetivo
<Qué problema resuelve y por qué merece la pena. Un párrafo.>

## Usuarios / actores
<Quiénes la utilizan.>

## Historias de usuario
- H1: Como <rol>, quiero <acción> para <beneficio>.

## Requisitos funcionales (criterios de aceptación en EARS)
- RF-1: ...

## Requisitos no funcionales
<Solo los que apliquen.>

## Casos límite
<Vacíos, duplicados, datos inválidos, límites, concurrencia, etc.>

## Fuera de alcance
<Qué no se implementará en esta iteración.>

## Criterios de finalización
<Cómo se comprobará que la funcionalidad está terminada.>

## Dudas abiertas
- [NECESITA ACLARACIÓN] <Duda pendiente.>
```

La spec describe el QUÉ y el POR QUÉ. Nada de stack, arquitectura ni nombres de archivos.

## Requisitos en EARS (en español)

- RF-x: CUANDO \<evento\>, EL SISTEMA \<respuesta\>.
- RF-x: SI \<condición no deseada\>, ENTONCES EL SISTEMA \<respuesta\>.
- RF-x: MIENTRAS \<estado\>, EL SISTEMA \<respuesta\>.
- RF-x: EL SISTEMA \<comportamiento permanente\>.

Cada RF debe ser verificable: nada de "rápido" o "bonito" sin un criterio medible.

## Plan (plan.md)

Componentes afectados (cliente/UI, host de emulación, netplay, voz por proximidad, backend) y su responsabilidad · Contratos entre componentes · Decisiones justificadas con su alternativa descartada · Estrategia de verificación con el stack acordado en `docs/CONTEXT.md` (compilación y pruebas; mediciones reproducibles de latencia o sincronización cuando apliquen). Indica qué RF cubre cada parte. Si el plan exige cambiar el stack o la plataforma, pregunta antes de fijarlo.

## Tareas (tasks.md)

```
- [ ] **Tn. <Descripción>.** RF-x, RF-y
- Hecho cuando: <comprobación verificable>.
```

Máximo 20-30 min por tarea, en orden de dependencia. Si salen más de 10, propón dividir la spec.

## Implementación

Una sola tarea cada vez: primero las pruebas definidas en `plan.md` (en rojo), luego el código hasta que pasen. Ejecuta la compilación y las pruebas y muestra el resultado real. Marca la tarea en `tasks.md`, ejecuta `codegraph sync`, actualiza `MEMORY.md` y PÁRATE. No modifiques `core_emulador/` ni añadas dependencias sin aprobación.

## Validación

Recorre la spec RF por RF: qué prueba o comprobación lo cubre y su resultado según la verificación definida en `plan.md`. Los RF de latencia, sincronización o rendimiento se verifican con mediciones o pruebas reproducibles. Si algún RF no está cubierto o falla, dilo claramente. Comprueba los criterios de finalización y da un veredicto: ¿la spec está cumplida?
