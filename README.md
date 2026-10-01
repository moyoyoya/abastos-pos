# Abastos Brew · Punto de venta

Offline web app for an Android tablet. No server: the code is hosted as a static site, and all data lives in the tablet's browser (IndexedDB).

## Files
- `index.html` – the whole app (UI + logic)
- `sw.js` – service worker: caches the app so it opens with no internet
- `manifest.webmanifest`, `icon-*.png`, `logo.png` – installable app, Abastos Brew branding

## How it runs (important)
The zip is only for you to upload. Your brother never opens it.
He opens a URL once in Chrome, installs it to the home screen, and from then on it works offline.

## Hosting (you, once)
1. Create a public GitHub repo, e.g. `abastos-pos`, and upload these files to the root.
2. Settings → Pages → Deploy from branch → `main` / root.
3. The app is at `https://<your-user>.github.io/abastos-pos/`.

## Where the data lives, and why updates are safe
- The database lives in Chrome on the tablet, tied to the site address (the "origin": `https://<your-user>.github.io`).
- An update only replaces the code files. Same address → same database → nothing is lost.
- **Never change the address** (repo name, GitHub user, or moving to another domain). A new address = an empty database. If you ever must move, use Respaldo → Enviar respaldo on the old one and Restaurar on the new one.
- Changing the data structure: bump `DB_VERSION` in `index.html` and add an `if (old < N)` step in `openDB()`. Existing data is upgraded in place.

## Publishing an update
1. Upload the changed files to the repo (GitHub Pages redeploys in about a minute).
2. Change `CACHE = 'abastos-pos-v7'` in `sw.js` to the next number. This tells the tablet there's a new version.
3. The next time the tablet opens the app with internet, it downloads the new code in the background; the version after that runs it. No action needed from your brother.

## Instrucciones para instalar (para tu hermano)
1. Con internet, abre el enlace en **Chrome**.
2. Menú (⋮) → **Instalar app** o **Agregar a pantalla de inicio**.
3. Desde ahora ábrela con el ícono de Abastos Brew. Funciona sin internet.
4. Cada 2 semanas: pestaña **Respaldo** → **Enviar respaldo** → elige Gmail → envíatelo a ti mismo.

## Tabs
Vender · Ventas · Productos · Inventario · Gastos · Cierre (cash-up + day/week/month summary) · Respaldo

## Primer uso
- The menu (16 drinks, 4 categories) is preloaded. Edit prices, add sizes (e.g. "Latte grande").
- Inventario: create items (Vaso 12 oz, Vaso 16 oz, Tapa…) with current stock and an alert level.
- Productos → Editar: add the items each product consumes (e.g. Latte grande → 1 Vaso 16 oz, 1 Tapa).
- Stock drops with each sale, comes back if a sale is voided. "+ Entrada" for deliveries, "Conteo" to correct after a physical count. Every change is logged in Movimientos.
