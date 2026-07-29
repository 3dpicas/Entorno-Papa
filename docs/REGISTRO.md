# Registro del proyecto — Entorno Digital para Papá

Bitácora de ejecución. Se rellena según `docs/PROTOCOLO.md`. Entradas nuevas SIEMPRE debajo de su bloque; «Estado actual» siempre al día.

---

## 📍 Estado actual

- **Fase:** 🔨 **Bloques 4 y 5 completos salvo lo presencial.** Las 22 tareas del plan están ejecutadas; lo que queda no es código.
- **Siguiente paso: todo lo pendiente es del autor y necesita estar delante de los equipos.** Por orden de importancia:
  1. Instalar en el segundo PC del padre (Tarea 19 Step 3) → cierra el Bloque 4, tag `bloque-4`.
  2. **Ver al padre usarlo diez minutos sin ayuda** (Tarea 22 Step 3): es la prueba de aceptación real de la v1, y lo único que puede decir si esto sirve de algo.
  3. Las 35 capturas de las guías (Tarea 20 Step 2) y una lectura del texto de las guías.
  4. Abrir las 13 URLs en un navegador de verdad: `curl` confirma que responden, no que la portada sea la correcta ni que no pidan registro.
- **Repos:** `entorno-app` @ `c6284eb` / tags `bloque-3`, `v0.1.0`, `v0.1.1`, `v0.1.2` · `entorno-contenido` @ `0872dc0` (contenido v6) / tag `bloque-0` · `Entorno-Papa` (docs) al día. Los tres en GitHub bajo `3dpicas`, rama `main`.
- **Primer PC del padre ya instalado** (2026-07-28): v0.1.1 funcionando; el único fallo que encontró usándola fue el enlace del solitario, ya corregido en el contenido v3. Queda el segundo equipo.
- **Entorno del padre (condiciona el contenido):** Gmail, Brave instalado y por defecto, móvil Android, prensa local de Burgos.
- **Releases publicadas:** `v0.1.0`, `v0.1.1` y `v0.1.2` en `3dpicas/entorno-app`, con instalador, firma y `latest.json`. **Instalador para llevar al segundo PC:** el `*-setup.exe` de la última release, en `https://github.com/3dpicas/entorno-app/releases`.
- **Clave de firma:** privada en `%USERPROFILE%\.tauri\entorno-papa.key`, contraseña en el gestor del autor, ambas también como secretos de `3dpicas/entorno-app`. Si se pierden, los PCs del padre dejan de aceptar actualizaciones y hay que reinstalar a mano.
- **GitHub:** ✅ los tres repos publicados en `3dpicas` (`Entorno-Papa`, `entorno-app`, `entorno-contenido`), públicos, rama `main`, con los tags de bloque subidos. Sync real verificado de punta a punta y CI en verde — ver «Publicación en GitHub y prueba del sync real» en el Bloque 3.
- **Última sesión:** 2026-07-28/29 — Bloques 4 y 5 enteros salvo lo presencial: icono e instalador NSIS en español, auto-actualización firmada (vista pasar sola de v0.1.0 a v0.1.1 y de v0.1.1 a v0.1.2), 5 guías reales, kiosco de prensa con Burgos, iconos en las tarjetas y sección Jugar. **El binario ya no es `app.exe` sino `entorno-papa.exe`.** 47 tests JS + 9 tests Rust en verde.
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
  1. ~~La sincronización real no se ha ejercitado nunca.~~ **RESUELTO el 2026-07-28**, ver «Publicación en GitHub» más abajo.
  2. ~~Las URLs llevan la rama `main` incrustada y los repos locales están en `master`.~~ **RESUELTO:** las tres ramas se renombraron a `main` al publicar.
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

### Publicación en GitHub y prueba del sync real — 2026-07-28

Hito, no tarea del plan: es lo que desbloquea todo el Bloque 3 y lo que más riesgo tenía.

- **Repos publicados** (usuario `3dpicas`, los tres públicos, rama `main`):
  - `3dpicas/Entorno-Papa` — documentación. Tenía ya un `README.md` creado desde la web, así que hubo que integrarlo con `git merge --allow-unrelated-histories` (commit `fd45706`) antes de poder subir.
  - `3dpicas/entorno-app` — 63 ficheros, con los tags `bloque-0`, `bloque-1` y `bloque-2`.
  - `3dpicas/entorno-contenido` — 10 ficheros, tag `bloque-0`.
