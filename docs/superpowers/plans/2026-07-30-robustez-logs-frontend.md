# Robustez de logs frontend — Plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Persistir en `entorno.log` los eventos y fallos del frontend relacionados con sincronización de contenido y actualización automática.

**Architecture:** Un adaptador `src/lib/registro.js` será la única frontera con `@tauri-apps/plugin-log`. El adaptador normaliza errores y absorbe fallos propios; `main.js` lo comparte con sync y updater, mientras el plugin Rust existente conserva archivo, formato y rotación.

**Tech Stack:** Tauri v2, `tauri-plugin-log` v2 ya registrado en Rust, `@tauri-apps/plugin-log` v2, Vite 8, Vitest 4, JavaScript ES modules, Rust 1.97.

## Global Constraints

- Windows 11; aplicación Tauri v2 con identificador `com.marquibel.entorno`.
- Padre nunca ve mensajes de error ni controles nuevos.
- Todo fallo del logger debe quedar absorbido; nunca bloquea arranque, sync o updater.
- Mantener `entorno.log`, nivel `Info`, máximo 1 MB y `KeepSome(2)` ya configurados en `src-tauri/src/lib.rs`.
- No modificar `src-tauri/src/sync.rs`, claves, firma, endpoint ni configuración criptográfica del updater.
- No redirigir globalmente `console.*`.
- No registrar contraseñas, claves, contenido de guías ni rutas privadas.
- Seguir TDD: test rojo, implementación mínima, test verde.
- Repos separados: plan/registro viven en repo raíz; código vive en repo `entorno-app`.
- Ejecutar código desde un worktree aislado del repo `entorno-app`, creado con `superpowers:using-git-worktrees`.
- Release `v0.1.3` prohibida hasta terminar verificación real de `entorno.log`.

---

### Task 1: Adaptador seguro y bindings oficiales

**Files:**
- Create: `entorno-app/src/lib/registro.js`
- Test: `entorno-app/tests/registro.test.js`
- Modify: `entorno-app/package.json`
- Modify: `entorno-app/package-lock.json`
- Modify: `entorno-app/src-tauri/capabilities/desktop.json`

**Interfaces:**
- Consumes: `info(message): Promise<void>` y `error(message): Promise<void>` de `@tauri-apps/plugin-log`.
- Produces: `crearRegistro({ info?, error? } = {}): { info(message): Promise<void>, error(context, cause): Promise<void> }`.
- Garantía: ambos métodos públicos siempre resuelven `undefined`, incluso si el plugin rechaza.

- [ ] **Step 1: Escribir test rojo del adaptador**

Crear `tests/registro.test.js`:

```js
import { describe, it, expect, vi } from 'vitest';
import { crearRegistro } from '../src/lib/registro.js';

describe('crearRegistro', () => {
  it('envía mensajes informativos al nivel info', async () => {
    const info = vi.fn(async () => {});
    const registro = crearRegistro({ info, error: vi.fn() });

    await registro.info('[sync] inicio');

    expect(info).toHaveBeenCalledWith('[sync] inicio');
  });

  it('convierte Error en texto y añade contexto', async () => {
    const error = vi.fn(async () => {});
    const registro = crearRegistro({ info: vi.fn(), error });

    await registro.error('[sync] error', new Error('sin red'));

    expect(error).toHaveBeenCalledWith('[sync] error · sin red');
  });

  it('conserva causas que ya son cadenas', async () => {
    const error = vi.fn(async () => {});
    const registro = crearRegistro({ info: vi.fn(), error });

    await registro.error('[updater] error', 'firma inválida');

    expect(error).toHaveBeenCalledWith('[updater] error · firma inválida');
  });

  it('convierte otros valores de forma segura', async () => {
    const error = vi.fn(async () => {});
    const registro = crearRegistro({ info: vi.fn(), error });

    await registro.error('[sync] error', 503);

    expect(error).toHaveBeenCalledWith('[sync] error · 503');
  });

  it('usa texto neutro si la causa no puede convertirse', async () => {
    const error = vi.fn(async () => {});
    const registro = crearRegistro({ info: vi.fn(), error });
    const imposible = { toString() { throw new Error('no convertir'); } };

    await registro.error('[sync] error', imposible);

    expect(error).toHaveBeenCalledWith('[sync] error · error desconocido');
  });

  it('absorbe rechazos del propio plugin', async () => {
    const falloPlugin = async () => { throw new Error('plugin roto'); };
    const registro = crearRegistro({ info: falloPlugin, error: falloPlugin });

    await expect(registro.info('mensaje')).resolves.toBeUndefined();
    await expect(registro.error('contexto', new Error('causa'))).resolves.toBeUndefined();
  });
});
```

