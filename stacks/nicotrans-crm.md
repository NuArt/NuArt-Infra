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

## Otevřené / `[OVĚŘIT]`
- Finální login potvrzen? Admin heslo změněno z iniciálního seedu? — nešlo ověřit bez přihlášení.
- `incidenty/crm-caddy-blok.md` — Caddy blok pro crm se historicky musel vložit ručně.
