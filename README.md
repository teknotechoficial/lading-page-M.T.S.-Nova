<!-- Rebrand a M.T.S. Nova - 2026-11-24 -->
# M.T.S. Nova ÔÇö Landing v3

Sitio est├ítico de M.T.S. Nova: landing p├║blica + panel interno de precios.

- **Producci├│n:** https://teknotech-services.pages.dev/ (Cloudflare Pages)
- **Panel interno:** https://teknotech-services.pages.dev/panel/ (protegido por Cloudflare Access)
- **Repositorio:** https://github.com/teknotechoficial/lading-page-M.T.S.-Nova

## Estructura

```
/
Ôö£ÔöÇÔöÇ index.html            Landing p├║blica
Ôö£ÔöÇÔöÇ css/styles.css        Estilos de la landing
Ôö£ÔöÇÔöÇ js/main.js            Men├║, scroll, formulario y wizard
Ôö£ÔöÇÔöÇ assets/               Logos, favicons e imagen Open Graph
Ôö£ÔöÇÔöÇ panel/
Ôöé   Ôö£ÔöÇÔöÇ index.html        Panel interno (noindex)
Ôöé   Ôö£ÔöÇÔöÇ panel.css         Estilos del panel
Ôöé   Ôö£ÔöÇÔöÇ panel.js          Filtros, tablas, ficha y exportaci├│n CSV
Ôöé   Ôö£ÔöÇÔöÇ marcas.js         Ilustraciones SVG de respaldo
Ôöé   Ôö£ÔöÇÔöÇ datos.json        ÔåÉ cat├ílogo y precios (lo que edita el panel)
Ôöé   Ôö£ÔöÇÔöÇ fichas.json       Fichas t├®cnicas reales, se cargan al abrir una ficha
Ôöé   Ôö£ÔöÇÔöÇ imagenes.json     Mapa "Marca|Modelo" ÔåÆ foto
Ôöé   ÔööÔöÇÔöÇ img/              Fotos de los equipos, una carpeta por marca
Ôö£ÔöÇÔöÇ robots.txt
Ôö£ÔöÇÔöÇ sitemap.xml
ÔööÔöÇÔöÇ _headers              Headers de Cloudflare Pages
```

## C├│mo actualizar los precios

Todo se edita en `panel/datos.json`:

- `actualizado`: fecha que se muestra en el panel.
- `celulares[]`: cada fila es un modelo con una capacidad. Tiene `marca`, `linea`, `modelo`,
  `variante` (almacenamiento, ej. `128GB`), `anio`, `gama`, `precio_nuevo` y `precio_usado`
  (ARS, sin decimales).
  - `precio_usado` es el valor de reventa; `precio_nuevo` es el de un equipo nuevo o sellado hoy.
    Siempre `precio_nuevo >= precio_usado`.
  - Valores v├ílidos de `gama`: `alta`, `media-alta`, `media`, `media-baja`, `baja`.
  - Un mismo modelo puede repetirse con distintas capacidades, cada una con su precio.
  - `anio` es el a├▒o de lanzamiento; el panel lo muestra y permite filtrar por ├®l.
  - `linea` es la familia comercial del modelo (`Galaxy S`, `iPhone Pro Max`, `Redmi Note`ÔÇª).
    Si la marca no tiene familias, coincide con la marca.
- `lineas`: orden en que se muestran las l├¡neas dentro de cada marca. Es un objeto
  `{ "Samsung": ["Galaxy S", "Galaxy Note", ÔÇª] }`; el panel respeta ese orden en la vista
  de cat├ílogo. Si agreg├ís una l├¡nea nueva y no la list├ís ac├í, va al final de su marca.
- `fichas` ya no vive en `datos.json`: est├í en `panel/fichas.json`, con la misma clave
  `"Marca|Modelo"` (ej. `"Apple|iPhone 15"`). El panel lo pide reci├®n cuando abr├¡s una ficha,
  as├¡ el listado no arrastra 1,8 MB extra.
  - Campos: `pantalla`, `chipset`, `ram`, `bateria`, `camara`, `frontal`, `antutu`, `soporte`.
  - `real: true` muestra ÔÇ£Datos de referencia del modeloÔÇØ; `false` muestra ÔÇ£Datos estimados seg├║n gama y a├▒oÔÇØ.
  - Si un modelo no est├í en `fichas.json`, el panel arma una ficha estimada con la gama y el a├▒o.
