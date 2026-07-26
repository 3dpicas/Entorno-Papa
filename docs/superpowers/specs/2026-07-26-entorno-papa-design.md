# Entorno Digital para Papá — Documento de Diseño

**Fecha:** 2026-07-26
**Estado:** Aprobado en sesión de brainstorming

## 1. Objetivo

Panel de escritorio centralizado, simplificado y visual para el padre del autor: elimina la frustración y el miedo a "romper algo" dándole un espacio seguro donde consultar guías, leer prensa y jugar, sin navegar por menús del sistema operativo. Instalable en sus dos equipos (portátil y torre, ambos Windows 11), con acceso directo en el escritorio.

## 2. Requisitos confirmados

- **Actualización remota automática:** el autor edita contenido desde su casa; los PCs del padre se sincronizan solos al tener internet.
- **Prensa y juegos abren en el navegador por defecto** (ventana normal). La app no embebe webs externas.
- **Offline:** la app y las guías funcionan sin internet (última versión cacheada). Prensa y juegos requieren conexión por naturaleza.
- **Guías:** el autor las escribe en Markdown con capturas; la app las presenta como asistente paso a paso (una pantalla por paso).
- **Accesibilidad:** letra grande (mínimo 24 px), alto contraste, todo feedback visual (no depender de audio), un solo clic para todo. Sin dificultades motrices especiales.
- **Arranque:** acceso directo en el escritorio; ventana maximizada. Sin autoarranque ni modo kiosco.
- **Modularidad:** añadir contenido (guías, botones, secciones) no requiere tocar código. Solo un tipo de tarjeta nuevo requiere actualizar la app.

## 3. Stack elegido

**Tauri v2** (frontend web + backend Rust mínimo).

- Instalador ~10 MB; usa WebView2, incluido de serie en Windows 11.
- Auto-actualización con `tauri-plugin-updater` contra GitHub Releases.
- Alternativas descartadas: Electron (instalador 80–150 MB, más RAM), carpeta estática + Edge `--app` (actualización y experiencia menos robustas).

Frontend: HTML/CSS/JS con tooling ligero (Vite). Sin framework pesado; la UI son tres pantallas dirigidas por datos.

## 4. Arquitectura general

Dos repos independientes:

```
┌──────────────────────────┐      ┌──────────────────────────┐
│ APP (repo: entorno-app)  │      │ CONTENIDO                │
│ Tauri v2                 │      │ (repo: entorno-contenido)│
│                          │      │                          │
│ - Shell + navegación     │◄────►│ - manifest.json          │
│ - Renderizador de        │ sync │ - guias/*.md             │
│   manifest               │      │ - img/ (capturas)        │
│ - Visor de guías paso a  │      │ - iconos/                │
│   paso                   │      └──────────────────────────┘
│ - Motor de sincronía     │
│ - Auto-updater           │
└──────────────────────────┘
```

**Separación clave:** la app no sabe qué botones existen; lee `manifest.json` y pinta. Añadir guía o periódico = editar contenido + push. La app solo cambia cuando hay funcionalidad nueva (tipo de tarjeta nuevo), y se actualiza sola.

**Flujo de arranque:**

1. La app abre maximizada mostrando el contenido cacheado local (instantáneo, sin internet).
2. En segundo plano comprueba GitHub: si hay contenido nuevo, lo descarga, valida y refresca silenciosamente.
3. Si hay versión nueva de la app, se descarga y aplica al siguiente arranque.

**Enlaces externos:** tarjeta → apertura con el navegador por defecto del sistema. Cerrar el navegador devuelve al entorno intacto.

## 5. Modelo de contenido

### 5.1 `manifest.json`

Plano del entorno. Define secciones, grupos opcionales y tarjetas:

```json
{
  "version": 3,
  "secciones": [
    {
      "id": "aprender",
      "titulo": "Aprender",
      "icono": "libro.svg",
      "color": "#2E7D32",
      "tarjetas": [
        { "tipo": "guia", "titulo": "Enviar un correo", "guia": "guias/correo-enviar.md" }
      ]
    },
    {
      "id": "prensa",
      "titulo": "Prensa",
      "icono": "periodico.svg",
      "color": "#1565C0",
      "grupos": [
        {
          "titulo": "Deportes",
          "tarjetas": [
            { "tipo": "enlace", "titulo": "Marca", "url": "https://www.marca.com", "icono": "marca.png" }
          ]
        }
      ]
    }
  ]
}
```

Reglas:

- Una sección tiene `tarjetas` directas o `grupos` (cada grupo con sus `tarjetas`), no ambos.
- `version` es un entero que crece con cada publicación de contenido; la app lo muestra en la pantalla admin.

### 5.2 Tipos de tarjeta

| Tipo | Acción | Estado |
|------|--------|--------|
| `enlace` | Abre `url` en el navegador por defecto | v1 |
| `guia` | Abre el visor paso a paso con el `.md` indicado | v1 |
| `programa` | Lanza un ejecutable local | futuro |
| `carpeta` | Abre una carpeta en el Explorador | futuro |

Un tipo desconocido para la app instalada se ignora sin romper el render (tolerancia hacia adelante: contenido nuevo no rompe apps viejas).

### 5.3 Formato de guías

Markdown con convención:

