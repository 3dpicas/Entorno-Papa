# Registro del proyecto — Entorno Digital para Papá

Bitácora de ejecución. Se rellena según `docs/PROTOCOLO.md`. Entradas nuevas SIEMPRE debajo de su bloque; «Estado actual» siempre al día.

---

## 📍 Estado actual

- **Fase:** 🔨 **Bloque 3 en curso** — Tareas 12, 13, 14 y 15 hechas; toca la Tarea 16 (la última del bloque).
- **Siguiente paso:** Bloque 3, Tarea 16 — log a archivo y pantalla admin oculta (`src/ui/admin.js` + `tests/admin.test.js`, comando `leer_log_reciente`, atajo Ctrl+Shift+A).
- **Repos:** `entorno-app` @ `6ac7107` (por delante del tag `bloque-2`) · `entorno-contenido` @ `d8d716d` / tag `bloque-0` · raíz: su HEAD sigue avanzando con los commits de documentación.
- **GitHub (pendiente del autor):** usuario `3dpicas`. Existe `3dpicas/Entorno-Papa` (público, rama `main`) que hará de repo de documentación; faltan por crear `entorno-contenido` (público) y `entorno-app`, vacíos, para que el primer push no choque. Hasta que existan, el sync y el updater no se pueden probar de verdad.
- **Última sesión:** 2026-07-27/28 — Bloques 1 y 2 completos (manifest, router, Inicio, Sección, enlaces al navegador, parser de guías y visor paso a paso) y Bloque 3 a medias: funciones puras de sync y descarga desde GitHub con `sync_now`. 34 tests JS + 8 tests Rust en verde.
- **Antes de escribir código:** `npm test` (entorno-app) y `npm run check` (entorno-contenido) en verde. Ver los comandos exactos en `CLAUDE.md` § «Comandos del día a día».
- **Entorno ya resuelto:** Node 24.15.0, cargo/rustc 1.97.1 (`stable-x86_64-pc-windows-msvc`), Visual Studio Build Tools 2026, WebView2 Runtime. `cargo test` en `src-tauri` responde `test result: ok. 0 passed` — correcto, todavía no hay código Rust propio que probar.

---

## Hitos de preparación

### Spec aprobada — 2026-07-26

- Decisiones clave: Tauri v2; contenido en repo GitHub público separado; prensa/juegos abren navegador normal; guías Markdown → visor paso a paso; letra ≥24 px; sin autoarranque, acceso directo en escritorio; Windows 11 ambos equipos.
- Documento: `docs/superpowers/specs/2026-07-26-entorno-papa-design.md`

### Plan aprobado — 2026-07-26

- 22 tareas en 6 bloques, TDD, commits por tarea.
- Documento: `docs/superpowers/plans/2026-07-26-entorno-papa.md`

---

## BLOQUE 0 — Scaffolding

### Tarea 1: estructura raíz + app Tauri — HECHA (pasos 1–4 y 6 el 2026-07-26; paso 5 el 2026-07-27)

- **Commits:** `4bb0e01` (raíz: `.gitignore` que ignora los dos repos anidados) · `8f4aa88` (entorno-app: commit raíz del scaffolding) · `d4087de` (entorno-app: `Cargo.lock` del primer build, más la normalización a `features = []` que cargo hace solo en `Cargo.toml`)
- **Verificado:**
  - `npm run build` → `built in 33ms`, genera `dist/index.html` + `dist/assets/index-*.js`.
  - `npm run tauri dev` → compiló 425 crates desde cero y abrió la ventana. Comprobado por API de Win32, no de oídas: `GetWindowPlacement` devolvió `showCmd = 3` (SW_SHOWMAXIMIZED), título exacto `Entorno de Papá`, rectángulo 1550×926 en una pantalla de 1536×960 (el desbordamiento de 7 px por lado es el normal de una ventana maximizada en Windows).
  - Captura de pantalla de la ventana revisada: se ve el texto placeholder «Entorno de Papá» y los acentos salen correctos, lo que confirma de paso que la cadena UTF-8 sobrevive de `main.js` al WebView2.
  - Toolchain con el que se compiló: `cargo 1.97.1`, `rustc 1.97.1`, `stable-x86_64-pc-windows-msvc`, Visual Studio Build Tools 2026.
- **Desviaciones del plan:**
  - `create-vite@9.1.1` genera un demo distinto al que suponía el plan. En vez de `src/counter.js` + `src/javascript.svg` + `public/vite.svg`, genera `src/counter.js`, `src/style.css`, `src/assets/` (hero.png, javascript.svg, vite.svg) y `public/` (favicon.svg, icons.svg). Se borró todo eso; `public/` quedó vacío y se eliminó también. Se quitó el `<link rel="icon">` de `index.html` porque apuntaba al favicon borrado.
  - `npx tauri init` necesitó `--ci` para no quedarse esperando entrada interactiva (la shell de la sesión no tiene stdin).
  - `tauri init` **no** añade el script `tauri` a `package.json`. Se añadió a mano (`"tauri": "tauri"`), imprescindible porque el plan usa `npm run tauri dev`.
  - En `tauri.conf.json` se descartaron las claves generadas `width: 800` / `height: 600` porque contradicen el `minWidth: 1024` que pide el plan. Se conservaron `resizable: true` y `fullscreen: false`.
