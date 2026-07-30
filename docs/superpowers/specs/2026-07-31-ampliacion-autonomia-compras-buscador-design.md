# Ampliación de autonomía: búsqueda, compras, juegos y ayudas

**Fecha:** 2026-07-31
**Estado:** aprobado en sesión de diseño
**Extiende:** `2026-07-26-entorno-papa-design.md`

## 1. Objetivo

Ampliar el entorno para que el padre del autor pueda buscar en Internet, entrar en tiendas conocidas, comprar con más seguridad y elegir entre varios solitarios sin pedir ayuda. Después se añadirán guías para sus tareas habituales con el ordenador: CaixaBank, YouTube e impresión.

La ampliación conserva los principios de la v1: letra grande, un clic, profundidad mínima, contenido actualizable a distancia, funcionamiento offline de las guías y degradación segura ante fallos.

## 2. Descomposición de la hoja de ruta

La petición se divide para no mezclar un cambio de aplicación, contenido comercial y guías bancarias en una sola entrega.

| Bloque | Entrega | Resultado verificable |
|---|---|---|
| 6 | Búsqueda con Brave, sección Compras, guía de compra segura y más solitarios | El padre abre Brave Search, entra en cada tienda y abre cada juego sin ayuda |
| 7 | Ayudas de ordenador: CaixaBank, YouTube e impresión | El padre completa los recorridos habituales siguiendo guías de ocho pasos o menos |
| Posterior | Personalización pendiente de entrevista | Se definirán valores bursátiles favoritos, WhatsApp para Windows y lista de tareas antes de diseñarlos |

El plan de implementación inmediato cubrirá solo el Bloque 6. El Bloque 7 tendrá especificación y plan propios después de terminar y observar el uso del Bloque 6.

## 3. Bloque 6: alcance funcional

### 3.1 Botón «BUSCAR EN INTERNET»

La pantalla Inicio mostrará un botón grande en la esquina superior izquierda, por encima del saludo y de la rejilla de secciones.

- Texto exacto: `BUSCAR EN INTERNET`.
- Icono: lupa reconocible y decorativa.
- Acción: abrir `https://search.brave.com/` con Brave para Windows.
- No abre una pantalla intermedia ni incrusta el buscador dentro de Tauri.
- El saludo, reloj e indicador de contenido conservan su comportamiento.
- A 1024 × 768 y 1366 × 768, las cuatro secciones se mostrarán en una rejilla 2 × 2. A resoluciones mayores, la rejilla podrá usar más columnas sin reducir los mínimos de accesibilidad.

El botón pertenece a la aplicación, no al `manifest.json`: tiene posición y comportamiento especiales que una sección normal no puede representar.

### 3.2 Lanzamiento controlado de Brave

Se añadirá un comando backend sin parámetros públicos, por ejemplo `abrir_busqueda_brave`. El propio backend contiene la URL fija de Brave Search; el frontend no puede pasar una ruta de programa, un comando ni una URL arbitraria.

El comando buscará `brave.exe` en estas instalaciones habituales:

1. `%PROGRAMFILES%\BraveSoftware\Brave-Browser\Application\brave.exe`
2. `%ProgramFiles(x86)%\BraveSoftware\Brave-Browser\Application\brave.exe`
3. `%LOCALAPPDATA%\BraveSoftware\Brave-Browser\Application\brave.exe`

Al encontrarlo, lo ejecutará con `https://search.brave.com/` como único argumento y devolverá estado `abierto`. Si Brave no aparece o no puede arrancar, devolverá estado `no_disponible`; el frontend registrará el diagnóstico y abrirá la misma URL con `openUrl`, mecanismo existente para el navegador predeterminado. En los dos equipos de destino, Brave deberá permanecer instalado y configurado antes de la aceptación.

No se añadirá un tipo genérico de tarjeta `programa`: sería más capacidad de la necesaria y permitiría una superficie de ejecución más amplia.

### 3.3 Sección «Compras»

El Inicio añadirá una cuarta sección:

- `id`: `compras`
- título: `Compras`
- color: `#7B1FA2`

La sección tendrá dos grupos:

1. **Antes de comprar**
   - Tarjeta de guía `Comprar con seguridad`.
2. **Tiendas**
   - Amazon — `https://www.amazon.es/`
   - AliExpress — `https://es.aliexpress.com/`
   - Temu — `https://www.temu.com/es`
   - El Corte Inglés — `https://www.elcorteingles.es/`
   - Carrefour — `https://www.carrefour.es/`

Cada tienda tendrá icono reconocible. Las tarjetas abrirán exclusivamente la portada oficial mediante el mecanismo de enlaces existente. No se guardarán cuentas, contraseñas, direcciones, tarjetas ni historial de compra dentro del entorno.

Amazon, AliExpress y Temu son mercados con vendedores y productos variables. La interfaz los presentará como accesos directos a dominios legítimos, no como garantía sobre cada oferta.

### 3.4 Guía «Comprar con seguridad»

La guía tendrá un máximo de ocho pantallas:

1. Entrar siempre desde la sección Compras del entorno.
2. Buscar el producto con palabras sencillas.
3. Revisar quién vende, puntuación y opiniones.
4. Comprobar descripción, tamaño, cantidad y estado.
5. Revisar fecha, coste de entrega y condiciones de devolución.
6. Añadir a la cesta y comprobar productos repetidos.
7. Revisar dirección e importe total antes de pagar.
8. Confirmar pago solo si todo coincide; detenerse ante urgencias, premios, precios imposibles o peticiones de claves.

