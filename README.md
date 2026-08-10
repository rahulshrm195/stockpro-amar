# StockPro — Amar Furniture

Offline-capable, installable PWA for daily stock management of hardware and
consumables (nails, screws, grinder paper, etc.). Single admin (you) +
manager (Ajay) roles. Cost/₹ data is hidden from the manager role — enforced
by Firestore security rules, not just the UI.

## 1. Firebase project setup

You can reuse the existing `amar-furniture-e4782` project (same one behind
QuotePro/LathePro/Umbra) or create a fresh project — either works, it's just
a config swap.

1. Go to the [Firebase console](https://console.firebase.google.com) → your project.
2. **Build → Authentication → Sign-in method** → enable **Email/Password**.
3. **Build → Firestore Database** → create database (production mode, closest region — e.g. `asia-south1`).
4. `firebaseConfig` in `index.html` is already wired up to your `amar-furniture-e4782` project — nothing to fill in here.
5. Deploy `firestore.rules` via the Firebase console (**Firestore → Rules** → paste the contents of `firestore.rules` → Publish), or with the CLI: `firebase deploy --only firestore:rules`.

## 2. First login (bootstrap admin)

1. In Firebase Console → Authentication → Users → **Add user** with your email (`rahulshrm195@gmail.com`) and a password.
2. Open the app and sign in with that email/password.
3. Because the email matches the one hardcoded in `firestore.rules` and `index.html` (`BOOTSTRAP_ADMIN_EMAIL`), your account is automatically set up as **Admin** on first login.
4. From there, go to **Users → Add manager** to create Ajay's login — no console work needed for that one.

## 3. Deploy (GitHub Pages + Cloudflare, same pattern as your other apps)

1. Push this folder to a new repo, e.g. `rahulshrm195/stockpro-amarfurniture`.
2. Repo → **Settings → Pages** → deploy from the `main` branch (root).
3. Add a `CNAME` file at the repo root containing: `stock.amarfurniture.in`
4. In Cloudflare DNS for `amarfurniture.in`, add a **CNAME** record: `stock` → `rahulshrm195.github.io`, proxied (orange cloud) or DNS-only — match whatever you used for `woodpro`/`umbra`.
5. In GitHub Pages settings, set the custom domain to `stock.amarfurniture.in` and enable **Enforce HTTPS** once the certificate provisions.

## 4. What's in v1

- **Items** (admin) — name, category, unit, reorder level, cost/unit, opening stock.
- **Receive / Issue** (admin + manager) — logs a movement and adjusts stock via `increment()`, so it queues correctly even offline.
- **Dashboard** — low-stock list, recent activity (manager sees only their own activity), inventory ₹ value (admin only).
- **Ledger** (admin) — last 300 movements, searchable, CSV export.
- **Reports** (admin) — issued-stock value grouped by project.
- **Users** (admin) — add manager accounts without needing the Firebase console.
- Installable, dark mode by default, works offline (Firestore's own offline queue + a service worker caching the app shell).

## 5. Known v1 trade-offs (worth knowing about)

- **Offline stock updates use `increment()`, not a transaction.** A strict "don't let stock go negative" check needs a live round-trip to the server, which defeats offline support. Instead, Issue shows a soft warning based on the last-known stock figure, but doesn't hard-block — so occasional negative stock is possible if two people issue the same item while both offline. For your team size (you + one manager) this should be rare; let me know if you'd rather it hard-block and skip true offline writes.
- **Reports use *current* cost/unit, not the price at the time of each movement.** If a cost changes, past project reports will reflect the new price, not what it actually cost back then. Fine for now since your item costs don't change often; if that becomes important I can snapshot cost per movement instead.
- **Icons are placeholder** (brass hex-bolt mark, matching the app's accent color) — swap `icons/icon-192.png`, `icons/icon-512.png`, `icons/icon-maskable-512.png` for real branding whenever you like.

## 6. Versioning (same pattern as your other apps)

When you change anything, bump **both**:
- `APP_VERSION` in `index.html` (and add a `CHANGELOG` entry so the What's New modal shows it)
- `CACHE_NAME` in `sw.js`

Safari rules followed: no nested backtick template literals, no inline
`onclick`, all interactions wired through `data-action` + delegated
listeners, syntax-checked with `node --input-type=module --check`.
