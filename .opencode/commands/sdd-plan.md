---
description: SDD · Genera el plan técnico de una spec aprobada
agent: plan
---

Lee docs/CONTEXT.md, docs/constitution.md, AGENTS.md y docs/specs/features/$1/spec.md. Usa la skill sdd. NO escribas código. Genera docs/specs/features/$1/plan.md con: componentes que se crean o modifican (cliente/UI, host de emulación, netplay, voz por proximidad, backend) y la responsabilidad de cada parte, contratos entre componentes, decisiones técnicas justificadas (con su alternativa descartada) y estrategia de verificación (compilación y pruebas del stack acordado; mediciones reproducibles de latencia o sincronización cuando apliquen). Todo debe respetar la constitución y cubrir todos los RF. Marca qué RF cubre cada parte. Si el plan exige cambiar el stack o la plataforma acordados en docs/CONTEXT.md, pregúntame antes de fijarlo. Si la spec no está aprobada o tiene dudas abiertas, para y avísame.