La creación inicial de cuentas, dirección y método de pago se hará una vez con el autor presente. Las capturas usarán datos ficticios u ocultos.

### 3.5 Sección «Jugar»

Se conservarán Tetris y el solitario actual. Se aclararán los nombres y se añadirán dos variantes:

- `Solitario clásico` — `https://www.solitr.com/`
- `Solitario Spider fácil` — `https://www.solitar.io/solitario-spider1`
- `Carta blanca` — `https://www.solitar.io/carta-blanca`
- `Tetris` — `https://tetris.com/play-tetris`

Spider usa la variante de un palo porque el propio sitio la presenta como versión para principiantes. No se añadirán más variantes en este bloque: cuatro juegos visibles ofrecen variedad sin crear una lista difícil de recordar.

## 4. Arquitectura y flujo de datos

La ampliación mantiene la separación actual:

- `entorno-app` recibe el botón especial y el comando seguro para Brave.
- `entorno-contenido` recibe sección Compras, guía, iconos y nuevas tarjetas de juego.

Flujo de búsqueda:

1. El padre pulsa `BUSCAR EN INTERNET`.
2. Frontend invoca comando sin argumentos.
3. Backend localiza Brave, intenta abrir la URL fija y devuelve `abierto` o `no_disponible`.
4. Ante `no_disponible`, frontend registra el fallo y usa `openUrl` como respaldo.

Flujo de compras y juegos:

1. App sincroniza y valida nueva versión del contenido.
2. Render existente pinta sección, grupos y tarjetas.
3. Una guía abre en el visor local; un enlace abre en navegador.
4. Un icono ausente se oculta sin romper la tarjeta, como en v1.

El contrato del `manifest.json` no cambia. El contenido subirá de versión y la semilla de la app se regenerará antes de publicar la nueva release.

## 5. Seguridad y errores

- Todas las URLs nuevas usan HTTPS y se revisan contra el dominio oficial.
- La guía prohíbe entrar en tiendas o banco desde enlaces de correos, SMS o anuncios.
- Ningún dato personal o bancario se guarda en el repositorio público.
- El comando de Brave no acepta datos del manifest ni del frontend.
- Un fallo al lanzar Brave no bloquea la aplicación.
- Una web caída o que cambie su interfaz no rompe el entorno; se corrige mediante actualización de contenido.
- Los fallos técnicos se registran en `entorno.log` con el saneamiento ya existente.
- No se afirmará que una tienda o vendedor garantiza todas las compras.

## 6. Pruebas y aceptación del Bloque 6

### Automatizadas

- Test frontend: botón existe, texto exacto, posición estructural antes del saludo y un clic invoca su callback.
- Tests Rust: orden de rutas candidatas, URL fija, elección de primera instalación disponible y camino de respaldo sin permitir argumentos arbitrarios.
- `npm test` en `entorno-app`.
- `cargo test` en `entorno-app/src-tauri`.
- `npm run check` en `entorno-contenido`.
- `npm run build` en `entorno-app`.

### Manuales

- Revisar Inicio en 1024 × 768, 1366 × 768 y 1920 × 1080: sin scroll horizontal, texto sin cortar y controles de al menos 120 px donde corresponda.
- Pulsar Buscar y comprobar por proceso/ventana que abre Brave en Brave Search.
- Abrir las cinco tiendas en Brave y confirmar dominio, portada y ausencia de redirecciones inesperadas.
- Abrir los cuatro juegos y empezar una partida sin registro obligatorio.
- Recorrer completa la guía de compra segura.
- Ver al padre buscar, entrar en una tienda y abrir cada juego sin ayuda.

La nueva versión de la app se publicará como release firmada. Después, el contenido se publicará mediante su repositorio y se comprobará la sincronización en producción.

## 7. Bloque 7: alcance aprobado para diseño posterior

La sección `Aprender` se reorganizará en grupos sin cambiar su identidad:

- **Banco:** entrar mediante `caixabank.es`, consultar saldo y movimientos, hacer transferencias, pagar recibos u operaciones habituales, descargar e imprimir justificantes y cerrar sesión.
- **Internet:** buscar y ver vídeos en YouTube; pausa, volumen, pantalla completa y subtítulos.
- **Impresión:** imprimir una página de Brave o un PDF con la Canon PIXMA MG5700 por Wi-Fi; elegir páginas, copias, color y cancelar.

Cada guía tendrá como máximo ocho pasos y una acción principal por paso. Toda captura bancaria deberá ocultar nombres, saldos, IBAN, identificadores, códigos y cualquier otro dato antes de entrar en el repositorio público.

## 8. Fuera de alcance actual

- Elegir accesos directos a valores bursátiles hasta preguntar cuáles consulta.
- Instalar y vincular WhatsApp para Windows para videollamadas.
- Diseñar lista de tareas hasta decidir quién crea las tareas y si existe sincronización remota.
- Incrustar webs, filtrar toda la navegación, gestionar pagos o guardar credenciales.

## 9. Referencias verificadas

- Brave Search: `https://search.brave.com/`
- Seguridad de CaixaBank: `https://www.caixabank.es/particular/seguridad/esquiva-las-nuevas-trampas-de-los-ciberdelincuentes.html`
- Compra segura de CaixaBank: `https://www.caixabank.es/particular/seguridad/tips-seguridad-tricicle.html`
- Solitario Spider fácil: `https://www.solitar.io/solitario-spider1`
- Carta blanca: `https://www.solitar.io/carta-blanca`
