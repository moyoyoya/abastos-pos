# Abastos Brew · Punto de venta

Web app hosted on GitHub Pages. Data lives in Firebase (Firestore) and every phone keeps an offline copy:
sales keep working without signal and upload when it returns. Several phones can sell at the same time.

## Files
- `index.html` – the whole app (UI + logic). Firebase config and `SUPER_UID` are at the top of the script.
- `firestore.rules` – security rules. Paste into Firebase Console → Firestore Database → Rules → Publish.
- `sw.js` – caches the app, fonts and the Firebase SDK so the app opens without internet.
- `manifest.webmanifest`, `icon-*.png`, `logo.png` – installable app, Abastos Brew branding.

## One-time setup
1. Firebase Console → Authentication → Users → **Add user** (your email + password). Copy its **User UID**.
2. (Done) Super user UID `pNHsoXTZelQyNl785pceyifj30b2` is set in both `index.html` and `firestore.rules`.
3. Publish `firestore.rules` in the console (Firestore Database → Rules → paste → Publish).
4. Authentication → Settings → Authorized domains → add `moyoyoya.github.io`.
5. Upload `index.html`, `sw.js`, `manifest.webmanifest`, `logo.png`, `icon-192.png`, `icon-512.png` to the GitHub repo.
6. Open the app, sign in as super user, then: Más → Respaldo y ajustes → **Usuarios** → create the other accounts.
   Create users from the app, not from the console (the app also saves their role).
7. Products: on an empty shop, Productos → "Restaurar menú base" loads the printed menu (29 items).

## Roles
| | Super | Admin | Cajero |
|---|---|---|---|
| Sell, inventory deliveries, expenses, close the day, see everything | ✓ | ✓ | ✓ |
| Void or edit sales, delete expenses, physical counts | ✓ | ✓ | — |
| Edit products/prices, create supplies, settings | ✓ | ✓ | — |
| Create users, change roles, set new passwords, deactivate (never the super user) | ✓ | ✓ | — |
| Restore backups | ✓ | — | — |

Enforced twice: the app hides what a role can't do, and `firestore.rules` rejects it on the server.
Deactivating a user (Usuarios → Desactivar) locks them out at once; their past sales stay.

## Logins
- Sign in with a **username** or an email. Creating a user only needs username, password and role; display name (defaults to the username capitalized) and email are optional.
- Username-only accounts use an internal address `username@abastosbrew-gdl.firebaseapp.com` (no mail is sent there).
  `logins/{username}` maps the username to that account; it can be read before signing in but reveals nothing without the password.
- Everyone can change their own password in Mi cuenta. Admins can set a new one for anyone except the super user
  (this creates a fresh login with the same username, email mapping, name and role; the old one is deactivated).

## Data model (Firestore)
`products`, `inventory` (stock changed only by increments, so concurrent sales add up), `movements`,
`sales` (each with `createdBy`), `dailyStats/{day}` (running totals for fast summaries), `expenses`,
`closures/{day}`, `settings/main`, `settings/menuBase`, `users/{uid}`.

## Time
- Everything uses Guadalajara time (`America/Mexico_City`), whatever the phone's own time zone.
- Each phone measures its clock against Firebase's server time when online (`clock/{uid}`) and corrects for the gap,
  so a phone with the wrong time still records sales on the right day. A banner warns if the phone is off by 2+ minutes.

## Automatic day close
- At 11:50 pm shop time, any phone with the app open closes the day if nobody did. Days missed that way are closed the
  next time any phone opens the app online.
- Automatic closes have no cash count: they assume the expected cash stayed in the register, so the next day's starting cash
  chains on. They refresh themselves if late sales sync in. Closing by hand ("Cerrar con conteo") replaces them.

## Editing a sale
Admins: Ventas → Editar loads the sale into the cart. Add/remove items or change payment, then "Guardar cambios".
Day totals and inventory are adjusted by the difference. Every edit keeps the version it replaced (`edits`), and "Ver cambios" (or tapping the "Editada" label) opens a popup: original order (time, seller) and each change with time, who, and what changed.

## Menu base
- `settings/menuBase` holds the shop's default menu (products, prices, flavours, supplies used).
- The first time an admin opens v19 online, the menu they have at that moment is saved as the base automatically.
- Productos → **Guardar como menú base** (admins) updates it after menu changes. **Restaurar menú base** puts back
  products that were deleted or changed since; products added later are left alone. It always asks first.
- The printed menu in the code (`MENU`) is only used when the shop has no products at all.

## Selling
Payment defaults to Efectivo (tap another method if needed); "efectivo recibido" isn't asked. A sale is: products → Cobrar.

## Receipt printer (per phone)
- Generic 58 mm Bluetooth thermal printers (ESC/POS). Más → Respaldo y ajustes → Impresora de tickets → Conectar impresora.
- Each sale prints 2 copies (COPIA CLIENTE, COPIA NEGOCIO) when "Imprimir al cobrar" is on. Ventas → Imprimir reprints.
- Uses Web Bluetooth: works in Chrome on Android. Browsers on iPhone don't support it; use the free "Bluefy" app there.
- A print failure never blocks or loses a sale.

## Offline behaviour
- First sign-in on each phone needs internet. After that the app opens and sells offline.
- The header shows "Sincronizado", "Subiendo…" or "Sin internet · cambios guardados aquí".
- Close the register only after every phone has synced, or their sales won't count yet.

## Publishing an update
1. Upload the changed files to the repo (GitHub Pages redeploys in about a minute).
2. Bump `CACHE = 'abastos-pos-v19'` in `sw.js` to the next number.
3. Phones pick up the new version the next times they open the app with internet.

## Backups
"Enviar respaldo" exports the cloud data (all phones) as a full backup `.txt` plus CSVs for sales,
inventory, movements, expenses and closures. The super user can re-upload a backup file with "Restaurar".

## PIN lock (optional, per phone)
Respaldo y ajustes → Seguridad. A quick lock on top of the login for a phone left on the counter.
