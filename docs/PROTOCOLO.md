# Protocolo de trabajo — Entorno Digital para Papá

Sistema de registro para que el proyecto sea documentable, escalable y retomable tras meses sin tocarlo. **Toda sesión de trabajo empieza y termina en estos documentos.**

## Los 4 documentos y su papel

| Documento | Qué es | Cuándo se toca |
|-----------|--------|----------------|
| `docs/superpowers/specs/2026-07-26-entorno-papa-design.md` | **Spec** — el QUÉ y el PORQUÉ. Decisiones de diseño. | Solo si cambia una decisión de diseño (y se anota el cambio en el Registro) |
| `docs/superpowers/plans/2026-07-26-entorno-papa.md` | **Plan** — el CÓMO. 22 tareas con pasos y checkboxes. | Al ejecutar: marcar `- [x]` cada paso completado |
| `docs/REGISTRO.md` | **Registro (bitácora)** — el QUÉ PASÓ. Historia real de la ejecución. | Al terminar cada tarea y cada bloque |
| `CLAUDE.md` (raíz) | **Punto de entrada** — orientación para retomar. | Solo si cambia la estructura del proyecto |

Regla de oro: **el Plan dice lo que se pensaba hacer; el Registro dice lo que de verdad se hizo.** Si difieren, el Registro manda.

## Al EMPEZAR una sesión de trabajo

1. Leer la sección **«Estado actual»** de `docs/REGISTRO.md` (30 segundos: dónde quedó todo).
2. Abrir el Plan por la tarea indicada como siguiente.
3. Verificar que el entorno arranca: `npm test` (en `entorno-app/`) y `npm run check` (en `entorno-contenido/`) en verde antes de escribir nada nuevo.

## Al EMPEZAR una tarea

1. Leer la tarea completa en el Plan (Files, Interfaces, Steps).
2. Comprobar que las tareas de las que depende (bloque «Consumes») están marcadas como hechas en el Registro.

## Al TERMINAR una tarea

1. **Verificar**: todos los tests en verde (`npm test` / `cargo test` según toque). Sin verde no hay tarea terminada.
2. **Commit** hecho según el paso final de la tarea (uno por tarea como mínimo).
3. **Marcar checkboxes** `- [x]` de todos los pasos de la tarea en el Plan.
4. **Añadir entrada al Registro** con la plantilla de tarea (ver abajo). Obligatorio rellenar «Desviaciones» aunque sea "Ninguna" — es el campo que salva el proyecto meses después.
5. **Actualizar «Estado actual»** del Registro: siguiente tarea a ejecutar.
6. Commit de la documentación en el repo raíz: `git commit -m "docs: registro tarea N"`.

## Al TERMINAR un bloque

1. Todas las tareas del bloque registradas y su **resultado verificable** (tabla de la spec §10) comprobado de verdad — no "debería funcionar", sino visto funcionar.
2. Añadir al Registro el **cierre de bloque** con su plantilla (ver abajo).
3. Etiquetar en los repos afectados: `git tag bloque-N` (facilita volver a un punto sano).
4. Actualizar «Estado actual».

## Plantillas

### Entrada de tarea (copiar en el Registro)

```markdown
### Tarea N: <nombre> — HECHA (AAAA-MM-DD)

- **Commits:** <hash(es) y repo>
- **Verificado:** <qué comando/comprobación se ejecutó y su resultado>
- **Desviaciones del plan:** <qué se hizo distinto y POR QUÉ, o "Ninguna">
- **Decisiones nuevas:** <decisiones tomadas que la spec no cubría, o "Ninguna">
- **Pendientes que deja:** <cabos sueltos para más adelante, o "Ninguno">
```

### Cierre de bloque (copiar en el Registro)

```markdown
## ✅ BLOQUE N cerrado (AAAA-MM-DD)

- **Resultado verificable de la spec:** <cita del criterio> → <cómo se comprobó>
- **Estado de los repos:** entorno-app @ <hash/tag> · entorno-contenido @ <hash/tag>
- **Qué sabe hacer la app ahora:** <2-3 frases en lenguaje llano>
- **Deuda/pendientes acumulados:** <lista o "Nada">
- **Notas para el futuro:** <cualquier cosa que el yo-del-futuro agradecerá saber>
```

## Reglas para la escalabilidad (después de v1)

- **Añadir contenido** (guía, periódico, sección): NO es una tarea del plan. Editar `entorno-contenido`, `npm run check`, push. Anotar una línea en el Registro solo si se aprendió algo.
- **Añadir funcionalidad** (nuevo tipo de tarjeta, nueva pantalla): mini-ciclo completo — añadir sección a la spec (o spec nueva en `docs/superpowers/specs/`), plan corto en `docs/superpowers/plans/`, ejecutar con este mismo protocolo, registrar.
- **Nunca** modificar la app directamente sin plan cuando el cambio toca sync, updater o el contrato del manifest: son las piezas que pueden romper los PCs de Papá en remoto.

## Convenciones fijas

- Commits: prefijos `feat:`, `fix:`, `test:`, `chore:`, `docs:` — en español, imperativo.
- Identidad git en todo repo del proyecto: `marquib3l` / `marquib3l@gmail.com`.
- Fechas siempre `AAAA-MM-DD`.
- Idioma de todo (código aparte): español.