- [ ] **Step 2: Ejecutar test y confirmar fallo**

Run:

```powershell
npx vitest run tests/registro.test.js
```

Expected: FAIL porque `../src/lib/registro.js` no existe.

- [ ] **Step 3: Instalar bindings JavaScript oficiales**

Run:

```powershell
npm install @tauri-apps/plugin-log@^2
```

Expected: `@tauri-apps/plugin-log` aparece en `dependencies`; `package-lock.json` queda actualizado.

- [ ] **Step 4: Autorizar comando del plugin**

En `src-tauri/capabilities/desktop.json`, dejar permisos así:

```json
"permissions": [
  "updater:default",
  "process:default",
  "log:default"
]
```

No ejecutar `tauri add log`: plugin Rust ya está instalado y configurado; ese comando podría reescribir `lib.rs`.

- [ ] **Step 5: Implementar adaptador mínimo**

Crear `src/lib/registro.js`:

```js
import { info as infoTauri, error as errorTauri } from '@tauri-apps/plugin-log';

function textoError(causa) {
  if (causa instanceof Error) return causa.message || causa.name;
  if (typeof causa === 'string') return causa;
  try {
    return String(causa);
  } catch {
    return 'error desconocido';
  }
}

async function escribirSeguro(escritor, mensaje) {
  try {
    await escritor(mensaje);
  } catch {
    // El log es diagnóstico: nunca puede romper la función diagnosticada.
  }
}

export function crearRegistro({
  info: escribirInfo = infoTauri,
  error: escribirError = errorTauri,
} = {}) {
  return {
    info(mensaje) {
      return escribirSeguro(escribirInfo, mensaje);
    },
    error(contexto, causa) {
      return escribirSeguro(escribirError, `${contexto} · ${textoError(causa)}`);
    },
  };
}
```

- [ ] **Step 6: Ejecutar test focal y suite completa**

Run:

```powershell
npx vitest run tests/registro.test.js
npm test
npm run build
```

Expected: `6 passed` en `registro.test.js`; suite completa verde; Vite genera `dist/` sin errores.

- [ ] **Step 7: Commit**

```powershell
git add package.json package-lock.json src-tauri/capabilities/desktop.json src/lib/registro.js tests/registro.test.js
git commit -m "feat: añadir registro frontend seguro"
```

---

### Task 2: Instrumentar actualización automática

**Files:**
- Modify: `entorno-app/src/lib/actualizacion.js`
- Modify: `entorno-app/tests/actualizacion.test.js`
- Modify: `entorno-app/src/main.js`

**Interfaces:**
- Consumes: `registro` producido por `crearRegistro`.
- Produces: `actualizarApp({ comprobar, relanzar, registro }): Promise<void>`.
- Eventos: comprobación, sin actualización, versión encontrada, instalación terminada y error.

- [ ] **Step 1: Sustituir tests por expectativas de eventos**

En `tests/actualizacion.test.js`, mantener imports y sustituir `describe` completo:

