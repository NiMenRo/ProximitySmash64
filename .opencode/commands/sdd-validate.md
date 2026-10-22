---
description: SDD · Valida la spec RF por RF según el plan
agent: build
---

Recorre docs/specs/features/$1/spec.md requisito por requisito. Para cada RF indica qué prueba o comprobación lo cubre y el resultado real de ejecutar la verificación definida en plan.md (compilación y pruebas). Los RF de latencia, sincronización o rendimiento, verifícalos con mediciones o pruebas reproducibles. Si algún RF no está cubierto o falla, dilo claramente. NO arregles nada todavía. Después comprueba los criterios de finalización y dame un veredicto: ¿la spec está cumplida?