- **Ramas renombradas** de `master` a `main` en los tres repos locales, para que casen con las URLs del sync.
- **Autenticación:** no hizo falta GitHub CLI. Git Credential Manager ya venía con Git 2.54 y tenía credenciales guardadas; se comprobó con `git push --dry-run` antes de subir nada.
- **CI estrenado:** el workflow `check.yml` del repo de contenido, que llevaba desde el Bloque 0 sin ejecutarse nunca, arrancó solo con el push y terminó en `success`.
- **Sync real, verificado de punta a punta** con el binario de release:
  1. Primer arranque sin `contenido_meta.json`: la app consultó `api.github.com/repos/3dpicas/entorno-contenido/commits/main`, se bajó el zip, lo validó, hizo el swap y escribió la meta con `sha = d8d716d821ae63902670252e063dcd9c0f2b33b2` (el commit real del repo) y `version = 1`.
  2. **La prueba de que el contenido viene de GitHub y no de la semilla:** `<appdata>/contenido` acabó conteniendo `.github/workflows/check.yml` y `.gitignore`, que la semilla no trae. No quedaron restos de `contenido.old` ni `descarga_tmp`.
  3. Segunda llamada a `sync_now` → `sin_cambios`, sin volver a descargar.
  4. Falseando el sha de la meta a ceros, `sync_now` → `actualizado`, con descarga, validación y swap otra vez.
  5. Con la meta caducada **y reiniciando la app**, el flujo de arranque completo: sync → `actualizado` → manifest recargado → `hashchange` → las 3 secciones repintadas, meta nueva escrita e indicador mostrando `Contenido v1 · 28/07/2026`. Es el recorrido exacto que hará el PC del padre.
- **Instalador probado por el autor:** instaló el NSIS en su máquina y la app abre y funciona.
- **Lo que sigue sin probarse:** que un *push nuevo* al repo de contenido lo recoja la app (aquí se forzó falseando la meta local, no publicando un cambio). Y todo el Bloque 4: updater, Releases y firma.

### Tarea 16: log a archivo + pantalla admin oculta — HECHA (2026-07-28)

- **Commits:** `ec24738` (entorno-app)
- **Verificado:**
  - TDD: `npm test` sin implementación → `Failed to resolve import "../src/ui/admin.js"`. Tras implementar → `Test Files 10 passed (10)`, `Tests 39 passed (39)`. `cargo test` sigue en `9 passed`.
  - **En dev:** Ctrl+Shift+A (evento de teclado despachado por CDP) abre el panel con `App: v0.1.0`, `Contenido: v1 (d8d716d)`, fecha del último sync y `Estado: sin_cambios`, más los botones «Forzar sincronización» y «Cerrar». El mismo atajo otra vez lo cierra. El log salía vacío, lo cual es correcto: en desarrollo nadie escribe en él.
  - **En producción, que es donde importa:** borrados `contenido`, `contenido_meta.json` y `entorno.log`, y arrancado el binario de release. El log se escribió solo con las dos líneas que hacen falta para diagnosticar a distancia:
    ```
    [2026-07-28][17:06:01][app_lib::contenido][INFO] contenido semilla copiado a C:\Users\marqu\AppData\Roaming\com.marquibel.entorno\contenido
    [2026-07-28][17:06:02][app_lib::sync][INFO] contenido actualizado a v1 (d8d716d821ae63902670252e063dcd9c0f2b33b2)
    ```
  - El panel admin muestra esas mismas líneas leídas por `leer_log_reciente`, y el botón «Forzar sincronización» ejecuta el sync y cierra el panel dejando la app intacta (3 secciones).
  - Ruta del log: `%LOCALAPPDATA%\com.marquibel.entorno\logs\entorno.log`.
- **Desviaciones del plan:**
  1. No se ejecutó `npm run tauri add log`: `tauri-plugin-log` ya era dependencia desde el scaffolding del Bloque 0. Solo se cambió **cómo** se registra.
  2. El plugin pasa de registrarse dentro de `.setup()` y solo bajo `debug_assertions`, a registrarse siempre en el builder con destino `LogDir`. En dev se le añade además `Stdout`. Sin este cambio no habría log en los PCs del padre, que es justamente donde no hay otra forma de ver qué pasó.
  3. Tampoco hizo falta añadir `log = "0.4"` a `Cargo.toml` (ya estaba) ni sustituir `eprintln!` (nunca se llegaron a usar, ver Tareas 13 y 14).
- **Decisiones nuevas:** Ninguna.
- **Pendientes que deja:**
  1. Queda un fichero huérfano `Entorno de Papa.log` (0 bytes) de cuando el plugin usaba el nombre por defecto. Inofensivo; se puede borrar a mano.
  2. El log **no rota ni se trunca nunca**. En años de uso puede crecer sin límite. `tauri-plugin-log` tiene `max_file_size` y política de rotación: conviene ponerlo antes de la entrega (Bloque 4).
  3. El atajo se verificó despachando un `KeyboardEvent` por CDP, no pulsando teclas físicas. Lo que se probó es el manejador y el panel; que Windows entregue esa combinación a la ventana no se ha comprobado, aunque no hay motivo para que falle.
  4. «Forzar sincronización» cierra el panel sin decir en qué acabó el sync. Para uso propio es suficiente, pero es un poco ciego: si falla, hay que reabrir el panel y mirar el log.

## ✅ BLOQUE 3 cerrado (2026-07-28)

