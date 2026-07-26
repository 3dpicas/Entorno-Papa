# Registro del proyecto — Entorno Digital para Papá

Bitácora de ejecución. Se rellena según `docs/PROTOCOLO.md`. Entradas nuevas SIEMPRE debajo de su bloque; «Estado actual» siempre al día.

---

## 📍 Estado actual

- **Fase:** Bloque 0 en curso — Tareas 2 y 3 hechas y verificadas; Tarea 1 hecha salvo su Paso 5 (verificación visual de la ventana Tauri)
- **Siguiente paso:** instalar Rust + MSVC Build Tools en el PC de desarrollo (no estaban), ejecutar `npm run tauri dev` y cerrar el Paso 5 de la Tarea 1. Después, Bloque 1 Tarea 4 (parser del manifest).
- **Repos:** raíz @ `4bb0e01` · `entorno-app` @ `6b3168e` · `entorno-contenido` @ `d8d716d`
- **Última sesión:** 2026-07-26 — scaffolding de los dos repos; Vitest en verde (5 tests); validador de contenido en verde y probado también en fallo. Bloque 0 **no cerrado**: falta compilar la app.

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

### Tarea 1: estructura raíz + app Tauri — PARCIAL, falta Paso 5 (2026-07-26)

- **Commits:** `4bb0e01` (raíz: `.gitignore` que ignora los dos repos anidados) · `8f4aa88` (entorno-app: commit raíz del scaffolding)
- **Verificado:** `npm run build` → `built in 33ms`, genera `dist/index.html` + `dist/assets/index-*.js`. `npx tauri --version` → `tauri-cli 2.11.4`. **Paso 5 NO verificado**: `npm run tauri dev` no se ha ejecutado.
- **Desviaciones del plan:**
  - `create-vite@9.1.1` genera un demo distinto al que suponía el plan. En vez de `src/counter.js` + `src/javascript.svg` + `public/vite.svg`, genera `src/counter.js`, `src/style.css`, `src/assets/` (hero.png, javascript.svg, vite.svg) y `public/` (favicon.svg, icons.svg). Se borró todo eso; `public/` quedó vacío y se eliminó también. Se quitó el `<link rel="icon">` de `index.html` porque apuntaba al favicon borrado.
  - `npx tauri init` necesitó `--ci` para no quedarse esperando entrada interactiva (la shell de la sesión no tiene stdin).
  - `tauri init` **no** añade el script `tauri` a `package.json`. Se añadió a mano (`"tauri": "tauri"`), imprescindible porque el plan usa `npm run tauri dev`.
  - En `tauri.conf.json` se descartaron las claves generadas `width: 800` / `height: 600` porque contradicen el `minWidth: 1024` que pide el plan. Se conservaron `resizable: true` y `fullscreen: false`.
- **Decisiones nuevas:** ninguna de diseño. Versiones fijadas por el entorno: Node v24.15.0, npm 11.14.1, Vite 8.1.5, tauri-cli 2.11.4, @tauri-apps/api 2.11.1.
- **Pendientes que deja:** **Paso 5 sin verificar** — el PC de desarrollo no tiene Rust ni MSVC Build Tools (sí tiene WebView2 Runtime). Sin ellos Tauri no compila. Hay que instalar `Microsoft.VisualStudio.2022.BuildTools` con la carga `Microsoft.VisualStudio.Workload.VCTools` y luego `Rustlang.Rustup`, en ese orden, y ejecutar `npm run tauri dev` para ver la ventana maximizada «Entorno de Papá».

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
