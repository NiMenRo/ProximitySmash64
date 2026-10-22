# ProximitySmash64 — Guía para agentes

## Descripción

Cliente de escritorio en C++ para emular **Super Smash Bros 64** (ROM del mod **Smash Remix**) sobre el core Libretro `mupen64plus-libretro-nx`, con netplay P2P sin abrir puertos, chat de voz por proximidad 3D y sistema de amigos/invitaciones con Supabase. El alcance completo y las decisiones aprobadas están en `docs/CONTEXT.md`.

## Stack (decidido)

- **Core:** `mupen64plus-libretro-nx` como submódulo Git de solo lectura; se compila como `.dll` y se carga dinámicamente con la API Libretro. En local se compila con su `Makefile` original; en CI se descarga temporalmente un release precompilado.
- **Cliente:** C++20, CMake, SDL2 + Dear ImGui + OpenGL.
- **Plataforma:** Windows x64 exclusivamente para v0.1–v0.3. Linux se evaluará después.
- **Dependencias:** submódulos Git o `CMake FetchContent`. No usar vcpkg por ahora.
- **Pruebas:** GoogleTest (gtest) vía `CMake FetchContent`.
- **Netplay:** `libjuice` sobre UDP; v0.2 con 2 jugadores Host–Guest y sincronización delay-based. Rollback y topología de 4 jugadores/TURN quedan para más adelante.
- **Voz:** Opus 48 kHz mono → estéreo espacial, push-to-talk configurable. Offsets de RAM (TODO v0.3) solo para Smash Remix v2.0.1.
- **Backend:** Supabase (Auth, PostgreSQL, Realtime) en v0.4; esquema y RLS se diseñan íntegramente entonces.
- **ROM:** Smash Remix v2.0.1. En v0.1 solo se imprime el CRC de la ROM cargada, sin bloquear.
- **Licencia:** GPLv3.

## Estado y estructura

- `docs/CONTEXT.md`: fuente de verdad del proyecto y de las decisiones aprobadas.
- `docs/constitution.md`: principios innegociables.
- `docs/specs/`: cada spec en `docs/specs/features/HU-XX-nombre/` con `spec.md`, `plan.md` y `tasks.md` (nunca archivos sueltos). `HU-01-base-local/` es el ejemplo de estructura y la primera spec.
- `.agents/skills/sdd/` y `.opencode/`: flujo Spec-Driven Development (agentes planner/implementer/reviewer y comandos `/sdd-*`).
- `core_emulador/`: submódulo del core (commit `12edd2c`, describe `mupen_next_old_gliden-458-g12edd2c`). Solo lectura.
- `src/`: esqueleto del cliente (`main.cpp`) con `emulation/`, `netplay/`, `audio/`, `backend/` y `ui/{components,views}`.
- `tests/`: pruebas unitarias e integración.
- Todavía no hay implementación: `main.cpp` es solo el punto de entrada y no existe `CMakeLists.txt`.

## Convenciones

- Código e identificadores técnicos en inglés.
- Documentación, especificaciones y textos de interfaz en español.
- Mantener la separación de responsabilidades entre componentes.
- No añadir complejidad ni dependencias innecesarias.
- La ROM la aporta el usuario: no se incluye, commitea ni distribuye.

## Arquitectura

Componentes:

- **Cliente / UI:** menús, lista de amigos, configuración y estados de partida. No implementa lógica de red ni de audio.
- **Host de emulación:** implementa la API Libretro y es el único punto que habla con el core (vídeo, audio, input, RAM, savestates).
- **Netplay:** transporte P2P con libjuice, sincronización delay-based y, más adelante, rollback.
- **Voz por proximidad:** captura Opus, push-to-talk, envío/recepción, cálculo espacial y mezcla estéreo.
- **Backend (Supabase):** identidad, amigos, presencia y signaling.

Reglas:

- El core es una dependencia binaria de solo lectura: solo se usa su API pública; no se parchea ni se accede a sus estructuras internas.
- Ningún componente del cliente accede directamente a PostgreSQL ni usa credenciales privilegiadas: todo pasa por Supabase (Auth/API/Realtime).
- Durante el MVP, UI y emulación comparten un solo hilo; solo la recepción UDP y los callbacks de audio tienen hilos propios.
- La ruta crítica del frame no se bloquea con red, disco, UI ni asignaciones de memoria.
- Las reglas de negocio no viven en la UI.
- Las posiciones de los jugadores solo se usan para la voz por proximidad; no se persisten sin decisión explícita.
- Si el CRC no coincide con Smash Remix v2.0.1: el cliente no debe crashear. Se permite el modo local (Offline/Fallback) con advertencia en ImGui y se desactivan red, Supabase y lectura de RAM/voz. Netplay y voz solo funcionan con CRC exacto.

## Forma de trabajar

1. Leer `MEMORY.md` al comenzar.
2. Consultar `docs/CONTEXT.md`, `docs/constitution.md` y la especificación correspondiente en `docs/specs/`.
3. Revisar los criterios de aceptación antes de implementar.
4. Seguir el flujo: spec → clarificación → plan → tareas → implementación → validación.
5. Hacer cambios pequeños y enfocados; una tarea a la vez.
6. No inventar reglas ni decisiones de stack o arquitectura; registrar dudas abiertas.
7. Al terminar, resumir los cambios y las verificaciones ejecutadas.

## Límites

Siempre:

- Respetar la frontera con el core, la plataforma Windows x64 y la separación entre componentes.
- Actualizar `MEMORY.md` al terminar cada tarea.
- Usar CodeGraph (`codegraph_explore` o `codegraph explore`) para entender y explorar el código antes de rastrear con grep/Read.
- Sincronizar el índice con `codegraph sync` después de cada cambio en el código.
- Usar el MCP Context7 cuando se necesite información sobre una librería (sintaxis, configuración, versiones).
- Verificar con compilación y pruebas lo que se pueda ejecutar.

Preguntar antes de:

- Cambiar el stack, la plataforma o el gestor de dependencias.
- Cambiar la arquitectura o los contratos entre componentes.
- Añadir dependencias fuera de submódulos Git o `CMake FetchContent`.
- Actualizar el submódulo del core o modificar cualquier archivo de `core_emulador/`.
- Implementar soporte para Linux.
- Persistir datos o añadir telemetría.

Nunca:

- Commitear la ROM, BIOS, saves ni material con derechos de autor.
- Exponer secretos (claves de Supabase, `service_role`, tokens, credenciales).
- Modificar `core_emulador/` ni sus archivos internos.
- Bloquear el bucle de emulación con E/S, red o audio.
- Usar vcpkg mientras no se apruebe.
- Afirmar que se ejecutaron pruebas o compilaciones si no se ejecutaron.

## Memoria

- Leer `MEMORY.md` al comenzar cada tarea.
- Actualizarlo al terminar con el estado actual, decisiones importantes y errores que se deben evitar.
- Mantenerlo breve, con un máximo aproximado de 50 líneas.
- Si una regla se vuelve permanente, proponer incorporarla a `AGENTS.md` o a la constitución.
- No guardar claves, tokens ni datos sensibles.
