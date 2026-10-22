# Especificaciones

## Regla de estructura

Cada spec vive en **su propia carpeta** dentro de `features/`:

```
docs/specs/
├── README.md
└── features/
    └── HU-XX-nombre/
        ├── spec.md
        ├── plan.md
        └── tasks.md
```

- Nunca se crean `spec.md`, `plan.md` ni `tasks.md` sueltos dentro de `features/`.
- Al crear una carpeta de spec, se crean **siempre los tres archivos**. Se rellena el que corresponda a la fase y se dejan los otros como estructura.
- `docs/specs/features/HU-01-base-local/` es el ejemplo de estructura y la primera spec (base local v0.1).
- Para una spec nueva, duplica esa carpeta y renómbrala a `HU-XX-<nombre>`, donde `XX` es el siguiente número libre.

| Archivo | Fase | Responde a |
|---|---|---|
| `spec.md` | Qué y por qué | Requisitos, actores, casos límite y criterios de aceptación |
| `plan.md` | Cómo | Componentes, contratos, decisiones técnicas y estrategia de verificación |
| `tasks.md` | En qué orden | Tareas pequeñas y verificables, cada una con sus RF |

## Cómo empezar una spec

1. Crea `docs/specs/features/HU-XX-nombre/` con los tres archivos copiados de `HU-01-base-local/`.
2. Ejecuta `/sdd-spec HU-XX-nombre <idea inicial>` y responde las preguntas del planner.
3. Sigue el flujo: spec → clarificación → plan → tareas → implementación → validación.

El flujo completo, la estructura de los documentos y el formato EARS están en `.agents/skills/sdd/SKILL.md`.

## Reglas

- La spec manda: lo que no está en ella no se implementa.
- La spec describe el QUÉ y el POR QUÉ; no incluye stack, arquitectura ni nombres de archivo. El stack está fijado en `docs/CONTEXT.md`; una spec no lo reabre.
- Todo requisito debe ser verificable.
- Ninguna spec se implementa sin aprobación explícita del usuario.
- `core_emulador/` es una dependencia externa: nunca se modifica desde una spec.
- La ROM, la BIOS y los saves no forman parte del repositorio.