```markdown
---
titulo: Enviar un correo
icono: sobre.svg
---

## Paso 1: Abre Gmail
Pulsa el botón azul que dice "Gmail".
![captura](img/correo-01.png)

## Paso 2: Pulsa "Redactar"
...
```

- Cada encabezado `## Paso` = una pantalla del asistente.
- Frontmatter opcional (título, icono); si falta, se usa el título de la tarjeta.
- Imágenes relativas al repo de contenido.

### 5.4 Validación de contenido

- JSON Schema del manifest en el repo de contenido.
- Script `npm run check`: valida manifest contra el schema, existencia de todos los archivos referenciados (guías, imágenes, iconos) y que cada guía tiene al menos un `## Paso`.
- GitHub Action ejecuta el check en cada push: un typo pone el CI en rojo y no llega a los PCs (la app además revalida antes de aplicar, ver §7).

## 6. Pantallas y UX

Tres pantallas; profundidad máxima 2 clics.

1. **Inicio:** saludo según hora del día + reloj; parrilla de secciones (tarjetas grandes con icono, título y color propios).
2. **Sección:** tarjetas de la sección, con encabezados de grupo si los hay. Botón «🏠 Inicio» grande, fijo, siempre visible arriba a la izquierda.
3. **Visor de guía:** asistente de un paso por pantalla; captura grande, texto grande, botones «Anterior»/«Siguiente» gigantes abajo, indicador "Paso 2 de 5", «🏠 Inicio» siempre presente.

Reglas de diseño (constantes CSS, un solo tema):

- Texto base mínimo 24 px; títulos 32–40 px.
- Botones y tarjetas de mínimo 120 px de alto; hover con borde grueso y cambio de color.
- Un solo clic para todo. Sin doble clic, arrastrar, menús desplegables ni scroll horizontal.
- Alto contraste, fondo claro, color identificativo por sección.
- Todo feedback es visual; ningún flujo depende de sonido.
- Ventana maximizada al abrir; sin estados ocultos que teclas accidentales (F11, Esc) puedan romper.

**Principio anti-miedo:** ninguna acción dentro de la app borra, configura ni instala nada. El peor caso posible es abrir el navegador.

## 7. Sincronización y actualizaciones

### 7.1 Contenido

- Repo `entorno-contenido` **público** en GitHub → sin tokens ni credenciales en los PCs del padre. No contiene nada sensible.
- Al arrancar y cada 6 horas: compara el SHA del último commit (API de GitHub) con el guardado localmente.
- Si difiere: descarga el ZIP del repo (`codeload`), lo extrae a carpeta temporal, **valida** (manifest parsea y cumple schema, guías referenciadas existen) y hace **swap atómico** de la carpeta cacheada en app-data.
- Si la descarga o validación falla: se conserva la versión anterior y se registra en el log. El padre nunca ve contenido roto.

### 7.2 App

- `tauri-plugin-updater` contra GitHub Releases. Comprobación silenciosa al arrancar; instala y aplica al siguiente arranque.
- GitHub Action en `entorno-app`: al hacer push de un tag, compila, firma el instalador (firma de updater de Tauri) y publica la release.

## 8. Manejo de errores

Principio: **el padre nunca ve un mensaje de error.**

| Situación | Comportamiento |
|-----------|----------------|
| Sin internet | Usa caché; cero diálogos. Indicador discreto opcional con fecha del contenido |
| Fallo de sync | Log local; reintento en el siguiente ciclo |
| ZIP corrupto / manifest inválido | Se descarta; permanece la versión anterior |
| Imagen de guía ausente | Placeholder; la guía continúa |
| URL rota | La gestiona el navegador; la app no interviene |
| Tipo de tarjeta desconocido | Se ignora en el render |

**Pantalla admin oculta** (Ctrl+Mayús+A): versión de app, versión de contenido, fecha del último sync, log reciente, botón "forzar sincronización". Pensada para asistencia telefónica: "pulsa Control-Mayúsculas-A y dime qué pone".

## 9. Testing

- **Unit (Vitest):** parser del manifest, troceado de guías por `## Paso`, lógica de comparación de versiones/SHA del sync.
- **Schema + CI de contenido:** GitHub Action con `npm run check` en cada push del repo de contenido.
- **Smoke manual** en la máquina del autor antes de cada release de la app.

## 10. Hoja de ruta

| Bloque | Entrega | Resultado verificable |
|--------|---------|----------------------|
| 0 | Repos, scaffolding Tauri v2 + Vite, CI básico | App vacía compila y abre ventana |
| 1 | Núcleo: shell, render del manifest, pantallas Inicio y Sección, tarjetas `enlace` | Panel navegable con prensa y juegos reales |
| 2 | Visor de guías paso a paso | Una guía de ejemplo completa funcionando |
| 3 | Sync de contenido + caché + validación + pantalla admin | Push en GitHub → la app refresca sola |
| 4 | Auto-update, instalador, icono, acceso directo | Instalado y actualizándose en ambos PCs |
| 5 | Contenido real v1: guías de ofimática, kiosco de prensa completo, juegos | El padre lo usa a diario |
| Futuro | Tipos `programa` y `carpeta`, nuevas guías, nuevas secciones | Solo ediciones de JSON o módulos pequeños |

Cada bloque se planifica e implementa por separado (spec → plan → implementación).
