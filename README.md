# otrosdias.com — v1.1

Sitio del colectivo **otrosdias** — plataforma de artistas profesionales.
Single-file HTML con **scene engine** interactivo y soporte **bilingüe ES/EN**.
Listo para GitHub Pages.

---

## Concepto

El sitio NO es una página de scroll tradicional. Es un **mundo horizontal** de cinco habitaciones conectadas. El visitante controla un personaje ASCII que las recorre con teclado (o d-pad en móvil). Cada habitación tiene **hotspots** (artistas, productos, transmisiones) que se iluminan al acercarse y se abren con ENTER.

```
[INICIO] ←→ [MANIFIESTO] ←→ [ARTISTAS] ←→ [TIENDA] ←→ [TRANSMISIONES]
```

---

## Mecánica de juego

### Movimiento
- `↑ ↓ ← →` o `WASD` — mueven al personaje en X/Y dentro de la escena
- Al tocar el borde izquierdo o derecho → transición automática a la habitación contigua (fade 350ms)
- `TAB` — viaje rápido a la siguiente escena
- Click en los **tabs superiores** o en las **puertas** laterales — teletransporte directo
- `ENTER` o `ESPACIO` — interactuar con el hotspot activo

### Hotspots
Cada elemento interactivo (artista, prenda, monitor) es un **hotspot** posicionado en coordenadas `%` de la escena. Cuando el personaje está dentro de un radio de 12 unidades, el hotspot:
1. Se ilumina (borde ámbar, glow)
2. Muestra `[ENTER]` parpadeando encima
3. Revela su label debajo
4. Queda como `activeHotspot`. Presionar ENTER abre el panel modal con detalles.

### Salidas (exits)
Las puertas izq/der se activan cuando el personaje pasa cerca de los bordes. También se pueden clickear directamente.

### Panel modal
- **Artistas:** nombre, rol, bio extendida, CTA a Instagram
- **Productos:** nombre editorial (italic serif), autor, descripción, precio, CTA a WhatsApp (link `wa.me/?text=Quiero+SERAFÍN`)
- **Transmisiones:** fecha, mensaje completo, badge "archivado"

---

## Bilingüe (ES / EN)

Sistema de i18n por diccionario centralizado en `I18N = { es: {...}, en: {...} }`.

- Cada elemento traducible tiene `data-i18n="clave.subclave"`
- El botón `EN` / `ES` en el nav superior derecho alterna el idioma completo
- Persiste en `localStorage` con la llave `od_lang`
- También cambia: textos del boot, mensajes aleatorios, contenido del panel modal, labels del HUD, todo

Para agregar un idioma nuevo:
1. Duplica el objeto `I18N.es` como `I18N.pt` (por ejemplo)
2. Traduce todos los valores
3. Modifica el ciclo del botón `langBtn`

---

## Estructura del archivo

Todo está en un solo `index.html` (~1100 líneas). Tres bloques principales:

1. **`<style>` (CSS) — tokens, layout, escenas, hotspots, panel, d-pad, responsive**
2. **`<body>` (HTML) — boot, overlays, wordmark, tabs, tools, HUD, world (5 escenas), panel, d-pad**
3. **`<script>` (JS) — i18n, boot, theme, HUD, character controller, scene transitions, hotspot proximity, panel logic, input (keyboard + d-pad), random transmissions**

### Estado global (un solo objeto)

```js
const STATE = {
  scene: 'inicio',         // escena actual
  x: 50, y: 75,            // posición % del personaje
  facing: 'right',         // dirección visual
  lang: 'es',              // idioma
  theme: 'dark',           // tema
  activeHotspot: null,     // hotspot bajo el personaje
  activeExit: null,        // puerta cercana
  transitioning: false,    // bloqueo durante fade
};
```

---

## Personalización rápida

### Agregar un artista
1. En la escena `scene-artistas`, agrega un nuevo `.hotspot.artist-frame` con `data-id="nuevo_id"` y posición `style="left:X%; top:Y%;"`
2. Agrega la entrada en `PANEL_DATA`:
```js
nuevo_id: { kind:'artist', name:'NUEVO_NOMBRE', role:'artist.nuevo_id.role', bio:'artist.nuevo_id.bio', tag:'PERFIL_A_00X' },
```
3. Agrega las traducciones en `I18N.es` y `I18N.en`:
```js
'artist.nuevo_id.role': 'ROL',
'artist.nuevo_id.bio': 'Bio del artista...',
```

### Agregar un producto
Igual que un artista pero en `scene-tienda` con `class="hotspot product-hotspot"`. En `PANEL_DATA` usa `kind:'product'`.

