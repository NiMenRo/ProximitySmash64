# Memoria del proyecto

## Estado actual

- Documento central: `docs/CONTEXT.md`. Cada spec vive en `docs/specs/features/HU-XX-nombre/`
  con sus tres archivos; `HU-01-base-local/` es el ejemplo y la primera spec.
- `core_emulador/` es un submódulo de solo lectura del core `mupen64plus-libretro-nx`
  (commit `12edd2c`). Se compila como DLL y se carga con la API Libretro.
- Existe el esqueleto `src/` (`main.cpp` + carpetas por componente) y `tests/`, pero no hay
  implementación ni `CMakeLists.txt`.

## Decisiones tomadas

- Cliente: C++20 + CMake + SDL2 + Dear ImGui + OpenGL. Windows x64 para v0.1–v0.3.
- Dependencias: submódulos Git o `CMake FetchContent`; sin vcpkg por ahora.
- **Test runner:** GoogleTest (gtest) mediante `CMake FetchContent`.
- **DLL del core:** build local con el `Makefile` original del submódulo; en CI se descarga
  temporalmente un release precompilado.
- Proyecto GPLv3; core GPLv2 cargado dinámicamente como biblioteca externa.
- **ROM objetivo:** Smash Remix **v2.0.1** (sustituye a la referencia v1.5.0).
- **v0.1:** se calcula e imprime el CRC de la ROM cargada en consola, sin bloquear la ejecución.
  Ese valor se usará para capturar y registrar el hash definitivo de v2.0.1.
- **CRC estricto:** netplay y chat de proximidad exigirán coincidencia exacta con v2.0.1.
- **Modo Offline/Fallback:** con CRC distinto el cliente no crashea; permite emular en local y
  desactiva red, Supabase y lectura de RAM/voz, mostrando una advertencia en ImGui.
- **TODO v0.3:** offsets de RAM X/Y/Z, búsqueda empírica estrictamente sobre v2.0.1.
- **TODO v0.2+:** topología de 4 jugadores y TURN; v0.2 arranca con 2P Host–Guest.
- **TODO v0.4:** diseño íntegro del esquema de Supabase y sus políticas RLS.
- Hilos: UI + emulación en un solo hilo; recepción UDP y callbacks de audio en hilos aparte.

## Pendiente

- Capturar el CRC real de Smash Remix v2.0.1 con la v0.1 y fijarlo como esperado.
- Definir el release precompilado del core para CI y qué pruebas son posibles sin ROM ni GPU.
- Diseñar la UI del modo Offline/Fallback y de los errores de red.

## Errores que se deben evitar

- No modificar `core_emulador/`; los cambios van por actualización del submódulo, con aprobación.
- No commitear ROMs, BIOS, saves, claves de Supabase ni tokens.
- No bloquear el bucle de emulación con red, disco o audio.
- No asumir offsets de memoria sin verificarlos contra Smash Remix v2.0.1.
- No hacer crashear el cliente por un CRC distinto: debe entrar en modo Offline/Fallback.
- No añadir vcpkg ni dependencias fuera de submódulos/FetchContent sin aprobación.
- No afirmar pruebas o compilaciones no ejecutadas.

## Próximos pasos

- Redactar la spec de v0.1: CMake + GoogleTest, carga de la DLL Libretro, ROM con impresión de
  CRC, vídeo OpenGL e input SDL2.
- Crear `CMakeLists.txt` y el cargador del core cuando la spec esté aprobada.