- `servicios[]`: cada rubro tiene `rubro`, `detalle` y `precio`.
  - Mientras `precio` sea `null`, el panel muestra ÔÇ£A completarÔÇØ.

El cat├ílogo actual tiene **10.114 filas (9.602 modelos) de 100 marcas (2010ÔÇô2026)**, con
8.417 fichas t├®cnicas reales. Los precios son estimados: el usado se calcula por gama, se ajusta
por capacidad y se deprecia seg├║n el a├▒o de lanzamiento, con un piso por gama para que un equipo
viejo no quede en un valor irreal. El nuevo se deriva del usado con un margen que crece con la
edad del modelo (`1,30 + 0,05 ├ù a├▒os`, tope a los 8 a├▒os). Verificar antes de cotizar.

### Origen de los datos

El cat├ílogo se genera con `build-catalog.py` a partir de:

- **GSMArena** (`gsmarena-raw.json`): las 128 marcas de su buscador, 12.685 equipos, de los
  cuales 10.826 son de 2010ÔÇô2026. Aporta modelo, a├▒o, tipo (smartphone / tablet / feature phone),
  pantalla, chipset, RAM, bater├¡a, c├ímara y almacenamiento.
- **Lista curada** (la secci├│n `CATALOGO` del generador): los modelos y gamas de las marcas
  principales, que tienen prioridad y variantes de capacidad verificadas a mano.

Reglas de armado:

- Se excluyen relojes, bandas y pulseras inteligentes (GSMArena los publica como ÔÇ£phoneÔÇØ).
- Se excluyen tablets, incluso las que GSMArena publica con tipo ÔÇ£phoneÔÇØ y delata el nombre
  (VivoTab, Ativ Tab, XPS 10, Lumia 2520, Wi10N, Surface salvo Duo).
- Se incluyen smartphones y feature phones; los feature phones van siempre a gama baja.
- La gama sale del chipset cuando est├í; si no, del a├▒o y del nombre, y la RAM solo ajusta dentro
  de su ├®poca (2 GB era gama alta en 2014 y baja en 2020). Un sufijo ÔÇ£Pro MaxÔÇØ/ÔÇ£UltraÔÇØ sube un
  escal├│n, pero no convierte por s├¡ solo un equipo de gama media en gama alta.
- El almacenamiento se normaliza a la capacidad comercial (`1024GB` ÔåÆ `1TB`, `13.3GB` ÔåÆ `16GB`).
  Si GSMArena no publica capacidad, el modelo queda con una sola fila y `variante: "s/d"`.
- Deduplicaci├│n por nombre exacto y por nombre base (ignora `5G`/`4G`/`LTE`).

### Orden del cat├ílogo

El panel abre en **Cat├ílogo (marca y l├¡nea)**, que ordena en este orden:

1. Marca, alfab├®tico (`localeCompare` en espa├▒ol).
2. L├¡nea, seg├║n el orden de `lineas` (ej. en Samsung: Galaxy S, Note, A, Z Fold, Z Flip, MÔÇª).
3. A├▒o, del m├ís nuevo al m├ís viejo.
4. Modelo, con orden natural cuando comparten a├▒o (`S9` antes que `S10`, `S26` antes que
   `S26+` antes que `S26 Ultra`).
5. Capacidad, de menor a mayor (`256GB`, `512GB`, `1TB`).

En ese modo la tabla muestra encabezados de grupo por marca y por l├¡nea. Los dem├ís ├│rdenes
(Mayor precio, Menor precio, M├ís nuevo, M├ís viejo, Marca, Gama) son planos, sin encabezados.

### Im├ígenes

La lista y la ficha muestran la foto del equipo cuando existe. `panel/imagenes.json` mapea
`"Marca|Modelo"` a un archivo en `panel/img/<marca>/<modelo>.jpg`, descargado de GSMArena y
normalizado a 160├ù212 px. Cobertura actual: **6.396 de 9.602 modelos (66,6%)**, 6.388 archivos,
59 MB, ~9,5 KB por foto. Si un modelo no tiene foto, `panel/marcas.js` dibuja un SVG de respaldo
con el color de la marca y la cantidad de c├ímaras (seg├║n `fichas.camara`, o seg├║n la gama si no
hay ficha). En m├│vil la miniatura de la tabla se oculta para no romper el ancho; la foto se ve
al abrir la ficha.

