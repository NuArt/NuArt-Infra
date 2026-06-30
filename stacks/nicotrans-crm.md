# Stack: nicotrans-crm ⭐

> Ověřeno 2026-06-30 (`docker ps`, Caddyfile, `.env` klíče, psql `\l`).

## Co to je
Nicotrans CRM Core — centrální správa uživatelů a práv nad moduly. **LIVE** na crm.nuart.cz (HTTPS).

- **Kontejner:** `nicotrans-crm`, interní port **3000**, **bez host mappingu** ✅ (healthy)
- **Stack:** `/opt/stacks/nicotrans-crm/`
- **Stack technologie:** Next.js + Prisma + auth (login / seed admin)
- **DB:** `crm_identity` (dedikovaný uživatel `crm_user`) ✅
- **Repo:** github.com/Endyz/Nicotrans-CRM → `/opt/stacks/nicotrans-crm`

## Caddy / routing
- `crm.nuart.cz` (root, BASE_PATH prázdné) → `nicotrans-crm:3000`
- `crm.nuart.cz/palety/*` → `nicotrans-palety:3000` (modul palety pod stejnou doménou)

## Konfigurace / secrets (umístění)
`.env`: `/opt/stacks/nicotrans-crm/.env` — klíče (bez hodnot):
`DATABASE_URL`, `SESSION_SECRET`, `BASE_PATH`, `CRM_HOST`,
`ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_NAME` (iniciální seed admin — dočasné).

## Deploy
- Migrace + seed přes `--profile tools`, pak `up -d --build app`.

## Stav účtů (ověřeno 2026-06-30 v `crm_identity`)
- **Login funguje** ✅ — 3 sessions, poslední 2026-06-30 11:51.
- **1 uživatel (admin):** `daniel.habrda@nuart.cz` (`is_admin`), vytvořen 2026-06-29 seedem.
- Registrovaný modul: `pallets`; role: „Plný přístup".
- ⚠️ **Admin heslo je STÁLE seed heslo** — `users.updated_at == created_at`, záznam se od
  seedu nezměnil. Tzn. běží na `ADMIN_PASSWORD` z `.env` (plaintext, zálohováno na Wedos).
  `[TODO: změnit admin heslo v aplikaci; poté ADMIN_PASSWORD v .env už neplatí (seed jen při initu)]`
  `[opraveno proti realitě 2026-06-30: dump měl „[OVĚŘIT login/heslo]" — login OK, heslo NEzměněno]`

## Související
- `incidenty/crm-caddy-blok.md` — Caddy blok pro crm se historicky musel vložit ručně.
- ⚠️ DB `crm_identity` se zatím **nezálohuje** (viz `proxmox/zalohy.md`).
