# Entorno Digital para Papá — Plan de Implementación (Bloques 0–5)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Panel de escritorio Tauri v2 para el padre del autor: guías paso a paso, kiosco de prensa y juegos, con contenido sincronizado desde GitHub y auto-actualización.

**Architecture:** Dos repos independientes: `entorno-app` (Tauri v2 + Vite vanilla JS) y `entorno-contenido` (manifest.json + guías Markdown). La app renderiza lo que dicta el manifest; un motor de sync en Rust descarga, valida y hace swap atómico del contenido cacheado. Enlaces externos abren el navegador por defecto.

**Tech Stack:** Tauri v2 (Rust), Vite + vanilla JS, Vitest + jsdom, marked, Ajv (CI contenido), reqwest + zip (sync), plugins Tauri: opener, updater, process, log.

## Global Constraints

- Windows 11 únicamente; ambos PCs destino con WebView2 de serie.
- Tauri v2 (no v1). Identificador de app: `com.marquibel.entorno`.
- Todo el texto de UI en español. Texto base mínimo **24 px**, títulos 32–40 px, tarjetas/botones mínimo **120 px** de alto.
- Un solo clic para todo; sin doble clic, arrastrar, desplegables ni scroll horizontal.
- El padre **nunca ve mensajes de error**: los fallos van al log y se degradan en silencio.
- La app funciona offline con el último contenido cacheado.
- Tipos de tarjeta v1: `enlace` y `guia`. Tipos desconocidos se ignoran sin romper el render.
- Repos GitHub: `marquib3l/entorno-app` (privado o público, con Releases públicas) y `marquib3l/entorno-contenido` (**público**, sin datos sensibles). Si el username real de GitHub difiere, se cambia en `src-tauri/src/config.rs` y en los workflows.
- Identidad git en cada repo nuevo: `user.name = marquib3l`, `user.email = marquib3l@gmail.com` (configurar sin preguntar).
- Directorio raíz de trabajo: `B:\01_Proyectos\Entorno-para-Papa`. Los dos repos viven como subcarpetas hermanas: `entorno-app/` y `entorno-contenido/`.

## Estructura de ficheros final

```
Entorno-para-Papa/               (repo raíz: solo docs)
├── docs/superpowers/...
├── .gitignore                   (ignora entorno-app/ y entorno-contenido/)
│
├── entorno-app/                 (repo git propio)
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── src/
│   │   ├── main.js              (arranque, router, sync loop)
│   │   ├── lib/
│   │   │   ├── manifest.js      (parseo/filtrado del manifest)
│   │   │   ├── guia.js          (parser de guías MD → pasos)
│   │   │   ├── saludo.js        (saludo según hora)
│   │   │   ├── router.js        (router hash)
│   │   │   ├── acciones.js      (dispatcher de tarjetas)
│   │   │   └── contenido.js     (puente a comandos Rust)
│   │   ├── ui/
│   │   │   ├── inicio.js        (pantalla Inicio)
│   │   │   ├── seccion.js       (pantalla Sección)
│   │   │   ├── guia.js          (visor asistente)
│   │   │   └── admin.js         (pantalla admin oculta)
│   │   └── styles/
│   │       ├── tokens.css       (tamaños, colores, constantes)
│   │       └── base.css         (layout, tarjetas, botones)
│   ├── tests/                   (Vitest)
│   ├── recursos/
│   │   ├── icono.svg / icono.png
│   │   └── contenido-semilla/   (copia del contenido para 1er arranque)
│   ├── scripts/
│   │   └── actualizar-semilla.mjs
│   └── src-tauri/
│       ├── tauri.conf.json
│       ├── capabilities/default.json
│       └── src/
│           ├── lib.rs           (registro de comandos, setup)
│           ├── config.rs        (owner/repo GitHub, ruta dev)
│           ├── contenido.rs     (rutas, lectura, semilla, copiar_dir)
│           └── sync.rs          (sha remoto, zip, validar, swap, meta)
│
└── entorno-contenido/           (repo git propio, público)
    ├── manifest.json
    ├── schema/manifest.schema.json
    ├── guias/*.md
    ├── img/                     (capturas de guías)
    ├── iconos/                  (iconos de secciones y tarjetas)
    ├── scripts/check.mjs        (validador)
    ├── scripts/descargar-iconos.mjs
    ├── package.json
    └── .github/workflows/check.yml
```

---

# BLOQUE 0 — Scaffolding

### Task 1: Estructura raíz + app Tauri que compila y abre

**Files:**
- Create: `.gitignore` (raíz)
- Create: `entorno-app/` completo (Vite + Tauri scaffolding)
- Modify: `entorno-app/src-tauri/tauri.conf.json`

**Interfaces:**
- Produces: proyecto `entorno-app` con `npm run tauri dev` funcional; ventana maximizada titulada "Entorno de Papá".

- [x] **Step 1: gitignore raíz y commit**

En `B:\01_Proyectos\Entorno-para-Papa` crear `.gitignore`:

```
entorno-app/
entorno-contenido/
```

```bash
git add .gitignore && git commit -m "chore: ignorar repos anidados"
```

- [x] **Step 2: Scaffold Vite vanilla**

```bash
cd /b/01_Proyectos/Entorno-para-Papa
npm create vite@latest entorno-app -- --template vanilla
cd entorno-app
npm install
```

Borrar demo: `src/counter.js`, `src/javascript.svg`, `public/vite.svg`. Dejar `src/main.js` mínimo:

```js
document.querySelector('#app').textContent = 'Entorno de Papá';
```

Y `index.html` con `<html lang="es">`, `<title>Entorno de Papá</title>`, `<div id="app"></div>` y el script module a `/src/main.js` (sin más).

- [x] **Step 3: Añadir Tauri v2**

```bash
npm install -D @tauri-apps/cli@^2
npm install @tauri-apps/api@^2
npx tauri init --app-name entorno-papa --window-title "Entorno de Papá" \
  --frontend-dist ../dist --dev-url http://localhost:5173 \
  --before-dev-command "npm run dev" --before-build-command "npm run build"
```

- [x] **Step 4: Configurar tauri.conf.json**

En `src-tauri/tauri.conf.json` ajustar:

```json
{
  "identifier": "com.marquibel.entorno",
  "productName": "Entorno de Papa",
  "app": {
    "windows": [
      {
        "title": "Entorno de Papá",
        "maximized": true,
        "minWidth": 1024,
        "minHeight": 700
      }
    ]
  }
}
```

(El resto de claves generadas se conservan.)

- [x] **Step 5: Verificar que compila y abre**

Run: `npm run tauri dev`
Expected: ventana maximizada "Entorno de Papá" con el texto placeholder. Cerrar.

- [x] **Step 6: git init + commit**

```bash
cd entorno-app
git init && git config user.email "marquib3l@gmail.com" && git config user.name "marquib3l"
git add -A && git commit -m "feat: scaffolding Tauri v2 + Vite vanilla"
```

### Task 2: Vitest operativo

**Files:**
- Modify: `entorno-app/package.json` (script `test`)
- Create: `entorno-app/vitest.config.js`
- Create: `entorno-app/src/lib/saludo.js`
- Test: `entorno-app/tests/saludo.test.js`

**Interfaces:**
- Produces: `saludo(hora: number): string` — "Buenos días, Papá" (6–13), "Buenas tardes, Papá" (14–20), "Buenas noches, Papá" (21–5). Comando `npm test` (vitest run).

- [x] **Step 1: Instalar y configurar Vitest**

```bash
npm install -D vitest jsdom
```

`vitest.config.js`:

```js
import { defineConfig } from 'vitest/config';
export default defineConfig({
  test: { environment: 'jsdom' },
});
```

En `package.json` añadir script: `"test": "vitest run"`.

- [x] **Step 2: Test que falla**

`tests/saludo.test.js`:

```js
import { describe, it, expect } from 'vitest';
import { saludo } from '../src/lib/saludo.js';

describe('saludo', () => {
  it('mañana', () => expect(saludo(9)).toBe('Buenos días, Papá'));
  it('límite mañana', () => expect(saludo(6)).toBe('Buenos días, Papá'));
  it('tarde', () => expect(saludo(14)).toBe('Buenas tardes, Papá'));
  it('noche', () => expect(saludo(21)).toBe('Buenas noches, Papá'));
  it('madrugada', () => expect(saludo(3)).toBe('Buenas noches, Papá'));
});
```

Run: `npm test` — Expected: FAIL (módulo no existe).

- [x] **Step 3: Implementar**

`src/lib/saludo.js`:

```js
export function saludo(hora) {
  if (hora >= 6 && hora < 14) return 'Buenos días, Papá';
  if (hora >= 14 && hora < 21) return 'Buenas tardes, Papá';
  return 'Buenas noches, Papá';
}
```

- [x] **Step 4: Verificar que pasa**

Run: `npm test` — Expected: 5 passed.

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "test: Vitest operativo con módulo saludo"
```

### Task 3: Repo de contenido con validador y CI

**Files:**
- Create: `entorno-contenido/manifest.json`
- Create: `entorno-contenido/schema/manifest.schema.json`
- Create: `entorno-contenido/guias/ejemplo-correo.md`
- Create: `entorno-contenido/scripts/check.mjs`
- Create: `entorno-contenido/package.json`
- Create: `entorno-contenido/.github/workflows/check.yml`
- Create: `entorno-contenido/iconos/.gitkeep`, `entorno-contenido/img/.gitkeep`

**Interfaces:**
- Produces: `manifest.json` conforme al schema (contrato para Tasks 4–8); convención de guías `## Paso ...` (contrato para Task 9); `npm run check` valida todo el contenido.

- [x] **Step 1: Estructura y package.json**

```bash
cd /b/01_Proyectos/Entorno-para-Papa
mkdir -p entorno-contenido/{schema,guias,img,iconos,scripts,.github/workflows}
cd entorno-contenido
git init && git config user.email "marquib3l@gmail.com" && git config user.name "marquib3l"
npm init -y && npm install -D ajv
```

En `package.json` añadir: `"type": "module"` y script `"check": "node scripts/check.mjs"`.

- [x] **Step 2: Schema del manifest**

`schema/manifest.schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["version", "secciones"],
  "properties": {
    "version": { "type": "integer", "minimum": 1 },
    "secciones": {
      "type": "array", "minItems": 1,
      "items": {
        "type": "object",
        "required": ["id", "titulo"],
        "properties": {
          "id": { "type": "string", "pattern": "^[a-z0-9-]+$" },
          "titulo": { "type": "string", "minLength": 1 },
          "icono": { "type": "string" },
          "color": { "type": "string", "pattern": "^#[0-9A-Fa-f]{6}$" },
          "tarjetas": { "$ref": "#/definitions/tarjetas" },
          "grupos": {
            "type": "array", "minItems": 1,
            "items": {
              "type": "object",
              "required": ["titulo", "tarjetas"],
              "properties": {
                "titulo": { "type": "string", "minLength": 1 },
                "tarjetas": { "$ref": "#/definitions/tarjetas" }
              }
            }
          }
        },
        "not": { "required": ["tarjetas", "grupos"] }
      }
    }
  },
  "definitions": {
    "tarjetas": {
      "type": "array", "minItems": 1,
      "items": {
        "type": "object",
        "required": ["tipo", "titulo"],
        "properties": {
          "tipo": { "type": "string" },
          "titulo": { "type": "string", "minLength": 1 },
          "url": { "type": "string", "pattern": "^https?://" },
          "guia": { "type": "string" },
          "icono": { "type": "string" }
        },
        "allOf": [
          { "if": { "properties": { "tipo": { "const": "enlace" } }, "required": ["tipo"] }, "then": { "required": ["url"] } },
          { "if": { "properties": { "tipo": { "const": "guia" } }, "required": ["tipo"] }, "then": { "required": ["guia"] } }
        ]
      }
    }
  }
}
```

- [x] **Step 3: manifest.json inicial**

```json
{
  "version": 1,
  "secciones": [
    {
      "id": "aprender",
      "titulo": "Aprender",
      "color": "#2E7D32",
      "tarjetas": [
        { "tipo": "guia", "titulo": "Enviar un correo", "guia": "guias/ejemplo-correo.md" }
      ]
    },
    {
      "id": "prensa",
      "titulo": "Prensa",
      "color": "#1565C0",
      "grupos": [
        {
          "titulo": "Nacional",
          "tarjetas": [
            { "tipo": "enlace", "titulo": "El País", "url": "https://elpais.com" },
            { "tipo": "enlace", "titulo": "El Mundo", "url": "https://www.elmundo.es" }
          ]
        },
        {
          "titulo": "Deportes",
          "tarjetas": [
            { "tipo": "enlace", "titulo": "Marca", "url": "https://www.marca.com" }
          ]
        }
      ]
    },
    {
      "id": "jugar",
      "titulo": "Jugar",
      "color": "#E65100",
      "tarjetas": [
        { "tipo": "enlace", "titulo": "Solitario", "url": "https://www.solitr.com/es" },
        { "tipo": "enlace", "titulo": "Tetris", "url": "https://tetris.com/play-tetris" }
      ]
    }
  ]
}
```

