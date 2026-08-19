# Guía de configuración — Control de Precios

Este paquete tiene tres archivos:

- **precios_supermercado_db.xlsx** — la base de datos (catálogo de productos, categorías, proveedores e historial de precios), lista para importar a Google Sheets.
- **control_precios_app.html** — la app web que se conecta a esa hoja para actualizar precios, márgenes y stock.
- **guia_configuracion.md** — esta guía.

La app se conecta directamente a tu Google Sheets usando las APIs de Google, sin servidor intermedio. Por eso hace falta un paso único de configuración en Google Cloud (gratis, 10-15 minutos).

## Paso 1 — Subir la base de datos a Google Sheets

1. Andá a [Google Drive](https://drive.google.com) y hacé clic en **Nuevo > Subir archivo**.
2. Subí `precios_supermercado_db.xlsx`.
3. Abrilo con doble clic — Google lo abre automáticamente en Google Sheets. También podés hacer clic derecho > Abrir con > Google Sheets si lo prefieres como copia convertida (Archivo > Guardar como Hojas de cálculo de Google).
4. Completá las hojas **Categorias** y **Proveedores** primero, y después cargá tus productos reales en **Productos** (las filas de ejemplo se pueden borrar). Las celdas amarillas son las que se completan a mano; el resto se calcula solo.

## Paso 2 — Copiar el ID de la hoja

En la URL de tu Google Sheet vas a ver algo así:

```
https://docs.google.com/spreadsheets/d/1AbCdEfGhIjKlMnOpQrStUvWxYz/edit#gid=0
```

El ID es la parte entre `/d/` y `/edit`: en el ejemplo, `1AbCdEfGhIjKlMnOpQrStUvWxYz`. Guardalo, lo vas a necesitar en la app.

## Paso 3 — Crear credenciales de Google (una sola vez)

La app necesita un "Client ID" de Google para poder pedir tu permiso y leer/escribir en la hoja.

1. Entrá a [Google Cloud Console](https://console.cloud.google.com/) con tu cuenta de Gmail.
2. Creá un proyecto nuevo (arriba a la izquierda, "Seleccionar proyecto" > "Proyecto nuevo"). Nombre sugerido: `control-precios-super`.
3. En el buscador de arriba, escribí **"Google Sheets API"**, entrá y hacé clic en **Habilitar**.
4. Andá a **APIs y servicios > Pantalla de consentimiento de OAuth**.
   - Tipo de usuario: **Externo**.
   - Completá nombre de la app, tu email, y guardá.
   - En "Usuarios de prueba" agregá tu propio email (así podés usar la app sin publicarla).
5. Andá a **APIs y servicios > Credenciales > Crear credenciales > ID de cliente de OAuth**.
   - Tipo de aplicación: **Aplicación web**.
   - En **Orígenes de JavaScript autorizados**, agregá la dirección desde donde vas a abrir la app (ver Paso 4). Por ejemplo `http://localhost:8000`.
   - Creá, y copiá el **Client ID** (termina en `.apps.googleusercontent.com`).

## Paso 4 — Alojar el archivo HTML

Google no permite el login abriendo el archivo directamente desde tu computadora (`file://`). Necesitás servirlo por `http://` o `https://`. Dos opciones:

**Opción rápida (uso local, en tu propia compu):**

1. Abrí una terminal en la carpeta donde está `control_precios_app.html`.
2. Ejecutá: `python3 -m http.server 8000`
3. Abrí en el navegador: `http://localhost:8000/control_precios_app.html`
4. En el Paso 3, el origen autorizado debe ser `http://localhost:8000`.

**Opción permanente (para que el equipo la use desde cualquier lado):**

Subí `control_precios_app.html` a un hosting gratuito como GitHub Pages, Netlify o Google Sites, y agregá esa URL como origen autorizado en el Paso 3 en lugar de `localhost`.

## Paso 5 — Conectar la app

1. Abrí la app en el navegador.
2. Hacé clic en **Conectar con Google** y aceptá los permisos (acceso a Google Sheets).
3. Deberías ver tu catálogo de productos cargado.

### Compartir el link sin que cada persona tenga que configurar nada

El **Client ID** y el **ID de la hoja** pueden quedar escritos directamente en el archivo `control_precios_app.html`, para que cualquiera que abra el link de la app (por ejemplo, alojado en GitHub Pages) se conecte directo, sin tener que tocar el botón ⚙ Configuración ni pegar ningún dato.

Para hacerlo, abrí el archivo y buscá estas dos líneas cerca del principio del `<script>`:

```js
const DEFAULT_CLIENT_ID = 'PEGAR_ACA_TU_CLIENT_ID.apps.googleusercontent.com';
const DEFAULT_SHEET_ID = 'PEGAR_ACA_EL_ID_DE_TU_HOJA';
```

Reemplazá los dos valores placeholder por tu Client ID real (Paso 3) y tu ID de hoja real (Paso 2), guardá el archivo y volvé a subirlo a tu hosting. A partir de ahí, el botón ⚙ Configuración solo se muestra si esos valores todavía no están cargados (o si alguien los borra manualmente en su navegador).

Esto es seguro: el Client ID de Google no es secreto — la seguridad la da la lista de "Orígenes de JavaScript autorizados" que configuraste en Google Cloud (Paso 3), no el Client ID en sí. El ID de la hoja tampoco otorga acceso por sí solo — quien abra la app igual tiene que iniciar sesión con una cuenta de Google que tenga permiso de Editor sobre esa hoja (ver Notas al final).

## Estructura de la hoja Productos (columnas usadas por la app)

A: SKU · B: Nombre · C: Categoría (se muestra en el catálogo como "Pasillo/Sector") · D: Proveedor · E: Unidad · F: Precio Costo · G: Precio Venta · H: Margen % (fórmula, no se muestra) · I: Stock Actual · J: Stock Mínimo · K: Estado Stock (fórmula, no se muestra ni se usa en la app) · L: Última Actualización · M: Código EAN · N: Código Proveedor · **P: Bloque**

La columna O (el "Pasillo/Sector" que se había agregado antes como columna aparte) ya no se usa — ahora ese nombre se muestra directamente sobre la columna C (Categoría). Si la habías completado, podés dejarla o borrarla, la app no la lee más.

## Estructura de la hoja Proveedores

A: Proveedor · B: Contacto · C: Teléfono · D: Email · E: Categorías que provee · **F: Markup promedio**

La columna F es nueva: agregá el encabezado "Markup promedio" y cargá, para cada proveedor, el markup esperado como número simple (por ejemplo `35` para 35% — no hace falta escribir el símbolo `%` ni usar formato de porcentaje en la celda). En el catálogo, la celda de Markup de cada producto se pinta de **verde** si su markup es igual o mayor al de su proveedor, y de **rojo** si es menor. Los proveedores sin markup cargado en esa columna no colorean la celda.

## Pestañas

Una vez conectado, el contenido se organiza en tres pestañas (arriba, debajo del cuadro de conexión):

- **📦 Catálogo:** los filtros/búsqueda y la tabla completa de productos — lo que se usa el día a día.
- **🔄 Actualizaciones masivas:** los cuatro cuadros para actualizar precios por proveedor, por archivo de venta, y de costo (Borla Hnos / VCC 365) — herramientas de uso ocasional, ahora fuera del camino del catálogo.
- **🏷️ Etiquetas:** generar el PDF de etiquetas de precio para góndola (ver sección siguiente).

## Etiquetas de precio para góndola

En la pestaña **🏷️ Etiquetas** podés generar un PDF con las etiquetas de precio listas para imprimir y cortar:

1. Filtrá por **Proveedor** y/o por fecha de **modificación de precio** (desde/hasta) — dejá los campos vacíos para no filtrar por eso. La fecha es la misma columna "Actualizado" que se ve en el catálogo.
2. Hacé clic en **Buscar productos**: aparece una lista con casillero por producto, todos tildados por defecto — destildá los que no quieras imprimir (o usá el casillero del encabezado para tildar/destildar todos de una).
3. Hacé clic en **Generar PDF de etiquetas**. Se descarga un PDF tamaño A4 con las etiquetas ubicadas de a 3 por fila (nombre en negrita, SKU centrado, precio de venta grande con el "$" a la izquierda), llenando cada hoja y pasando a la siguiente hasta completar todos los productos elegidos, siguiendo el modelo que nos pasaste. Imprimilo y cortá las etiquetas por las líneas.

## Uso diario

- **Editar un precio:** cambiá el valor de Costo o Venta en la fila del producto y hacé clic en el ✓. El markup y la fecha se actualizan solos, y queda un registro en la hoja Historial_Precios.
- **Guardar varios cambios de una vez:** si editaste varias filas del catálogo (precios, markup, EAN, código proveedor, pasillo/sector, proveedor, bloque) y no querés hacer clic en el ✓ de cada una, usá el botón **"💾 Guardar todo"** (al lado de "+ Agregar producto") — revisa todas las filas visibles según los filtros activos, guarda de una sola vez los cambios de todas las que hayan cambiado, y te pide confirmación antes de aplicar.
- **Editar el Markup directamente:** la columna Markup del catálogo también es editable. Si escribís un nuevo Markup, la Venta se recalcula sola (Costo × (1 + Markup%)) — y si en cambio editás la Venta (o el Costo), el Markup mostrado se recalcula y recolorea al toque. En los dos casos, lo único que se guarda en la hoja al hacer clic en **Guardar** es el precio de Venta resultante; el Markup nunca se guarda como tal, siempre se calcula a partir de Costo y Venta. Cuando la Venta sale de un Markup (acá o en la actualización masiva por proveedor), siempre se redondea hacia arriba a la decena más cercana (por ejemplo, $1231 queda en $1240); si escribís la Venta a mano, se guarda tal cual la tipeaste, sin redondear.
- **Editar pasillo/sector y proveedor:** en el catálogo, ambas columnas son listas desplegables — elegís un valor existente (cargado en las hojas Categorias/Proveedores) y hacés clic en **Guardar** en esa fila.
- **Agregar un producto nuevo:** el botón "+ Agregar producto" (al lado de "Exportar a Excel") abre un formulario con SKU, Nombre, Pasillo/Sector, Proveedor, Bloque, EAN, Código Proveedor, Costo y Venta. El SKU y el Nombre son obligatorios, y no te deja cargar un SKU que ya exista. Al confirmar, el producto se agrega al final de la hoja Productos y el catálogo se recarga solo, mostrando el producto nuevo filtrado por su SKU.
- **Filtrar:** por nombre/SKU/EAN, pasillo/sector (categoría), proveedor, bloque, o solo productos con markup bajo (los que aparecen en rojo, por debajo del Markup promedio de su proveedor).
- **Usar una pistola lectora de código de barras:** las pistolas USB o Bluetooth funcionan como un teclado — no necesitan instalación ni configuración especial en la app. Conectá la pistola a la compu (por cable o emparejada por Bluetooth), hacé clic una vez en el campo "Buscar" y escaneá: el código EAN aparece solo y filtra el catálogo. El campo queda enfocado automáticamente después de conectar o recargar datos, así que podés seguir escaneando sin volver a hacer clic. Si un escaneo encuentra un único producto, la fila se resalta en amarillo y se lleva a la vista. Si mientras tanto hiciste clic en otra parte de la página (por ejemplo, para editar un precio), volvé a hacer clic en el campo "Buscar" antes de escanear de nuevo.
- **Escanear con la cámara del celular/tablet:** el botón "📷 Escanear con cámara" (al lado del campo "Buscar") abre la cámara del dispositivo dentro de la misma página — no hace falta instalar ninguna app aparte. La primera vez, el navegador va a pedir permiso para usar la cámara; aceptalo. Apuntá al código de barras del producto y, apenas lo reconoce, el código se carga solo en "Buscar" y filtra el catálogo. Esto requiere que la app esté abierta por `https://` (GitHub Pages ya lo es) o en `http://localhost`; por seguridad, los navegadores no dan acceso a la cámara en otro tipo de conexión.
- **Exportar a Excel:** el botón "Exportar a Excel" (junto a "Recargar datos") descarga un .xlsx con la misma estructura que usa tu sistema para importar precios: columnas `CODIGO`, `DESCRIPCION`, `COSTO`, `VENTA`, pestaña "ActualizacioPrecio.rpt". Exporta los productos que estén visibles según los filtros activos en ese momento — si querés todo el catálogo, limpiá los filtros antes de exportar.
- **Actualización masiva de precio de venta por proveedor:** en el panel correspondiente, elegí el proveedor y el **Método**:
  - **Por markup deseado:** calcula el nuevo precio de venta como Costo × (1 + markup%) para cada producto de ese proveedor (por ejemplo `35` para 35%). Los productos sin costo cargado quedan afuera automáticamente.
  - **Por % de aumento sobre el precio actual:** calcula el nuevo precio de venta como Precio de venta actual × (1 + aumento%) — no depende del costo, sirve para aumentar directamente sobre lo que ya tenés cargado (por ejemplo `10` para un aumento del 10%). Los productos sin precio de venta actual cargado quedan afuera.

  En los dos casos el resultado se redondea hacia arriba a la decena más cercana. Igual que las demás actualizaciones masivas, primero previsualiza los cambios (podés destildar productos puntuales) y recién después de confirmar los aplica y los registra en el historial.
- **Actualizar precio de venta desde un archivo:** en el panel "Actualizar precio de venta desde archivo", subí un .xls/.xlsx/.csv con el código (SKU) en la columna A y el precio de venta en la columna E (una fila por producto; se ignoran automáticamente filas de encabezado o categoría). Hacé clic en "Previsualizar cambios" para revisar qué productos se van a actualizar antes de confirmar — la app te muestra precio actual vs. nuevo y podés destildar filas puntuales. Por defecto ignora precios en $0 del archivo para evitar borrar precios reales por error.
- **Actualizar precio de costo por proveedor desde un archivo:** hay un botón dedicado por proveedor, sin nada que configurar más que elegir el archivo:
  - **"Actualizar costo — Borla Hnos"**: código en columna D, precio de costo en columna I.
  - **"Actualizar costo — VCC 365"**: código en columna E, precio de costo en columna S.

  Cada botón cruza esa columna del archivo contra la columna N (Código Proveedor) de tu catálogo — no contra el SKU — y solo actualiza productos que ya tengan asignado ese proveedor exacto ("Borla Hnos" / "VCC 365") en la hoja Productos. Si el proveedor no tiene productos cargados, o esos productos no tienen Código Proveedor completado, la previsualización va a mostrar 0 coincidencias.
- **Actualizar costo desde factura (otros proveedores):** para cualquier proveedor que no sea Borla Hnos ni VCC 365 (esos dos ya tienen su botón dedicado arriba), usá el cuadro "Actualizar costo desde factura (otros proveedores)":
  1. Elegí el **Proveedor** de la factura.
  2. Conseguí una **foto de la factura**, de dos formas: con el botón **"📷 Usar cámara"** (abre la cámara del dispositivo dentro de la misma página, pedí permiso la primera vez, y "Capturar foto" saca la imagen), o eligiendo un archivo ya existente con el selector de "Foto de la factura". Cualquiera de las dos queda mostrada en pantalla como referencia mientras cargás los datos a mano — no se lee el texto automáticamente: ninguna herramienta de reconocimiento de texto es confiable con fotos de facturas reales, así que preferimos que cargues los números vos, mirando la foto al lado.
  3. Arriba de la tabla, marcá si el **precio de la factura incluye IVA** (si no lo incluye, completá el % de IVA para que se sume) y si el **precio es por unidad** o por bulto — estas dos condiciones aplican a toda la factura.
  4. Cargá una fila por ítem: **Código Proveedor** (se cruza contra la columna N del catálogo, igual que Borla/VCC), la **Descripción** es solo para tu referencia visual y no se guarda, el **Precio** tal cual figura en la factura, la **Cantidad** del bulto (dejalo en 1 si el precio ya es por unidad), y el **% de descuento** de esa línea en particular — a diferencia del IVA y el precio unitario, el descuento sí se carga línea por línea, porque no siempre aplica igual a todos los ítems de una factura. La columna "Costo final" se recalcula sola con cada cambio.
  5. Hacé clic en **"+ Agregar línea"** si necesitás más filas.
  6. Hacé clic en **Previsualizar cambios**, revisá la tabla (podés destildar productos puntuales) y confirmá — igual que las demás actualizaciones masivas, queda registrado en el historial.

  El costo final que se guarda en la hoja siempre queda con IVA incluido, para que sea comparable con el resto del catálogo (Markup se calcula igual para todos los productos, vengan de donde vengan).
- **Historial:** cada cambio de precio (individual o por archivo) se guarda automáticamente en la hoja Historial_Precios, con fecha, precios antes/después y motivo.

## Agregar otro usuario de Gmail

Para que otra persona pueda entrar a la app con su propia cuenta de Gmail, hacen falta dos pasos (los dos son necesarios, uno solo no alcanza):

1. **Compartir la hoja de Google Sheets:** abrí la hoja > botón **Compartir** (arriba a la derecha) > agregá el email de Gmail de esa persona con permiso **Editor** > Enviar.
2. **Agregarla como usuario de prueba en Google Cloud** (obligatorio mientras la pantalla de consentimiento OAuth esté en modo "Prueba", que es el modo normal para uso interno):
   - Entrá a [Google Cloud Console](https://console.cloud.google.com/) con la cuenta donde creaste el proyecto (Paso 3 de esta guía).
   - Andá a **APIs y servicios > Pantalla de consentimiento de OAuth**.
   - En la sección **Usuarios de prueba**, hacé clic en **Add users**, agregá el email de Gmail de la persona y guardá.

Sin el paso 2, esa persona va a ver un error de Google diciendo que la app no está verificada o que no tiene acceso, aunque ya tenga permiso de Editor en la hoja. El modo "Prueba" permite hasta 100 usuarios agregados así, más que suficiente para un equipo chico. Si en algún momento necesitás que cualquier cuenta de Gmail pueda entrar sin agregarla a mano, se puede publicar la app (requiere verificación de Google), pero para uso interno no es necesario.

## Notas

- Los permisos que pide la app están limitados a Google Sheets (no accede a Gmail, Drive completo, etc.).
- El token de acceso vive solo en la sesión del navegador; no se guarda en ningún servidor.
- Si varias personas van a usar la app, cada una necesita "Editor" en la hoja de Google Sheets compartida, y además estar agregada como "Usuario de prueba" en Google Cloud (ver sección anterior).
- El catálogo carga todos los productos que tengas cargados en la hoja Productos, sin límite de cantidad de filas.