El emparejamiento con GSMArena no es solo por nombre exacto: `emparejar.py` prueba reglas
conservadoras en orden (sin espacios, marca incluida en el nombre, a├▒o pegado al modelo,
generaci├│n, c├│digo de modelo, conectividad) y prefiere no asignar foto antes que asignar la de
otro equipo. Con eso se recuperaron 214 modelos que antes quedaban sin imagen.

Los 3.206 modelos sin foto son, en su mayor├¡a, equipos viejos o de marcas menores cuyo archivo
en GSMArena no se llama igual que la ficha, y el sitio bloquea el HTML (`HTTP 429`) para
averiguarlo. Cuando se libere el bloqueo, el descargador completa el resto abriendo la ficha de
cada modelo.

Las fotos son renders de prensa de GSMArena, con copyright: se usan solo en este panel interno.
Para agregar una marca nueva al SVG de respaldo, sumar su color al objeto `COLORES` de `marcas.js`.

Despu├®s de editar, commit y push: Cloudflare Pages redeploya solo.

## Desarrollo local

El panel carga `datos.json` por `fetch`, as├¡ que necesita un servidor HTTP:

```bash
python -m http.server 8080
```

Luego abrir:

- http://localhost:8080/ ÔåÆ landing
- http://localhost:8080/panel/ ÔåÆ panel

## Deploy en Cloudflare Pages

1. Cloudflare Dashboard ÔåÆ Workers & Pages ÔåÆ Create ÔåÆ Pages ÔåÆ Connect to Git.
2. Elegir el repo `teknotechoficial/TeknoTech-Services`.
3. Branch: `main`. Framework preset: `None`. Build command: vac├¡o. Output directory: `/`.
4. Deploy. La URL queda como `https://<nombre-del-proyecto>.pages.dev`.
5. Si el nombre del proyecto no es `teknotech-services`, actualizar la URL en:
   - `index.html` (canonical, `og:url`, `og:image`, Twitter y JSON-LD)
   - `sitemap.xml`
   - `robots.txt`

## Proteger el panel con Cloudflare Access

1. Cloudflare Zero Trust ÔåÆ Access ÔåÆ Applications ÔåÆ Add an application ÔåÆ Self-hosted.
2. Dominio: `<proyecto>.pages.dev`, path `panel`.
3. Policy: `Allow`, regla `Emails` con el email del due├▒o.
4. Login method: One-time PIN (email).
5. Guardar. La primera visita a `/panel/` pide el c├│digo enviado por mail.

> Nota: sin Access, el panel es p├║blico aunque tenga `noindex`. `noindex` solo evita que Google lo indexe, no que alguien entre.

## Entrada al panel desde el sitio

- **Cinco clics seguidos** (menos de 2 segundos entre uno y otro) sobre el logo del encabezado
  o del pie abren el pedido de contrase├▒a.
- Con la contrase├▒a correcta se abre `/panel/`.
- Si alguien entra directo a `/panel/` sin pasar por el logo, el panel pide la contrase├▒a
  antes de cargar nada.
- La contrase├▒a no est├í en texto plano: se guarda como hash SHA-256 en `js/access-core.js`.
  Para cambiarla:

  ```bash
  node -e "console.log(require('crypto').createHash('sha256').update('NUEVA_CLAVE').digest('hex'))"
  ```

  Y reemplazar el valor de `HASH` en `js/access-core.js`.

> Esta contrase├▒a es una barrera de comodidad, no seguridad real: el c├│digo es p├║blico y
> cualquiera puede leerlo y probar sin l├¡mite. La protecci├│n efectiva es Cloudflare Access.

## Pendientes conocidos

- Precios de servicio t├®cnico sin cargar (18 rubros con `precio: null`).
- Fichas t├®cnicas: 8.417 modelos con datos reales de GSMArena; el resto se estima seg├║n gama y a├▒o.
- El precio nuevo de modelos viejos es te├│rico: ya no se consiguen sellados, sirve solo como referencia.
- La marca de agua a pantalla completa del `index.html` se mantiene por decisi├│n del due├▒o; pesa ~470 KB en base64.
- Sin dominio propio: cuando haya uno, actualizar las URLs listadas arriba.
