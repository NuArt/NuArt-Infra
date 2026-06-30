# Stack: nuart-backoffice

> Ověřeno 2026-06-30 (`docker ps`, Caddyfile, `.env` klíče, git status repa).

## Co to je
Interní firemní backoffice NuArt. Veřejně na **office.nuart.cz** (má login).

- **Kontejner:** `nuart-backoffice`, interní port 3000, host mapping **3020** ✅
- **Stack:** `/opt/stacks/nuart-backoffice/`
- **Stack technologie:** Next.js 16.2.6 + Better Auth + Prisma + Postgres
- **DB:** `nuart` (uživatel `nuart_user`)
- **Caddy:** `office.nuart.cz` → `nuart-backoffice:3000`

## Konfigurace / secrets (umístění)
`.env`: `/opt/stacks/nuart-backoffice/.env` — klíče (bez hodnot):
`DATABASE_URL`, `DIRECT_URL`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`,
`S3_ENDPOINT`, `S3_REGION`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_FORCE_PATH_STYLE`.
Auth config: `src/lib/auth/auth.ts` (`trustedOrigins`).

## Úskalí
- **Better Auth „Invalid origin"** na produkční doméně — viz `incidenty/better-auth-origin.md`.
  Fix: `BETTER_AUTH_URL=https://office.nuart.cz` + `trustedOrigins` (office + IP + localhost)
  + **REBUILD** (`up -d --build`, ne restart).

## Stav gitu (ověřeno 2026-06-30)
- Repo je v sync s `origin/main` — commit `0d98968 feat(auth): trustedOrigins pro produkci
  i lokální přístup` je **pushnutý** (žádný „ahead").
  `[opraveno proti realitě 2026-06-30: dump měl otazník „nepushnutý commit?"; commit JE pushnutý.
   Lokálně je netrackovaný jen soubor docs/BETTER_AUTH_PROXY.md (archivován v tomto repu)]`