- **Decisiones nuevas:** ninguna de diseño. Versiones fijadas por el entorno: Node v24.15.0, npm 11.14.1, Vite 8.1.5, tauri-cli 2.11.4, @tauri-apps/api 2.11.1.
- **Pendientes que deja:**
  - `src-tauri/Cargo.toml` se quedó con los valores por defecto de la plantilla: `name = "app"`, `[lib] name = "app_lib"`, `description = "A Tauri App"`, `authors = ["you"]`. El flag `--app-name entorno-papa` de `tauri init` **no** los tocó. No rompe nada (el nombre del ejecutable y del instalador los manda `productName` de `tauri.conf.json`), pero conviene arreglar nombre/descripción/autor al llegar al Bloque 4 (distribución), donde esos metadatos acaban en las propiedades del .exe.
  - La plantilla trae ya `tauri-plugin-log` como dependencia en `Cargo.toml`, pero **no** está registrado en `lib.rs`. El plan lo necesita más adelante.
- **Nota de entorno (importante para retomar):** el PC de desarrollo no tenía Rust ni MSVC Build Tools; se instalaron el 2026-07-27 (`Microsoft.VisualStudio.2022.BuildTools` con la carga `Microsoft.VisualStudio.Workload.VCTools`, y `Rustlang.Rustup`). WebView2 Runtime ya venía de serie. `cargo` **no** está en el PATH de las shells nuevas de la sesión: hay que prefijar `$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"` o abrir una terminal nueva.

### Tarea 2: Vitest operativo — HECHA (2026-07-26)

- **Commits:** `6b3168e` (entorno-app)
- **Verificado:** ciclo TDD completo. Primero `npm test` con el test escrito y sin implementación → `FAIL ... Failed to resolve import "../src/lib/saludo.js"`. Tras crear `src/lib/saludo.js` → `Test Files 1 passed (1)`, `Tests 5 passed (5)`.
- **Desviaciones del plan:** ninguna. Vitest 4.1.10 + jsdom 29.1.1.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:** Ninguno.

### Tarea 3: repo de contenido con validador y CI — HECHA (2026-07-26)

- **Commits:** `d8d716d` (entorno-contenido, commit raíz)
- **Verificado:** `npm run check` → `Contenido OK` (exit 0). Prueba negativa: cambiando la tarjeta guía a `guias/no-existe.md` → `guía no existe: guias/no-existe.md` con `EXIT=1`. Cambio deshecho y `npm run check` de nuevo en verde antes de commitear.
- **Desviaciones del plan:** `npm init -y` genera `type: commonjs`, un script `test` que falla a propósito y campos vacíos (`main`, `keywords`, `author`). Se reescribió `package.json` entero en vez de solo añadir claves: `type: module`, `private: true`, script `check`, descripción real. Después se regeneró `package-lock.json` con `npm install --package-lock-only` para que el `npm ci` del workflow no choque con el `package.json` editado.
- **Decisiones nuevas:** Ninguna. El schema, el manifest, la guía de ejemplo y `check.mjs` son literalmente los del plan (contrato para las Tareas 4–9).
- **Pendientes que deja:** El workflow `check.yml` no se ha ejecutado nunca de verdad: el repo aún no está en GitHub, así que el CI está sin estrenar. `iconos/` e `img/` están vacíos con `.gitkeep`.

## ✅ BLOQUE 0 cerrado (2026-07-27)

- **Resultado verificable de la spec:** §10 → «App vacía compila y abre ventana». Comprobado ejecutando `npm run tauri dev`: la compilación de 425 crates terminó sin errores y la ventana abrió maximizada (`GetWindowPlacement` → `showCmd = 3`) con el título `Entorno de Papá` y el texto placeholder visible en una captura de pantalla. No es «debería funcionar»: está visto funcionando.
- **Estado de los repos:** raíz @ `335373c` / tag `bloque-0` · entorno-app @ `d4087de` / tag `bloque-0` · entorno-contenido @ `d8d716d` / tag `bloque-0`
- **Qué sabe hacer la app ahora:** abre una ventana maximizada en blanco con un texto de prueba. Nada más: no lee el manifest, no pinta tarjetas, no navega. Lo que sí existe es el andamio completo — Tauri v2 compila, Vitest corre tests, y el repo de contenido tiene su schema y su validador funcionando.
- **Deuda/pendientes acumulados:**
  1. Metadatos por defecto en `src-tauri/Cargo.toml` (`name = "app"`, `authors = ["you"]`) — arreglar en Bloque 4.
  2. `tauri-plugin-log` está como dependencia pero sin registrar en `lib.rs`.
  3. Ningún repo está subido a GitHub, así que el workflow `check.yml` sigue sin estrenar y no hay Releases para el updater.
  4. `recursos/`, `scripts/` y `src/styles/` de `entorno-app` no existen todavía (los crean bloques posteriores).
- **Notas para el futuro:**
  - `cargo` no está en el PATH por defecto en esta máquina: `$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"` antes de cualquier comando de Tauri, o terminal nueva.
  - El primer `npm run tauri dev` tarda varios minutos (descarga y compila 425 crates). Los siguientes son rápidos porque `src-tauri/target/` queda cacheado — no borrarlo a la ligera.
  - `create-vite` y `tauri init` generan plantillas que cambian entre versiones: el plan describe la de su fecha, no la que te vas a encontrar. Comparar antes de borrar ficheros del demo.

---

## BLOQUE 1 — Núcleo del panel

### Tarea 4: parser del manifest (JS) — HECHA (2026-07-27)

