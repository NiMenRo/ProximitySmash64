# Constitución del proyecto ProximitySmash64

1. El core de emulación (`mupen64plus-libretro-nx`) es una dependencia externa de solo lectura: se integra únicamente a través de la API Libretro pública, sin parchearlo ni acceder a sus estructuras internas.
2. Mantener el stack acordado: C++20, CMake, SDL2, Dear ImGui y OpenGL, con Windows x64 como plataforma objetivo de v0.1 a v0.3. Cambiarlo requiere aprobación explícita.
3. Mantener la separación de responsabilidades entre cliente/UI, host de emulación, netplay, voz por proximidad y backend; ningún componente asume tareas de otro.
4. El cliente nunca accede directamente a PostgreSQL ni maneja credenciales privilegiadas: la identidad, la persistencia y el signaling pasan por Supabase (Auth/API/Realtime).
5. El bucle de emulación tiene prioridad: durante el MVP comparte hilo con la UI, y solo la recepción UDP y los callbacks de audio usan hilos propios. Red, disco, UI y asignaciones de memoria no deben bloquear la ruta crítica del frame.
6. La ROM, la BIOS, los saves y cualquier material con derechos de autor no se almacenan ni se distribuyen en el repositorio. La versión objetivo es Smash Remix v2.0.1; netplay y voz exigen que su CRC coincida. Con un CRC distinto, el cliente entra en modo Offline/Fallback (emulación local sin red, Supabase ni lectura de RAM/voz) y nunca crashea.
7. Definir requisitos y criterios de aceptación medibles (latencia, sincronización, calidad de audio, frame time) antes de implementar.
8. Verificar los cambios con compilación y pruebas apropiadas; no afirmar verificaciones no realizadas.
9. Proteger secretos y datos: sin claves de Supabase, tokens ni credenciales en el repositorio. Las posiciones de los jugadores solo se usan para la voz por proximidad salvo decisión explícita.
10. Respetar la licencia GPLv3 del proyecto y las licencias de los componentes de terceros; no sugerir afiliación con Nintendo.
11. Evitar dependencias, abstracciones y funcionalidades innecesarias. Las dependencias nuevas se añaden por submódulo Git o CMake FetchContent y se aprueban antes de integrarlas.

Cualquier modificación de estos principios debe revisarse y aprobarse por el equipo.
