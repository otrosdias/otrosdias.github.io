# otrosdias · index v4

Sitio web interactivo de **otrosdias** — colectivo de artistas y plataforma de ropa de serigrafía fundada por Randy García Mejía, Puebla, México.

---

## Descripción general

Un archivo HTML único, sin dependencias externas ni frameworks. El sitio funciona como un videojuego de exploración 2D: el usuario mueve un personaje por cinco salas navegables, cada una representando una sección del colectivo. La estética es CRT/terminal con tipografía mono y efectos de ruido de pantalla.

La tienda carga su catálogo dinámicamente desde Google Sheets y abre un modal de producto de pantalla completa al hacer clic en cualquier prenda.

---

## Estructura del proyecto

```
index_otrosdias_v4.html   ← único archivo, todo incluido
otrosdias_catalogo.xlsx   ← catálogo de productos (conecta con la web)
```

No requiere servidor, compilador ni instalación. Se abre directamente en el navegador.

---

## Las 5 salas

| # | ID | Contenido |
|---|-----|-----------|
| 1 | `inicio` | Página de entrada con frase del colectivo |
| 2 | `manifiesto` | Texto fundacional de otrosdias |
| 3 | `artistas` | Grid de los 33 miembros del colectivo |
| 4 | `tienda` | Catálogo dinámico de prendas desde Google Sheets |
| 5 | `transmisiones` | Bitácora pública de 6 entradas archivadas |

La navegación entre salas es lateral: el personaje camina hasta el borde de la pantalla y cruza hacia la siguiente sala, o bien se usan los tabs de navegación rápida en la parte superior.

---

## Controles

### Desktop (teclado)

| Tecla | Acción |
|-------|--------|
| `←` `→` `↑` `↓` o `WASD` | Mover personaje |
| `ENTER` o `SPACE` | Interactuar con hotspot cercano |
| `ESC` | Cerrar panel abierto |
| `TAB` | Viaje rápido a la siguiente sala |

### Móvil (D-pad táctil)

D-pad de 5 botones fijo en la esquina inferior derecha: `↑ ↓ ← → ⏎`. Aparece automáticamente en pantallas menores a 900px. Soporta hold para movimiento continuo.

---

## Funcionalidades

### Boot sequence
Al cargar la página por primera vez en la sesión, aparece una terminal que escribe líneas de texto animado estilo CRT antes de revelar el sitio. Se muestra una sola vez por sesión (`sessionStorage`).

### Personaje
ASCII art de `(lol)` que camina por la escena activa, anima las piernas al moverse y se voltea según la dirección. Tamaño `20px` con sombra ámbar.

### Hotspots y proximidad
En las salas Artistas y Transmisiones, al acercarse a un elemento aparece el indicador `[↵]` y se puede interactuar con `ENTER`. En Tienda y Artistas (grid), el clic directo también abre el panel.

### Panel de artistas (pestañas tipo Arkham Asylum)
Al interactuar con cualquier miembro del colectivo se abre un panel con tres pestañas:

- **ATRIBUTOS** — nombre, rol y biografía
- **HECHOS** — ficha técnica en grid (disciplina, estado, proyectos)
- **HISTORIA** — biografía completa + slot para subir una foto PNG/JPG del artista (se guarda en memoria de sesión por artista)

### Tienda dinámica (Google Sheets)
La tienda carga el catálogo desde un CSV público de Google Sheets. Si no hay URL configurada, usa 4 productos de demostración. Las tarjetas de producto muestran imagen real o placeholder SVG de camiseta.

### Modal de producto
Al hacer clic en una prenda se abre un modal de pantalla completa con:
- Imagen principal (frente)
- Thumbnail del reverso (si existe) para cambiar vista
- Nombre en tipografía grande, autor, precio
- Descripción de compra
- Botón de WhatsApp que abre una conversación con mensaje prellenado

### Popup de transmisiones aleatorio
Cada 30–70 segundos aparece un mensaje emergente en la parte inferior con un texto del catálogo `random.01`–`random.08`. Primera aparición a los 18 segundos.

### Modo invertido (colores)
El botón `INVERTIR` en la esquina superior derecha alterna entre fondo negro (oscuro) y fondo crema (claro). Preferencia guardada en `localStorage`.

### Idioma ES / EN
El botón `EN` / `ES` alterna todos los textos del sitio. Los mensajes del popup de transmisiones se mantienen en español en ambos idiomas. Preferencia guardada en `localStorage`.

### HUD
Dos bloques de información fijos en la pantalla:
- Inferior izquierdo: reloj en tiempo real, `Cada día · hoy`
- Inferior derecho: sala actual `XX / 05`, posición `X,Y`, señal `ESTABLE`

---

## Configuración inicial (dos pasos)

### 1. Conectar WhatsApp

En el archivo, busca las dos apariciones de `wa.me` y reemplaza la URL:

```javascript
// Antes
window.open('https://wa.me/?text=' + msg, '_blank');

// Después (con tu número mexicano de 10 dígitos)
window.open('https://wa.me/52TUNUMERO?text=' + msg, '_blank');
```

Ejemplo: si tu número es `2221234567` → escribe `522221234567`