```js
describe('actualizarApp', () => {
  function crearRegistroFalso() {
    return {
      info: vi.fn(async () => {}),
      error: vi.fn(async () => {}),
    };
  }

  it('sin actualización registra comprobación y ausencia', async () => {
    const relanzar = vi.fn();
    const registro = crearRegistroFalso();

    await actualizarApp({ comprobar: async () => null, relanzar, registro });

    expect(registro.info.mock.calls).toEqual([
      ['[updater] comprobación'],
      ['[updater] sin actualización'],
    ]);
    expect(registro.error).not.toHaveBeenCalled();
    expect(relanzar).not.toHaveBeenCalled();
  });

  it('con actualización registra versión, instala y relanza', async () => {
    const descargarEInstalar = vi.fn(async () => {});
    const relanzar = vi.fn(async () => {});
    const registro = crearRegistroFalso();

    await actualizarApp({
      comprobar: async () => ({ version: '0.1.3', downloadAndInstall: descargarEInstalar }),
      relanzar,
      registro,
    });

    expect(registro.info.mock.calls).toEqual([
      ['[updater] comprobación'],
      ['[updater] encontrada v0.1.3'],
      ['[updater] instalada; relanzando'],
    ]);
    expect(descargarEInstalar).toHaveBeenCalledOnce();
    expect(relanzar).toHaveBeenCalledOnce();
  });

  it('si la comprobación falla registra error y no lanza', async () => {
    const relanzar = vi.fn();
    const registro = crearRegistroFalso();
    const causa = new Error('sin red');

    await expect(actualizarApp({
      comprobar: async () => { throw causa; },
      relanzar,
      registro,
    })).resolves.toBeUndefined();

    expect(registro.error).toHaveBeenCalledWith('[updater] error', causa);
    expect(relanzar).not.toHaveBeenCalled();
  });

  it('si la instalación falla registra error y no relanza', async () => {
    const relanzar = vi.fn();
    const registro = crearRegistroFalso();
    const causa = new Error('descarga rota');

    await expect(actualizarApp({
      comprobar: async () => ({
        version: '0.1.3',
        downloadAndInstall: async () => { throw causa; },
      }),
      relanzar,
      registro,
    })).resolves.toBeUndefined();

    expect(registro.error).toHaveBeenCalledWith('[updater] error', causa);
    expect(relanzar).not.toHaveBeenCalled();
  });
});
```

- [ ] **Step 2: Ejecutar tests y confirmar fallo**

Run:

```powershell
npx vitest run tests/actualizacion.test.js
```

Expected: FAIL porque implementación todavía no usa `registro`.

- [ ] **Step 3: Implementar eventos updater**

Sustituir función de `src/lib/actualizacion.js`:

```js
export async function actualizarApp({ comprobar, relanzar, registro }) {
  await registro.info('[updater] comprobación');
  try {
    const actualizacion = await comprobar();
    if (!actualizacion) {
      await registro.info('[updater] sin actualización');
      return;
    }
    await registro.info(`[updater] encontrada v${actualizacion.version}`);
    await actualizacion.downloadAndInstall();
    await registro.info('[updater] instalada; relanzando');
    await relanzar();
  } catch (e) {
    await registro.error('[updater] error', e);
  }
}
```

- [ ] **Step 4: Crear registro compartido en main**

En `src/main.js` añadir:

```js
import { crearRegistro } from './lib/registro.js';
```

Junto a constantes globales:

```js
const registro = crearRegistro();
```

Cambiar llamada final:

```js
actualizarApp({ comprobar: check, relanzar: relaunch, registro });
```

- [ ] **Step 5: Ejecutar test focal, suite y build**

Run:

```powershell
npx vitest run tests/actualizacion.test.js
npm test
npm run build
```

Expected: `4 passed` en test focal; suite completa verde; build verde.

- [ ] **Step 6: Commit**

```powershell
git add src/lib/actualizacion.js tests/actualizacion.test.js src/main.js
git commit -m "feat: registrar eventos del updater"
```

---

### Task 3: Instrumentar sincronización de contenido

**Files:**
- Modify: `entorno-app/src/lib/registro.js`
- Modify: `entorno-app/tests/registro.test.js`
- Modify: `entorno-app/src/main.js`

**Interfaces:**
- Consumes: `registro.info`, `registro.error` y resultado `EstadoSync` de Rust.
- Produces: `registrarResultadoSync(registro, resultado): Promise<void>`.
- Estados soportados: `actualizado`, `sin_cambios`, `error`, `dev`; cualquier otro estado seguro se trata como sin cambios.

- [ ] **Step 1: Añadir tests rojos de resultados sync**

