# Registro del proyecto — Entorno Digital para Papá

Bitácora de ejecución. Se rellena según `docs/PROTOCOLO.md`. Entradas nuevas SIEMPRE debajo de su bloque; «Estado actual» siempre al día.

---

## 📍 Estado actual

- **Fase:** ✅ **Bloque 0 cerrado** (Tareas 1, 2 y 3 hechas y verificadas)
- **Siguiente paso:** Bloque 1, Tarea 4 — parser del manifest (`src/lib/manifest.js` + `tests/manifest.test.js`), TDD.
- **Repos:** raíz @ `335373c` (tag `bloque-0`) · `entorno-app` @ `d4087de` (tag `bloque-0`) · `entorno-contenido` @ `d8d716d` (tag `bloque-0`)
- **Última sesión:** 2026-07-26/27 — scaffolding de los dos repos; Vitest en verde (5 tests); validador de contenido en verde y probado también en fallo; se instaló el toolchain de Rust + MSVC y la app compiló y abrió ventana maximizada.
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

*(sin tareas ejecutadas aún)*

---

## BLOQUE 2 — Visor de guías

*(sin tareas ejecutadas aún)*

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