- **Resultado verificable de la spec:** §10 → «Push en GitHub → la app refresca sola». Comprobado con el binario de release contra `3dpicas/entorno-contenido`: en el primer arranque la app consultó el sha real del repo, se bajó el zip, lo validó, hizo el swap y escribió la meta; el contenido resultante incluye `.github/workflows/check.yml`, que la semilla no trae, así que vino de GitHub y no del paquete local. Con la meta caducada y reiniciando, el ciclo completo se repitió y la interfaz se repintó sola con `Contenido v1 · 28/07/2026`. Matiz honesto: la parte de «push» se ejerció con el sha de un commit real ya publicado, no publicando un cambio nuevo durante la prueba.
- **Estado de los repos:** entorno-app @ `ec24738` / tag `bloque-3` · entorno-contenido @ `d8d716d` / tag `bloque-0` · Entorno-Papa (docs) al día. Los tres publicados en GitHub bajo `3dpicas`, ramas `main`.
- **Qué sabe hacer la app ahora:** al abrirse mira si hay contenido nuevo en GitHub y, si lo hay, se lo descarga y se actualiza sola sin decir nada; repite la comprobación cada seis horas. Si no hay red, sigue funcionando con lo último que tenga guardado, y en un PC recién instalado arranca con el contenido que viene dentro del instalador. Abajo a la derecha, en letra pequeña, indica qué versión de contenido tiene. Con Ctrl+Shift+A aparece una pantalla de mantenimiento, invisible para el usuario normal, con las versiones, el estado del último sync, el log reciente y un botón para forzar la actualización.
- **Deuda/pendientes acumulados:**
  1. Metadatos por defecto en `src-tauri/Cargo.toml` (`name = "app"`, `authors = ["you"]`) — Bloque 4.
  2. El log no rota; puede crecer sin límite.
  3. La semilla no se regenera sola al compilar (`npm run semilla` es manual) y arrastra ficheros que la app no usa.
  4. `validar_contenido` (Rust) valida menos que el `check.mjs` del repo de contenido: es una red de última hora, no el validador de verdad.
  5. `extraer_zip` no protege contra rutas `../` dentro del zip (zip slip). Fuente controlada, riesgo teórico.
  6. El camino de imágenes de las guías sigue sin ejercitarse (`contenido_ruta` devuelve rutas con prefijo `\\?\`).
  7. `https://www.solitr.com/es` da 404 en el manifest de ejemplo.
  8. El HTML de las guías va a `innerHTML` sin sanear.
  9. Sin probar: que un push nuevo al repo de contenido lo recoja la app sola.
- **Notas para el futuro:**
  - `cargo build --release` **no** da un binario de producción: `tauri-build` mantiene `cfg(dev)` y la ventana intenta cargar `localhost:5173`. Usar siempre `npm run tauri build`.
  - Cerrar la app antes de compilar, o el enlazado falla con `Acceso denegado (os error 5)`.
  - Para diagnosticar cualquier cosa en el PC del padre: Ctrl+Shift+A, o directamente `%LOCALAPPDATA%\com.marquibel.entorno\logs\entorno.log`.

---

## BLOQUE 4 — Distribución e instalación

### Tarea 17: icono e instalador NSIS en español — HECHA (2026-07-28)

- **Commits:** `622568b` (entorno-app)
- **Verificado:**
  - `node scripts/generar-icono.mjs` → `icono.png generado`; `npx tauri icon recursos/icono.png` repobló `src-tauri/icons/`. Revisado `128x128.png`: casa blanca sobre cuadrado verde redondeado, que es el diseño del SVG.
  - `npm run tauri build` → un solo bundle, `Entorno de Papa_0.1.0_x64-setup.exe` (ya no sale el MSI). El `beforeBuildCommand` ejecutó la semilla solo: `Semilla actualizada desde entorno-contenido` antes del `vite build`.
  - **Instalador en español, comprobado leyendo la propia ventana** (no de oídas): lanzado el setup y volcados sus controles por Win32 → título `Instalación de Entorno de Papa`, botones `&Siguiente >` y `Cancelar`, texto `Bienvenido al Asistente de Instalación de Entorno de Papa` y el párrafo completo en español. No apareció selector de idioma. Pie con `© 2026 marquib3l`, que confirma el `copyright` del bundle.
  - **Instalación real** con `/S` (exit 0). Creó `C:\Users\marqu\Desktop\Entorno de Papa.lnk` y `…\Start Menu\Programs\Entorno de Papa.lnk`. El acceso directo apunta a `B:\03_Recursos\Software\Entorno de Papa\entorno-papa.exe` — NSIS reutilizó la ruta de la instalación anterior del autor, y no quedó ningún `app.exe` huérfano del nombre viejo.
  - **App instalada abierta:** título `Entorno de Papá`, `showCmd = 3` (maximizada), 1938×1158 físicos. Captura revisada: saludo «Buenas tardes, Papá», reloj `19:39`, las 3 tarjetas con sus colores, icono de casa verde en la barra de título y, abajo a la derecha, `Contenido v1 · 28/07/2026` — o sea que el sync desde GitHub también funcionó en esta build.
  - Icono del ejecutable extraído con `ExtractAssociatedIcon` y revisado: la casa verde.
  - Tests sin regresiones: `npm test` → 39, `cargo test` → 9.
- **Desviaciones del plan:**
  1. **`npx tauri icon` genera además `icons/android/` e `icons/ios/`** (unos 40 ficheros). El proyecto es solo Windows, así que se borraron antes de commitear.
  2. Del array `bundle.icon` se quitó `icons/icon.icns` (macOS) por lo mismo. `icons/64x64.png` sí se conserva porque lo genera la herramienta y no estorba.
  3. Se añadieron al bundle `publisher`, `copyright`, `shortDescription` y `longDescription`, que el plan no pedía: son los campos que acaban a la vista en el instalador y en las propiedades del `.exe`.
  4. **Se saldó de paso deuda acumulada de bloques anteriores**, porque toca exactamente los mismos ficheros de build:
     - `Cargo.toml`: `name = "entorno-papa"`, `description` y `authors` reales, `repository` apuntando a `3dpicas/entorno-app`. **Efecto secundario a recordar: el binario ya no es `target/release/app.exe` sino `entorno-papa.exe`.** El `[lib] name = "app_lib"` no se tocó, así que `main.rs` sigue igual.
     - Log acotado: `.max_file_size(1_000_000)` + `RotationStrategy::KeepSome(2)`. Máximo ~3 MB en total en vez de crecer sin límite.
     - Semilla automática: `beforeBuildCommand` pasa a `npm run compilar` (= `npm run semilla && npm run build`), y `actualizar-semilla.mjs` ya no aborta si falta el repo hermano: si hay semilla commiteada, avisa y sigue. Esto es lo que hace que el `|| true` del workflow de la Tarea 18 sobre.
- **Decisiones nuevas:**
  - `recursos/icono.png` va al `.gitignore`: es un derivado de `icono.svg` que regenera `npm`. La fuente de verdad es el SVG.
  - `bundle.targets` pasa de `"all"` a `["nsis"]`. El MSI no aporta nada aquí y el updater de la Tarea 18 trabaja con los artefactos NSIS.
- **Pendientes que deja:**
  1. El instalador **no está firmado con certificado de código**, así que Windows SmartScreen avisará al ejecutarlo en una máquina nueva. La firma del updater (Tarea 18) es otra cosa distinta: sirve para que la app confíe en la actualización, no para que Windows confíe en el instalador.
  2. Queda por comprobar la instalación **en una máquina limpia**: aquí NSIS reutilizó la ruta de una instalación previa (`B:\03_Recursos\Software\…`), no la de por defecto.
  3. La rotación del log está configurada pero no ejercitada: haría falta un log de más de 1 MB para verla rotar. Además `leer_log_reciente` solo lee `entorno.log`, así que tras rotar no verá lo que quedó en los ficheros con fecha.
  4. **Corrección a la deuda de la Tarea 16:** el `Entorno de Papa.log` huérfano **no se puede borrar «a mano y ya está»**: se comprobó borrándolo y reabriendo la app, y `tauri-plugin-log` lo vuelve a crear vacío en cada arranque, junto al `entorno.log` de verdad. Son 0 bytes y no molestan; lo que no vale es la instrucción de borrarlo como si fuera un resto de una sola vez.

### Tarea 18: auto-actualización de la app — HECHA (2026-07-28)

- **Commits:** `c3e841f` (entorno-app) · tag `v0.1.0`
- **Verificado:**
  - TDD del frontend: `npm test` con el test escrito y sin implementación → `Failed to resolve import "../src/lib/actualizacion.js"`, `Test Files 1 failed | 10 passed (11)`. Tras crear `src/lib/actualizacion.js` → `Test Files 11 passed (11)`, `Tests 43 passed (43)`. `cargo test` sigue en `9 passed`.
  - **Firma, probada antes de tocar GitHub:** `npm run tauri build` en local con las variables de firma puestas → `Finished 1 updater signature at: …\Entorno de Papa_0.1.0_x64-setup.exe.sig` (428 bytes).
  - **Release publicada de verdad:** workflow `release` en verde en 7 m 45 s; `v0.1.0` (`Entorno de Papá v0.1.0`, no borrador) con los tres artefactos: `Entorno.de.Papa_0.1.0_x64-setup.exe` (3,62 MB), su `.sig` (428 B) y `latest.json` (1,3 KB).
  - **El endpoint que consultará la app, consultado:** `curl` a `https://github.com/3dpicas/entorno-app/releases/latest/download/latest.json` devuelve `version: 0.1.0`, la firma en base64 y la URL de descarga, con las dos claves de plataforma `windows-x86_64` y `windows-x86_64-nsis`.
- **Desviaciones del plan:**
  1. **Los repos de GitHub ya existían** (se publicaron en el Bloque 3) y están bajo `3dpicas`, no `marquib3l`. Del Step 4 solo quedaban los secretos, puestos con `gh secret set` sobre `3dpicas/entorno-app`. `gh` resultó estar instalado y autenticado como `3dpicas`.
  2. **`actualizarApp` no vive en `main.js`, sino en `src/lib/actualizacion.js` con `comprobar` y `relanzar` inyectados.** Tal como lo escribía el plan (llamando directamente a `check`/`relaunch` importados) no había forma de probarlo sin Tauri delante, y este proyecto va por TDD. `main.js` solo lo cablea: `actualizarApp({ comprobar: check, relanzar: relaunch })`. De paso el test cubre un caso que el plan no contemplaba: que falle la *instalación* (no solo la comprobación) tampoco debe lanzar ni relanzar.
  3. `npm run tauri add updater` creó un `capabilities/desktop.json` nuevo en vez de tocar `default.json`; se le añadió a mano `process:default`, que el CLI no puso.
  4. El workflow no lleva el `npm run semilla || true` del plan: sobra desde la Tarea 17, porque el `beforeBuildCommand` ya la ejecuta y el script conserva la semilla commiteada cuando no encuentra el repo hermano.
  5. `node-version: 22` en vez de 20, por el mínimo que pide Vite 8.
  6. `npm run tauri add` volvió a dejar el fichero basura `src-tauri/2` y a pasar `cargo fmt` por `contenido.rs` y `sync.rs`. Se borró el fichero y se revisó el reformateo con `git diff -w`: solo saltos de línea, ni una línea de lógica cambiada — importante, porque `sync.rs` es de las piezas que el protocolo prohíbe tocar sin plan.
- **Decisiones nuevas:**
  - **Clave de firma con contraseña aleatoria** (elegido por el autor frente a la opción sin contraseña). Privada en `%USERPROFILE%\.tauri\entorno-papa.key`, fuera de todo repo; pública en `tauri.conf.json`; ambas en los secretos del repo. **Si se pierden la clave o la contraseña, los PCs del padre dejan de aceptar actualizaciones y hay que reinstalar a mano.**
  - `bundle.createUpdaterArtifacts: true`, que es lo que hace que el bundler emita el `.sig`.
- **Pendientes que deja:**
  1. **Trampa que costó un release fallido** (el primer intento del workflow murió con `failed to decode secret key: incorrect updater private key password: Wrong password for that key`): la contraseña se había guardado con `Set-Content -Encoding utf8` de **PowerShell 5.1, que escribe BOM**. `gh secret set --body "$(cat …)"` desde bash mandó los tres bytes `EF BB BF` pegados delante de la contraseña. En local no se notaba porque `Get-Content -Raw` sí quita el BOM, así que la build local firmaba bien y la de CI no. Comprobado con `od -c`. **Para cualquier secreto que salga de un fichero escrito por PowerShell: verificar los primeros bytes.**
  2. La actualización automática **está publicada pero todavía no vista funcionar**: eso es la Tarea 19, que necesita una v0.1.1 que descargar.
  3. El instalador no lleva firma de código (certificado): SmartScreen avisará en un equipo nuevo. La firma del updater es otra cosa, sirve para que la app confíe en la actualización.
  4. Los `actions/checkout@v4` y `setup-node@v4` avisan de que Node 20 está obsoleto en los runners. Es aviso, no error.

### Tarea 19: instalación y ciclo completo — STEPS 1 Y 2 HECHOS (2026-07-28); STEP 3 PENDIENTE DEL AUTOR

- **Commits:** `59d0ec1` (entorno-app, versión 0.1.1) · `7595bed` (entorno-contenido, ABC + `version: 2`) · tag `v0.1.1`
- **Verificado — Step 1, la app se actualiza sola:**
  - Punto de partida honesto: se **descargó e instaló el `setup.exe` de la release v0.1.0** (la instalación previa era la build local de la Tarea 17, que no lleva updater). Comprobado en disco: `FileVersion = 0.1.0`, fecha `20:17:10`, la de la compilación en CI.
  - Publicada la v0.1.1 (subida de versión en `tauri.conf.json` y `Cargo.toml`, tag `v0.1.1`); workflow en verde y release con instalador, `.sig` y `latest.json` apuntando ya a `0.1.1`.
  - Abierta la app instalada (v0.1.0) a las 20:42:27 con pid 40452. **75 segundos después, sin tocar nada:** el ejecutable en disco era `FileVersion = 0.1.1` (fecha `20:34:04`, la de la build de CI), el proceso 40452 había desaparecido y corría uno nuevo, pid 1908. Es decir: descargó, instaló y se relanzó sola.
  - Confirmado desde dentro con Ctrl+Shift+A: `App: v0.1.1`.
- **Verificado — Step 2, el contenido llega sin reinstalar:**
  - Añadida la tarjeta ABC a Prensa/Nacional y subido `version` a 2 en `entorno-contenido`; `npm run check` → `Contenido OK`; push a `main`.
  - Con la app **ya abierta y sin reiniciarla**: `contenido_meta.json` pasó de `sha d8d716d / version 1` a `sha 7595bed / version 2` — y `7595bed` es exactamente el commit que se acababa de empujar. El indicador de la esquina cambió a `Contenido v2 · 28/07/2026` (captura revisada).
  - La tarjeta nueva, vista en pantalla: navegando a Prensa por CDP, `tarjetas = ["El País","El Mundo","ABC","Marca"]` y captura del grupo Nacional con las tres tarjetas.
  - **Esto cierra el pendiente nº 9 del Bloque 3** («sin probar: que un push nuevo al repo de contenido lo recoja la app sola»). Ya no se falsea el sha local: se publicó un commit de verdad y la app lo recogió.
- **Desviaciones del plan:** el plan daba por hecho que la máquina ya tenía instalada la v0.1.0 «buena»; había que instalar antes la de la release, porque la build local de la Tarea 17 es anterior al updater y nunca habría comprobado actualizaciones. Los dos Steps se solaparon en el tiempo (el contenido se probó mientras compilaba la v0.1.1), pero las evidencias son independientes: el sha del contenido por un lado, la versión del ejecutable por otro.
- **Decisiones nuevas:** Ninguna. ABC es una tarjeta de prueba con URL real; el contenido de verdad se cura en el Bloque 5.
- **Pendientes que deja:**
  1. **Step 3 sin hacer: instalar en los dos PCs del padre.** Es trabajo presencial del autor. Checklist en el plan: acceso directo con icono verde, contenido visible al abrir, Ctrl+Shift+A correcto, un periódico abriendo en el navegador que use él, y la prueba sin WiFi.
  2. **La actualización de la app no deja rastro en `entorno.log`.** `actualizarApp` usa `console.info`/`console.error`, que se quedan en la consola del WebView; el log a fichero solo recoge lo que escribe Rust. Lo mismo le pasa a `sincronizar()`. Si una actualización falla en el PC del padre, el log —que es la única ventana que hay a esa máquina— no dirá nada. No es regresión de esta tarea (viene de antes), pero contradice el espíritu de la spec §8 y conviene resolverlo antes de dar la v1 por buena.
  3. El instalador sigue sin firma de código: en los equipos del padre saldrá el aviso de SmartScreen la primera vez.

### Contenido v3: arreglada la URL del solitario — 2026-07-28

Cambio de contenido, no tarea del plan (`PROTOCOLO.md` § escalabilidad). Se anota porque **cierra el pendiente nº 7 del Bloque 3**, arrastrado desde la Tarea 8.

- **Commit:** `a6eead9` (entorno-contenido) · CI `check` en verde.
- **Qué pasó:** lo detectó el padre usando la app en su PC, no una prueba: era el único enlace que no abría. `https://www.solitr.com/es` **nunca ha existido** —da 404, y también `/solitario`—; la que funciona es la raíz `https://www.solitr.com/`. Elegida por el autor entre tres candidatas: es la más limpia (sin anuncios ni registro, cartas grandes, la partida ya montada al entrar) a cambio de que sus tres botones estén en inglés, cosa que no hace falta leer para jugar.
- **Verificado:** `npm run check` → `Contenido OK`, y **las seis URLs del manifest comprobadas una a una con `curl`, todas 200** (elpais, elmundo, abc, marca, solitr, tetris). Es lo que debería haberse hecho en el Bloque 0, cuando las URLs se escribieron sin comprobar.
- **Aprendido:** el validador `check.mjs` valida el *formato* de las URLs (`^https?://`) pero nunca las visita. Un enlace roto pasa el CI sin problema. Comprobarlas de verdad es candidato claro para el Bloque 5, donde entra el contenido definitivo.

---

## BLOQUE 5 — Contenido real v1

### Tarea 20: guías de ofimática — HECHA salvo las capturas (2026-07-29)

- **Commits:** `1decff1` (entorno-contenido, contenido v4)
- **Datos que condicionaron el texto, confirmados por el autor:** el padre usa **Gmail**, **Brave** ya instalado y puesto como navegador predeterminado, y móvil **Android**. Sin esto las guías se habrían escrito a ciegas y no habrían servido: los botones, los iconos y los nombres de carpeta cambian con cada combinación.
- **Verificado:**
  - `npm run check` → `Contenido OK`.
  - **Parseadas con el parser real de la app** (`src/lib/guia.js`, no una comprobación aparte): 5 guías, 36 pasos, **ninguno vacío y ninguno con el título mal formado**. 7+7+7+7+8 pasos.
  - En la app en marcha: la sección Aprender pinta las 5 tarjetas nuevas, y abriendo «Fotos del móvil al PC» el visor da `h1 = Pasar las fotos del móvil al ordenador`, `Paso 1 de 8` y **0 imágenes rotas** — el degradado silencioso de las capturas que aún no existen funciona. Captura revisada.
- **Desviaciones del plan:**
  1. **Las guías llevan escritas las 35 referencias `![captura](img/…)` aunque los ficheros no existan.** El plan decía «se commitean sin imágenes»; dejarlas escritas convierte la guía en la lista exacta de capturas que hay que hacer, y el visor las quita en silencio mientras tanto. Comprobado que `check.mjs` no valida las imágenes del cuerpo de las guías (solo los iconos), así que el CI pasa.
  2. La regla de «no arrastrar» del plan (solo para `carpetas-archivos`) se aplicó a todas: mover ficheros y copiar fotos van con botón derecho y menú. En Windows 11 eso obliga a describir **dibujos** (tijeras, portapapeles) en vez de palabras, porque el menú nuevo es de iconos.
  3. `drive-basico` abre los archivos con botón derecho → **Vista previa** en vez de doble clic, por la misma regla.
- **Decisiones nuevas:** el Paso 1 de `drive-basico` no es una acción sino una explicación de qué es Drive («un armario tuyo, pero guardado en internet»). Se salta la regla de «un paso = una acción» a propósito: sin saber qué es, los seis pasos siguientes no significan nada.
- **Pendientes que deja:**
  1. **Las 35 capturas** (Step 2). El autor las hará más adelante; las guías ya se leen sin ellas. Nombres exactos: `img/correo-enviar-01..07`, `correo-leer-01..07`, `drive-basico-02..07`, `carpetas-archivos-01..07`, `fotos-movil-pc-01..08`.
  2. El texto **no lo ha revisado todavía el autor**, que es quien conoce a su padre. Se publicó con su permiso explícito porque corregir contenido es inmediato y no requiere reinstalar nada.

### Tarea 21: kiosco de prensa completo con iconos — HECHA (2026-07-29)

- **Commits:** `b89f87e` (entorno-contenido, contenido v5) · `e47a82b` + `f3dd33e` (entorno-app) · release `v0.1.2`
- **Verificado:**
  - **Las 11 URLs comprobadas una a una con `curl`: todas 200.** Burgos (Diario de Burgos, El Correo de Burgos, BURGOSconecta), Nacional (El País, El Mundo, ABC), Internacional (BBC Mundo, Euronews), Deportes (Marca, AS), Bolsa (Expansión).
  - `node scripts/descargar-iconos.mjs` → 11 iconos descargados; revisados de uno en uno: son los logos reales, no el globo genérico de reserva (el de la BBC son sus tres cuadrados negros, y pesa solo 252 bytes, que era lo que hacía sospechar).
  - TDD en la app: `npm test` con los tests escritos y sin implementación → 2 fallos, los dos que necesitaban la funcionalidad. Tras implementar → `Test Files 11 passed`, `Tests 47 passed`.
  - **En la app en marcha:** los 5 grupos, las 11 tarjetas y **11 de 11 imágenes con `naturalWidth > 0`**, o sea cargadas de verdad. Captura revisada con los logos.
  - **Esto cierra el pendiente nº 6 del Bloque 3:** el camino `contenido_ruta` → `convertFileSrc` → `assetProtocol` llevaba sin ejercitarse desde el Bloque 1 por la duda del prefijo `\\?\` de `canonicalize`. Funciona.
- **Desviaciones del plan:**
  1. **Grupo local = Burgos**, que el plan dejaba a rellenar. Elegidas las tres cabeceras de la provincia.
  2. **Cinco Días queda fuera.** `cincodias.elpais.com` responde **403** a las comprobaciones automáticas y no se ha podido verificar; entrará cuando se abra en un navegador de verdad. Bolsa se queda con Expansión sola.
  3. `renderSeccion` recibe `resolverImagen` como **opcional**: si una tarjeta trae icono pero nadie pasa el resolvedor, se pinta sin icono en vez de romper. Hay test para ese caso y para el de icono que no carga.
- **Decisiones nuevas:**
  - **Compactada la pantalla de sección** (`f3dd33e`): márgenes de los títulos de grupo 32/16 → 14/8, cabecera 32 → 16, padding vertical de `main` 32 → 20. Con cinco grupos, Prensa desbordaba 306 px y Deportes quedaba cortado; ahora desborda 154 y solo queda Bolsa por debajo. **No se tocaron el alto mínimo de tarjeta (120 px) ni el tamaño del título de grupo (32 px): son mínimos de la spec.**
  - **Se acepta el scroll vertical**, decisión del autor. La spec solo prohíbe el horizontal.
  - La semilla de `entorno-app` se regeneró al contenido v5 antes de publicar, para que una instalación nueva arranque ya con la prensa de Burgos y las cinco guías aunque no haya red.
- **Aprendido, y es lo importante de esta tarea:** al ver el desborde empecé a recortar CSS hasta que cupiera **en el monitor de desarrollo**. El autor lo paró: la app puede acabar en varios ordenadores, así que cuadrar píxeles contra una pantalla concreta produce un diseño que se rompe en la siguiente. Enfoque correcto, y el que se aplicó: **probar a varios tamaños y validar invariantes**, no una medida. Resultados con la ventana redimensionada por Win32 y medida por CDP:

  | Ventana | Columnas | Scroll horizontal | Texto recortado | Alto de tarjeta |
  |---|---|---|---|---|
  | 1024×768 | 2 | no | no | 120 px |
  | 1366×768 | 2 | no | no | 120 px |
  | 1600×900 | 3 | no | no | 120 px |
  | 1920×1080 | 4 | no | no | 120 px |

  La rejilla se adapta sola y ninguna de las invariantes se rompe; lo único que cambia es cuánto hay que bajar. Script de apoyo: `resoluciones.ps1` en el scratchpad de la sesión.
- **Pendientes que deja:**
  1. Con cinco grupos **no existe forma de que Prensa quepa sin scroll** respetando la spec: 5 × (título 42 + tarjeta 120) + cabecera 96 + padding 40 = **946 px mínimos**, aunque se quiten todos los márgenes. Si algún día molesta, la salida es fusionar grupos (Internacional dentro de Nacional ahorra 184 px), no seguir recortando CSS.
  2. Cinco Días sin verificar (403).
  3. Los iconos vienen del servicio de favicons de Google: si algún medio cambia de logo, hay que volver a ejecutar el script.

### Tarea 22: juegos, revisión final y v1 — HECHA salvo la verificación en los PCs (2026-07-29)

- **Commits:** `0872dc0` (entorno-contenido, contenido v6)
- **Verificado:**
  - `npm run check` → `Contenido OK`. `npm test` → 47. `cargo test` → 9.
  - **En la app instalada de producción, que es el camino real:** tras el push, la app sincronizó sola a `sha 0872dc0` / `version 6` —el commit que se acababa de publicar—, y pintó **12 tarjetas de prensa con 12 iconos cargados** y Jugar con sus 2, indicador `Contenido v6 · 29/07/2026`. Ningún título recortado en ninguna sección.
  - **Una guía recorrida entera** en el visor (`carpetas-archivos`): los 7 pasos con contenido real (entre 82 y 225 caracteres cada uno), indicador correcto en todos, y el botón final pasa a `✔ Terminar`.
- **Desviaciones del plan:**
  1. **Cinco Días vuelve a Bolsa.** Se había descartado en la Tarea 21 por un 403; reintentando resulta que responde 200. El 403 es antibot, no una URL rota: `elpais.com` alterna 200/403/200 desde la misma máquina en el mismo minuto. **La lección: una sola comprobación con `curl` produce falsos negativos en medios con protección antibot; hay que reintentar antes de descartar nada.**
  2. **El solitario lleva icono dibujado a mano.** `solitr.com` no publica favicon: el servicio de Google da 404, `/favicon.ico` responde 200 pero con **0 bytes**, y el HTML no trae ningún `<link rel=icon>`. Se dibujó uno (dos cartas y un corazón sobre verde) y se dejó el SVG fuente junto al PNG. `descargar-iconos.mjs` avisa «sin favicon» en cada ejecución pero **no lo sobrescribe**, que es el comportamiento correcto.
  3. La URL del solitario del plan (`solitr.com/es`) estaba mal desde el Bloque 0; corregida en el plan y en el contenido.
- **Decisiones nuevas:** Ninguna.
- **Trampa importante que costó un rato, y que volverá a pasar:** con el updater ya activo, **arrancar un binario de desarrollo compilado con una versión anterior dispara una actualización de verdad**. El `target/debug/entorno-papa.exe` se había compilado siendo la v0.1.1; al lanzarlo vio la release v0.1.2, se actualizó, y el proceso que quedó vivo fue **la app instalada**, no la de desarrollo. Durante un rato estuve midiendo producción creyendo medir dev, y las cifras no cuadraban (11 tarjetas en vez de 12) porque producción lee de appdata. Señales para reconocerlo: el proceso lanzado muere al instante, `Get-Process | Select Path` apunta a la ruta de instalación, y el target de CDP es `http://tauri.localhost/` en vez de `http://localhost:5173`. **Solución: recompilar el binario de dev tras cada subida de versión.** De rebote, es una confirmación más de que el updater funciona.
- **Pendientes que deja:**
  1. **Step 3 a medias:** el push está hecho y verificado en la máquina del autor con el binario de producción, pero falta abrirla en los PCs del padre y, sobre todo, **verle usarlo diez minutos sin ayuda**, que es la prueba de aceptación real de la v1.
  2. Las URLs se han comprobado con `curl`, no abriéndolas en un navegador: eso confirma que responden, no que la portada sea la correcta ni que no pidan registro o cookies. El plan pedía abrirlas.
  3. Siguen pendientes las 35 capturas de las guías.

---

## Post-v1 (mejoras y contenido tras la entrega)

*(vacío)*
