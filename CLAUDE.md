# Entorno Digital para Papá

Panel de escritorio (Tauri v2, Windows 11) para el padre del autor: guías paso a paso, kiosco de prensa y juegos. Contenido sincronizado desde GitHub, app auto-actualizable.

## Para retomar el proyecto (orden de lectura)

1. `docs/REGISTRO.md` → sección **«Estado actual»**: dónde quedó todo y cuál es el siguiente paso.
2. `docs/PROTOCOLO.md`: cómo se trabaja aquí (registro por tarea/bloque, plantillas, reglas).
3. `docs/superpowers/plans/2026-07-26-entorno-papa.md`: el plan de 22 tareas con checkboxes.
4. `docs/superpowers/specs/2026-07-26-entorno-papa-design.md`: decisiones de diseño y porqués.

## Estructura

- Este repo raíz: solo documentación.
- `entorno-app/`: repo git propio — la aplicación Tauri.
- `entorno-contenido/`: repo git propio (público en GitHub) — manifest.json, guías, iconos.

## Reglas imprescindibles

- Seguir `docs/PROTOCOLO.md`: cada tarea terminada se marca en el plan y se registra en REGISTRO.md; cada bloque cerrado se etiqueta `bloque-N`.
- Verificación antes de dar nada por hecho: `npm test` (entorno-app), `cargo test` (src-tauri), `npm run check` (entorno-contenido).
- Identidad git: `marquib3l` / `marquib3l@gmail.com` (repo-local, sin preguntar).
- Todo en español; commits con prefijos `feat:`/`fix:`/`test:`/`chore:`/`docs:`.
- No tocar sync, updater ni contrato del manifest sin plan previo (riesgo de romper los PCs del padre en remoto).
