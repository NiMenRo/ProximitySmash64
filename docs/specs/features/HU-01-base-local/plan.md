# Plan HU-01 — Base local (v0.1)

Spec base: `docs/specs/features/HU-01-base-local/spec.md` (Estado: borrador, <N> RF).
<Alcance de la verificación y stack usado.>

## 1. Componentes afectados

| Componente | Responsabilidad | RF que cubre |
|---|---|---|
| <cliente/UI> | <...> | <RF-x> |
| <host de emulación> | <...> | <RF-x> |
| <netplay / voz / backend> | <...> | <RF-x> |

<Indica qué componentes se crean y cuáles se modifican.>

## 2. Contratos entre componentes

<Interfaces, formatos de mensaje, estructuras compartidas y quién es el único punto de acceso a cada recurso.>

## 3. Decisiones justificadas

**D1. <Decisión>.** <Justificación y qué RF cubre.> *Alternativa descartada: <alternativa y motivo>.*

**D2. <Decisión>.** <Justificación.> *Alternativa descartada: <alternativa y motivo>.*

## 4. Estrategia de verificación

- <Compilación y pruebas del stack acordado (CTest/GoogleTest, pytest u otro).>
- <Pruebas o mediciones reproducibles para RF de latencia, sincronización o rendimiento.>
- <Comprobación manual cuando no exista runner todavía.>
- <Cobertura RF por RF.>
