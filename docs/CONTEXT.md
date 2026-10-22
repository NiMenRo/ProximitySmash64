# ProximitySmash64 — Contexto del proyecto

Documento central para agentes de IA. Las decisiones aquí registradas están aprobadas; cualquier cambio de stack, arquitectura o alcance requiere aprobación explícita.

## Descripción general

Cliente de escritorio personalizado para emular **Super Smash Bros 64** con la ROM del mod **Smash Remix**, con multijugador P2P sin abrir puertos (NAT traversal), chat de voz por proximidad 3D basado en las coordenadas de los personajes y sistema de amigos/invitaciones con un backend ligero.

## 1. Núcleo de emulación

- **Core:** `mupen64plus-libretro-nx`, integrado como submódulo Git de solo lectura (`core_emulador/`, commit fijado `12edd2c`).
- Se compila por separado como DLL y el cliente la carga dinámicamente mediante la API estándar de Libretro. **No se modifica el código del emulador.**
- **Build:** en local se compila el submódulo con su `Makefile` original; en CI se descarga temporalmente un release precompilado.
- El host del cliente implementa los callbacks de Libretro (vídeo, audio, input, entorno) y controla el bucle `retro_run()`.

## 2. Aplicación cliente

- **Lenguaje:** C++20.
- **Build:** CMake.
- **UI y render:** SDL2 (ventana, input y contexto) + Dear ImGui (interfaz) + OpenGL (backend gráfico).
- **Plataforma:** Windows x64 exclusivamente para v0.1–v0.3. Linux se evaluará más adelante.
- **Dependencias:** submódulos Git para librerías externas ligeras o `CMake FetchContent`. No se usa vcpkg por ahora, para agilizar el CI/CD.
- **Pruebas:** GoogleTest (gtest) gestionado con `CMake FetchContent`.

Estructura del repositorio:

```
src/
├── main.cpp
├── emulation/        # Carga del core y lectura de RAM
├── netplay/          # P2P, sincronización y rollback futuro
├── audio/            # Captura de voz y audio espacial
├── backend/          # Supabase: auth, amigos y signaling
└── ui/
    ├── components/   # Widgets reutilizables
    └── views/        # Menú, configuración y juego
tests/                # Pruebas unitarias e integración
core_emulador/        # Submódulo del core (solo lectura)
docs/
├── CONTEXT.md        # Este documento
├── constitution.md   # Principios del proyecto
└── specs/
    ├── README.md
    └── features/
        └── HU-01-base-local/   # spec.md, plan.md y tasks.md
```

## 3. Red multijugador (P2P)

- **Transporte:** `libjuice` (ICE/STUN/TURN) sobre UDP, para conexión directa sin abrir puertos.
- **v0.2:** 2 jugadores en topología estricta Host–Guest con sincronización **delay-based** (más simple para el MVP).
- **4 jugadores y TURN:** TODO a partir de v0.2; la topología definitiva (malla P2P o host autoritativo) y el servidor TURN se decidirán entonces.
- **Rollback:** se evaluará después, apoyado en los savestates de Libretro (`retro_serialize` / `retro_unserialize`).
- **Hilos:** recepción UDP en un hilo separado del bucle de emulación.
- **Signaling:** Supabase Realtime intercambia SDP/ICE; el tráfico de juego no pasa por Supabase.

## 4. Chat de voz por proximidad

- **Códec:** Opus a 48 kHz, mono en captura y procesado a estéreo según la posición 3D.
- **Micrófono:** push-to-talk con tecla configurable por defecto, para evitar eco sin depender todavía de cancelación de eco de WebRTC.
- **Lectura de RAM:** extraer las coordenadas X, Y, Z de los jugadores desde la memoria del juego.
- **Cálculo espacial:** distancia euclidiana `d = sqrt((x2-x1)² + (y2-y1)² + (z2-z1)²)`.
- **Mezcla:** atenuación de volumen según `d` y paneo estéreo según la posición relativa.
- **Offsets de RAM:** TODO para v0.3. Se buscarán empíricamente y corresponderán estrictamente a Smash Remix **v2.0.1**.

## 5. Backend, autenticación y amigos

