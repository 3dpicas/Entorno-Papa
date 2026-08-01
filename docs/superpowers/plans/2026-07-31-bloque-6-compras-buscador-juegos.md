# Bloque 6: compras, buscador y juegos — Plan de Implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Añadir acceso directo a Brave Search, sección de compras seguras y dos variantes de solitario, manteniendo accesibilidad, actualización remota y seguridad.

**Architecture:** Frontend añade botón especial e integra una función aislada para búsqueda. Backend Rust localiza `brave.exe` y abre una URL fija sin aceptar argumentos externos; frontend usa `openUrl` como respaldo. Compras, guía y juegos siguen siendo contenido dirigido por `manifest.json`, sin cambiar su schema.

**Tech Stack:** Tauri v2, Rust 2021, Vite 8, JavaScript ES modules, Vitest 4 + jsdom, Markdown, JSON Schema/Ajv, `tauri-plugin-opener`.

## Global Constraints

- Windows 11; resolución mínima de ventana 1024 × 700.
- Texto base mínimo 24 px; controles grandes; un clic; sin scroll horizontal.
- Texto exacto del botón: `BUSCAR EN INTERNET`.
- URL fija: `https://search.brave.com/`.
- Sección Compras: `id = compras`, título `Compras`, color `#7B1FA2`.
- Tiendas: Amazon, AliExpress, Temu, El Corte Inglés y Carrefour; solo portadas HTTPS oficiales.
- Juegos finales: Solitario clásico, Solitario Spider fácil, Carta blanca y Tetris.
- App no guarda credenciales, tarjetas, direcciones ni historial comercial.
- Fallos quedan en `entorno.log`; ninguna excepción bloquea Inicio.
- Contrato de `manifest.json` sin cambios.
- Bloque 7, valores bursátiles, WhatsApp y lista de tareas quedan fuera de este plan.
- Repos independientes: raíz documental, `entorno-app` y `entorno-contenido`.
- Antes de editar: `npm test` en `entorno-app` y `npm run check` en `entorno-contenido` deben estar verdes.

---

## Mapa de archivos

| Archivo | Responsabilidad |
|---|---|
| `entorno-app/src/ui/inicio.js` | Pintar botón y secciones; emitir callback de búsqueda |
| `entorno-app/src/styles/base.css` | Posición, tamaño, color y rejilla adaptable de Inicio |
| `entorno-app/src/lib/busqueda.js` | Orquestar comando Tauri, logs y respaldo con `openUrl` |
| `entorno-app/src-tauri/src/busqueda.rs` | Localizar y lanzar Brave con URL constante |
| `entorno-app/src-tauri/src/lib.rs` | Registrar comando Tauri |
| `entorno-app/src/main.js` | Conectar UI, `invoke`, `openUrl` y registro |
| `entorno-app/tests/inicio.test.js` | Contrato visual/funcional del botón |
| `entorno-app/tests/busqueda.test.js` | Estados abierto, respaldo y errores |
| `entorno-contenido/manifest.json` | Sección Compras y tarjetas nuevas de Jugar |
| `entorno-contenido/guias/comprar-seguro.md` | Guía de ocho pasos |
| `entorno-contenido/iconos/*.png` | Favicons reconocibles de tiendas |
| `entorno-app/recursos/contenido-semilla/` | Contenido v8 para primera instalación/offline |
| `docs/REGISTRO.md` | Resultado real, desviaciones y aceptación |

---

### Task 1: Botón accesible en Inicio

**Files:**
- Modify: `entorno-app/tests/inicio.test.js`
- Modify: `entorno-app/src/ui/inicio.js`
- Modify: `entorno-app/src/styles/base.css`

**Interfaces:**
- Consumes: `manifest.secciones`, `navegarA(hash: string)`, `alBuscarInternet(): void | Promise<void>`
- Produces: `renderInicio(manifest, { navegarA, alBuscarInternet }): HTMLElement`, selector `.boton-buscar-internet`

- [x] **Step 1: Confirmar base verde**

Run:

```powershell
cd B:\01_Proyectos\Entorno-para-Papa\entorno-app
npm test
```

Expected: 60 tests verdes antes del cambio.

- [x] **Step 2: Escribir test rojo del botón**

Reemplazar `tests/inicio.test.js` por:

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

function opciones(cambios = {}) {
  return {
    navegarA: vi.fn(),
    alBuscarInternet: vi.fn(),
    ...cambios,
  };
}