- [x] **Step 4: Guía de ejemplo**

`guias/ejemplo-correo.md`:

```markdown
---
titulo: Enviar un correo
---

## Paso 1: Abre el navegador
Haz clic en el icono del navegador en la barra de abajo.

## Paso 2: Entra en Gmail
Escribe gmail.com en la barra de arriba y pulsa Enter.

## Paso 3: Pulsa "Redactar"
Es el botón grande de la izquierda con un lápiz.
```

- [x] **Step 5: Validador check.mjs**

`scripts/check.mjs`:

```js
import { readFileSync, existsSync } from 'node:fs';
import { join, dirname } from 'node:path';
import { fileURLToPath } from 'node:url';
import Ajv from 'ajv';

const raiz = join(dirname(fileURLToPath(import.meta.url)), '..');
const manifest = JSON.parse(readFileSync(join(raiz, 'manifest.json'), 'utf8'));
const schema = JSON.parse(readFileSync(join(raiz, 'schema/manifest.schema.json'), 'utf8'));

const ajv = new Ajv({ allErrors: true });
if (!ajv.validate(schema, manifest)) {
  console.error('manifest.json no cumple el schema:');
  console.error(ajv.errors);
  process.exit(1);
}

const errores = [];
const tarjetas = manifest.secciones.flatMap(s =>
  (s.tarjetas ?? []).concat((s.grupos ?? []).flatMap(g => g.tarjetas)));

for (const t of tarjetas) {
  if (t.tipo === 'guia') {
    const ruta = join(raiz, t.guia);
    if (!existsSync(ruta)) errores.push(`guía no existe: ${t.guia}`);
    else if (!/^##\s+Paso/im.test(readFileSync(ruta, 'utf8')))
      errores.push(`guía sin pasos (## Paso ...): ${t.guia}`);
  }
  if (t.icono && !existsSync(join(raiz, 'iconos', t.icono)))
    errores.push(`icono no existe: iconos/${t.icono}`);
}
for (const s of manifest.secciones)
  if (s.icono && !existsSync(join(raiz, 'iconos', s.icono)))
    errores.push(`icono no existe: iconos/${s.icono}`);

if (errores.length) { console.error(errores.join('\n')); process.exit(1); }
console.log('Contenido OK');
```

- [x] **Step 6: Probar el validador — pasa y falla**

Run: `npm run check` — Expected: `Contenido OK`.
Romper a propósito: cambiar `"guia": "guias/no-existe.md"` → `npm run check` — Expected: exit 1 con `guía no existe`. Deshacer el cambio.

- [x] **Step 7: GitHub Action**

`.github/workflows/check.yml`:

```yaml
name: check
on: [push, pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run check
```

- [x] **Step 8: Commit**

```bash
echo "node_modules/" > .gitignore
git add -A && git commit -m "feat: contenido inicial con schema, validador y CI"
```

---

# BLOQUE 1 — Núcleo del panel

### Task 4: Parser del manifest (JS)

**Files:**
- Create: `entorno-app/src/lib/manifest.js`
- Test: `entorno-app/tests/manifest.test.js`

**Interfaces:**
- Produces: `parsearManifest(textoJson: string): { version, secciones }` — lanza `Error` con mensaje claro si es inválido; filtra tarjetas de tipo desconocido. `TIPOS_CONOCIDOS = ['enlace', 'guia']`.
- Consumes: formato manifest de Task 3.

- [x] **Step 1: Tests que fallan**

`tests/manifest.test.js`:

```js
import { describe, it, expect } from 'vitest';
import { parsearManifest, TIPOS_CONOCIDOS } from '../src/lib/manifest.js';

const valido = JSON.stringify({
  version: 1,
  secciones: [
    { id: 'a', titulo: 'A', tarjetas: [{ tipo: 'enlace', titulo: 'X', url: 'https://x.com' }] },
    { id: 'b', titulo: 'B', grupos: [{ titulo: 'G', tarjetas: [{ tipo: 'guia', titulo: 'Y', guia: 'guias/y.md' }] }] },
  ],
});

describe('parsearManifest', () => {
  it('parsea manifest válido', () => {
    const m = parsearManifest(valido);
    expect(m.version).toBe(1);
    expect(m.secciones).toHaveLength(2);
  });
  it('rechaza JSON roto', () => {
    expect(() => parsearManifest('{oops')).toThrow(/JSON/);
  });
  it('rechaza manifest sin secciones', () => {
    expect(() => parsearManifest('{"version":1,"secciones":[]}')).toThrow(/secciones/);
  });
  it('rechaza sección con tarjetas y grupos a la vez', () => {
    const malo = JSON.stringify({ version: 1, secciones: [{ id: 'a', titulo: 'A', tarjetas: [], grupos: [] }] });
    expect(() => parsearManifest(malo)).toThrow(/a la vez/);
  });
  it('ignora tipos de tarjeta desconocidos', () => {
    const conFuturo = JSON.stringify({
      version: 2,
      secciones: [{ id: 'a', titulo: 'A', tarjetas: [
        { tipo: 'holograma3d', titulo: 'Futuro' },
        { tipo: 'enlace', titulo: 'X', url: 'https://x.com' },
      ] }],
    });
    const m = parsearManifest(conFuturo);
    expect(m.secciones[0].tarjetas).toHaveLength(1);
    expect(m.secciones[0].tarjetas[0].tipo).toBe('enlace');
  });
  it('expone los tipos conocidos', () => {
    expect(TIPOS_CONOCIDOS).toEqual(['enlace', 'guia']);
  });
});
```

Run: `npm test` — Expected: FAIL (módulo no existe).

- [x] **Step 2: Implementación**

`src/lib/manifest.js`:

```js
export const TIPOS_CONOCIDOS = ['enlace', 'guia'];

export function parsearManifest(textoJson) {
  let datos;
  try {
    datos = JSON.parse(textoJson);
  } catch (e) {
    throw new Error(`manifest.json no es JSON válido: ${e.message}`);
  }
  if (!Number.isInteger(datos.version)) throw new Error('manifest: falta "version" entera');
  if (!Array.isArray(datos.secciones) || datos.secciones.length === 0)
    throw new Error('manifest: "secciones" vacío o ausente');

  const secciones = datos.secciones.map((s) => {
    if (!s.id || !s.titulo) throw new Error('manifest: sección sin id o titulo');
    if (s.tarjetas && s.grupos) throw new Error(`sección ${s.id}: tarjetas y grupos a la vez`);
    return {
      ...s,
      tarjetas: s.tarjetas ? filtrarTarjetas(s.tarjetas) : undefined,
      grupos: s.grupos
        ? s.grupos.map((g) => ({ ...g, tarjetas: filtrarTarjetas(g.tarjetas) }))
        : undefined,
    };
  });
  return { version: datos.version, secciones };
}

function filtrarTarjetas(tarjetas = []) {
  return tarjetas.filter((t) => TIPOS_CONOCIDOS.includes(t.tipo));
}
```

- [x] **Step 3: Verificar**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: parser del manifest con tolerancia a tipos futuros"
```

### Task 5: Router hash + pantalla Inicio

**Files:**
- Create: `entorno-app/src/lib/router.js`
- Create: `entorno-app/src/ui/inicio.js`
- Test: `entorno-app/tests/router.test.js`, `entorno-app/tests/inicio.test.js`

**Interfaces:**
- Consumes: `saludo()` (Task 2), manifest parseado (Task 4).
- Produces: `registrarRuta(patron: RegExp, handler)`, `iniciarRouter(raiz: HTMLElement)`, `navegarA(hash: string)`; `renderInicio(manifest, { navegarA }): HTMLElement`. Rutas: `#/` inicio, `#/seccion/<id>`, `#/guia/<rutaCodificada>`.

- [x] **Step 1: Tests que fallan**

`tests/router.test.js`:

```js
import { describe, it, expect, beforeEach } from 'vitest';
import { registrarRuta, iniciarRouter, limpiarRutas } from '../src/lib/router.js';

describe('router', () => {
  beforeEach(() => { limpiarRutas(); location.hash = ''; });

  it('renderiza la ruta que coincide', () => {
    const raiz = document.createElement('div');
    registrarRuta(/^#\/$/, () => Object.assign(document.createElement('p'), { textContent: 'inicio' }));
    iniciarRouter(raiz);
    expect(raiz.textContent).toBe('inicio');
  });

  it('pasa grupos capturados al handler', () => {
    const raiz = document.createElement('div');
    location.hash = '#/seccion/prensa';
    registrarRuta(/^#\/$/, () => document.createElement('p'));
    registrarRuta(/^#\/seccion\/([a-z0-9-]+)$/, (id) =>
      Object.assign(document.createElement('p'), { textContent: id }));
    iniciarRouter(raiz);
    expect(raiz.textContent).toBe('prensa');
  });

  it('hash desconocido redirige a inicio', () => {
    const raiz = document.createElement('div');
    location.hash = '#/nada';
    registrarRuta(/^#\/$/, () => Object.assign(document.createElement('p'), { textContent: 'inicio' }));
    iniciarRouter(raiz);
    expect(location.hash).toBe('#/');
  });
});
```

`tests/inicio.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { renderInicio } from '../src/ui/inicio.js';

const manifest = {
  version: 1,
  secciones: [
    { id: 'aprender', titulo: 'Aprender', color: '#2E7D32', tarjetas: [] },
    { id: 'prensa', titulo: 'Prensa', color: '#1565C0', grupos: [] },
  ],
};

describe('renderInicio', () => {
  it('pinta una tarjeta por sección con su título', () => {
    const el = renderInicio(manifest, { navegarA: () => {} });
    const botones = el.querySelectorAll('button.tarjeta-seccion');
    expect(botones).toHaveLength(2);
    expect(botones[0].textContent).toContain('Aprender');
  });

  it('clic en sección navega a su ruta', () => {
    const navegarA = vi.fn();
    const el = renderInicio(manifest, { navegarA });
    el.querySelectorAll('button.tarjeta-seccion')[1].click();
    expect(navegarA).toHaveBeenCalledWith('#/seccion/prensa');
  });

  it('incluye saludo y reloj', () => {
    const el = renderInicio(manifest, { navegarA: () => {} });
    expect(el.querySelector('.saludo').textContent).toMatch(/Buen[oa]s/);
    expect(el.querySelector('.reloj')).toBeTruthy();
  });
});
```

Run: `npm test` — Expected: FAIL.

- [x] **Step 2: Implementar router**

`src/lib/router.js`:

```js
let rutas = [];

export function limpiarRutas() { rutas = []; }

export function registrarRuta(patron, handler) { rutas.push({ patron, handler }); }

export function navegarA(hash) { location.hash = hash; }

export function iniciarRouter(raiz) {
  const resolver = () => {
    const hash = location.hash || '#/';
    for (const { patron, handler } of rutas) {
      const m = hash.match(patron);
      if (m) { raiz.replaceChildren(handler(...m.slice(1))); return; }
    }
    location.hash = '#/';
  };
  window.addEventListener('hashchange', resolver);
  resolver();
}
```

- [x] **Step 3: Implementar pantalla Inicio**

`src/ui/inicio.js`:

```js
import { saludo } from '../lib/saludo.js';

export function renderInicio(manifest, { navegarA }) {
  const el = document.createElement('main');
  el.className = 'pantalla-inicio';

  const cabecera = document.createElement('header');
  const elSaludo = document.createElement('h1');
  elSaludo.className = 'saludo';
  elSaludo.textContent = saludo(new Date().getHours());
  const reloj = document.createElement('p');
  reloj.className = 'reloj';
  const pintarHora = () => {
    reloj.textContent = new Date().toLocaleTimeString('es-ES', { hour: '2-digit', minute: '2-digit' });
  };
  pintarHora();
  setInterval(pintarHora, 30_000);
  cabecera.append(elSaludo, reloj);

  const parrilla = document.createElement('div');
  parrilla.className = 'parrilla-secciones';
  for (const seccion of manifest.secciones) {
    const boton = document.createElement('button');
    boton.className = 'tarjeta-seccion';
    boton.style.setProperty('--color-seccion', seccion.color ?? '#455A64');
    boton.textContent = seccion.titulo;
    boton.addEventListener('click', () => navegarA(`#/seccion/${seccion.id}`));
    parrilla.append(boton);
  }

  el.append(cabecera, parrilla);
  return el;
}
```

- [x] **Step 4: Verificar**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: router hash y pantalla de inicio"
```