Cambiar import de `tests/registro.test.js`:

```js
import { crearRegistro, registrarResultadoSync } from '../src/lib/registro.js';
```

Añadir:

```js
describe('registrarResultadoSync', () => {
  function crearRegistroFalso() {
    return {
      info: vi.fn(async () => {}),
      error: vi.fn(async () => {}),
    };
  }

  it('registra que no hay cambios', async () => {
    const registro = crearRegistroFalso();

    await registrarResultadoSync(registro, { estado: 'sin_cambios' });

    expect(registro.info).toHaveBeenCalledWith('[sync] sin cambios');
  });

  it('registra versión y primeros siete caracteres del SHA', async () => {
    const registro = crearRegistroFalso();

    await registrarResultadoSync(registro, {
      estado: 'actualizado',
      version: 6,
      sha: '0872dc06d5b78dde',
    });

    expect(registro.info).toHaveBeenCalledWith(
      '[sync] actualizado · versión 6 · SHA 0872dc0'
    );
  });

  it('registra estado de error devuelto por Rust', async () => {
    const registro = crearRegistroFalso();

    await registrarResultadoSync(registro, {
      estado: 'error',
      detalle: 'GitHub no responde',
    });

    expect(registro.error).toHaveBeenCalledWith('[sync] error', 'GitHub no responde');
  });

  it('distingue sync desactivado en desarrollo', async () => {
    const registro = crearRegistroFalso();

    await registrarResultadoSync(registro, { estado: 'dev' });

    expect(registro.info).toHaveBeenCalledWith('[sync] omitido en desarrollo');
  });
});
```

- [ ] **Step 2: Ejecutar tests y confirmar fallo**

Run:

```powershell
npx vitest run tests/registro.test.js
```

Expected: FAIL porque `registrarResultadoSync` no está exportada.

- [ ] **Step 3: Implementar clasificador de resultado**

Añadir a `src/lib/registro.js`:

```js
export async function registrarResultadoSync(registro, resultado) {
  if (resultado?.estado === 'error') {
    await registro.error('[sync] error', resultado.detalle ?? 'error desconocido');
    return;
  }
  if (resultado?.estado === 'actualizado') {
    const version = resultado.version ?? 'desconocida';
    const sha = typeof resultado.sha === 'string' && resultado.sha
      ? resultado.sha.slice(0, 7)
      : 'desconocido';
    await registro.info(`[sync] actualizado · versión ${version} · SHA ${sha}`);
    return;
  }
  if (resultado?.estado === 'dev') {
    await registro.info('[sync] omitido en desarrollo');
    return;
  }
  await registro.info('[sync] sin cambios');
}
```

- [ ] **Step 4: Cablear eventos en main**

Cambiar import:

```js
import { crearRegistro, registrarResultadoSync } from './lib/registro.js';
```

Sustituir `sincronizar()`:

```js
async function sincronizar() {
  await registro.info('[sync] inicio');
  try {
    const resultado = await invoke('sync_now');
    await registrarResultadoSync(registro, resultado);
    if (resultado.estado === 'actualizado') {
      manifest = await cargarManifest();
      window.dispatchEvent(new HashChangeEvent('hashchange'));
    }
    const previo = document.querySelector('.indicador-contenido');
    const estado = resultado.version ? resultado : await invoke('estado_sync');
    previo?.replaceWith(renderIndicador(estado));
  } catch (e) {
    await registro.error('[sync] error', e);
  }
}
```

- [ ] **Step 5: Ejecutar test focal, suite, build y Rust**

Run:

```powershell
npx vitest run tests/registro.test.js
npm test
npm run build
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
cargo test --manifest-path src-tauri/Cargo.toml
```

Expected: `10 passed` en `registro.test.js`; suite JS completa verde; build verde; 9 tests Rust verdes.

- [ ] **Step 6: Confirmar que motor protegido no cambió**

Run:

```powershell
git diff -- src-tauri/src/sync.rs src-tauri/src/lib.rs src-tauri/tauri.conf.json
```

Expected: sin salida.

- [ ] **Step 7: Commit**