### Cambiar el SVG de una prenda
Cada producto tiene un SVG inline que representa la prenda. Sustitúyelo por tu propia ilustración (mismo `viewBox="0 0 200 240"`).

### Cambiar transmisiones (la bitácora pública)
En `scene-transmisiones`, edita los monitores. Cada uno tiene una fecha y un mensaje corto. El mensaje extendido va en `I18N` como `trans.full.0X`.

**Importante:** este es el contenido que cambia con frecuencia para que la gente regrese. Considera moverlo a Firestore (ya tienes `firebase-config.js` en el repo) para poder editarlo sin redeployar.

---

## Cosas que NO incluí (intencionalmente)

- **Carrito de compras**: la primera versión usa link a WhatsApp porque es lo más ágil para validar. Si las ventas crecen, conviene Shopify o Stripe Checkout (sin migrar a SaaS pesado).
- **Backend de regalías**: el manifiesto dice "cada autor recibe regalías". Implementarlo de verdad requiere Stripe Connect o un sistema custom. Tu stack actual (FastAPI + PostgreSQL del CafeteriaOS) podría reutilizarse.
- **Imágenes reales de productos**: usé SVGs placeholder. Cuando tengas fotos, sustituye los `<svg>` por `<img src="...">` con object-fit cover.
- **Audios**: tu `index_1.html` tenía audios aleatorios. Los quité porque autoplay con sonido genera fricción inmediata (los browsers lo bloquean y suena spam). Si los quieres, ponles un toggle explícito.

---

## Deployment — GitHub Pages + Namecheap

### Paso 1: Subir el archivo

```bash
git clone https://github.com/otrosdias/otrosdias.github.io.git
cd otrosdias.github.io
cp /ruta/a/index.html .
echo "otrosdias.com" > CNAME      # ← CRÍTICO para custom domain
git add .
git commit -m "v1.1 scene engine + i18n"
git push origin main
```

> El repo `otrosdias.github.io` sirve la raíz del dominio. Si usas otro nombre (ej. `otrosdias/web`), tendrás que activar Pages manualmente desde `Settings → Pages`.

### Paso 2: Activar GitHub Pages

1. `https://github.com/otrosdias/otrosdias.github.io/settings/pages`
2. Source: `Deploy from a branch` → `main` / `/ (root)`
3. **Custom domain**: `otrosdias.com`
4. **Enforce HTTPS** (después de propagación DNS)

### Paso 3: DNS en Namecheap

Panel Namecheap → `otrosdias.com` → **Advanced DNS**:

| Type    | Host | Value                  | TTL  |
|---------|------|------------------------|------|
| A       | @    | `185.199.108.153`      | Auto |
| A       | @    | `185.199.109.153`      | Auto |
| A       | @    | `185.199.110.153`      | Auto |
| A       | @    | `185.199.111.153`      | Auto |
| CNAME   | www  | `otrosdias.github.io.` | Auto |

> ⚠️ Borra los registros `URL Redirect` o `Parking Page` por default.

Verificación: `dig otrosdias.com +short` debe responder los 4 IPs `185.199.108-111.153`.

---

## Migración futura a Vite/React

Tu screenshot mostraba un setup Vite con `Artists.jsx`, `ASCIINavigator.jsx`, `Hero.jsx`. Cuando el contenido crezca, vale la pena migrar este HTML a componentes:

| HTML actual                    | Componente sugerido         |
|--------------------------------|-----------------------------|
| `<div class="boot">`           | `<BootSequence />`          |
| HUD + tools + tabs             | `<TopHUD />` + `<SceneTabs />` |
| `<div id="world">`             | `<World scenes={...} />`    |
| Cada `.scene`                  | `<SceneInicio />`, `<SceneArtistas />`, etc. |
| `<div id="char">`              | `<Character state={...} />` |
| `<div class="hotspot">`        | `<Hotspot data={...} />`    |
| Panel modal                    | `<InfoPanel />`             |
| `I18N` diccionario             | `react-i18next` o context propio |
| `PANEL_DATA`                   | JSON desde Firestore        |
| Posiciones de hotspots         | JSON `artists.json`, `products.json` |

Una vez en React, podrías agregar:
- **CMS visual** para editar transmisiones desde un panel
- **Animaciones más complejas** con Framer Motion (el personaje deslizándose entre escenas en vez de fade)
- **Multi-step character pathfinding** (clickeas un cuadro, el personaje camina hasta allí solo)
- **Sonido espacial** (cada escena con un ambient track, atenuado por distancia)

---

## Archivos

- `index.html` — sitio completo (~1100 líneas, 0 dependencias JS externas)
- `README.md` — este archivo

Solo se cargan **Google Fonts** desde CDN. Todo lo demás es local.