describe('renderInicio', () => {
  it('pinta una tarjeta por sección con su título', () => {
    const el = renderInicio(manifest, opciones());
    const botones = el.querySelectorAll('button.tarjeta-seccion');
    expect(botones).toHaveLength(2);
    expect(botones[0].textContent).toContain('Aprender');
  });

  it('clic en sección navega a su ruta', () => {
    const navegarA = vi.fn();
    const el = renderInicio(manifest, opciones({ navegarA }));
    el.querySelectorAll('button.tarjeta-seccion')[1].click();
    expect(navegarA).toHaveBeenCalledWith('#/seccion/prensa');
  });

  it('incluye saludo y reloj', () => {
    const el = renderInicio(manifest, opciones());
    expect(el.querySelector('.saludo').textContent).toMatch(/Buen[oa]s/);
    expect(el.querySelector('.reloj')).toBeTruthy();
  });

  it('buscar aparece primero y ejecuta su callback con un clic', () => {
    const alBuscarInternet = vi.fn();
    const el = renderInicio(manifest, opciones({ alBuscarInternet }));
    const boton = el.querySelector('button.boton-buscar-internet');

    expect(boton).toBeTruthy();
    expect(boton.textContent).toContain('BUSCAR EN INTERNET');
    expect(el.firstElementChild).toBe(boton);

    boton.click();
    expect(alBuscarInternet).toHaveBeenCalledOnce();
  });
});
```

- [x] **Step 3: Ejecutar test y comprobar fallo correcto**

Run:

```powershell
npm test -- tests/inicio.test.js
```

Expected: FAIL en `buscar aparece primero...` porque `.boton-buscar-internet` no existe.

- [x] **Step 4: Implementar UI mínima**

Reemplazar `src/ui/inicio.js` por:

```js
import { saludo } from '../lib/saludo.js';

