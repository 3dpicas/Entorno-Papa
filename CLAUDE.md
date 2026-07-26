# Entorno Digital para Papá

Panel de escritorio (Tauri v2, Windows 11) para el padre del autor: guías paso a paso, kiosco de prensa y juegos. Contenido sincronizado desde GitHub, app auto-actualizable.

## Para retomar el proyecto (orden de lectura)

1. `docs/REGISTRO.md` → sección **«Estado actual»**: dónde quedó todo y cuál es el siguiente paso.
2. `docs/PROTOCOLO.md`: cómo se trabaja aquí (registro por tarea/bloque, plantillas, reglas).
3. `docs/superpowers/plans/2026-07-26-entorno-papa.md`: el plan de 22 tareas con checkboxes.
4. `docs/superpowers/specs/2026-07-26-entorno-papa-design.md`: decisiones de diseño y porqués.

## Estructura

- Este repo raíz: solo documentación. Ignora las dos subcarpetas por `.gitignore`, así que son **tres repos git independientes**; cada uno tiene su propio tag `bloque-N`.
- `entorno-app/`: repo git propio — la aplicación Tauri.
- `entorno-contenido/`: repo git propio (público en GitHub) — manifest.json, guías, iconos.

Ninguno de los tres está subido a GitHub todavía (a fecha del Bloque 0 cerrado).

## Requisitos del entorno de desarrollo

Windows 11. Hace falta todo esto instalado antes de poder compilar:

| Qué | Por qué | Comprobar con |
|-----|---------|---------------|
| Node.js 20+ | Vite, Vitest, validador de contenido | `node -v` |
| Rust (rustup, toolchain `stable-x86_64-pc-windows-msvc`) | Backend de Tauri | `cargo -V` |
| VS Build Tools con la carga «Desarrollo para el escritorio con C++» | El linker MSVC que Rust necesita; sin él Tauri no compila | `vswhere.exe -products *` |
| WebView2 Runtime | El motor que renderiza la interfaz | Viene de serie en Windows 11 |

**Trampa conocida:** `cargo` no queda en el PATH de las terminales ya abiertas. Antes de cualquier comando de Tauri, en PowerShell:

```powershell
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
```

El primer `npm run tauri dev` tarda varios minutos porque descarga y compila ~425 crates. Los siguientes son rápidos gracias a `src-tauri/target/`: no borrar esa carpeta sin motivo.

## Comandos del día a día

```powershell
# Tests del frontend (desde entorno-app/)
npm test

# Arrancar la app en desarrollo (desde entorno-app/, con el PATH ya ajustado)
npm run tauri dev

# Tests del backend Rust (desde entorno-app/src-tauri/)
cargo test

# Validar el contenido (desde entorno-contenido/)
npm run check
```

## Reglas imprescindibles

- Seguir `docs/PROTOCOLO.md`: cada tarea terminada se marca en el plan y se registra en REGISTRO.md; cada bloque cerrado se etiqueta `bloque-N`.
- Verificación antes de dar nada por hecho: los comandos de § «Comandos del día a día», ejecutados y con su salida leída. Nada de «debería funcionar»: si la tarea dice que se abre una ventana, hay que verla abierta. (`cargo test` da `0 passed` mientras no haya código Rust propio; eso es correcto, no un fallo.)
- Identidad git: `marquib3l` / `marquib3l@gmail.com` (repo-local, sin preguntar).
- Todo en español; commits con prefijos `feat:`/`fix:`/`test:`/`chore:`/`docs:`.
- No tocar sync, updater ni contrato del manifest sin plan previo (riesgo de romper los PCs del padre en remoto).