```powershell
git add src/lib/registro.js tests/registro.test.js src/main.js
git commit -m "feat: registrar eventos de sincronización"
```

---

### Task 4: Versión 0.1.3 y verificación real

**Files:**
- Modify: `entorno-app/src-tauri/tauri.conf.json`
- Modify: `entorno-app/src-tauri/Cargo.toml`
- Modify: `entorno-app/src-tauri/Cargo.lock` si Cargo actualiza versión del paquete raíz

**Interfaces:**
- Consumes: Tasks 1–3 completas.
- Produces: build local `0.1.3` con evidencia real de eventos frontend en `entorno.log`.
- Release: solo tras integrar rama y verificar archivo/panel.

- [ ] **Step 1: Subir versión a 0.1.3**

En `src-tauri/tauri.conf.json`:

```json
"version": "0.1.3"
```

En `src-tauri/Cargo.toml`:

```toml
version = "0.1.3"
```

- [ ] **Step 2: Regenerar semilla y ejecutar verificación automática completa**

Run:

```powershell
npm run semilla
npm test
npm run build
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
cargo test --manifest-path src-tauri/Cargo.toml
git diff --check
```

Expected:

- contenido semilla actualizado o confirmado sin cambios;
- suite JavaScript completa verde;
- 9 tests Rust verdes;
- build Vite verde;
- `git diff --check` sin errores.

- [ ] **Step 3: Arrancar Tauri y producir eventos reales**

Run:

```powershell
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"
npm run tauri dev
```

Expected:

- app abre maximizada;
- log recibe `[sync] inicio`;
- build de desarrollo recibe `[sync] omitido en desarrollo`;
- updater recibe `[updater] comprobación`;
- si `v0.1.3` aún no está publicada, recibe `[updater] sin actualización`.

- [ ] **Step 4: Verificar panel admin**

Con app abierta:

1. Pulsar `Ctrl+Shift+A`.
2. Confirmar líneas `[sync]` y `[updater]`.
3. Confirmar app sigue mostrando contenido normal.
4. Cerrar panel y app.

- [ ] **Step 5: Verificar archivo real**

Run:

```powershell
$log = Join-Path $env:LOCALAPPDATA 'com.marquibel.entorno\logs\entorno.log'
Get-Content -LiteralPath $log -Tail 50
```

Expected: mismas líneas `[sync]` y `[updater]` observadas en panel.

- [ ] **Step 6: Commit de versión**

Revisar cambios:

```powershell
git status --short
git diff -- src-tauri/tauri.conf.json src-tauri/Cargo.toml src-tauri/Cargo.lock
```

Commit:

```powershell
git add src-tauri/tauri.conf.json src-tauri/Cargo.toml src-tauri/Cargo.lock recursos/contenido-semilla
git commit -m "chore: preparar versión 0.1.3"
```

Si `Cargo.lock` o semilla no cambiaron, `git add` los ignora sin error.

- [ ] **Step 7: Gate de publicación**

No crear tag todavía. Ejecutar `superpowers:finishing-a-development-branch`, integrar rama elegida por usuario y comprobar `main`.

Solo después:

```powershell
git tag v0.1.3
git push origin main
git push origin v0.1.3
gh run watch --repo 3dpicas/entorno-app
gh release view v0.1.3 --repo 3dpicas/entorno-app
```

Expected: workflow verde; release no borrador con instalador, firma y `latest.json`.

---

## Cierre documental

Tras publicar:

1. Marcar checkboxes ejecutados en este plan.
2. Añadir entrada bajo `Post-v1` en `docs/REGISTRO.md` con:
   - hashes exactos de commits de `entorno-app`;
   - conteos reales de tests;
   - evidencia de líneas vistas en `entorno.log`;
   - release `v0.1.3`;
   - desviaciones y pendientes.
3. Mantener capturas de guías como pendiente independiente.
4. Actualizar `Estado actual`.
5. Commit en repo raíz:

```powershell
git add docs/superpowers/plans/2026-07-30-robustez-logs-frontend.md docs/REGISTRO.md
git commit -m "docs: registrar robustez de logs frontend"
```