export function renderInicio(manifest, { navegarA, alBuscarInternet }) {
  const el = document.createElement('main');
  el.className = 'pantalla-inicio';

  const botonBuscar = document.createElement('button');
  botonBuscar.type = 'button';
  botonBuscar.className = 'boton-buscar-internet';

  const iconoBuscar = document.createElement('span');
  iconoBuscar.setAttribute('aria-hidden', 'true');
  iconoBuscar.textContent = '🔎';

  const textoBuscar = document.createElement('span');
  textoBuscar.textContent = 'BUSCAR EN INTERNET';

  botonBuscar.append(iconoBuscar, textoBuscar);
  botonBuscar.addEventListener('click', alBuscarInternet);

  const cabecera = document.createElement('header');
  const elSaludo = document.createElement('h1');
  elSaludo.className = 'saludo';
  elSaludo.textContent = saludo(new Date().getHours());
  const reloj = document.createElement('p');
  reloj.className = 'reloj';
  const pintarHora = () => {
    reloj.textContent = new Date().toLocaleTimeString('es-ES', {
      hour: '2-digit',
      minute: '2-digit',
    });
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

  el.append(botonBuscar, cabecera, parrilla);
  return el;
}
```

- [x] **Step 5: Aplicar estilos aprobados**

Añadir antes de `.pantalla-inicio header` en `src/styles/base.css`:

```css
.boton-buscar-internet {
  --color-seccion: #44219E;
  display: inline-flex;
  align-items: center;
  gap: 12px;
  min-height: 88px;
  margin-bottom: 24px;
  padding: 0 32px;
  font-size: 28px;
  font-weight: 800;
  color: #fff;
  background: #6D3EE8;
}
```

Reemplazar el bloque `.parrilla-secciones` por:

```css
.parrilla-secciones {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 32px;
}

@media (min-width: 1600px) {
  .parrilla-secciones {
    grid-template-columns: repeat(4, minmax(280px, 1fr));
  }
}
```

- [x] **Step 6: Verificar UI aislada y build**

Run:

```powershell
npm test -- tests/inicio.test.js
npm run build
```

Expected: 4 tests de Inicio verdes; Vite termina con `built in`.

- [x] **Step 7: Commit**

```powershell
git add tests/inicio.test.js src/ui/inicio.js src/styles/base.css
git commit -m "feat: añadir botón de búsqueda en Inicio"
```

---

### Task 2: Comando Rust seguro para Brave

**Files:**
- Create: `entorno-app/src-tauri/src/busqueda.rs`
- Modify: `entorno-app/src-tauri/src/lib.rs`
- Test: tests internos en `busqueda.rs`

**Interfaces:**
- Consumes: variables `ProgramFiles`, `ProgramFiles(x86)`, `LOCALAPPDATA`
- Produces: comando Tauri `abrir_busqueda_brave() -> String`, estados `abierto` y `no_disponible`

- [x] **Step 1: Añadir módulo con tests todavía sin implementación**

Crear `src-tauri/src/busqueda.rs`:

```rust
#[cfg(test)]
mod tests {
    use super::{localizar_brave, rutas_brave_desde, URL_BUSQUEDA};
    use std::fs;
    use std::path::{Path, PathBuf};
    use tempfile::tempdir;

    #[test]
    fn construye_rutas_en_orden_de_preferencia() {
        let rutas = rutas_brave_desde(
            Some(Path::new(r"C:\Program Files")),
            Some(Path::new(r"C:\Program Files (x86)")),
            Some(Path::new(r"C:\Users\Papa\AppData\Local")),
        );

        assert_eq!(
            rutas,
            vec![
                PathBuf::from(
                    r"C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
                PathBuf::from(
                    r"C:\Program Files (x86)\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
                PathBuf::from(
                    r"C:\Users\Papa\AppData\Local\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
            ]
        );
    }

    #[test]
    fn elige_primera_ruta_que_existe() {
        let temp = tempdir().unwrap();
        let ausente = temp.path().join("uno").join("brave.exe");
        let presente = temp.path().join("dos").join("brave.exe");
        fs::create_dir_all(presente.parent().unwrap()).unwrap();
        fs::write(&presente, b"prueba").unwrap();

        assert_eq!(
            localizar_brave(&[ausente, presente.clone()]),
            Some(presente)
        );
    }

    #[test]
    fn url_de_busqueda_es_fija() {
        assert_eq!(URL_BUSQUEDA, "https://search.brave.com/");
    }
}
```

Añadir junto a los módulos de `src-tauri/src/lib.rs`:

```rust
mod busqueda;
```

- [x] **Step 2: Ejecutar test y comprobar rojo**

Desde `entorno-app/src-tauri`:

```powershell
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
cargo test busqueda
```

Expected: FAIL de compilación por imports sin definir en `busqueda.rs`.

- [x] **Step 3: Implementar localización y lanzamiento**

Reemplazar `src-tauri/src/busqueda.rs` por:

```rust
use std::env;
use std::path::{Path, PathBuf};
use std::process::Command;

pub const URL_BUSQUEDA: &str = "https://search.brave.com/";
const RUTA_RELATIVA: &str = r"BraveSoftware\Brave-Browser\Application\brave.exe";

fn rutas_brave_desde(
    program_files: Option<&Path>,
    program_files_x86: Option<&Path>,
    local_app_data: Option<&Path>,
) -> Vec<PathBuf> {
    [program_files, program_files_x86, local_app_data]
        .into_iter()
        .flatten()
        .map(|base| base.join(RUTA_RELATIVA))
        .collect()
}

fn rutas_brave() -> Vec<PathBuf> {
    let program_files = env::var_os("ProgramFiles").map(PathBuf::from);
    let program_files_x86 = env::var_os("ProgramFiles(x86)").map(PathBuf::from);
    let local_app_data = env::var_os("LOCALAPPDATA").map(PathBuf::from);

    rutas_brave_desde(
        program_files.as_deref(),
        program_files_x86.as_deref(),
        local_app_data.as_deref(),
    )
}

fn localizar_brave(rutas: &[PathBuf]) -> Option<PathBuf> {
    rutas.iter().find(|ruta| ruta.is_file()).cloned()
}

#[tauri::command]
pub fn abrir_busqueda_brave() -> String {
    let Some(ruta) = localizar_brave(&rutas_brave()) else {
        log::warn!("[busqueda] Brave no encontrado");
        return "no_disponible".into();
    };

    match Command::new(&ruta).arg(URL_BUSQUEDA).spawn() {
        Ok(_) => {
            log::info!("[busqueda] Brave abierto");
            "abierto".into()
        }
        Err(error) => {
            log::warn!("[busqueda] Brave no pudo abrirse: {error}");
            "no_disponible".into()
        }
    }
}

#[cfg(test)]
mod tests {
    use super::{localizar_brave, rutas_brave_desde, URL_BUSQUEDA};
    use std::fs;
    use std::path::{Path, PathBuf};
    use tempfile::tempdir;

    #[test]
    fn construye_rutas_en_orden_de_preferencia() {
        let rutas = rutas_brave_desde(
            Some(Path::new(r"C:\Program Files")),
            Some(Path::new(r"C:\Program Files (x86)")),
            Some(Path::new(r"C:\Users\Papa\AppData\Local")),
        );

        assert_eq!(
            rutas,
            vec![
                PathBuf::from(
                    r"C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
                PathBuf::from(
                    r"C:\Program Files (x86)\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
                PathBuf::from(
                    r"C:\Users\Papa\AppData\Local\BraveSoftware\Brave-Browser\Application\brave.exe"
                ),
            ]
        );
    }

    #[test]
    fn elige_primera_ruta_que_existe() {
        let temp = tempdir().unwrap();
        let ausente = temp.path().join("uno").join("brave.exe");
        let presente = temp.path().join("dos").join("brave.exe");
        fs::create_dir_all(presente.parent().unwrap()).unwrap();
        fs::write(&presente, b"prueba").unwrap();

        assert_eq!(
            localizar_brave(&[ausente, presente.clone()]),
            Some(presente)
        );
    }

    #[test]
    fn url_de_busqueda_es_fija() {
        assert_eq!(URL_BUSQUEDA, "https://search.brave.com/");
    }
}
```

- [x] **Step 4: Registrar comando**

Dentro de `tauri::generate_handler!` en `src-tauri/src/lib.rs`, añadir como primera entrada:

```rust
busqueda::abrir_busqueda_brave,
```

El bloque empezará así:

```rust
.invoke_handler(tauri::generate_handler![
    busqueda::abrir_busqueda_brave,
    contenido::contenido_leer,
    contenido::contenido_ruta,
    sync::sync_now,
    sync::estado_sync,
    leer_log_reciente
])
```

- [x] **Step 5: Formatear y verificar**

```powershell
cargo fmt
cargo fmt --check
cargo test busqueda
cargo test
```

Expected: 3 tests de búsqueda y suite Rust completa verdes.

- [x] **Step 6: Commit**

```powershell
git add src-tauri/src/busqueda.rs src-tauri/src/lib.rs
git commit -m "feat: abrir Brave Search con comando seguro"
```

---

### Task 3: Integración con respaldo y logs

**Files:**
- Create: `entorno-app/src/lib/busqueda.js`
- Create: `entorno-app/tests/busqueda.test.js`
- Modify: `entorno-app/src/main.js`

**Interfaces:**
- Consumes: `invocar(nombre: string): Promise<string>`, `abrirUrl(url: string): Promise<void>`, `registro.info`, `registro.error`
- Produces: `abrirBusquedaInternet({ invocar, abrirUrl, registro }): Promise<void>`

- [x] **Step 1: Escribir tests rojos**

Crear `tests/busqueda.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import {
  abrirBusquedaInternet,
  URL_BUSQUEDA,
} from '../src/lib/busqueda.js';

function crearRegistroFalso() {
  return {
    info: vi.fn(async () => {}),
    error: vi.fn(async () => {}),
  };
}

describe('abrirBusquedaInternet', () => {
  it('si Rust abre Brave no usa respaldo', async () => {
    const invocar = vi.fn(async () => 'abierto');
    const abrirUrl = vi.fn(async () => {});
    const registro = crearRegistroFalso();

    await abrirBusquedaInternet({ invocar, abrirUrl, registro });

    expect(invocar).toHaveBeenCalledWith('abrir_busqueda_brave');
    expect(abrirUrl).not.toHaveBeenCalled();
    expect(registro.info).toHaveBeenCalledWith('[busqueda] Brave abierto');
  });

  it('si Brave no está disponible abre URL fija con navegador predeterminado', async () => {
    const abrirUrl = vi.fn(async () => {});
    const registro = crearRegistroFalso();

    await abrirBusquedaInternet({
      invocar: async () => 'no_disponible',
      abrirUrl,
      registro,
    });

    expect(abrirUrl).toHaveBeenCalledWith(URL_BUSQUEDA);
    expect(registro.info).toHaveBeenCalledWith(
      '[busqueda] Brave no disponible; usando navegador predeterminado'
    );
  });

  it('si invoke falla registra error y usa respaldo', async () => {
    const causa = new Error('comando ausente');
    const abrirUrl = vi.fn(async () => {});
    const registro = crearRegistroFalso();

    await abrirBusquedaInternet({
      invocar: async () => { throw causa; },
      abrirUrl,
      registro,
    });

    expect(registro.error).toHaveBeenCalledWith(
      '[busqueda] error al abrir Brave',
      causa
    );
    expect(abrirUrl).toHaveBeenCalledWith(URL_BUSQUEDA);
  });

  it('si respaldo falla registra error y nunca rechaza', async () => {
    const causa = new Error('sin navegador');
    const registro = crearRegistroFalso();

    await expect(abrirBusquedaInternet({
      invocar: async () => 'no_disponible',
      abrirUrl: async () => { throw causa; },
      registro,
    })).resolves.toBeUndefined();

    expect(registro.error).toHaveBeenCalledWith(
      '[busqueda] error en navegador predeterminado',
      causa
    );
  });
});
```

- [x] **Step 2: Ejecutar tests y comprobar rojo**

```powershell
npm test -- tests/busqueda.test.js
```

Expected: FAIL porque `src/lib/busqueda.js` no existe.

- [x] **Step 3: Implementar orquestador**

Crear `src/lib/busqueda.js`:

```js
export const URL_BUSQUEDA = 'https://search.brave.com/';

export async function abrirBusquedaInternet({ invocar, abrirUrl, registro }) {
  try {
    const estado = await invocar('abrir_busqueda_brave');
    if (estado === 'abierto') {
      await registro.info('[busqueda] Brave abierto');
      return;
    }
    await registro.info(
      '[busqueda] Brave no disponible; usando navegador predeterminado'
    );
  } catch (causa) {
    await registro.error('[busqueda] error al abrir Brave', causa);
  }

  try {
    await abrirUrl(URL_BUSQUEDA);
  } catch (causa) {
    await registro.error('[busqueda] error en navegador predeterminado', causa);
  }
}
```

- [x] **Step 4: Conectar Inicio en `main.js`**

Añadir import:

```js
import { abrirBusquedaInternet } from './lib/busqueda.js';
```

Debajo de `alPulsarTarjeta`, añadir:

```js
const alBuscarInternet = () => abrirBusquedaInternet({
  invocar: invoke,
  abrirUrl: openUrl,
  registro,
});
```

Cambiar registro de ruta Inicio a:

```js
registrarRuta(/^#\/$/, () =>
  renderInicio(manifest, { navegarA, alBuscarInternet }));
```

- [x] **Step 5: Verificar integración**

```powershell
npm test -- tests/busqueda.test.js tests/inicio.test.js
npm test
npm run build
```

Expected: 65 tests totales verdes y build Vite verde.

- [x] **Step 6: Commit**

```powershell
git add src/lib/busqueda.js tests/busqueda.test.js src/main.js
git commit -m "feat: integrar búsqueda con respaldo y logs"
```

---

### Task 4: Sección Compras y guía segura

**Files:**
- Modify: `entorno-contenido/manifest.json`
- Create: `entorno-contenido/guias/comprar-seguro.md`
- Create: `entorno-contenido/iconos/amazon.png`
- Create: `entorno-contenido/iconos/aliexpress.png`
- Create: `entorno-contenido/iconos/temu.png`
- Create: `entorno-contenido/iconos/elcorteingles.png`
- Create: `entorno-contenido/iconos/carrefour.png`

**Interfaces:**
- Consumes: tipos existentes `guia` y `enlace`, grupos existentes, descargador de favicons
- Produces: contenido v7 con sección `compras` y guía de ocho pasos

- [x] **Step 1: Confirmar contenido base**

```powershell
cd B:\01_Proyectos\Entorno-para-Papa\entorno-contenido
npm run check
```

Expected: `Contenido OK`.

- [x] **Step 2: Añadir guía completa**

Crear `guias/comprar-seguro.md`:

```markdown
---
titulo: Comprar por internet con seguridad
---

## Paso 1: Entra desde el Entorno
Abre **Compras** y pulsa la tienda que quieras usar.

No entres desde enlaces que lleguen por correo, mensaje o anuncio. Así siempre empiezas en la página oficial.

## Paso 2: Busca el producto
Pulsa la barra de búsqueda de la tienda. Escribe qué necesitas con pocas palabras y pulsa **Enter**.

Por ejemplo: *pilas AA recargables*.

## Paso 3: Mira quién lo vende
Abre un producto y busca el nombre del vendedor, su puntuación y las opiniones de otros compradores.

Si casi no tiene opiniones o muchas son malas, vuelve atrás y elige otro.

## Paso 4: Comprueba qué vas a recibir
Lee descripción, tamaño, cantidad, color y estado. Comprueba que sea exactamente lo que necesitas.

Una foto bonita no basta: manda lo que pone escrito.

## Paso 5: Revisa entrega y devolución
Mira fecha de entrega, gastos de envío y si permite devolución.

Si tardará demasiado o devolverlo parece difícil, no lo compres.

## Paso 6: Revisa la cesta
Pulsa **Cesta** o **Carrito**. Comprueba que no haya productos repetidos y que cada cantidad sea correcta.

Quita cualquier producto que no reconozcas.

## Paso 7: Comprueba dirección e importe
Antes de pagar, lee dirección de entrega e importe total, incluidos envío e impuestos.

Si dirección o total no coinciden, vuelve atrás. Nada se compra hasta confirmar.

## Paso 8: Confirma solo si todo coincide
Confirma el pago desde la propia tienda y desde tu móvil cuando CaixaBank lo pida.

Nunca digas contraseñas o códigos por teléfono o mensaje. Si aparece una urgencia, premio o precio imposible, cancela la compra. Cancelar no estropea nada.
```

- [x] **Step 3: Añadir sección al manifest y provocar fallo por iconos**

Cambiar `"version": 6` a `"version": 7`. Insertar después de sección `aprender`:

```json
{
  "id": "compras",
  "titulo": "Compras",
  "color": "#7B1FA2",
  "grupos": [
    {
      "titulo": "Antes de comprar",
      "tarjetas": [
        {
          "tipo": "guia",
          "titulo": "Comprar con seguridad",
          "guia": "guias/comprar-seguro.md"
        }
      ]
    },
    {
      "titulo": "Tiendas",
      "tarjetas": [
        {
          "tipo": "enlace",
          "titulo": "Amazon",
          "url": "https://www.amazon.es/",
          "icono": "amazon.png"
        },
        {
          "tipo": "enlace",
          "titulo": "AliExpress",
          "url": "https://es.aliexpress.com/",
          "icono": "aliexpress.png"
        },
        {
          "tipo": "enlace",
          "titulo": "Temu",
          "url": "https://www.temu.com/es",
          "icono": "temu.png"
        },
        {
          "tipo": "enlace",
          "titulo": "El Corte Inglés",
          "url": "https://www.elcorteingles.es/",
          "icono": "elcorteingles.png"
        },
        {
          "tipo": "enlace",
          "titulo": "Carrefour",
          "url": "https://www.carrefour.es/",
          "icono": "carrefour.png"
        }
      ]
    }
  ]
}
```

Run:

```powershell
npm run check
```

Expected: FAIL enumerando cinco iconos ausentes.

- [x] **Step 4: Descargar iconos y revisar archivos**

```powershell
node scripts/descargar-iconos.mjs
Get-Item iconos\amazon.png,iconos\aliexpress.png,iconos\temu.png,iconos\elcorteingles.png,iconos\carrefour.png |
  Select-Object Name,Length
```

Expected: cinco archivos con `Length` mayor que cero. Abrirlos y confirmar que cada uno identifica su tienda; favicon genérico o incorrecto no se acepta.

- [x] **Step 5: Validar contenido y guía**

```powershell
npm run check
node --input-type=module -e "import { readFileSync } from 'node:fs'; import { parsearGuia } from '../entorno-app/src/lib/guia.js'; const guia = parsearGuia(readFileSync('guias/comprar-seguro.md', 'utf8')); if (guia.pasos.length !== 8 || guia.pasos.some((paso) => !paso.html.trim())) process.exit(1); console.log('Guía OK: 8 pasos');"
```

Expected: `Contenido OK` y `Guía OK: 8 pasos`.

- [x] **Step 6: Commit de contenido**

```powershell
git add manifest.json guias/comprar-seguro.md iconos/amazon.png iconos/aliexpress.png iconos/temu.png iconos/elcorteingles.png iconos/carrefour.png
git commit -m "feat: añadir sección de compras seguras"
```

---

### Task 5: Variantes de solitario

**Files:**
- Modify: `entorno-contenido/manifest.json`

**Interfaces:**
- Consumes: sección `jugar`, tipo `enlace`, `iconos/solitario.png`
- Produces: contenido v8 con cuatro juegos en orden fijo

- [x] **Step 1: Actualizar versión y tarjetas**

Cambiar manifest a `"version": 8`. Reemplazar tarjetas de sección `jugar` por:

```json
"tarjetas": [
  {
    "tipo": "enlace",
    "titulo": "Solitario clásico",
    "url": "https://www.solitr.com/",
    "icono": "solitario.png"
  },
  {
    "tipo": "enlace",
    "titulo": "Solitario Spider fácil",
    "url": "https://www.solitar.io/solitario-spider1",
    "icono": "solitario.png"
  },
  {
    "tipo": "enlace",
    "titulo": "Carta blanca",
    "url": "https://www.solitar.io/carta-blanca",
    "icono": "solitario.png"
  },
  {
    "tipo": "enlace",
    "titulo": "Tetris",
    "url": "https://tetris.com/play-tetris",
    "icono": "tetris.png"
  }
]
```

- [x] **Step 2: Validar contrato exacto**

```powershell
npm run check
node --input-type=module -e "import { readFileSync } from 'node:fs'; const m = JSON.parse(readFileSync('manifest.json', 'utf8')); const jugar = m.secciones.find((s) => s.id === 'jugar'); const titulos = jugar.tarjetas.map((t) => t.titulo); const esperado = ['Solitario clásico','Solitario Spider fácil','Carta blanca','Tetris']; if (m.version !== 8 || JSON.stringify(titulos) !== JSON.stringify(esperado)) process.exit(1); console.log('Juegos OK');"
```

Expected: `Contenido OK` y `Juegos OK`.

- [ ] **Step 3: Abrir juegos en Brave**

Abrir cada URL directamente en Brave. Para cada una:

- confirmar HTTPS y dominio exacto;
- empezar una partida;
- comprobar que no exige registro;
- en Spider y Carta blanca, probar `Pista` y `Deshacer`;
- confirmar cartas legibles a zoom normal.

Expected: cuatro juegos utilizables; ninguna redirección inesperada.

- [x] **Step 4: Commit**

```powershell
git add manifest.json
git commit -m "feat: añadir Spider y Carta blanca"
```

---

### Task 6: Integración, versión 0.1.4 y release

**Files:**
- Modify: `entorno-app/src-tauri/Cargo.toml`
- Modify: `entorno-app/src-tauri/Cargo.lock`
- Modify: `entorno-app/src-tauri/tauri.conf.json`
- Modify: `entorno-app/recursos/contenido-semilla/**`

**Interfaces:**
- Consumes: app con Tasks 1–3; contenido v8 con Tasks 4–5
- Produces: release firmada `v0.1.4` con semilla v8

- [x] **Step 1: Subir versión**

En `src-tauri/Cargo.toml`:

```toml
version = "0.1.4"
```

En `src-tauri/tauri.conf.json`:

```json
"version": "0.1.4"
```

- [x] **Step 2: Regenerar semilla**

Desde `entorno-app`:

```powershell
npm run semilla
node -e "const fs=require('fs'); const m=JSON.parse(fs.readFileSync('recursos/contenido-semilla/manifest.json','utf8')); if(m.version!==8) process.exit(1); console.log('Semilla v8');"
```

Expected: `Semilla actualizada desde entorno-contenido` y `Semilla v8`.

- [x] **Step 3: Ejecutar verificación automatizada completa**

```powershell
npm test
npm run build
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
cd src-tauri
cargo fmt --check
cargo test
cd ..\..\entorno-contenido
npm run check
```

Expected:

- 65 tests JS verdes;
- build Vite verde;
- 12 tests Rust verdes;
- `Contenido OK`;
- `Cargo.lock` actualizado a paquete `entorno-papa` v0.1.4.

- [x] **Step 4: Verificar UI real en tres resoluciones**

Arrancar desde `entorno-app`:

```powershell
cd B:\01_Proyectos\Entorno-para-Papa\entorno-app
npm run tauri dev
```

Redimensionar ventana a 1024 × 768, 1366 × 768 y 1920 × 1080. En cada tamaño, medir en DevTools/CDP:

```js
({
  ancho: innerWidth,
  overflowHorizontal:
    document.documentElement.scrollWidth >
    document.documentElement.clientWidth,
  columnas: getComputedStyle(
    document.querySelector('.parrilla-secciones')
  ).gridTemplateColumns.split(' ').length,
  buscarPrimero:
    document.querySelector('.pantalla-inicio').firstElementChild
      .classList.contains('boton-buscar-internet'),
  buscarTexto:
    document.querySelector('.boton-buscar-internet').textContent.trim(),
  secciones:
    [...document.querySelectorAll('.tarjeta-seccion')]
      .map((boton) => boton.textContent.trim()),
})
```

Expected:

- `overflowHorizontal: false` siempre;
- 2 columnas a 1024 y 1366;
- 4 columnas a 1920;
- `buscarPrimero: true`;
- texto contiene `BUSCAR EN INTERNET`;
- secciones: Aprender, Compras, Prensa, Jugar.

- [x] **Step 5: Verificar flujos reales**

Con app Tauri abierta:

1. Pulsar `BUSCAR EN INTERNET`; confirmar pestaña `https://search.brave.com/` en proceso Brave.
2. Abrir Compras; recorrer los ocho pasos de guía.
3. Abrir cinco tiendas; confirmar portadas y dominios oficiales.
4. Abrir cuatro juegos; comenzar partida.
5. Ctrl+Mayús+A; confirmar que log incluye `[busqueda] Brave abierto`.
6. Cerrar app y Brave.

Expected: ningún diálogo técnico, enlace roto, icono ausente o título cortado.

- [x] **Step 6: Commit de versión y semilla**

Desde `entorno-app`:

```powershell
git add src-tauri/Cargo.toml src-tauri/Cargo.lock src-tauri/tauri.conf.json recursos/contenido-semilla
git commit -m "chore: preparar versión 0.1.4 con contenido v8"
```

- [ ] **Step 7: Publicar contenido y app**

```powershell
cd B:\01_Proyectos\Entorno-para-Papa\entorno-contenido
git push origin main

cd ..\entorno-app
git push origin main
git tag v0.1.4
git push origin v0.1.4
```

- [ ] **Step 8: Verificar GitHub Actions y assets**

```powershell
$releaseRun = gh run list --workflow release.yml --limit 1 --json databaseId --jq '.[0].databaseId'
gh run watch $releaseRun --exit-status
gh release view v0.1.4 --json isDraft,isPrerelease,tagName,targetCommitish,assets
```

Expected:

- workflow termina `success`;
- release no es draft ni prerelease;
- contiene instalador `*-setup.exe`, firma `.sig` y `latest.json`.

---

### Task 7: Aceptación en equipos y cierre documental

**Files:**
- Modify: `docs/superpowers/plans/2026-07-31-bloque-6-compras-buscador-juegos.md`
- Modify: `docs/REGISTRO.md`

**Interfaces:**
- Consumes: release `v0.1.4`, contenido v8, dos PCs Windows 11 con Brave
- Produces: Bloque 6 cerrado y tags `bloque-6`

- [ ] **Step 1: Verificar actualización en cada PC**

En portátil y torre:

1. Abrir app instalada y esperar actualización/reinicio.
2. Ctrl+Mayús+A: confirmar app v0.1.4 y contenido v8.
3. Pulsar Buscar y confirmar Brave Search.
4. Abrir cada tienda y cada juego.
5. Desconectar Wi-Fi, reabrir app y recorrer guía de compra segura.

Expected: app y guía funcionan offline; enlaces requieren red; cero mensajes técnicos.

- [ ] **Step 2: Prueba de aceptación con el padre**

Sin indicarle dónde pulsar, pedirle:

1. buscar información en Internet;
2. entrar en Amazon desde Entorno;
3. abrir guía de compra segura;
4. jugar Solitario Spider;
5. volver a Inicio.

Expected: completa cinco recorridos sin ayuda. Si se bloquea, anotar punto exacto y no cerrar bloque hasta corregirlo mediante nuevo ciclo TDD.

- [ ] **Step 3: Recoger evidencia**

Ejecutar:

```powershell
cd B:\01_Proyectos\Entorno-para-Papa
git -C entorno-app rev-parse --short HEAD
git -C entorno-contenido rev-parse --short HEAD
git -C entorno-app tag --points-at HEAD
git -C entorno-contenido show -s --format="%h %s"
```

Guardar salidas reales para Registro. Añadir comprobaciones observadas: resoluciones, URLs, versión, contenido, resultado del padre y cualquier desviación.

- [ ] **Step 4: Actualizar plan y Registro**

Marcar todos los pasos completados con `- [x]`. En `docs/REGISTRO.md`:

- actualizar `Estado actual`;
- añadir entrada `Bloque 6 — búsqueda, compras y juegos`;
- registrar commits reales de ambos repos;
- registrar comandos y resultados exactos;
- incluir desviaciones o `Ninguna`;
- incluir pendientes independientes del Bloque 7.

- [ ] **Step 5: Verificar y commit documental**

```powershell
git diff --check
git add docs/superpowers/plans/2026-07-31-bloque-6-compras-buscador-juegos.md docs/REGISTRO.md
git commit -m "docs: registrar cierre del bloque 6"
```

- [ ] **Step 6: Etiquetar cierre**

```powershell
git -C entorno-app tag bloque-6
git -C entorno-app push origin bloque-6
git -C entorno-contenido tag bloque-6
git -C entorno-contenido push origin bloque-6
git tag bloque-6
git push origin main bloque-6
```

Expected: tres repos limpios, tags publicados y Registro apuntando al Bloque 7 como siguiente ciclo.