### Task 6: Pantalla Sección

**Files:**
- Create: `entorno-app/src/ui/seccion.js`
- Test: `entorno-app/tests/seccion.test.js`

**Interfaces:**
- Consumes: manifest parseado (Task 4).
- Produces: `renderSeccion(seccion, { alPulsarTarjeta, navegarA }): HTMLElement` — pinta tarjetas directas o grupos con encabezado; botón «🏠 Inicio» fijo que navega a `#/`.

- [x] **Step 1: Tests que fallan**

`tests/seccion.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { renderSeccion } from '../src/ui/seccion.js';

const conGrupos = {
  id: 'prensa', titulo: 'Prensa', color: '#1565C0',
  grupos: [
    { titulo: 'Deportes', tarjetas: [{ tipo: 'enlace', titulo: 'Marca', url: 'https://www.marca.com' }] },
    { titulo: 'Bolsa', tarjetas: [{ tipo: 'enlace', titulo: 'Expansión', url: 'https://www.expansion.com' }] },
  ],
};

describe('renderSeccion', () => {
  it('pinta encabezados de grupo y tarjetas', () => {
    const el = renderSeccion(conGrupos, { alPulsarTarjeta: () => {}, navegarA: () => {} });
    const grupos = [...el.querySelectorAll('h2.titulo-grupo')].map((h) => h.textContent);
    expect(grupos).toEqual(['Deportes', 'Bolsa']);
    expect(el.querySelectorAll('button.tarjeta')).toHaveLength(2);
  });

  it('clic en tarjeta llama a alPulsarTarjeta con la tarjeta', () => {
    const alPulsarTarjeta = vi.fn();
    const el = renderSeccion(conGrupos, { alPulsarTarjeta, navegarA: () => {} });
    el.querySelector('button.tarjeta').click();
    expect(alPulsarTarjeta).toHaveBeenCalledWith(conGrupos.grupos[0].tarjetas[0]);
  });

  it('botón Inicio navega a #/', () => {
    const navegarA = vi.fn();
    const el = renderSeccion(conGrupos, { alPulsarTarjeta: () => {}, navegarA });
    el.querySelector('button.boton-inicio').click();
    expect(navegarA).toHaveBeenCalledWith('#/');
  });

  it('sección con tarjetas directas (sin grupos)', () => {
    const simple = { id: 'jugar', titulo: 'Jugar', tarjetas: [{ tipo: 'enlace', titulo: 'Solitario', url: 'https://s.com' }] };
    const el = renderSeccion(simple, { alPulsarTarjeta: () => {}, navegarA: () => {} });
    expect(el.querySelectorAll('button.tarjeta')).toHaveLength(1);
    expect(el.querySelectorAll('h2.titulo-grupo')).toHaveLength(0);
  });
});
```

Run: `npm test` — Expected: FAIL.

- [x] **Step 2: Implementación**

`src/ui/seccion.js`:

```js
export function renderSeccion(seccion, { alPulsarTarjeta, navegarA }) {
  const el = document.createElement('main');
  el.className = 'pantalla-seccion';
  el.style.setProperty('--color-seccion', seccion.color ?? '#455A64');

  const cabecera = document.createElement('header');
  const botonInicio = document.createElement('button');
  botonInicio.className = 'boton-inicio';
  botonInicio.textContent = '🏠 Inicio';
  botonInicio.addEventListener('click', () => navegarA('#/'));
  const titulo = document.createElement('h1');
  titulo.textContent = seccion.titulo;
  cabecera.append(botonInicio, titulo);
  el.append(cabecera);

  const pintarTarjetas = (tarjetas, contenedor) => {
    for (const tarjeta of tarjetas) {
      const boton = document.createElement('button');
      boton.className = 'tarjeta';
      boton.textContent = tarjeta.titulo;
      boton.addEventListener('click', () => alPulsarTarjeta(tarjeta));
      contenedor.append(boton);
    }
  };

  if (seccion.grupos) {
    for (const grupo of seccion.grupos) {
      const tituloGrupo = document.createElement('h2');
      tituloGrupo.className = 'titulo-grupo';
      tituloGrupo.textContent = grupo.titulo;
      const parrilla = document.createElement('div');
      parrilla.className = 'parrilla-tarjetas';
      pintarTarjetas(grupo.tarjetas, parrilla);
      el.append(tituloGrupo, parrilla);
    }
  } else {
    const parrilla = document.createElement('div');
    parrilla.className = 'parrilla-tarjetas';
    pintarTarjetas(seccion.tarjetas ?? [], parrilla);
    el.append(parrilla);
  }
  return el;
}
```

- [x] **Step 3: Verificar**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: pantalla de sección con grupos y tarjetas"
```

### Task 7: Dispatcher de acciones + apertura de enlaces

**Files:**
- Create: `entorno-app/src/lib/acciones.js`
- Test: `entorno-app/tests/acciones.test.js`
- Modify: `entorno-app/src-tauri/capabilities/default.json`

**Interfaces:**
- Consumes: tarjetas del manifest (Task 4).
- Produces: `ejecutarTarjeta(tarjeta, { abrirUrl, navegarA })` — `enlace` → `abrirUrl(url)`; `guia` → `navegarA('#/guia/' + encodeURIComponent(ruta))`. En producción `abrirUrl` = `openUrl` de `@tauri-apps/plugin-opener`.

- [x] **Step 1: Tests que fallan**

`tests/acciones.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { ejecutarTarjeta } from '../src/lib/acciones.js';

describe('ejecutarTarjeta', () => {
  it('enlace abre la URL', () => {
    const abrirUrl = vi.fn();
    ejecutarTarjeta({ tipo: 'enlace', titulo: 'X', url: 'https://x.com' }, { abrirUrl, navegarA: () => {} });
    expect(abrirUrl).toHaveBeenCalledWith('https://x.com');
  });

  it('guia navega al visor con la ruta codificada', () => {
    const navegarA = vi.fn();
    ejecutarTarjeta({ tipo: 'guia', titulo: 'Y', guia: 'guias/mi guía.md' }, { abrirUrl: () => {}, navegarA });
    expect(navegarA).toHaveBeenCalledWith(`#/guia/${encodeURIComponent('guias/mi guía.md')}`);
  });

  it('tipo desconocido no hace nada ni lanza', () => {
    expect(() =>
      ejecutarTarjeta({ tipo: 'holograma3d', titulo: 'Z' }, { abrirUrl: () => {}, navegarA: () => {} })
    ).not.toThrow();
  });
});
```

Run: `npm test` — Expected: FAIL.

- [x] **Step 2: Implementación**

`src/lib/acciones.js`:

```js
export function ejecutarTarjeta(tarjeta, { abrirUrl, navegarA }) {
  if (tarjeta.tipo === 'enlace') return abrirUrl(tarjeta.url);
  if (tarjeta.tipo === 'guia') return navegarA(`#/guia/${encodeURIComponent(tarjeta.guia)}`);
}
```

- [x] **Step 3: Verificar tests**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 4: Añadir plugin opener**

```bash
npm run tauri add opener
```

En `src-tauri/capabilities/default.json`, sustituir el permiso `opener:default` por permiso con alcance de URLs web:

```json
{
  "identifier": "opener:allow-open-url",
  "allow": [{ "url": "https://*" }, { "url": "http://*" }]
}
```

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: dispatcher de acciones y plugin opener"
```

### Task 8: Estilos accesibles + integración con contenido real (dev)

**Files:**
- Create: `entorno-app/src/styles/tokens.css`, `entorno-app/src/styles/base.css`
- Create: `entorno-app/src/lib/contenido.js`
- Create: `entorno-app/src-tauri/src/config.rs`, `entorno-app/src-tauri/src/contenido.rs`
- Modify: `entorno-app/src-tauri/src/lib.rs`, `entorno-app/src/main.js`, `entorno-app/index.html`
- Modify: `entorno-app/src-tauri/tauri.conf.json` (assetProtocol)

**Interfaces:**
- Consumes: Tasks 4–7.
- Produces:
  - Rust command `contenido_leer(rel: String) -> Result<String, String>` (lee texto del dir de contenido activo, con guarda anti path-traversal).
  - Rust command `contenido_ruta(rel: String) -> Result<String, String>` (ruta absoluta para `convertFileSrc`, misma guarda).
  - JS `cargarManifest(): Promise<manifest>` y `urlRecurso(rel): Promise<string>` en `src/lib/contenido.js`.
  - En dev (`debug_assertions`) el dir de contenido es `config::DIR_CONTENIDO_DEV`; en prod es `<appdata>/contenido` (Task 14 lo puebla).

- [x] **Step 1: Rust config.rs y contenido.rs**

`src-tauri/src/config.rs`:

```rust
pub const GITHUB_OWNER: &str = "marquib3l";
pub const REPO_CONTENIDO: &str = "entorno-contenido";

#[cfg(debug_assertions)]
pub const DIR_CONTENIDO_DEV: &str = r"B:\01_Proyectos\Entorno-para-Papa\entorno-contenido";
```

`src-tauri/src/contenido.rs`:

```rust
use std::path::PathBuf;
use tauri::Manager;

pub fn dir_contenido(app: &tauri::AppHandle) -> Result<PathBuf, String> {
    #[cfg(debug_assertions)]
    {
        let _ = app;
        Ok(PathBuf::from(crate::config::DIR_CONTENIDO_DEV))
    }
    #[cfg(not(debug_assertions))]
    {
        Ok(app
            .path()
            .app_data_dir()
            .map_err(|e| e.to_string())?
            .join("contenido"))
    }
}

fn resolver_seguro(app: &tauri::AppHandle, rel: &str) -> Result<PathBuf, String> {
    let base = dir_contenido(app)?
        .canonicalize()
        .map_err(|e| format!("dir de contenido no disponible: {e}"))?;
    let ruta = base
        .join(rel)
        .canonicalize()
        .map_err(|e| format!("recurso no existe: {rel}: {e}"))?;
    if !ruta.starts_with(&base) {
        return Err(format!("ruta fuera del contenido: {rel}"));
    }
    Ok(ruta)
}

#[tauri::command]
pub fn contenido_leer(app: tauri::AppHandle, rel: String) -> Result<String, String> {
    std::fs::read_to_string(resolver_seguro(&app, &rel)?).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn contenido_ruta(app: tauri::AppHandle, rel: String) -> Result<String, String> {
    Ok(resolver_seguro(&app, &rel)?.to_string_lossy().into_owned())
}
```

En `src-tauri/src/lib.rs` declarar módulos y registrar comandos:

```rust
mod config;
mod contenido;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_opener::init())
        .invoke_handler(tauri::generate_handler![
            contenido::contenido_leer,
            contenido::contenido_ruta
        ])
        .run(tauri::generate_context!())
        .expect("error al arrancar la app");
}
```

- [x] **Step 2: Asset protocol para imágenes**

En `src-tauri/tauri.conf.json`, dentro de `app.security`:

```json
"assetProtocol": {
  "enable": true,
  "scope": [
    "$APPDATA/contenido/**",
    "B:\\01_Proyectos\\Entorno-para-Papa\\entorno-contenido\\**"
  ]
}
```

- [x] **Step 3: Puente JS**

`src/lib/contenido.js`:

```js
import { invoke, convertFileSrc } from '@tauri-apps/api/core';
import { parsearManifest } from './manifest.js';

export async function cargarManifest() {
  const texto = await invoke('contenido_leer', { rel: 'manifest.json' });
  return parsearManifest(texto);
}

export async function leerTexto(rel) {
  return invoke('contenido_leer', { rel });
}

export async function urlRecurso(rel) {
  const abs = await invoke('contenido_ruta', { rel });
  return convertFileSrc(abs);
}
```

- [x] **Step 4: main.js integrando todo**

`src/main.js`:

```js
import './styles/tokens.css';
import './styles/base.css';
import { openUrl } from '@tauri-apps/plugin-opener';
import { registrarRuta, iniciarRouter, navegarA } from './lib/router.js';
import { ejecutarTarjeta } from './lib/acciones.js';
import { cargarManifest } from './lib/contenido.js';
import { renderInicio } from './ui/inicio.js';
import { renderSeccion } from './ui/seccion.js';

let manifest;

const alPulsarTarjeta = (tarjeta) => ejecutarTarjeta(tarjeta, { abrirUrl: openUrl, navegarA });

async function arrancar() {
  manifest = await cargarManifest();
  const raiz = document.querySelector('#app');

  registrarRuta(/^#\/$/, () => renderInicio(manifest, { navegarA }));
  registrarRuta(/^#\/seccion\/([a-z0-9-]+)$/, (id) => {
    const seccion = manifest.secciones.find((s) => s.id === id);
    if (!seccion) { navegarA('#/'); return document.createElement('div'); }
    return renderSeccion(seccion, { alPulsarTarjeta, navegarA });
  });

  iniciarRouter(raiz);
}

arrancar();
```

- [x] **Step 5: Estilos**

`src/styles/tokens.css`:

```css
:root {
  --texto-base: 24px;
  --texto-titulo: 40px;
  --texto-grupo: 32px;
  --alto-tarjeta: 120px;
  --fondo: #FAFAF5;
  --tinta: #1A1A1A;
  --color-seccion: #455A64;
  --radio: 16px;
  --sombra: 0 2px 8px rgba(0, 0, 0, 0.15);
}
```

`src/styles/base.css`:

```css
* { box-sizing: border-box; margin: 0; }

html, body { height: 100%; overflow-x: hidden; }

body {
  font-family: 'Segoe UI', system-ui, sans-serif;
  font-size: var(--texto-base);
  background: var(--fondo);
  color: var(--tinta);
}

main { padding: 32px; }

h1 { font-size: var(--texto-titulo); }
h2.titulo-grupo { font-size: var(--texto-grupo); margin: 32px 0 16px; }

button {
  font: inherit;
  cursor: pointer;
  border: 4px solid transparent;
  border-radius: var(--radio);
  box-shadow: var(--sombra);
}

button:hover, button:focus-visible {
  border-color: var(--color-seccion);
  outline: none;
  filter: brightness(1.05);
}

.pantalla-inicio header { text-align: center; margin-bottom: 48px; }
.reloj { font-size: var(--texto-grupo); color: #555; }

.parrilla-secciones {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 32px;
}

.tarjeta-seccion {
  min-height: 200px;
  font-size: var(--texto-titulo);
  font-weight: 700;
  color: #fff;
  background: var(--color-seccion);
}

.pantalla-seccion header {
  display: flex;
  align-items: center;
  gap: 32px;
  margin-bottom: 32px;
}

.boton-inicio {
  min-height: 80px;
  padding: 0 32px;
  font-size: 28px;
  font-weight: 700;
  background: #fff;
}

.parrilla-tarjetas {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 24px;
}

.tarjeta {
  min-height: var(--alto-tarjeta);
  font-size: 28px;
  font-weight: 600;
  background: #fff;
  border-left: 12px solid var(--color-seccion);
}
```

- [x] **Step 6: Verificación completa**

Run: `npm test` — Expected: todos los tests pasan.
Run: `npm run tauri dev` — Expected (checklist manual):
- Inicio muestra saludo, reloj y 3 tarjetas de sección con sus colores.
- Clic en Prensa → grupos Nacional/Deportes con tarjetas; botón «🏠 Inicio» vuelve.
- Clic en El País → se abre el navegador por defecto.
- Texto legible a distancia; nada requiere doble clic.

- [x] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: estilos accesibles e integración con contenido real en dev"
```

---

# BLOQUE 2 — Visor de guías

### Task 9: Parser de guías Markdown

**Files:**
- Create: `entorno-app/src/lib/guia.js`
- Test: `entorno-app/tests/guia.test.js`

**Interfaces:**
- Consumes: convención `## Paso ...` + frontmatter (Task 3).
- Produces: `parsearGuia(md: string): { titulo?, icono?, pasos: [{ titulo, html }] }` — lanza `Error` si no hay pasos. Los `src` de imágenes quedan tal cual (relativos); la UI los resuelve.

- [x] **Step 1: Instalar marked**

```bash
npm install marked
```

- [x] **Step 2: Tests que fallan**

`tests/guia.test.js`:

```js
import { describe, it, expect } from 'vitest';
import { parsearGuia } from '../src/lib/guia.js';

const md = `---
titulo: Enviar un correo
icono: sobre.svg
---

Texto introductorio que no es un paso.

## Paso 1: Abre Gmail
Pulsa el botón azul.

![captura](img/correo-01.png)

## Paso 2: Redactar
Pulsa **Redactar**.
`;

describe('parsearGuia', () => {
  it('extrae frontmatter', () => {
    const g = parsearGuia(md);
    expect(g.titulo).toBe('Enviar un correo');
    expect(g.icono).toBe('sobre.svg');
  });

  it('trocea por ## Paso', () => {
    const g = parsearGuia(md);
    expect(g.pasos).toHaveLength(2);
    expect(g.pasos[0].titulo).toBe('Paso 1: Abre Gmail');
    expect(g.pasos[1].titulo).toBe('Paso 2: Redactar');
  });

  it('convierte el cuerpo de cada paso a HTML', () => {
    const g = parsearGuia(md);
    expect(g.pasos[0].html).toContain('<img');
    expect(g.pasos[0].html).toContain('img/correo-01.png');
    expect(g.pasos[1].html).toContain('<strong>Redactar</strong>');
  });

  it('funciona sin frontmatter', () => {
    const g = parsearGuia('## Paso 1: Hola\nTexto.');
    expect(g.titulo).toBeUndefined();
    expect(g.pasos).toHaveLength(1);
  });

  it('lanza si no hay pasos', () => {
    expect(() => parsearGuia('# Título\nSin pasos.')).toThrow(/pasos/);
  });
});
```

Run: `npm test` — Expected: FAIL.

- [x] **Step 3: Implementación**

`src/lib/guia.js`:

```js
import { marked } from 'marked';

export function parsearGuia(md) {
  const { meta, cuerpo } = separarFrontmatter(md);
  const pasos = cuerpo
    .split(/^##\s+/m)
    .filter((b) => /^paso/i.test(b.trim()))
    .map((bloque) => {
      const [primera, ...resto] = bloque.split('\n');
      return { titulo: primera.trim(), html: marked.parse(resto.join('\n')) };
    });
  if (pasos.length === 0) throw new Error('La guía no tiene pasos (## Paso ...)');
  return { titulo: meta.titulo, icono: meta.icono, pasos };
}

function separarFrontmatter(md) {
  const m = md.match(/^---\r?\n([\s\S]*?)\r?\n---\r?\n?/);
  if (!m) return { meta: {}, cuerpo: md };
  const meta = {};
  for (const linea of m[1].split('\n')) {
    const i = linea.indexOf(':');
    if (i > 0) meta[linea.slice(0, i).trim()] = linea.slice(i + 1).trim();
  }
  return { meta, cuerpo: md.slice(m[0].length) };
}
```

- [x] **Step 4: Verificar**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: parser de guías markdown a pasos"
```

### Task 10: Visor asistente paso a paso

**Files:**
- Create: `entorno-app/src/ui/guia.js`
- Test: `entorno-app/tests/ui-guia.test.js`
- Modify: `entorno-app/src/styles/base.css` (estilos del visor)

**Interfaces:**
- Consumes: `parsearGuia` (Task 9).
- Produces: `renderGuia(guia, { navegarA, resolverImagen }): HTMLElement` — `resolverImagen(rel: string): Promise<string>` convierte src relativo en URL cargable (en producción `urlRecurso` de Task 8; en tests un stub).

- [x] **Step 1: Tests que fallan**

`tests/ui-guia.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { renderGuia } from '../src/ui/guia.js';

const guia = {
  titulo: 'Enviar un correo',
  pasos: [
    { titulo: 'Paso 1: Abre Gmail', html: '<p>Pulsa el botón azul.</p>' },
    { titulo: 'Paso 2: Redactar', html: '<p>Pulsa Redactar.</p>' },
    { titulo: 'Paso 3: Enviar', html: '<p>Pulsa Enviar.</p>' },
  ],
};

const deps = () => ({ navegarA: vi.fn(), resolverImagen: vi.fn(async (rel) => `asset://${rel}`) });