- **Commits:** `1da03ba` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` con el test escrito y sin implementación → `Failed to resolve import "../src/lib/manifest.js"`, `Test Files 1 failed | 1 passed (2)`. Tras crear `src/lib/manifest.js` → `Test Files 2 passed (2)`, `Tests 11 passed (11)`.
- **Desviaciones del plan:** Ninguna. El test y la implementación son literalmente los del plan.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:** El parser no valida `url`/`guia` de cada tarjeta (eso lo cubre el schema Ajv del repo de contenido, no la app). La app confía en que el contenido pasó el CI; si llegara un manifest con una tarjeta `enlace` sin `url`, `parsearManifest` la dejaría pasar y el fallo saldría al pulsar. Aceptable en v1 porque el contenido es propio y validado en origen.

### Tarea 5: router hash + pantalla Inicio — HECHA (2026-07-27)

- **Commits:** `e7d83b3` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` con los dos tests escritos y sin implementación → `Failed to resolve import "../src/ui/inicio.js"` y `Failed to resolve import "../src/lib/router.js"`, `Test Files 2 failed | 2 passed (4)`. Tras crear `src/lib/router.js` y `src/ui/inicio.js` → `Test Files 4 passed (4)`, `Tests 17 passed (17)`.
- **Desviaciones del plan:** Ninguna. Router e Inicio son literalmente los del plan.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. `iniciarRouter` registra un listener de `hashchange` en cada llamada y nunca lo quita. En la app da igual (se llama una sola vez al arrancar), pero en los tests los listeners se acumulan entre casos; hoy no molesta porque `resolver` lee siempre el array `rutas` actual y las aserciones son síncronas. Si un test futuro se vuelve asíncrono, hará falta un `pararRouter()`.
  2. `renderInicio` arranca un `setInterval` de 30 s para el reloj y no lo cancela. Solo se monta una vez, así que no hay fuga real; a vigilar si el Inicio pasa a re-renderizarse.
  3. Todavía no hay CSS: las clases (`tarjeta-seccion`, `saludo`, `reloj`, `parrilla-secciones`) existen sin estilos hasta la Tarea 8.

### Tarea 6: pantalla Sección — HECHA (2026-07-27)

- **Commits:** `cf19a7a` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` con el test escrito y sin implementación → `Failed to resolve import "../src/ui/seccion.js"`, `Test Files 1 failed | 4 passed (5)`. Tras crear `src/ui/seccion.js` → `Test Files 5 passed (5)`, `Tests 21 passed (21)`.
- **Desviaciones del plan:** Ninguna.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:** El emoji 🏠 del botón Inicio va incrustado en el JS, no en el CSS ni en el contenido. Si algún día hay que cambiar el icono habrá que tocar `seccion.js` y `guia.js` (Tarea 10 usa el mismo botón).

### Tarea 7: dispatcher de acciones + plugin opener — HECHA (2026-07-27)

- **Commits:** `fb90941` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` sin implementación → `Failed to resolve import "../src/lib/acciones.js"`, `Test Files 1 failed | 5 passed (6)`. Tras crear `src/lib/acciones.js` → `Test Files 6 passed (6)`, `Tests 24 passed (24)`. `npm run tauri add opener` terminó añadiendo `tauri-plugin-opener 2.5.4` a `Cargo.toml`, `@tauri-apps/plugin-opener@^2.5.4` a `package.json`, `.plugin(tauri_plugin_opener::init())` a `lib.rs` y el permiso a `capabilities/default.json` (luego sustituido por el de alcance).
- **Desviaciones del plan:**
  - `npm run tauri add opener` dejó un fichero basura `src-tauri/2` (139 bytes con la salida de `npm install`): el CLI lanza el `npm install` en una shell donde el `2>&1` acabó creando un fichero llamado `2`. Borrado antes de commitear.
  - El mismo comando ejecuta `cargo fmt` al final, y eso **reformateó `build.rs`, `main.rs` y `lib.rs`** de 2 espacios (estilo de la plantilla de Tauri) a los 4 de rustfmt. Se conserva el reformateo, que es el estándar de Rust; por eso el commit toca ficheros que la tarea no menciona.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. **Sin verificar en ejecución:** el alcance `{"url": "https://*"}` es un glob, y está por confirmar si casa con URLs que llevan ruta (`https://www.solitr.com/es`, `https://tetris.com/play-tetris`) o solo con el dominio pelado. Se comprueba en el checklist manual de la Tarea 8; si falla, hay que ampliar el patrón (p. ej. `https://**`) y anotarlo.
  2. Corrección a la deuda anotada al cerrar el Bloque 0: `tauri-plugin-log` **sí** está registrado en `lib.rs`, dentro de `.setup()` y bajo `cfg!(debug_assertions)`; es decir, funciona en dev pero no en la build de producción. Que funcione en producción es lo que hará falta más adelante (§ del plan sobre logs), no el registro en sí.

### Tarea 8: estilos accesibles + integración con contenido real (dev) — HECHA (2026-07-27)

