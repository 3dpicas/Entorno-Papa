# Registro del proyecto — Entorno Digital para Papá

Bitácora de ejecución. Se rellena según `docs/PROTOCOLO.md`. Entradas nuevas SIEMPRE debajo de su bloque; «Estado actual» siempre al día.

---

## 📍 Estado actual

- **Fase:** 🔨 **Bloque 2 en curso** — Tareas 9 y 10 hechas; toca la Tarea 11 (la última del bloque).
- **Siguiente paso:** Bloque 2, Tarea 11 — registrar la ruta `#/guia/...` en `main.js` y verificar el visor en la app real.
- **Repos:** `entorno-app` @ `761d620` (por delante del tag `bloque-1`) · `entorno-contenido` @ `d8d716d` / tag `bloque-0` · raíz: su HEAD sigue avanzando con los commits de documentación.
- **Última sesión:** 2026-07-27 — Bloque 1 entero (parser del manifest, router, pantallas Inicio y Sección, dispatcher con el plugin `opener`, integración con el contenido real) y arranque del Bloque 2 con el parser de guías. 29 tests en verde.
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
  5. **Caché.** El `app.exe` de dev sirve `dist/`, no vite; y WebView2 cachea `index.html` en `%LOCALAPPDATA%\com.marquibel.entorno`. Tras un `npm run build`, si la app no refleja el cambio, borrar esa carpeta.
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
  3. `.boton-siguiente` usa `var(--color-seccion, #2E7D32)`, pero el visor no fija `--color-seccion` (lo hace `renderSeccion` en su propio elemento). Al entrar en una guía el botón saldrá con el verde de reserva, no con el color de la sección de la que viene. Se verá en la Tarea 11.

---

## BLOQUE 3 — Sincronización de contenido

*(sin tareas ejecutadas aún)*

---

## BLOQUE 4 — Distribución e instalación

*(sin tareas ejecutadas aún)*

---

## BLOQUE 5 — Contenido real v1

*(sin tareas ejecutadas aún)*

---

## Post-v1 (mejoras y contenido tras la entrega)

*(vacío)*
