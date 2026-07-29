# Robustez de logs frontend — Diseño

**Fecha:** 2026-07-30  
**Estado:** aprobado para planificación

## Objetivo

Registrar en `entorno.log` los eventos importantes y los fallos de sincronización de contenido y actualización automática que hoy solo aparecen en la consola del WebView.

La mejora será invisible para el padre: no añade botones, avisos ni pantallas. El panel de administración existente seguirá mostrando las últimas líneas del mismo archivo.

## Alcance

- Integrar las bindings JavaScript oficiales `@tauri-apps/plugin-log`.
- Autorizar `log:default` para la ventana principal.
- Mantener la configuración Rust existente:
  - archivo `entorno.log`;
  - nivel máximo `Info`;
  - rotación a 1 MB;
  - dos rotaciones conservadas.
- Añadir un adaptador frontend pequeño en `src/lib/registro.js`.
- Registrar eventos de sincronización y actualización.
- Convertir errores desconocidos en texto seguro y legible.
- Silenciar cualquier fallo del propio sistema de log para que nunca interrumpa la app.

## Fuera de alcance

- Cambiar el motor Rust de sincronización.
- Cambiar updater, claves, firma o endpoint de releases.
- Redirigir globalmente todos los métodos `console.*`.
- Añadir exportación, copia o envío remoto del log.
- Mostrar errores al padre.
- Cambiar el panel de administración.

## Arquitectura

`src/lib/registro.js` será la única frontera frontend con `@tauri-apps/plugin-log`. Recibirá las funciones `info` y `error` del plugin y expondrá operaciones seguras para:

- registrar información;
- registrar errores con contexto;
- normalizar valores lanzados que no sean instancias de `Error`;
- absorber rechazos producidos por el propio plugin.

`src/main.js` construirá el adaptador y lo usará durante la sincronización. También lo inyectará en `actualizarApp`, siguiendo el patrón de dependencias inyectadas que esa función ya utiliza para `comprobar` y `relanzar`.

El plugin Rust ya configurado seguirá escribiendo todos los registros en `entorno.log`. No habrá segundo archivo ni escritura manual desde JavaScript.

## Flujo de eventos

### Sincronización

1. Antes de invocar `sync_now`: `[sync] inicio`.
2. Si no hay contenido nuevo: `[sync] sin cambios`.
3. Si hay actualización: `[sync] actualizado · versión 6 · SHA 0872dc0` (valores reales; SHA limitado a sus primeros 7 caracteres).
4. Si `sync_now` devuelve estado de error: `[sync] error · conexión rechazada` (detalle real normalizado).
5. Si la invocación falla antes de devolver estado: mismo formato de error.
6. En una build de desarrollo, donde el sync remoto está desactivado: `[sync] omitido en desarrollo`.

### Actualización de la app

1. Antes de comprobar releases: `[updater] comprobación`.
2. Si no existe versión nueva: `[updater] sin actualización`.
3. Si existe: `[updater] encontrada v0.1.3` (versión real recibida).
4. Tras instalar y antes de relanzar: `[updater] instalada; relanzando`.
5. Si falla comprobación, descarga o instalación: `[updater] error · conexión rechazada` (detalle real normalizado).

Los mensajes no incluirán contraseñas, claves de firma, contenido de guías ni rutas privadas.

## Manejo de fallos

- Fallo de `info()` o `error()` se captura y se ignora.
- Fallo de log nunca cambia el resultado de sync o updater.
- Fallo de sync o updater continúa degradándose en silencio para el padre.
- Los mensajes se esperan antes de operaciones que pueden terminar el proceso, especialmente `relaunch`, para maximizar probabilidad de persistencia.
- Errores se normalizan así:
  - `Error` → `message`;
  - cadena → misma cadena;
  - otro valor → conversión segura a texto;
  - conversión imposible → `error desconocido`.

## Pruebas

### Unitarias

- Adaptador envía mensajes al nivel correcto.
- Adaptador normaliza `Error`, cadenas y valores desconocidos.
- Rechazo del plugin queda silenciado.
- Updater registra secuencia correcta sin actualización.
- Updater registra secuencia correcta con actualización.
- Updater registra error sin lanzar.
- Sync registra inicio y resultado sin cambios.
- Sync registra actualización con versión y SHA corto.
- Sync registra tanto el estado `error` devuelto por Rust como un rechazo de la invocación.
- Sync registra `dev` sin describirlo erróneamente como “sin cambios”.

### Regresión

- Toda suite JavaScript permanece verde.
- Nueve tests Rust permanecen verdes.
- Build Vite termina sin errores.

### Verificación real

1. Arrancar aplicación Tauri.
2. Ejecutar sincronización.
3. Abrir panel con `Ctrl+Shift+A`.
4. Confirmar eventos `[sync]` en panel.
5. Leer `entorno.log` en directorio de logs y confirmar mismas líneas.
6. Verificar al menos un evento `[updater]` tras arranque.

## Criterios de aceptación

- Eventos frontend definidos aparecen en `entorno.log`.
- Panel admin los muestra mediante `leer_log_reciente`.
- Ningún error del logger bloquea arranque, sync o updater.
- Padre no ve interfaz ni mensajes nuevos.
- No se modifica lógica Rust de sync ni configuración criptográfica del updater.