- **Commits:** `5993372` (entorno-app)
- **Verificado:**
  - `npm test` → `Test Files 6 passed (6)`, `Tests 24 passed (24)`. `npm run build` → `built in 95ms`. `cargo check` sin warnings; `cargo test` → `test result: ok. 0 passed` (aún no hay código Rust propio con tests).
  - App ejecutándose (`app.exe` del perfil dev, con vite aparte). Ventana `Entorno de Papá`, `showCmd = 3` (maximizada), **1938×1158** físicos.
  - **Inicio:** captura con saludo «Buenas noches, Papá» (a las 22:57, coherente con `saludo()`), reloj `22:57` y las 3 tarjetas de sección con sus colores del manifest (#2E7D32 verde, #1565C0 azul, #E65100 naranja).
  - **Sin scroll horizontal:** medido dentro de la página, `scrollWidth = clientWidth = 1536`; parrilla 1472 px y tarjeta 469 px, que es exactamente lo que predice el CSS (3×469 + 2×32 = 1471).
  - **Navegación** (clics reales en el DOM vía CDP): pulsar «Prensa» → `hash=#/seccion/prensa`, `h1=Prensa`, grupos `Nacional,Deportes`, tarjetas `El País,El Mundo,Marca`. Pulsar «🏠 Inicio» → `hash=#/`, vuelve el saludo y las 3 secciones.
  - **Enlaces al navegador:** pulsar la tarjeta «El País» abrió una pestaña nueva en el Edge del usuario; su título pasó de `...y 14 páginas más` a `EL PAÍS: el periódico global y 15 páginas más`. Además, invocando el comando del plugin directamente, `https://elpais.com` y `https://www.solitr.com/es` devolvieron ambos OK.
  - **Contenido real:** todo lo anterior sale de `entorno-contenido/manifest.json` leído por el comando Rust `contenido_leer`, no de datos de prueba.
- **Desviaciones del plan:**
  - **`protocol-asset` era obligatorio y el plan no lo dice.** Con el `assetProtocol` del Step 2 pero sin la feature, `cargo check` falla: `The tauri dependency features on the Cargo.toml file does not match the allowlist defined under tauri.conf.json. Please run tauri dev or tauri build or add the protocol-asset feature`. Se puso `tauri = { version = "2.11.3", features = ["protocol-asset"] }`.
  - `lib.rs` **no** se sustituyó por el del plan: se conservó el bloque `.setup()` con `tauri-plugin-log` que ya traía la plantilla y se le añadieron `mod config; mod contenido;` y el `invoke_handler`. El plan proponía un `lib.rs` que habría borrado el logger.
  - En `contenido.rs`, `use tauri::Manager;` se movió dentro de la rama `#[cfg(not(debug_assertions))]`: en dev no se usa y dejaba un warning de import sin usar.
  - `config.rs` lleva `#[allow(dead_code)]` en `GITHUB_OWNER` y `REPO_CONTENIDO` con un comentario que explica que los usa el sync del Bloque 3. Sin eso, `cargo check` suelta dos warnings en cada compilación.
  - `index.html` figuraba como «Modify» en la tarea, pero ya tenía lo necesario del Bloque 0 (lang, título, `#app`, script module). No se tocó.
- **Decisiones nuevas:** Ninguna de diseño.
- **Cómo verificar la UI sin fiarse de la vista (aprendido a golpes, vale para las tareas 10 y 16):**
  1. **DPI.** El monitor es 1920×1200 al 125%. PowerShell 5.1 es *DPI-unaware*, así que Win32 le devuelve coordenadas lógicas (ventana «1550×926») de una ventana que en realidad mide 1938×1158. Capturar con esas medidas recorta el 20 % derecho: eso me hizo dar por bueno un desbordamiento horizontal que **no existía**. Hay que llamar a `SetProcessDpiAwarenessContext(-4)` al principio de cada llamada de PowerShell (cada llamada es un proceso nuevo).
  2. **Captura.** `CopyFromScreen` fotografía lo que haya encima (me sacó el navegador del usuario). Usar `PrintWindow(h, hdc, PW_RENDERFULLCONTENT=2)`, que le pide el contenido a la propia ventana.
  3. **Clics.** Los clics sintéticos (`mouse_event`) **no** son fiables aquí: la ventana de Claude Code recupera el foco y los clics caen en el escritorio del usuario. Lo que sí funciona: lanzar la app con `WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS=--remote-debugging-port=9222` y hablar por CDP (`http://127.0.0.1:9222/json` + WebSocket, `Runtime.evaluate`). Así se pulsan botones por selector y se leen `location.hash`, textos y medidas. Script de apoyo en el scratchpad de la sesión (`uiauto.ps1`, `cdp.ps1`).
  4. **Ciclo de vida.** `npm run tauri dev` muere en cuanto la shell cierra stdin. Para tener la app viva entre comandos: `npm run dev` (vite) por un lado y `target\debug\app.exe` por otro, cada uno como tarea de fondo.
  5. **Caché.** WebView2 cachea agresivamente en `%LOCALAPPDATA%\com.marquibel.entorno`; si un cambio del frontend no aparece, borrar esa carpeta y reabrir. (Corrección hecha en la Tarea 11: el `app.exe` de dev **sí** carga `http://localhost:5173`, o sea vite, no `dist/`. Sin vite levantado, la ventana muestra `ERR_CONNECTION_REFUSED`. Lo que me despistó en la Tarea 8 fue la caché, no la fuente.)
- **Pendientes que deja:**
  1. Pulsar una tarjeta de guía lleva a `#/guia/<ruta>`, el router no tiene esa ruta y redirige a Inicio. Comprobado. Es lo esperado hasta la Tarea 10.
  2. `contenido_ruta` devuelve la ruta canonicalizada de Windows, que en rutas de red o largas lleva el prefijo `\\?\`. Está por ver si `convertFileSrc` y el `scope` del `assetProtocol` lo tragan; se sabrá en la Tarea 10, la primera que carga imágenes.
  3. Contenido: `https://www.solitr.com/es` responde **404** y `https://tetris.com/play-tetris` responde 308 (redirección, los navegadores la siguen). Son URLs de ejemplo del Bloque 0; hay que sustituir la de solitr al llegar al contenido real (Bloque 5).
  4. La app arranca sin manejo de errores: si `cargarManifest()` fallara, la pantalla se quedaría en blanco. La spec §8 dice que el padre nunca ve errores; el degradado silencioso está previsto para el Bloque 3 junto con la caché.

## ✅ BLOQUE 1 cerrado (2026-07-27)

- **Resultado verificable de la spec:** §10 → «Panel navegable con prensa y juegos reales». Comprobado con la app en marcha y contenido real de `entorno-contenido`: Inicio pinta saludo, reloj y las 3 secciones del manifest; pulsar «Prensa» lleva a `#/seccion/prensa` con los grupos Nacional y Deportes y sus tarjetas; «🏠 Inicio» vuelve a `#/`; y pulsar «El País» abrió una pestaña nueva en el navegador del sistema (título `EL PAÍS: el periódico global`, contador de pestañas de Edge 14 → 15). Nada de esto es «debería funcionar»: son capturas y valores leídos de la página en ejecución.
- **Estado de los repos:** entorno-app @ `5993372` / tag `bloque-1` · entorno-contenido @ `d8d716d` / tag `bloque-0` (el Bloque 1 no lo tocó)
- **Qué sabe hacer la app ahora:** abre maximizada, lee el `manifest.json` del repo de contenido y pinta una pantalla de inicio con saludo según la hora, reloj y una tarjeta grande por sección. Al pulsar una sección muestra sus tarjetas (agrupadas si el manifest lo dice) y un botón de Inicio para volver. Las tarjetas de tipo enlace abren la web en el navegador de siempre. Las de guía todavía no llevan a ningún sitio: eso es el Bloque 2.
- **Deuda/pendientes acumulados:**
  1. Metadatos por defecto en `src-tauri/Cargo.toml` (`name = "app"`, `authors = ["you"]`) — arreglar en Bloque 4.
  2. `tauri-plugin-log` solo está activo en dev (`cfg!(debug_assertions)`); para producción hay que registrarlo también.
  3. Ningún repo está subido a GitHub: el workflow `check.yml` sigue sin estrenar y no hay Releases para el updater.
  4. Sin manejo de errores en el arranque: si el manifest no cargara, pantalla en blanco (spec §8 lo prohíbe; toca en el Bloque 3).
  5. `contenido_ruta` devuelve rutas canonicalizadas (`\\?\`) sin verificar contra `convertFileSrc` ni contra el `scope` del `assetProtocol` — primera prueba real en la Tarea 10.
  6. `https://www.solitr.com/es` da 404 en el manifest de ejemplo.
- **Notas para el futuro:**
  - Verificar la interfaz por CDP (`--remote-debugging-port=9222`), no con clics sintéticos ni capturas de pantalla a ciegas. Está detallado en la entrada de la Tarea 8; ahorra la hora que costó averiguarlo.
  - Todo comando de PowerShell que mida o capture ventanas tiene que declararse DPI-aware antes de nada, o las cifras mienten en un 25 % en esta máquina.
  - `npm run tauri add <plugin>` ejecuta `cargo fmt` y reformatea ficheros que no son suyos, y deja un fichero basura llamado `2` en `src-tauri/`. Revisar `git status` antes de commitear.

---

## BLOQUE 2 — Visor de guías

### Tarea 9: parser de guías Markdown — HECHA (2026-07-27)

- **Commits:** `a246b99` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` sin implementación → `Failed to resolve import "../src/lib/guia.js"`, `Test Files 1 failed | 6 passed (7)`. Tras crear `src/lib/guia.js` → `Test Files 7 passed (7)`, `Tests 29 passed (29)`. `marked` 16.x instalado como dependencia de producción.
- **Desviaciones del plan:** Ninguna.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. El HTML de cada paso sale de `marked` **sin sanear** y la Tarea 10 lo mete con `innerHTML`. No es un agujero hoy porque el contenido es propio y pasa por el CI del repo de contenido, pero conviene recordarlo si alguna vez se aceptan guías de terceros.
  2. El frontmatter se parsea a mano (`clave: valor` por línea), no con YAML de verdad: no admite listas, anidamiento ni comillas. Suficiente para `titulo` e `icono`.
  3. El troceo exige literalmente `## Paso ...`; un `### Paso` o un `## Etapa` se ignoran en silencio. Es el contrato acordado en la Tarea 3 y el validador del repo de contenido lo comprueba.

### Tarea 10: visor asistente paso a paso — HECHA (2026-07-27)

- **Commits:** `761d620` (entorno-app)
- **Verificado:** ciclo TDD completo. `npm test` sin implementación → `Failed to resolve import "../src/ui/guia.js"`, `Test Files 1 failed | 7 passed (8)`. Tras crear `src/ui/guia.js` y añadir los estilos del visor a `base.css` → `Test Files 8 passed (8)`, `Tests 34 passed (34)`. Los cinco casos cubren: primer paso e indicador, Siguiente/Anterior, Anterior oculto en el paso 1, «✔ Terminar» en el último paso navegando a `#/`, botón Inicio, y resolución de `src` de imágenes.
- **Desviaciones del plan:** Ninguna.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. Sin verificar en la app real: eso es la Tarea 11, que registra la ruta `#/guia/...`. Hasta entonces el visor solo existe para los tests.
  2. Las imágenes se resuelven de forma asíncrona tras pintar el paso, así que hay un instante con el `src` relativo (roto). El `addEventListener('error', ...)` borra la imagen si no carga — degradado silencioso, coherente con la spec §8, pero significa que un fallo de ruta se traduce en «la captura desaparece» sin más aviso que el log.
  3. `.boton-siguiente` usa `var(--color-seccion, #2E7D32)`, pero el visor no fija `--color-seccion` (lo hace `renderSeccion` en su propio elemento). Medido en la Tarea 11: el botón sale `rgb(69, 90, 100)`, es decir `#455A64` —el valor que `tokens.css` da a `--color-seccion` en `:root`—, no el verde de reserva, que nunca llega a usarse. El botón no hereda el color de la sección de la que viene la guía; queda así a propósito hasta que alguien decida lo contrario.

### Tarea 11: integrar el visor en la app — HECHA (2026-07-27)

- **Commits:** `be17c9d` (entorno-app)
- **Verificado:** `npm test` → `Test Files 8 passed (8)`, `Tests 34 passed (34)` (sin regresiones). En la app en marcha, pilotada por CDP:
  - Inicio → «Aprender» → tarjeta «Enviar un correo»: `hash=#/guia/guias%2Fejemplo-correo.md`, `h1=Enviar un correo`, `Paso 1: Abre el navegador`, indicador `Paso 1 de 3`, botón `Siguiente ➡`, Anterior oculto.
  - Siguiente → `Paso 2 de 3` / `Paso 2: Entra en Gmail`, Anterior ya visible, botón Inicio presente. Anterior → vuelve a `Paso 1 de 3`.
  - Dos veces Siguiente → `Paso 3 de 3`, `Paso 3: Pulsa "Redactar"`, el botón pasa a `✔ Terminar`. Pulsarlo → `hash=#/`, saludo y las 3 secciones otra vez.
  - Captura del visor en el paso 3 revisada: cabecera con «🏠 Inicio» y el título de la guía, texto del paso, y el pie con Anterior / «Paso 3 de 3» / Terminar.
  - **Guía inexistente:** `#/guia/guias%2Fno-existe.md` → el `catch` registra el fallo en consola y devuelve a `#/`; el visor no llega a montarse y el padre no ve ningún error. Es el comportamiento que pide la spec §8.
  - El texto sale de `guias/ejemplo-correo.md` del repo de contenido leído por `contenido_leer`, no de datos de prueba.
- **Desviaciones del plan:** Ninguna en el código. En la verificación sí: el plan dice `npm run tauri dev`, pero ese comando muere al cerrarse stdin en esta shell, así que se levantó vite y `app.exe` por separado y se comprobó por CDP (ver las notas de la Tarea 8).
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. **`resolverImagen` sigue sin probarse contra ficheros reales:** la guía de ejemplo no tiene imágenes, así que el camino `contenido_ruta` → `convertFileSrc` → `assetProtocol` no se ha ejercitado nunca. Sigue vivo el riesgo del prefijo `\\?\` de `canonicalize`. Hará falta una guía con captura (Bloque 5, o una prueba suelta antes).
  2. El visor no recuerda por qué sección se entró: «Terminar» y «Inicio» llevan los dos a `#/`, nunca de vuelta a la sección. Es lo que dice el plan; anotado por si al usarlo se ve incómodo.

## ✅ BLOQUE 2 cerrado (2026-07-27)

- **Resultado verificable de la spec:** §10 → «Una guía de ejemplo completa funcionando». Comprobado con `guias/ejemplo-correo.md` del repo de contenido: se abre desde su tarjeta, recorre los 3 pasos hacia delante y hacia atrás, el botón final dice «✔ Terminar» y devuelve al inicio. Valores leídos de la página en ejecución, más una captura del visor.
- **Estado de los repos:** entorno-app @ `be17c9d` / tag `bloque-2` · entorno-contenido @ `d8d716d` / tag `bloque-0` (el Bloque 2 tampoco lo tocó)
- **Qué sabe hacer la app ahora:** todo lo del Bloque 1, y además abrir guías. Al pulsar una tarjeta de guía aparece un asistente a pantalla completa que enseña un paso cada vez, con botones grandes de Anterior y Siguiente, un «Paso N de M» y un botón de Inicio siempre a la vista. Si la guía no existe o está rota, vuelve al inicio sin decir nada.
- **Deuda/pendientes acumulados:**
  1. Metadatos por defecto en `src-tauri/Cargo.toml` (`name = "app"`, `authors = ["you"]`) — Bloque 4.
  2. `tauri-plugin-log` solo activo en dev.
  3. Ningún repo subido a GitHub: `check.yml` sin estrenar, sin Releases para el updater.
  4. Sin manejo de errores en el arranque (`cargarManifest`): pantalla en blanco si fallara — Bloque 3.
  5. **El camino de imágenes de las guías sigue sin ejercitarse** (`contenido_ruta` devuelve rutas con prefijo `\\?\`; `convertFileSrc` y el `scope` del `assetProtocol` están por validar).
  6. `https://www.solitr.com/es` da 404 en el manifest de ejemplo.
  7. El HTML de las guías va a `innerHTML` sin sanear (aceptable mientras el contenido sea propio y pase el CI).
- **Notas para el futuro:**
  - Para verificar en la app: levantar `npm run dev` y `target\debug\app.exe` por separado (con `WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS=--remote-debugging-port=9222`) y pilotar por CDP. `npm run tauri dev` no sobrevive a la shell.
  - El binario de dev carga vite, no `dist/`: sin vite levantado la ventana muestra `ERR_CONNECTION_REFUSED`.

---

## BLOQUE 3 — Sincronización de contenido

### Tarea 12: validación y swap atómico (Rust, funciones puras) — HECHA (2026-07-27)

- **Commits:** `f200d29` (entorno-app)
- **Verificado:** ciclo TDD completo, esta vez en Rust. `cargo test` con solo el módulo de tests → `error[E0425]: cannot find function 'validar_contenido' in this scope` (y lo mismo para `reemplazar_contenido`, `leer_meta`, `guardar_meta`, `MetaContenido`). Tras implementar → `test result: ok. 7 passed; 0 failed`, con los 7 casos: manifest válido devuelve `version=7`, manifest roto y guía ausente dan error (el mensaje incluye `no-existe.md`), swap con y sin contenido previo, y meta de ida y vuelta más meta ausente que devuelve el `Default`. `npm test` sigue en 34.
- **Desviaciones del plan:** `sync.rs` lleva `#![allow(dead_code)]` con un comentario explicando que quien usará estas funciones es el `sync_now` de la Tarea 13. Sin él, `cargo test` suelta 6 warnings de «never used» en cada compilación, porque de momento solo las llaman los tests.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. **`validar_contenido` valida menos que el CI del repo de contenido.** Aquí solo se comprueba que el JSON parsea, que hay `version` y `secciones`, y que los ficheros de guía existen. El `check.mjs` del repo de contenido además valida contra el schema Ajv (patrones de `id`, formato de color, `url` obligatoria en las tarjetas de enlace, exclusión mutua de `tarjetas`/`grupos`) y comprueba los iconos. Es una red de seguridad de última hora, no el validador de verdad: si alguien sube contenido saltándose el CI, la app lo aceptaría.
  2. `reemplazar_contenido` hace dos `rename` y no es atómico de verdad: si el proceso muere justo entre apartar lo viejo y activar lo nuevo, queda `contenido.old` y ningún `contenido`. El rollback cubre el fallo del segundo `rename`, no un corte de luz. Aceptable porque el primer arranque repuebla desde la semilla (Tarea 14).
  3. `chrono` está en `Cargo.toml` pero todavía no se usa: la fecha de la meta la rellena la Tarea 13.
  4. **Flake de entorno:** el primer `cargo test` falló con `LINK : fatal error LNK1104: no se puede abrir el archivo ...\app-*.exe` (binario de test aún bloqueado por la ejecución anterior). Repetir el comando lo arregló; no hay que tocar nada.

### Tarea 13: descarga desde GitHub y comando sync_now — HECHA (2026-07-28)

- **Commits:** `6f5cb27` (entorno-app)
- **Verificado:**
  - TDD: `cargo test` con el test de `extraer_zip` y sin implementación → `error[E0425]: cannot find function 'extraer_zip' in this scope`. Tras implementar → `test result: ok. 8 passed; 0 failed`. `cargo check` sin errores ni warnings con `reqwest 0.12.28` y `zip 2` compilados.
  - **Comandos registrados de verdad, no solo compilando:** con la app en marcha, invocados desde la página por CDP → `estado_sync` devuelve `{"estado":"sin_datos","sha":"","version":0,"fecha":"","detalle":null}` (aún no hay meta guardada) y `sync_now` devuelve `{"estado":"dev",...,"detalle":"sync desactivado en dev"}`, que es exactamente la salida prevista para una build de depuración.
- **Desviaciones del plan:** el plan avisaba de usar `eprintln!` en vez de `log::warn!`/`log::info!` hasta la Tarea 16. No hizo falta: el crate `log` ya es dependencia desde el scaffolding, así que las macros compilan. Lo que la Tarea 16 tiene que resolver es que haya un logger instalado en producción (hoy `tauri-plugin-log` solo se registra bajo `debug_assertions`), no las llamadas.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. **La sincronización real no se ha ejercitado nunca.** `sync_now` corta en seco en dev y `entorno-contenido` no está en GitHub, así que `obtener_sha_remoto`, `descargar_zip_remoto` y el camino completo de `sincronizar` no han hecho una sola petición. La primera prueba de verdad exige subir el repo de contenido y una build de release: es el mayor riesgo abierto del proyecto.
  2. Las URLs de GitHub llevan la rama `main` incrustada (`/commits/main`, `/zip/refs/heads/main`). Los repos locales están en `master`. Al subirlos hay que renombrar la rama a `main` o cambiar estas dos cadenas; si no, el sync fallará silenciosamente con un 404.
  3. `sincronizar` descarga el zip entero en memoria (`Vec<u8>`) antes de extraerlo. Con un repo de contenido pequeño da igual; si algún día se llena de imágenes, conviene ir a fichero temporal.
  4. `extraer_zip` llama a `ZipArchive::extract`, que no protege contra entradas con rutas tipo `../` (zip slip). La fuente es un zip generado por GitHub a partir de un repo propio, así que el riesgo es teórico, pero queda dicho.
  5. `estado_sync` devuelve `sin_datos` con `version: 0`; la pantalla de admin (Tarea 15) tendrá que distinguir ese caso del de un contenido de versión 0.

### Tarea 14: contenido semilla para el primer arranque — HECHA (2026-07-28)

- **Commits:** `f3b3f5f` (entorno-app) · antes, `e4f3046` corrige el usuario de GitHub (ver «Decisiones nuevas»)
- **Verificado:**
  - `npm run semilla` → `Semilla actualizada desde entorno-contenido`, con `recursos/contenido-semilla/manifest.json`, `guias/ejemplo-correo.md`, `schema/`, `iconos/`, `img/`.
  - TDD: `cargo test` sin implementación → `error[E0425]: cannot find function 'copiar_dir'`. Tras implementar → `test result: ok. 9 passed; 0 failed`.
  - `npm run tauri build` genera los dos instaladores: `Entorno de Papa_0.1.0_x64-setup.exe` (2,58 MB, NSIS) y `Entorno de Papa_0.1.0_x64_en-US.msi` (4,01 MB, WiX; el CLI se descargó solo WiX 3.14 y NSIS 3.11 la primera vez).
  - **Primer arranque de verdad:** borrado `%APPDATA%\com.marquibel.entorno\contenido` y ejecutado el binario de producción. La carpeta se recreó sola con `manifest.json`, `guias/ejemplo-correo.md` y el resto de la semilla, y la ventana pintó el panel completo —saludo «Buenas noches, Papá», reloj `03:06` y las 3 tarjetas de sección con sus colores— sin repo de desarrollo ni red. Captura revisada.
  - La instalación del `.exe` en la máquina queda para el autor; lo verificado aquí es el mismo binario que el instalador empaqueta.
- **Desviaciones del plan:**
  1. **El glob `../recursos/contenido-semilla/**` no vale:** el build script aborta con `glob pattern ../recursos/contenido-semilla/** path not found or didn't match any files`. Hay que escribir `../recursos/contenido-semilla/**/*`.
  2. **La ruta de la semilla dentro del paquete no es la del plan.** Tauri mete los recursos declarados con `../` bajo `_up_/`, así que quedan en `<recursos>/_up_/recursos/contenido-semilla`. El `resolve("recursos/contenido-semilla", Resource)` del plan nunca los habría encontrado y el primer arranque se habría quedado sin contenido. Ahora se buscan las dos rutas (`_up_/...` y la del plan) y se elige la que tenga `manifest.json`, para no depender de esa convención.
  3. El plan añadía un `.setup()` nuevo al builder; se metió la llamada dentro del `.setup()` que ya existía, porque Tauri solo admite uno y el segundo habría sustituido al del logger.
  4. `copiar_dir` lleva `#[cfg_attr(debug_assertions, allow(dead_code))]`: en dev nadie la llama, porque `asegurar_contenido_inicial` solo compila su cuerpo en release.
  5. `asegurar_contenido_inicial` usa `log::warn!`/`log::info!` en vez del `eprintln!` del plan, por lo mismo que en la Tarea 13.
- **Decisiones nuevas:**
  - **Usuario de GitHub: `3dpicas`, no `marquib3l`.** Confirmado con el autor y con la API (`marquib3l` da 404; existe `3dpicas/Entorno-Papa`, público, rama por defecto `main`). `GITHUB_OWNER` cambiado en `config.rs` (`e4f3046`) y corregido el plan. `marquib3l` sigue siendo la identidad de los commits: son dos cosas distintas.
  - **Reparto de repos, decidido por el autor:** los tres del plan. `3dpicas/Entorno-Papa` = documentación (este repo raíz); quedan por crear `entorno-contenido` (público) y `entorno-app`.
- **Pendientes que deja:**
  1. La semilla arrastra `package.json`, `package-lock.json` y `scripts/check.mjs` del repo de contenido, que no le sirven de nada a la app. El filtro del script solo excluye `node_modules` y `.git`. Son unos pocos KB; se puede afinar cuando moleste.
  2. **La semilla no se regenera sola:** si alguien edita `entorno-contenido` y compila sin acordarse de `npm run semilla`, el instalador sale con contenido viejo. Conviene encadenarlo al `beforeBuildCommand` en el Bloque 4.
  3. **Trampa importante para el futuro:** `cargo build --release` **no** produce un binario de producción. `tauri-build` sigue poniendo `cfg(dev)`, así que la ventana intenta cargar `http://localhost:5173` y muestra `ERR_CONNECTION_REFUSED`, mientras que `debug_assertions` sí está apagado (o sea, el contenido se lee de appdata). Para probar producción hay que usar `npm run tauri build`.
  4. Otra trampa: si hay un `app.exe` abierto, `tauri build` falla con `error: failed to remove file ...\target\release\app.exe / Acceso denegado. (os error 5)`. Cerrar la app antes de compilar.
  5. Al lanzar el ejecutable desde una shell no interactiva, la ventana abre **minimizada** (`showCmd = 2`) pese al `maximized: true`. Es cosa del `STARTUPINFO` heredado, no de la configuración; desde un acceso directo normal abre maximizada. Para capturarla hay que llamar antes a `ShowWindow(h, 3)`.
  6. El proceso lanzado en segundo plano muere cuando termina la shell que lo lanzó: para inspeccionarlo hay que arrancarlo y capturarlo en la misma llamada.

### Tarea 15: bucle de sync en el frontend + indicador discreto — HECHA (2026-07-28)

- **Commits:** `6ac7107` (entorno-app)
- **Verificado:**
  - TDD: `npm test` sin implementación → `Failed to resolve import "../src/ui/indicador.js"`. Tras implementar, un test seguía en rojo (ver desviaciones); arreglada la implementación → `Test Files 9 passed (9)`, `Tests 36 passed (36)`.
  - Con la app en marcha (dev, por CDP): el indicador **existe en el DOM y su texto está vacío**, que es justo lo que toca en desarrollo. La cadena es `sync_now` → `{"estado":"dev",...,"version":null}` → como no trae versión, se consulta `estado_sync` → `{"estado":"sin_datos","version":0,...}` → `renderIndicador` no pinta nada porque `version` es 0. Las 3 secciones siguen renderizando y la navegación no se altera.
  - Ambos comandos invocados desde la página responden sin lanzar: son exactamente los dos que hace `sincronizar()`.
- **Desviaciones del plan:**
  - **La implementación del plan no pasaba su propio test.** `new Date(fecha).toLocaleDateString('es-ES')` devuelve `26/7/2026`, y el test pide `26/07/2026`. Se arregló la implementación (no el test) pasando `{ day: '2-digit', month: '2-digit', year: 'numeric' }`.
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. **No se instrumentó la captura de errores de consola.** El criterio del plan («consola sin errores») se comprobó de forma indirecta: la app renderiza, y los dos `invoke` que hace el bucle responden bien. Para capturar `console.error` de verdad haría falta ampliar el cliente CDP con `Page.addScriptToEvaluateOnNewDocument`.
  2. El indicador solo se repinta si ya había uno (`previo?.replaceWith(...)`). Como se añade siempre al arrancar, hoy funciona; si algún día se monta condicionalmente, el primer refresco se perdería.
  3. **El refresco tras «actualizado» no se ha probado nunca**, porque en dev nunca llega ese estado. `manifest = await cargarManifest()` + `dispatchEvent(new HashChangeEvent('hashchange'))` está escrito pero no ejercitado; si el usuario estuviera dentro de una guía en ese momento, el re-render la reiniciaría en el paso 1.
  4. El `setInterval` de 6 h no se cancela nunca (no hace falta: vive lo que vive la ventana).

---

## BLOQUE 4 — Distribución e instalación

*(sin tareas ejecutadas aún)*

---

## BLOQUE 5 — Contenido real v1

*(sin tareas ejecutadas aún)*

---

## Post-v1 (mejoras y contenido tras la entrega)

*(vacío)*