- **Tecnología:** Supabase (v0.4).
- **Auth:** registro e inicio de sesión.
- **PostgreSQL:** perfiles, amigos y salas.
- **Realtime:** presencia (quién está online) y signaling para invitaciones y establecimiento P2P.
- **Esquema y RLS:** TODO para v0.4. Se diseñarán íntegramente en esa fase.
- El cliente nunca accede directamente a PostgreSQL ni usa `service_role`; solo claves públicas y políticas RLS.

## 6. ROM y compatibilidad

- **Versión oficial objetivo:** Smash Remix **v2.0.1**, la versión actual del proyecto. Sustituye a la referencia anterior v1.5.0.
- **v0.1 (sin bloqueo):** el cliente calcula e imprime en consola el CRC de la ROM cargada. Ese valor se usará para capturar y registrar el hash definitivo de una ROM generada con Smash Remix v2.0.1.
- **Validación futura:** el CRC/hash esperado se fijará contra Smash Remix v2.0.1. **Netplay y chat de proximidad dependerán de que el CRC coincida estrictamente** con esa versión, porque los offsets de RAM del audio 3D corresponden solo a ella.
- **Modo Offline/Fallback:** si el CRC no coincide, el cliente **no debe crashear**. Permitirá emular el juego localmente y desactivará:
  - los módulos de red;
  - la conexión con Supabase;
  - la lectura de RAM y el chat de voz.
  
  La UI de ImGui mostrará una advertencia clara del modo degradado.
- La ROM la aporta el usuario final: no se incluye ni se distribuye en el repositorio.

## 7. Roadmap

| Versión | Alcance |
|---|---|
| **v0.1 — Base local** | CMake + GoogleTest, carga de la DLL Libretro, carga de ROM e impresión de su CRC sin bloqueo, vídeo OpenGL y mando local con SDL2. Sin red. |
| **v0.2 — Netplay P2P** | 2 jugadores Host–Guest con `libjuice` y sincronización delay-based. Validación estricta de CRC contra v2.0.1. |
| **v0.3 — Voz por proximidad** | Opus, push-to-talk, offsets X/Y/Z de v2.0.1 y atenuación/paneo estéreo. |
| **v0.4 — Supabase** | Cuentas, amigos, salas, matchmaking y diseño completo de esquema/RLS. |

## 8. Hilos y frame pacing

- Un solo hilo para UI (ImGui) y emulación (`retro_run`) durante el MVP, para simplificar el frame pacing.
- Hilos separados únicamente para:
  - recepción UDP de red;
  - callbacks de audio.
- La ruta crítica del frame no debe bloquearse con red, disco, UI ni asignaciones de memoria.

## 9. Licencia

- El proyecto se distribuye bajo **GPLv3**.
- El core se carga dinámicamente como biblioteca externa GPLv2; se reconoce y respeta la licencia y autoría de todos los componentes de terceros.
- No se distribuyen ROMs, BIOS ni material con derechos de autor, y no se sugiere afiliación con Nintendo.

## 10. Decisiones resueltas y pendientes por fase

### Resueltas para v0.1

- **Test runner:** GoogleTest (gtest) vía `CMake FetchContent`.
- **DLL del core:** compilación local con el `Makefile` original del submódulo; en CI se descarga temporalmente un release precompilado.
- **ROM:** Smash Remix v2.0.1 como objetivo. En v0.1 solo se imprime el CRC de la ROM cargada para capturarlo, sin bloquear la ejecución.

### Aplazadas (TODO)

| Tema | Fase | Nota |
|---|---|---|
| Capturar y fijar el CRC/hash definitivo de v2.0.1 | v0.1 → v0.2 | Se registra con la impresión de CRC de v0.1 y se valida antes de netplay/voz. |
| Offsets de RAM X/Y/Z | v0.3 | Búsqueda empírica, estrictamente sobre Smash Remix v2.0.1. |
| Topología de 4 jugadores y servidor TURN | v0.2+ | v0.2 arranca estricto con 2P Host–Guest. |
| Esquema de Supabase y políticas RLS | v0.4 | Diseño íntegro en esa fase. |

### Otros pendientes

- Definir el release precompilado del core que usará CI y su origen.
- Definir qué pruebas son posibles sin ROM ni GPU.
- Diseñar el flujo de UI para el modo Offline/Fallback y errores de red.
