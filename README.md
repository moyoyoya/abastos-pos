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
7. Products: either "Cargar el menú de Abastos Brew", or on the phone that used the old version:
   sign in as admin → Más → Respaldo y ajustes → **Subir a la nube** (one time only).

## Roles
| | Super | Admin | Cajero |
|---|---|---|---|
| Sell, inventory deliveries, expenses, close the day, see everything | ✓ | ✓ | ✓ |
| Void sales, delete expenses, physical counts | ✓ | ✓ | — |
| Edit products/prices, create supplies, settings | ✓ | ✓ | — |
| Create users and assign roles, restore backups | ✓ | — | — |

Enforced twice: the app hides what a role can't do, and `firestore.rules` rejects it on the server.
Deactivating a user (Usuarios → Desactivar) locks them out at once; their past sales stay.

## Data model (Firestore)
`products`, `inventory` (stock changed only by increments, so concurrent sales add up), `movements`,
`sales` (each with `createdBy`), `dailyStats/{day}` (running totals for fast summaries), `expenses`,
`closures/{day}`, `settings/main`, `users/{uid}`.

## Offline behaviour
- First sign-in on each phone needs internet. After that the app opens and sells offline.
- The header shows "Sincronizado", "Subiendo…" or "Sin internet · cambios guardados aquí".
- Close the register only after every phone has synced, or their sales won't count yet.

## Publishing an update
1. Upload the changed files to the repo (GitHub Pages redeploys in about a minute).
2. Bump `CACHE = 'abastos-pos-v9'` in `sw.js` to the next number.
3. Phones pick up the new version the next times they open the app with internet.

## Backups
"Enviar respaldo" exports the cloud data (all phones) as a full backup `.txt` plus CSVs for sales,
inventory, movements, expenses and closures. The super user can re-upload a backup file with "Restaurar".

## PIN lock (optional, per phone)
Respaldo y ajustes → Seguridad. A quick lock on top of the login for a phone left on the counter.