Hay dos instancias: una en el panel de artistas (botón CONTACTAR) y una en el modal de producto (botón COMPRAR VÍA WHATSAPP).

### 2. Conectar el catálogo de Google Sheets

```javascript
// Línea ~2342 del archivo
const SHEET_CSV_URL = 'TU_URL_CSV_AQUI';
```

Reemplaza `'TU_URL_CSV_AQUI'` con la URL CSV de tu hoja publicada (ver sección **Catálogo** más abajo).

---

## Catálogo de productos (Google Sheets)

### Estructura de la hoja `catalogo`

| Columna | Descripción |
|---------|-------------|
| `id` | Identificador único sin espacios. Ej: `fragildevocipn` |
| `nombre` | Nombre visible en mayúsculas. Ej: `FRÁGIL DEVOCIÓN` |
| `autor` | Nombre del artista. Ej: `secret.eyes888` |
| `descripcion` | Texto del modal (máx. 200 caracteres recomendado) |
| `precio` | Con signo. Ej: `$180 MXN` |
| `estado` | `disponible` · `agotado` · `proximo` |
| `imagen_frente` | URL directa de la foto delantera |
| `imagen_reverso` | URL directa de la foto trasera (puede quedar vacío) |

### Publicar el Sheet como CSV

1. Abre tu Google Sheet
2. **Archivo → Compartir → Publicar en la web**
3. Selecciona: hoja `catalogo` → formato **CSV** → clic en **Publicar**
4. Copia el enlace generado
5. Pégalo en `SHEET_CSV_URL` dentro del HTML

La tienda se actualiza automáticamente cada vez que un usuario carga la página. No hay que tocar el HTML de nuevo.

### URLs de imágenes desde Google Drive

1. Sube la imagen a Google Drive
2. Clic derecho → **Compartir** → *Cualquier persona con el enlace puede ver*
3. Del URL copia solo el ID: `drive.google.com/file/d/`**`ESTE_ID`**`/view`
4. La URL final para la hoja queda: `https://drive.google.com/uc?export=view&id=ESTE_ID`

---

## Tipografías

Cargadas desde Google Fonts (requiere conexión a internet):

| Variable | Fuente | Uso |
|----------|--------|-----|
| `--font-mono` | JetBrains Mono | Texto general, HUD, botones |
| `--font-crt` | VT323 | Títulos de salas, nombres de artistas, precios |
| `--font-serif` | Instrument Serif | Texto del manifiesto, títulos de panel |

---

## Paleta de colores

| Variable | Valor | Uso |
|----------|-------|-----|
| `--bg` | `#0a0a0a` | Fondo principal |
| `--bg-2` | `#111111` | Fondo de elementos |
| `--fg` | `#e8e3d3` | Texto principal |
| `--fg-dim` | `#8a8578` | Texto secundario |
| `--fg-muted` | `#4a4a44` | Texto apagado, bordes |
| `--amber` | `#f0b81b` | Acento dorado (precio, activo) |
| `--magenta` | `#ff2d6f` | Acento rojo (agotado, pulso) |
| `--line` | `#1f1f1f` | Bordes y separadores |

En modo invertido (`body.inverted`) el fondo y texto se invierten a crema/negro.

---

## Agregar o editar artistas

Los datos de los 33 artistas viven en el objeto `ARTIST_DATA` dentro del `<script>`. Cada entrada tiene esta forma:

```javascript
nombre_id: {
  name: 'Nombre visible',
  role: 'Disciplina',
  tag: 'A01',
  bio: 'Biografía corta o completa.',
  attrs: {
    'Campo': 'Valor',      // aparece en pestaña HECHOS
  },
  hechos: {
    'Campo': 'Valor',      // aparece en pestaña HISTORIA → ficha
  }
}
```

Para agregar un artista nuevo también hay que:
1. Añadir su tarjeta en el HTML dentro de `.artistas-grid`
2. Añadir su ID en `PANEL_DATA`
3. Añadir sus datos en `ARTIST_DATA`

---

## Responsividad

| Breakpoint | Comportamiento |
|-----------|----------------|
| > 900px | Layout completo, cursor crosshair, sin D-pad |
| ≤ 900px | D-pad visible, tabs en segunda línea, world ajustado |
| ≤ 600px | Títulos de sala en múltiples líneas, tienda en 2 columnas, modal apilado verticalmente |
| ≤ 380px | Tarjetas mínimas, tabs comprimidos |

---

## Despliegue

El archivo es 100% estático. Opciones de publicación:

- **GitHub Pages** — sube el archivo como `index.html` en un repositorio público y activa Pages
- **Netlify Drop** — arrastra el archivo a [app.netlify.com/drop](https://app.netlify.com/drop)
- **Vercel** — conecta el repositorio o sube manualmente
- **Servidor propio** — cualquier hosting de archivos estáticos

No requiere Node.js, PHP ni base de datos.

---

## Créditos

**Concepto y dirección:** Randy García Mejía / otrosdias  
**Sitio:** [marlonstevegm.com](https://marlonstevegm.com)  
**Colectivo:** @otrosdias  
**Año:** 2026

---

*otrosdias · todos los derechos reservados · 2020–2026*