describe('renderGuia', () => {
  it('muestra el primer paso y el indicador', () => {
    const el = renderGuia(guia, deps());
    expect(el.querySelector('.titulo-paso').textContent).toBe('Paso 1: Abre Gmail');
    expect(el.querySelector('.indicador-paso').textContent).toBe('Paso 1 de 3');
  });

  it('Siguiente avanza y Anterior retrocede', () => {
    const el = renderGuia(guia, deps());
    el.querySelector('.boton-siguiente').click();
    expect(el.querySelector('.indicador-paso').textContent).toBe('Paso 2 de 3');
    el.querySelector('.boton-anterior').click();
    expect(el.querySelector('.indicador-paso').textContent).toBe('Paso 1 de 3');
  });

  it('Anterior oculto en el primer paso; en el último, Siguiente dice Terminar y va a inicio', () => {
    const d = deps();
    const el = renderGuia(guia, d);
    expect(el.querySelector('.boton-anterior').hidden).toBe(true);
    el.querySelector('.boton-siguiente').click();
    el.querySelector('.boton-siguiente').click();
    const siguiente = el.querySelector('.boton-siguiente');
    expect(siguiente.textContent).toContain('Terminar');
    siguiente.click();
    expect(d.navegarA).toHaveBeenCalledWith('#/');
  });

  it('botón Inicio siempre visible', () => {
    const d = deps();
    const el = renderGuia(guia, d);
    el.querySelector('.boton-inicio').click();
    expect(d.navegarA).toHaveBeenCalledWith('#/');
  });

  it('resuelve los src de las imágenes', async () => {
    const conImagen = {
      titulo: 'G', pasos: [{ titulo: 'Paso 1: X', html: '<img src="img/a.png">' }],
    };
    const d = deps();
    const el = renderGuia(conImagen, d);
    await Promise.resolve();
    expect(d.resolverImagen).toHaveBeenCalledWith('img/a.png');
  });
});
```

Run: `npm test` — Expected: FAIL.

- [x] **Step 2: Implementación**

`src/ui/guia.js`:

```js
export function renderGuia(guia, { navegarA, resolverImagen }) {
  const el = document.createElement('main');
  el.className = 'pantalla-guia';
  let indice = 0;

  const cabecera = document.createElement('header');
  const botonInicio = document.createElement('button');
  botonInicio.className = 'boton-inicio';
  botonInicio.textContent = '🏠 Inicio';
  botonInicio.addEventListener('click', () => navegarA('#/'));
  const titulo = document.createElement('h1');
  titulo.textContent = guia.titulo ?? '';
  cabecera.append(botonInicio, titulo);

  const tituloPaso = document.createElement('h2');
  tituloPaso.className = 'titulo-paso';
  const cuerpo = document.createElement('article');
  cuerpo.className = 'cuerpo-paso';

  const pie = document.createElement('footer');
  pie.className = 'pie-guia';
  const botonAnterior = document.createElement('button');
  botonAnterior.className = 'boton-anterior';
  botonAnterior.textContent = '⬅ Anterior';
  const indicador = document.createElement('p');
  indicador.className = 'indicador-paso';
  const botonSiguiente = document.createElement('button');
  botonSiguiente.className = 'boton-siguiente';
  pie.append(botonAnterior, indicador, botonSiguiente);

  const pintar = () => {
    const paso = guia.pasos[indice];
    tituloPaso.textContent = paso.titulo;
    cuerpo.innerHTML = paso.html;
    for (const img of cuerpo.querySelectorAll('img')) {
      const rel = img.getAttribute('src');
      if (rel && !rel.includes('://')) {
        resolverImagen(rel).then((url) => { img.src = url; });
        img.addEventListener('error', () => img.remove(), { once: true });
      }
    }
    indicador.textContent = `Paso ${indice + 1} de ${guia.pasos.length}`;
    botonAnterior.hidden = indice === 0;
    const ultimo = indice === guia.pasos.length - 1;
    botonSiguiente.textContent = ultimo ? '✔ Terminar' : 'Siguiente ➡';
  };

  botonAnterior.addEventListener('click', () => { indice -= 1; pintar(); });
  botonSiguiente.addEventListener('click', () => {
    if (indice === guia.pasos.length - 1) { navegarA('#/'); return; }
    indice += 1;
    pintar();
  });

  pintar();
  el.append(cabecera, tituloPaso, cuerpo, pie);
  return el;
}
```

Estilos a añadir en `base.css`:

```css
.pantalla-guia { display: flex; flex-direction: column; min-height: 100vh; }
.titulo-paso { font-size: var(--texto-grupo); margin: 24px 0; }
.cuerpo-paso { flex: 1; font-size: var(--texto-base); line-height: 1.6; }
.cuerpo-paso img { max-width: 100%; max-height: 50vh; border-radius: var(--radio); box-shadow: var(--sombra); }
.pie-guia { display: flex; align-items: center; justify-content: space-between; gap: 24px; padding: 24px 0; }
.boton-anterior, .boton-siguiente { min-height: 100px; min-width: 260px; font-size: 32px; font-weight: 700; background: #fff; }
.boton-siguiente { background: var(--color-seccion, #2E7D32); color: #fff; }
.indicador-paso { font-size: 28px; color: #555; }
```

- [x] **Step 3: Verificar**

Run: `npm test` — Expected: todos pasan.

- [x] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: visor de guías paso a paso"
```

### Task 11: Integrar el visor en la app

**Files:**
- Modify: `entorno-app/src/main.js` (ruta `#/guia/...`)

**Interfaces:**
- Consumes: `renderGuia` (Task 10), `leerTexto`/`urlRecurso` (Task 8), `parsearGuia` (Task 9).

- [x] **Step 1: Registrar la ruta del visor**

En `src/main.js`, añadir imports y ruta:

```js
import { parsearGuia } from './lib/guia.js';
import { renderGuia } from './ui/guia.js';
import { leerTexto, urlRecurso } from './lib/contenido.js';
```

Dentro de `arrancar()`, tras las otras rutas:

```js
registrarRuta(/^#\/guia\/(.+)$/, (rutaCodificada) => {
  const contenedor = document.createElement('div');
  const rel = decodeURIComponent(rutaCodificada);
  leerTexto(rel)
    .then((md) => {
      const guia = parsearGuia(md);
      contenedor.replaceChildren(renderGuia(guia, { navegarA, resolverImagen: urlRecurso }));
    })
    .catch((e) => {
      console.error(`No se pudo abrir la guía ${rel}:`, e);
      navegarA('#/');
    });
  return contenedor;
});
```

- [x] **Step 2: Verificación manual**

Run: `npm run tauri dev` — Expected:
- Aprender → «Enviar un correo» abre el asistente con "Paso 1 de 3".
- Siguiente/Anterior funcionan; «Terminar» vuelve al inicio.
- «🏠 Inicio» visible en todo momento.

Run: `npm test` — Expected: todos pasan (sin regresiones).

- [x] **Step 3: Commit**

```bash
git add -A && git commit -m "feat: ruta del visor de guías integrada"
```

---

# BLOQUE 3 — Sincronización de contenido

### Task 12: Validación y swap atómico (Rust, funciones puras)

**Files:**
- Create: `entorno-app/src-tauri/src/sync.rs` (parte 1: validar, swap, meta)
- Modify: `entorno-app/src-tauri/src/lib.rs` (declarar `mod sync;`)
- Modify: `entorno-app/src-tauri/Cargo.toml`

**Interfaces:**
- Produces (usadas por Task 13):
  - `validar_contenido(dir: &Path) -> Result<u64, String>` (devuelve `version` del manifest).
  - `reemplazar_contenido(dir_contenido: &Path, dir_nuevo: &Path) -> Result<(), String>` (swap con rollback).
  - `MetaContenido { sha: String, version: u64, fecha: String }` + `leer_meta(ruta) -> MetaContenido` + `guardar_meta(ruta, &MetaContenido) -> Result<(), String>`.

- [x] **Step 1: Dependencias**

En `src-tauri/Cargo.toml` añadir:

```toml
[dependencies]
# ... existentes ...
serde_json = "1"
chrono = "0.4"

[dev-dependencies]
tempfile = "3"
```

- [x] **Step 2: Tests que fallan**

En `src-tauri/src/sync.rs`, escribir primero el módulo de tests:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::fs;

    fn contenido_valido(dir: &std::path::Path) {
        fs::create_dir_all(dir.join("guias")).unwrap();
        fs::write(dir.join("guias/a.md"), "## Paso 1: Hola\nTexto.").unwrap();
        fs::write(
            dir.join("manifest.json"),
            r#"{"version":7,"secciones":[{"id":"x","titulo":"X","tarjetas":[{"tipo":"guia","titulo":"A","guia":"guias/a.md"}]}]}"#,
        )
        .unwrap();
    }

    #[test]
    fn valida_contenido_correcto() {
        let dir = tempfile::tempdir().unwrap();
        contenido_valido(dir.path());
        assert_eq!(validar_contenido(dir.path()).unwrap(), 7);
    }

    #[test]
    fn rechaza_manifest_invalido() {
        let dir = tempfile::tempdir().unwrap();
        fs::write(dir.path().join("manifest.json"), "{rotisimo").unwrap();
        assert!(validar_contenido(dir.path()).is_err());
    }

    #[test]
    fn rechaza_guia_ausente() {
        let dir = tempfile::tempdir().unwrap();
        fs::write(
            dir.path().join("manifest.json"),
            r#"{"version":1,"secciones":[{"id":"x","titulo":"X","tarjetas":[{"tipo":"guia","titulo":"A","guia":"guias/no-existe.md"}]}]}"#,
        )
        .unwrap();
        let err = validar_contenido(dir.path()).unwrap_err();
        assert!(err.contains("no-existe.md"));
    }

    #[test]
    fn swap_reemplaza_y_borra_lo_viejo() {
        let base = tempfile::tempdir().unwrap();
        let actual = base.path().join("contenido");
        let nuevo = base.path().join("nuevo");
        fs::create_dir_all(&actual).unwrap();
        fs::write(actual.join("marca.txt"), "viejo").unwrap();
        fs::create_dir_all(&nuevo).unwrap();
        fs::write(nuevo.join("marca.txt"), "nuevo").unwrap();

        reemplazar_contenido(&actual, &nuevo).unwrap();
        assert_eq!(fs::read_to_string(actual.join("marca.txt")).unwrap(), "nuevo");
        assert!(!base.path().join("contenido.old").exists());
    }

    #[test]
    fn swap_funciona_sin_contenido_previo() {
        let base = tempfile::tempdir().unwrap();
        let actual = base.path().join("contenido");
        let nuevo = base.path().join("nuevo");
        fs::create_dir_all(&nuevo).unwrap();
        fs::write(nuevo.join("marca.txt"), "nuevo").unwrap();
        reemplazar_contenido(&actual, &nuevo).unwrap();
        assert!(actual.join("marca.txt").exists());
    }

    #[test]
    fn meta_ida_y_vuelta() {
        let dir = tempfile::tempdir().unwrap();
        let ruta = dir.path().join("contenido_meta.json");
        let meta = MetaContenido { sha: "abc123".into(), version: 7, fecha: "2026-07-26T10:00:00Z".into() };
        guardar_meta(&ruta, &meta).unwrap();
        let leida = leer_meta(&ruta);
        assert_eq!(leida.sha, "abc123");
        assert_eq!(leida.version, 7);
    }

    #[test]
    fn meta_ausente_devuelve_default() {
        let leida = leer_meta(std::path::Path::new("Z:/no/existe/meta.json"));
        assert_eq!(leida.sha, "");
    }
}
```

Run: `cd src-tauri && cargo test` — Expected: FAIL (funciones no definidas).

- [x] **Step 3: Implementación**

En `src-tauri/src/sync.rs`, encima de los tests:

```rust
use std::fs;
use std::path::Path;

#[derive(serde::Serialize, serde::Deserialize, Default, Clone)]
pub struct MetaContenido {
    pub sha: String,
    pub version: u64,
    pub fecha: String,
}

pub fn leer_meta(ruta: &Path) -> MetaContenido {
    fs::read_to_string(ruta)
        .ok()
        .and_then(|t| serde_json::from_str(&t).ok())
        .unwrap_or_default()
}

pub fn guardar_meta(ruta: &Path, meta: &MetaContenido) -> Result<(), String> {
    let texto = serde_json::to_string_pretty(meta).map_err(|e| e.to_string())?;
    fs::write(ruta, texto).map_err(|e| e.to_string())
}

pub fn validar_contenido(dir: &Path) -> Result<u64, String> {
    let texto = fs::read_to_string(dir.join("manifest.json"))
        .map_err(|e| format!("no se puede leer manifest.json: {e}"))?;
    let v: serde_json::Value =
        serde_json::from_str(&texto).map_err(|e| format!("manifest.json inválido: {e}"))?;
    let version = v["version"].as_u64().ok_or("manifest sin 'version' entera")?;
    let secciones = v["secciones"]
        .as_array()
        .filter(|s| !s.is_empty())
        .ok_or("manifest sin 'secciones'")?;
    for seccion in secciones {
        for tarjeta in tarjetas_de(seccion) {
            if tarjeta["tipo"] == "guia" {
                let rel = tarjeta["guia"].as_str().ok_or("tarjeta guia sin ruta")?;
                if !dir.join(rel).is_file() {
                    return Err(format!("guía referenciada no existe: {rel}"));
                }
            }
        }
    }
    Ok(version)
}

fn tarjetas_de(seccion: &serde_json::Value) -> Vec<&serde_json::Value> {
    let mut res = Vec::new();
    if let Some(ts) = seccion["tarjetas"].as_array() {
        res.extend(ts);
    }
    if let Some(gs) = seccion["grupos"].as_array() {
        for g in gs {
            if let Some(ts) = g["tarjetas"].as_array() {
                res.extend(ts);
            }
        }
    }
    res
}

pub fn reemplazar_contenido(dir_contenido: &Path, dir_nuevo: &Path) -> Result<(), String> {
    let viejo = dir_contenido.with_extension("old");
    if viejo.exists() {
        fs::remove_dir_all(&viejo).map_err(|e| e.to_string())?;
    }
    if dir_contenido.exists() {
        fs::rename(dir_contenido, &viejo)
            .map_err(|e| format!("no se pudo apartar el contenido actual: {e}"))?;
    }
    match fs::rename(dir_nuevo, dir_contenido) {
        Ok(_) => {
            let _ = fs::remove_dir_all(&viejo);
            Ok(())
        }
        Err(e) => {
            if viejo.exists() {
                let _ = fs::rename(&viejo, dir_contenido);
            }
            Err(format!("no se pudo activar el contenido nuevo: {e}"))
        }
    }
}
```

Y en `src-tauri/src/lib.rs` añadir `mod sync;`.

- [x] **Step 4: Verificar**

Run: `cargo test` (en `src-tauri/`) — Expected: 7 tests pasan.

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: validación de contenido, swap atómico y meta (sync parte 1)"
```

### Task 13: Descarga desde GitHub y comando sync_now

**Files:**
- Modify: `entorno-app/src-tauri/src/sync.rs` (parte 2: red + orquestación)
- Modify: `entorno-app/src-tauri/src/lib.rs` (registrar comandos)
- Modify: `entorno-app/src-tauri/Cargo.toml`

**Interfaces:**
- Consumes: Task 12 (`validar_contenido`, `reemplazar_contenido`, meta), `dir_contenido` (Task 8), `config` (Task 8).
- Produces:
  - Command `sync_now(app) -> EstadoSync` — `{ estado: "actualizado"|"sin_cambios"|"error"|"dev", version?, sha?, fecha?, detalle? }`.
  - Command `estado_sync(app) -> EstadoSync` (lee meta local sin tocar red).
  - `extraer_zip(bytes: &[u8], destino: &Path) -> Result<PathBuf, String>` (devuelve carpeta raíz interna del zip).

- [x] **Step 1: Dependencias de red**

En `src-tauri/Cargo.toml`:

```toml
reqwest = { version = "0.12", features = ["json"] }
zip = "2"
```

- [x] **Step 2: Test de extraer_zip que falla**

Añadir al módulo de tests de `sync.rs`:

```rust
    #[test]
    fn extrae_zip_y_devuelve_raiz_interna() {
        let mut buf = Vec::new();
        {
            let mut zw = zip::ZipWriter::new(std::io::Cursor::new(&mut buf));
            let opciones: zip::write::SimpleFileOptions = Default::default();
            zw.add_directory("repo-main/", opciones).unwrap();
            zw.start_file("repo-main/manifest.json", opciones).unwrap();
            std::io::Write::write_all(&mut zw, b"{}").unwrap();
            zw.finish().unwrap();
        }
        let destino = tempfile::tempdir().unwrap();
        let raiz = extraer_zip(&buf, destino.path()).unwrap();
        assert!(raiz.ends_with("repo-main"));
        assert!(raiz.join("manifest.json").is_file());
    }
```

Run: `cargo test` — Expected: FAIL.

- [x] **Step 3: Implementación red + orquestación**

Añadir a `sync.rs`:

```rust
use std::path::PathBuf;

#[derive(serde::Serialize, Clone)]
pub struct EstadoSync {
    pub estado: String,
    pub sha: Option<String>,
    pub version: Option<u64>,
    pub fecha: Option<String>,
    pub detalle: Option<String>,
}

pub fn extraer_zip(bytes: &[u8], destino: &Path) -> Result<PathBuf, String> {
    let mut archivo =
        zip::ZipArchive::new(std::io::Cursor::new(bytes)).map_err(|e| e.to_string())?;
    archivo.extract(destino).map_err(|e| e.to_string())?;
    fs::read_dir(destino)
        .map_err(|e| e.to_string())?
        .filter_map(|e| e.ok())
        .map(|e| e.path())
        .find(|p| p.is_dir())
        .ok_or_else(|| "zip sin carpeta raíz".to_string())
}

async fn obtener_sha_remoto() -> Result<String, String> {
    let url = format!(
        "https://api.github.com/repos/{}/{}/commits/main",
        crate::config::GITHUB_OWNER,
        crate::config::REPO_CONTENIDO
    );
    let resp = reqwest::Client::new()
        .get(&url)
        .header("User-Agent", "entorno-papa")
        .send()
        .await
        .map_err(|e| e.to_string())?
        .error_for_status()
        .map_err(|e| e.to_string())?;
    let v: serde_json::Value = resp.json().await.map_err(|e| e.to_string())?;
    v["sha"]
        .as_str()
        .map(String::from)
        .ok_or_else(|| "respuesta de GitHub sin sha".to_string())
}

async fn descargar_zip_remoto() -> Result<Vec<u8>, String> {
    let url = format!(
        "https://codeload.github.com/{}/{}/zip/refs/heads/main",
        crate::config::GITHUB_OWNER,
        crate::config::REPO_CONTENIDO
    );
    let resp = reqwest::Client::new()
        .get(&url)
        .header("User-Agent", "entorno-papa")
        .send()
        .await
        .map_err(|e| e.to_string())?
        .error_for_status()
        .map_err(|e| e.to_string())?;
    Ok(resp.bytes().await.map_err(|e| e.to_string())?.to_vec())
}

fn ruta_meta(app: &tauri::AppHandle) -> Result<PathBuf, String> {
    use tauri::Manager;
    Ok(app
        .path()
        .app_data_dir()
        .map_err(|e| e.to_string())?
        .join("contenido_meta.json"))
}

#[tauri::command]
pub fn estado_sync(app: tauri::AppHandle) -> EstadoSync {
    match ruta_meta(&app) {
        Ok(ruta) => {
            let meta = leer_meta(&ruta);
            EstadoSync {
                estado: if meta.sha.is_empty() { "sin_datos".into() } else { "sin_cambios".into() },
                sha: Some(meta.sha),
                version: Some(meta.version),
                fecha: Some(meta.fecha),
                detalle: None,
            }
        }
        Err(e) => EstadoSync { estado: "error".into(), sha: None, version: None, fecha: None, detalle: Some(e) },
    }
}

#[tauri::command]
pub async fn sync_now(app: tauri::AppHandle) -> EstadoSync {
    if cfg!(debug_assertions) {
        return EstadoSync { estado: "dev".into(), sha: None, version: None, fecha: None, detalle: Some("sync desactivado en dev".into()) };
    }
    match sincronizar(&app).await {
        Ok(estado) => estado,
        Err(e) => {
            log::warn!("sync falló: {e}");
            EstadoSync { estado: "error".into(), sha: None, version: None, fecha: None, detalle: Some(e) }
        }
    }
}

async fn sincronizar(app: &tauri::AppHandle) -> Result<EstadoSync, String> {
    use tauri::Manager;
    let ruta_meta = ruta_meta(app)?;
    let meta = leer_meta(&ruta_meta);

    let sha = obtener_sha_remoto().await?;
    if sha == meta.sha {
        return Ok(EstadoSync {
            estado: "sin_cambios".into(),
            sha: Some(sha), version: Some(meta.version), fecha: Some(meta.fecha), detalle: None,
        });
    }

    let bytes = descargar_zip_remoto().await?;
    let temporal = app.path().app_data_dir().map_err(|e| e.to_string())?.join("descarga_tmp");
    if temporal.exists() {
        fs::remove_dir_all(&temporal).map_err(|e| e.to_string())?;
    }
    fs::create_dir_all(&temporal).map_err(|e| e.to_string())?;

    let raiz_nueva = extraer_zip(&bytes, &temporal)?;
    let version = validar_contenido(&raiz_nueva)?;

    let dir_actual = crate::contenido::dir_contenido(app)?;
    reemplazar_contenido(&dir_actual, &raiz_nueva)?;
    let _ = fs::remove_dir_all(&temporal);

    let fecha = chrono::Utc::now().to_rfc3339();
    let nueva_meta = MetaContenido { sha: sha.clone(), version, fecha: fecha.clone() };
    guardar_meta(&ruta_meta, &nueva_meta)?;

    log::info!("contenido actualizado a v{version} ({sha})");
    Ok(EstadoSync { estado: "actualizado".into(), sha: Some(sha), version: Some(version), fecha: Some(fecha), detalle: None })
}
```

Registrar en `lib.rs` (`invoke_handler`): `sync::sync_now, sync::estado_sync`.

Nota: `log::` requiere el plugin de log de Task 16; hasta entonces usar `eprintln!` y cambiarlo en Task 16. Añadir mientras tanto:

```rust
// en lugar de log::warn!/log::info! hasta Task 16:
eprintln!("sync: ...");
```

- [x] **Step 4: Verificar**

Run: `cargo test` — Expected: 8 tests pasan.
Run: `cargo check` — Expected: sin errores (comandos registrados).

- [x] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: descarga desde GitHub y comando sync_now (sync parte 2)"
```

### Task 14: Contenido semilla para el primer arranque

**Files:**
- Create: `entorno-app/scripts/actualizar-semilla.mjs`
- Create: `entorno-app/recursos/contenido-semilla/` (generado por el script)
- Modify: `entorno-app/src-tauri/src/contenido.rs` (copiar_dir + asegurar_contenido_inicial)
- Modify: `entorno-app/src-tauri/src/lib.rs` (setup hook)
- Modify: `entorno-app/src-tauri/tauri.conf.json` (bundle resources)
- Modify: `entorno-app/package.json` (script)

**Interfaces:**
- Consumes: `dir_contenido` (Task 8).
- Produces: `copiar_dir(origen: &Path, destino: &Path) -> Result<(), String>`; `asegurar_contenido_inicial(app: &tauri::AppHandle)` llamada en `setup` — si `<appdata>/contenido` no existe, copia la semilla empaquetada.

- [ ] **Step 1: Script de semilla**

`scripts/actualizar-semilla.mjs`:

```js
import { cpSync, rmSync, existsSync } from 'node:fs';
import { join, dirname } from 'node:path';
import { fileURLToPath } from 'node:url';

const raiz = join(dirname(fileURLToPath(import.meta.url)), '..');
const origen = join(raiz, '..', 'entorno-contenido');
const destino = join(raiz, 'recursos', 'contenido-semilla');

if (!existsSync(origen)) {
  console.error(`No existe ${origen}`);
  process.exit(1);
}
rmSync(destino, { recursive: true, force: true });
cpSync(origen, destino, {
  recursive: true,
  filter: (src) => !src.includes('node_modules') && !src.includes(`${origen}\\.git`) && !src.includes(`${origen}/.git`),
});
console.log('Semilla actualizada desde entorno-contenido');
```

En `package.json`: `"semilla": "node scripts/actualizar-semilla.mjs"`.

Run: `npm run semilla` — Expected: `recursos/contenido-semilla/manifest.json` existe.

- [ ] **Step 2: Test Rust de copiar_dir que falla**

En `contenido.rs`, módulo de tests:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::fs;

    #[test]
    fn copia_directorio_recursivo() {
        let origen = tempfile::tempdir().unwrap();
        fs::create_dir_all(origen.path().join("sub")).unwrap();
        fs::write(origen.path().join("a.txt"), "hola").unwrap();
        fs::write(origen.path().join("sub/b.txt"), "adios").unwrap();

        let destino = tempfile::tempdir().unwrap();
        let dest = destino.path().join("copia");
        copiar_dir(origen.path(), &dest).unwrap();
        assert_eq!(fs::read_to_string(dest.join("a.txt")).unwrap(), "hola");
        assert_eq!(fs::read_to_string(dest.join("sub/b.txt")).unwrap(), "adios");
    }
}
```

Run: `cargo test` — Expected: FAIL.

- [ ] **Step 3: Implementar copiar_dir y semilla**

En `contenido.rs`:

```rust
pub fn copiar_dir(origen: &std::path::Path, destino: &std::path::Path) -> Result<(), String> {
    std::fs::create_dir_all(destino).map_err(|e| e.to_string())?;
    for entrada in std::fs::read_dir(origen).map_err(|e| e.to_string())? {
        let entrada = entrada.map_err(|e| e.to_string())?;
        let dest = destino.join(entrada.file_name());
        if entrada.path().is_dir() {
            copiar_dir(&entrada.path(), &dest)?;
        } else {
            std::fs::copy(entrada.path(), &dest).map_err(|e| e.to_string())?;
        }
    }
    Ok(())
}

pub fn asegurar_contenido_inicial(app: &tauri::AppHandle) {
    #[cfg(not(debug_assertions))]
    {
        let Ok(destino) = dir_contenido(app) else { return };
        if destino.join("manifest.json").exists() {
            return;
        }
        let Ok(semilla) = app
            .path()
            .resolve("recursos/contenido-semilla", tauri::path::BaseDirectory::Resource)
        else { return };
        if let Err(e) = copiar_dir(&semilla, &destino) {
            eprintln!("no se pudo copiar la semilla: {e}");
        }
    }
    #[cfg(debug_assertions)]
    let _ = app;
}
```

En `lib.rs`, añadir hook de setup al builder:

```rust
.setup(|app| {
    contenido::asegurar_contenido_inicial(&app.handle());
    Ok(())
})
```

En `tauri.conf.json`, en `bundle`:

```json
"resources": ["../recursos/contenido-semilla/**"]
```

- [ ] **Step 4: Verificar**

Run: `cargo test` — Expected: todos pasan.
Run: `npm run tauri build` (primera build completa) — Expected: instalador NSIS generado en `src-tauri/target/release/bundle/nsis/`. Instalarlo en la máquina del autor: la app abre con el contenido semilla (sin repo dev).

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: contenido semilla empaquetado para primer arranque"
```

### Task 15: Bucle de sync en el frontend + indicador discreto

**Files:**
- Modify: `entorno-app/src/main.js`
- Create: `entorno-app/src/ui/indicador.js`
- Test: `entorno-app/tests/indicador.test.js`

**Interfaces:**
- Consumes: commands `sync_now`/`estado_sync` (Task 13), `cargarManifest` (Task 8).
- Produces: `renderIndicador(estado: EstadoSync): HTMLElement` — pie discreto "Contenido v7 · 26/07/2026". Sync al arrancar y cada 6 h; si `estado === 'actualizado'`, recarga manifest y re-renderiza la ruta actual.

- [ ] **Step 1: Test del indicador que falla**

`tests/indicador.test.js`:

```js
import { describe, it, expect } from 'vitest';
import { renderIndicador } from '../src/ui/indicador.js';

describe('renderIndicador', () => {
  it('muestra versión y fecha', () => {
    const el = renderIndicador({ estado: 'sin_cambios', version: 7, fecha: '2026-07-26T10:00:00Z' });
    expect(el.textContent).toContain('v7');
    expect(el.textContent).toContain('26/07/2026');
  });

  it('sin datos no muestra nada raro', () => {
    const el = renderIndicador({ estado: 'sin_datos' });
    expect(el.textContent).toBe('');
  });
});
```

Run: `npm test` — Expected: FAIL.

- [ ] **Step 2: Implementación**

`src/ui/indicador.js`:

```js
export function renderIndicador(estado) {
  const el = document.createElement('footer');
  el.className = 'indicador-contenido';
  if (estado?.version && estado?.fecha) {
    const fecha = new Date(estado.fecha).toLocaleDateString('es-ES');
    el.textContent = `Contenido v${estado.version} · ${fecha}`;
  }
  return el;
}
```

Añadir a `base.css`:

```css
.indicador-contenido {
  position: fixed;
  bottom: 8px;
  right: 16px;
  font-size: 16px;
  color: #999;
}
```

En `src/main.js`:

```js
import { invoke } from '@tauri-apps/api/core';
import { renderIndicador } from './ui/indicador.js';

const SEIS_HORAS = 6 * 60 * 60 * 1000;

async function sincronizar() {
  try {
    const resultado = await invoke('sync_now');
    if (resultado.estado === 'actualizado') {
      manifest = await cargarManifest();
      window.dispatchEvent(new HashChangeEvent('hashchange'));
    }
    const previo = document.querySelector('.indicador-contenido');
    const estado = resultado.version ? resultado : await invoke('estado_sync');
    previo?.replaceWith(renderIndicador(estado));
  } catch (e) {
    console.error('sync falló:', e);
  }
}
```

Y al final de `arrancar()`:

```js
document.body.append(renderIndicador(await invoke('estado_sync').catch(() => ({}))));
sincronizar();
setInterval(sincronizar, SEIS_HORAS);
```

- [ ] **Step 3: Verificar**

Run: `npm test` — Expected: todos pasan.
Run: `npm run tauri dev` — Expected: indicador ausente o discreto (en dev `sync_now` devuelve `dev`); consola sin errores.

- [ ] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: bucle de sync con refresco silencioso e indicador"
```

### Task 16: Log a archivo + pantalla admin oculta

**Files:**
- Create: `entorno-app/src/ui/admin.js`
- Test: `entorno-app/tests/admin.test.js`
- Modify: `entorno-app/src-tauri/src/lib.rs` (plugin log, comando leer_log_reciente)
- Modify: `entorno-app/src/main.js` (atajo Ctrl+Shift+A)

**Interfaces:**
- Consumes: `estado_sync`, `sync_now` (Task 13).
- Produces: command `leer_log_reciente() -> Vec<String>` (últimas 50 líneas del log); `renderAdmin({ versionApp, estado, lineasLog, alForzarSync, alCerrar }): HTMLElement`.

- [ ] **Step 1: Plugin de log**

```bash
npm run tauri add log
```

En `lib.rs`, añadir al builder:

```rust
.plugin(
    tauri_plugin_log::Builder::new()
        .target(tauri_plugin_log::Target::new(
            tauri_plugin_log::TargetKind::LogDir { file_name: Some("entorno".into()) },
        ))
        .build(),
)
```

Sustituir los `eprintln!` de Tasks 13–14 por `log::warn!` / `log::info!` (añadir `log = "0.4"` a Cargo.toml).

Comando para leer el log (en `lib.rs` o módulo nuevo):

```rust
#[tauri::command]
fn leer_log_reciente(app: tauri::AppHandle) -> Vec<String> {
    use tauri::Manager;
    let Ok(dir) = app.path().app_log_dir() else { return vec![] };
    let ruta = dir.join("entorno.log");
    let Ok(texto) = std::fs::read_to_string(&ruta) else { return vec![] };
    texto.lines().rev().take(50).map(String::from).collect::<Vec<_>>().into_iter().rev().collect()
}
```

Registrarlo en `invoke_handler`.

- [ ] **Step 2: Tests del panel admin que fallan**

`tests/admin.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { renderAdmin } from '../src/ui/admin.js';

const datos = {
  versionApp: '0.1.0',
  estado: { estado: 'sin_cambios', version: 7, sha: 'abc123', fecha: '2026-07-26T10:00:00Z' },
  lineasLog: ['linea 1', 'linea 2'],
  alForzarSync: vi.fn(),
  alCerrar: vi.fn(),
};

describe('renderAdmin', () => {
  it('muestra versiones, sha y log', () => {
    const el = renderAdmin(datos);
    expect(el.textContent).toContain('0.1.0');
    expect(el.textContent).toContain('v7');
    expect(el.textContent).toContain('abc123');
    expect(el.textContent).toContain('linea 2');
  });

  it('botón de forzar sync llama al callback', () => {
    const el = renderAdmin(datos);
    el.querySelector('.boton-forzar-sync').click();
    expect(datos.alForzarSync).toHaveBeenCalled();
  });

  it('botón cerrar llama al callback', () => {
    const el = renderAdmin(datos);
    el.querySelector('.boton-cerrar-admin').click();
    expect(datos.alCerrar).toHaveBeenCalled();
  });
});
```

Run: `npm test` — Expected: FAIL.

- [ ] **Step 3: Implementación del panel**

`src/ui/admin.js`:

```js
export function renderAdmin({ versionApp, estado, lineasLog, alForzarSync, alCerrar }) {
  const el = document.createElement('div');
  el.className = 'panel-admin';

  const fecha = estado?.fecha ? new Date(estado.fecha).toLocaleString('es-ES') : '—';
  const info = document.createElement('pre');
  info.textContent = [
    `App:        v${versionApp}`,
    `Contenido:  v${estado?.version ?? '—'} (${estado?.sha?.slice(0, 7) ?? '—'})`,
    `Último sync: ${fecha}`,
    `Estado:     ${estado?.estado ?? '—'}`,
  ].join('\n');

  const log = document.createElement('pre');
  log.className = 'log-admin';
  log.textContent = (lineasLog ?? []).join('\n');

  const forzar = document.createElement('button');
  forzar.className = 'boton-forzar-sync';
  forzar.textContent = 'Forzar sincronización';
  forzar.addEventListener('click', alForzarSync);

  const cerrar = document.createElement('button');
  cerrar.className = 'boton-cerrar-admin';
  cerrar.textContent = 'Cerrar';
  cerrar.addEventListener('click', alCerrar);

  el.append(info, log, forzar, cerrar);
  return el;
}
```

Estilos en `base.css`:

```css
.panel-admin {
  position: fixed;
  inset: 5%;
  z-index: 100;
  background: #1E1E1E;
  color: #DDD;
  padding: 24px;
  border-radius: var(--radio);
  font-size: 16px;
  overflow: auto;
}
.panel-admin pre { font-size: 14px; white-space: pre-wrap; }
.log-admin { max-height: 40vh; overflow: auto; border: 1px solid #444; padding: 8px; margin: 16px 0; }
.panel-admin button { min-height: 48px; margin-right: 16px; padding: 0 24px; font-size: 16px; }
```

- [ ] **Step 4: Atajo de teclado en main.js**

```js
import { getVersion } from '@tauri-apps/api/app';
import { renderAdmin } from './ui/admin.js';

window.addEventListener('keydown', async (ev) => {
  if (ev.ctrlKey && ev.shiftKey && ev.code === 'KeyA') {
    const existente = document.querySelector('.panel-admin');
    if (existente) { existente.remove(); return; }
    const [versionApp, estado, lineasLog] = await Promise.all([
      getVersion(),
      invoke('estado_sync').catch(() => null),
      invoke('leer_log_reciente').catch(() => []),
    ]);
    const panel = renderAdmin({
      versionApp, estado, lineasLog,
      alForzarSync: async () => { await invoke('sync_now'); panel.remove(); },
      alCerrar: () => panel.remove(),
    });
    document.body.append(panel);
  }
});
```

- [ ] **Step 5: Verificar**

Run: `npm test` — Expected: todos pasan.
Run: `npm run tauri dev` → Ctrl+Shift+A — Expected: panel con versión de app, estado `dev`, log (posiblemente vacío). Ctrl+Shift+A otra vez lo cierra.

- [ ] **Step 6: Commit**

```bash
git add -A && git commit -m "feat: log a archivo y pantalla admin oculta"
```

---

# BLOQUE 4 — Distribución e instalación

### Task 17: Icono e instalador NSIS en español

**Files:**
- Create: `entorno-app/recursos/icono.svg`
- Create: `entorno-app/scripts/generar-icono.mjs`
- Modify: `entorno-app/src-tauri/tauri.conf.json` (bundle)
- Modify: `entorno-app/package.json` (scripts, devDependency sharp)

**Interfaces:**
- Produces: iconos en `src-tauri/icons/` generados con `npx tauri icon`; instalador NSIS en español con acceso directo de escritorio.

- [ ] **Step 1: Icono fuente**

`recursos/icono.svg` (casa blanca sobre fondo verde, simple y reconocible):

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="1024" height="1024">
  <rect width="1024" height="1024" rx="180" fill="#2E7D32"/>
  <path d="M512 200 L860 500 L780 500 L780 820 L580 820 L580 620 L444 620 L444 820 L244 820 L244 500 L164 500 Z" fill="#FFFFFF"/>
</svg>
```

`scripts/generar-icono.mjs`:

```js
import sharp from 'sharp';
import { join, dirname } from 'node:path';
import { fileURLToPath } from 'node:url';

const raiz = join(dirname(fileURLToPath(import.meta.url)), '..');
await sharp(join(raiz, 'recursos/icono.svg'))
  .resize(1024, 1024)
  .png()
  .toFile(join(raiz, 'recursos/icono.png'));
console.log('icono.png generado');
```

```bash
npm install -D sharp
node scripts/generar-icono.mjs
npx tauri icon recursos/icono.png
```

Expected: `src-tauri/icons/` poblado (icon.ico, varios png).

- [ ] **Step 2: Configurar bundle NSIS**

En `tauri.conf.json`, en `bundle`:

```json
{
  "active": true,
  "targets": ["nsis"],
  "icon": ["icons/32x32.png", "icons/128x128.png", "icons/128x128@2x.png", "icons/icon.ico"],
  "resources": ["../recursos/contenido-semilla/**"],
  "windows": {
    "nsis": {
      "languages": ["Spanish"],
      "displayLanguageSelector": false,
      "installMode": "currentUser"
    }
  }
}
```

- [ ] **Step 3: Verificar instalador**

```bash
npm run semilla
npm run tauri build
```

Expected: `src-tauri/target/release/bundle/nsis/Entorno de Papa_0.1.0_x64-setup.exe`. Instalarlo: instalador en español, crea acceso directo en escritorio y menú inicio, la app abre maximizada con icono de casa verde.

- [ ] **Step 4: Commit**

```bash
git add -A && git commit -m "feat: icono e instalador NSIS en español"
```

### Task 18: Auto-actualización de la app

**Files:**
- Modify: `entorno-app/src-tauri/tauri.conf.json` (plugin updater)
- Modify: `entorno-app/src-tauri/capabilities/default.json`
- Modify: `entorno-app/src/main.js` (comprobación al arrancar)
- Create: `entorno-app/.github/workflows/release.yml`

**Interfaces:**
- Consumes: GitHub Releases del repo `entorno-app`.
- Produces: app que se auto-actualiza al arrancar (silencioso, relanza tras instalar); workflow que publica release + `latest.json` al pushear un tag `v*`.

- [ ] **Step 1: Plugins y claves de firma**

```bash
npm run tauri add updater
npm run tauri add process
npx tauri signer generate -w "$HOME/.tauri/entorno-papa.key"
```

Guardar la clave privada y su contraseña fuera del repo (el archivo `.key` NO se commitea nunca). Copiar la clave pública que imprime el comando.

- [ ] **Step 2: Configuración del updater**

En `tauri.conf.json`:

```json
"plugins": {
  "updater": {
    "pubkey": "<CLAVE_PUBLICA_GENERADA_EN_STEP_1>",
    "endpoints": [
      "https://github.com/marquib3l/entorno-app/releases/latest/download/latest.json"
    ]
  }
},
"bundle": {
  "createUpdaterArtifacts": true
}
```

(`<CLAVE_PUBLICA_GENERADA_EN_STEP_1>` se sustituye por el valor real impreso en Step 1 — es el único dato que no puede escribirse de antemano en este plan.)

En `capabilities/default.json` añadir permisos: `"updater:default"`, `"process:default"`.

- [ ] **Step 3: Comprobación al arrancar (frontend)**

En `src/main.js`:

```js
import { check } from '@tauri-apps/plugin-updater';
import { relaunch } from '@tauri-apps/plugin-process';

async function actualizarApp() {
  try {
    const actualizacion = await check();
    if (actualizacion) {
      await actualizacion.downloadAndInstall();
      await relaunch();
    }
  } catch (e) {
    console.error('comprobación de actualización falló:', e);
  }
}
```

Llamar `actualizarApp()` al final de `arrancar()` (tras pintar la UI: si hay update, la app se relanza sola en segundos; si no, no se nota nada).

- [ ] **Step 4: Crear repos en GitHub y secretos**

```bash
cd /b/01_Proyectos/Entorno-para-Papa/entorno-contenido
gh repo create marquib3l/entorno-contenido --public --source . --push

cd ../entorno-app
gh repo create marquib3l/entorno-app --public --source . --push
gh secret set TAURI_SIGNING_PRIVATE_KEY < "$HOME/.tauri/entorno-papa.key"
gh secret set TAURI_SIGNING_PRIVATE_KEY_PASSWORD --body "<contraseña elegida en Step 1>"
```

- [ ] **Step 5: Workflow de release**

`.github/workflows/release.yml`:

```yaml
name: release
on:
  push:
    tags: ['v*']
jobs:
  build:
    runs-on: windows-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - uses: dtolnay/rust-toolchain@stable
      - run: npm ci
      - run: npm run semilla || true
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
          TAURI_SIGNING_PRIVATE_KEY_PASSWORD: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY_PASSWORD }}
        with:
          tagName: ${{ github.ref_name }}
          releaseName: 'Entorno de Papá ${{ github.ref_name }}'
          releaseBody: 'Actualización automática.'
```

Nota: `npm run semilla` falla en CI porque `entorno-contenido` no está como carpeta hermana; por eso el `|| true`. La semilla se commitea en el repo de la app (quitar `recursos/contenido-semilla` del `.gitignore` si estuviera) para que CI la empaquete tal cual. Ejecutar `npm run semilla` + commit antes de cada tag.

- [ ] **Step 6: Verificar**

```bash
git add -A && git commit -m "feat: auto-actualización con updater firmado"
git tag v0.1.0 && git push origin master --tags
```

Expected: Action `release` en verde; release `v0.1.0` en GitHub con `*-setup.exe`, `*-setup.nsis.zip.sig` y `latest.json`.

### Task 19: Instalación en los PCs del padre + prueba del ciclo completo

**Files:** ninguno nuevo (verificación de extremo a extremo).

**Interfaces:**
- Consumes: release v0.1.0 (Task 18), sync (Bloque 3).

- [ ] **Step 1: Prueba del ciclo de actualización de app**

En la máquina del autor, con v0.1.0 instalada: subir un cambio trivial (p. ej. versión 0.1.1 en `tauri.conf.json` y `Cargo.toml`), `npm run semilla`, commit, `git tag v0.1.1 && git push --tags`. Abrir la app instalada.
Expected: la app se relanza sola en <1 min ya en v0.1.1 (verificar con Ctrl+Shift+A).

- [ ] **Step 2: Prueba del ciclo de contenido**

Editar `entorno-contenido/manifest.json` (añadir un periódico), subir `version` a 2, `npm run check`, commit y push. En la app instalada: forzar sync desde el panel admin (o esperar al arranque).
Expected: la tarjeta nueva aparece sin reinstalar nada.

- [ ] **Step 3: Instalar en los dos PCs del padre**

Checklist por equipo (portátil y torre):
- Descargar el `*-setup.exe` de la última release (o llevarlo en USB) e instalar.
- Comprobar acceso directo en escritorio con icono verde; renombrarlo si hace falta a "Entorno".
- Abrir: contenido actual visible (semilla + primer sync).
- Ctrl+Shift+A: versión app y contenido correctas, sync OK.
- Clic en un periódico: abre el navegador por defecto (comprobar que el navegador por defecto es el que usa el padre).
- Desconectar WiFi y reabrir la app: todo funciona salvo enlaces (guías incluidas).

- [ ] **Step 4: Commit de cualquier ajuste + cierre del bloque**

```bash
git add -A && git commit -m "chore: ajustes post-instalación en equipos de destino"
```

---

# BLOQUE 5 — Contenido real v1

### Task 20: Guías de ofimática

**Files:**
- Create: `entorno-contenido/guias/correo-enviar.md`
- Create: `entorno-contenido/guias/correo-leer.md`
- Create: `entorno-contenido/guias/drive-basico.md`
- Create: `entorno-contenido/guias/carpetas-archivos.md`
- Create: `entorno-contenido/guias/fotos-movil-pc.md`
- Create: `entorno-contenido/img/` (capturas reales)
- Modify: `entorno-contenido/manifest.json`
- Delete: `entorno-contenido/guias/ejemplo-correo.md`

**Interfaces:**
- Consumes: convención de guías (Task 3), visor (Bloque 2).

- [ ] **Step 1: Redactar los borradores de las 5 guías**

El implementador escribe el contenido completo de cada guía (borrador real, no esqueleto), siguiendo estas reglas de redacción:
- Un paso = una sola acción concreta ("Pulsa el botón azul que dice Redactar").
- Lenguaje cotidiano: "la barra de abajo", "la rueda dentada", nunca "taskbar" ni "configuración avanzada".
- Máximo 8 pasos por guía; si hace falta más, dividir en dos guías.
- Cada paso que involucre la pantalla lleva captura (`![captura](img/<guia>-<nn>.png)`).

Guías y alcance:
1. `correo-enviar.md` — abrir Gmail desde el navegador, redactar, destinatario, asunto, enviar.
2. `correo-leer.md` — abrir la bandeja, distinguir no leídos, abrir un correo, volver a la bandeja, responder.
3. `drive-basico.md` — qué es Drive, abrirlo, ver los archivos, abrir un documento, volver.
4. `carpetas-archivos.md` — abrir el Explorador, crear una carpeta en Documentos, mover un archivo arrastrando NO: con cortar/pegar del menú contextual (regla de un clic: usar clic derecho + opciones, no arrastrar).
5. `fotos-movil-pc.md` — conectar el móvil por cable, permitir acceso en el móvil, abrir el Explorador, copiar fotos de DCIM a la carpeta Imágenes.

- [ ] **Step 2: Capturas**

El autor (usuario) hace las capturas reales en el PC del padre o en uno equivalente (mismo Windows 11 y navegador). Guardarlas como `img/correo-enviar-01.png`, etc. Hasta tenerlas, las guías se commitean sin imágenes (el visor las tolera).

- [ ] **Step 3: Actualizar manifest y validar**

Sección `aprender` con las 5 tarjetas `guia`. Subir `version`. Borrar `ejemplo-correo.md` y su tarjeta.

Run: `npm run check` — Expected: `Contenido OK`.

- [ ] **Step 4: Revisión del usuario y push**

El usuario revisa y ajusta el texto de las guías (conoce a su padre). Después:

```bash
git add -A && git commit -m "feat: guías de ofimática v1" && git push
```

Expected: CI verde; en la app instalada, tras sync, aparecen las 5 guías.

### Task 21: Kiosco de prensa completo con iconos

**Files:**
- Create: `entorno-contenido/scripts/descargar-iconos.mjs`
- Create: `entorno-contenido/iconos/*.png`
- Modify: `entorno-contenido/manifest.json`

**Interfaces:**
- Consumes: manifest (Task 3), render de tarjetas (Task 6 — las tarjetas con `icono` muestran la imagen si la app la soporta; si aún no, el icono queda para una mejora futura del render y no rompe nada).

- [ ] **Step 1: Confirmar la lista de medios con el usuario**

Lista por defecto (el usuario la ajusta a los gustos reales de su padre — en particular la prensa LOCAL de su provincia, que solo él conoce):
- Local: (a rellenar por el usuario — 2 o 3 cabeceras de su zona)
- Nacional: El País, El Mundo, ABC
- Internacional: BBC News Mundo, Euronews
- Deportes: Marca, AS
- Bolsa: Expansión, Cinco Días

- [ ] **Step 2: Script de iconos**

`scripts/descargar-iconos.mjs`:

```js
import { writeFileSync, readFileSync } from 'node:fs';
import { join, dirname } from 'node:path';
import { fileURLToPath } from 'node:url';

const raiz = join(dirname(fileURLToPath(import.meta.url)), '..');
const manifest = JSON.parse(readFileSync(join(raiz, 'manifest.json'), 'utf8'));
const tarjetas = manifest.secciones.flatMap(s =>
  (s.tarjetas ?? []).concat((s.grupos ?? []).flatMap(g => g.tarjetas)));

for (const t of tarjetas.filter(t => t.tipo === 'enlace' && t.icono)) {
  const dominio = new URL(t.url).hostname;
  const resp = await fetch(`https://www.google.com/s2/favicons?domain=${dominio}&sz=128`);
  if (!resp.ok) { console.warn(`sin favicon: ${dominio}`); continue; }
  writeFileSync(join(raiz, 'iconos', t.icono), Buffer.from(await resp.arrayBuffer()));
  console.log(`descargado: ${t.icono}`);
}
```

- [ ] **Step 3: Manifest de prensa completo**

Actualizar la sección `prensa` con los 5 grupos y sus tarjetas, cada una con `icono: "<slug>.png"`. Ejecutar:

```bash
node scripts/descargar-iconos.mjs
npm run check
```

Expected: iconos descargados, `Contenido OK`.

- [ ] **Step 4: Mostrar iconos en las tarjetas (app)**

En `entorno-app/src/ui/seccion.js`, dentro de `pintarTarjetas`, si la tarjeta tiene `icono`:

```js
if (tarjeta.icono) {
  const img = document.createElement('img');
  img.className = 'icono-tarjeta';
  img.alt = '';
  resolverImagen(`iconos/${tarjeta.icono}`).then((url) => { img.src = url; });
  img.addEventListener('error', () => img.remove(), { once: true });
  boton.prepend(img);
}
```

`renderSeccion` gana el parámetro `resolverImagen` (mismo contrato que Task 10); `main.js` le pasa `urlRecurso`. Test nuevo en `tests/seccion.test.js`:

```js
it('tarjeta con icono pinta la imagen', async () => {
  const conIcono = { id: 'p', titulo: 'P', tarjetas: [{ tipo: 'enlace', titulo: 'X', url: 'https://x.com', icono: 'x.png' }] };
  const resolverImagen = vi.fn(async () => 'asset://x.png');
  const el = renderSeccion(conIcono, { alPulsarTarjeta: () => {}, navegarA: () => {}, resolverImagen });
  await Promise.resolve();
  expect(resolverImagen).toHaveBeenCalledWith('iconos/x.png');
  expect(el.querySelector('img.icono-tarjeta')).toBeTruthy();
});
```

CSS en `base.css`: `.icono-tarjeta { width: 48px; height: 48px; margin-right: 16px; vertical-align: middle; }`

Run: `npm test` — Expected: todos pasan (los tests existentes de `renderSeccion` siguen pasando: `resolverImagen` es opcional cuando ninguna tarjeta tiene icono).

- [ ] **Step 5: Commit y push de ambos repos**

```bash
# entorno-contenido
git add -A && git commit -m "feat: kiosco de prensa completo con iconos" && git push
# entorno-app
git add -A && git commit -m "feat: iconos en tarjetas de enlace" && git push
```

Para la app: publicar release nueva (subir versión, tag) para que llegue a los PCs.

### Task 22: Juegos, revisión final y v1

**Files:**
- Modify: `entorno-contenido/manifest.json`

**Interfaces:**
- Consumes: todo lo anterior.

- [ ] **Step 1: Sección Jugar definitiva**

Tarjetas con URLs verificadas ese día (abrirlas y comprobar que cargan sin registro):
- Solitario: `https://www.solitr.com/es`
- Tetris: `https://tetris.com/play-tetris`
Con iconos vía `descargar-iconos.mjs`. Subir `version`.

- [ ] **Step 2: Revisión completa del contenido**

Run: `npm run check` — Expected: `Contenido OK`.
Revisión manual con checklist:
- Cada URL del manifest abre y es la portada correcta.
- Cada guía se lee entera en el visor sin pasos vacíos.
- Los títulos de tarjeta son cortos (caben en la tarjeta sin cortarse).

- [ ] **Step 3: Push final y verificación en los PCs del padre**

```bash
git add -A && git commit -m "feat: contenido v1 completo" && git push
```

En cada PC del padre: abrir la app (o forzar sync) y verificar que aparece todo. Ver al padre usarlo 10 minutos sin ayuda: esa es la prueba de aceptación real de v1.

---

## Notas para el ejecutor

- Las versiones de dependencias (`@tauri-apps/cli@^2`, `reqwest 0.12`, `zip 2`, etc.) son las vigentes a fecha del plan; si `npm`/`cargo` resuelven versiones mayores nuevas con breaking changes, consultar la documentación oficial antes de fijar una versión anterior.
- Los comandos bash asumen Git Bash en Windows (rutas `/b/...`). En PowerShell, adaptar las rutas.
- `npm run tauri dev` la primera vez compila Rust (~2-5 min); las siguientes son incrementales.
- Si el username de GitHub no es `marquib3l`, cambiarlo en: `config.rs`, `tauri.conf.json` (endpoint updater), Task 18 Step 4 y este documento.
