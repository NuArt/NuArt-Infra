# Stack: nicotrans-palety

> Ověřeno 2026-06-30 (`docker ps`, Caddyfile, `next.config.ts`, `.env` klíče).

## Co to je
Paletový systém Nicotrans (modul). Next.js + Prisma — vzor pro další moduly.

- **Kontejner:** `nicotrans-palety`, interní port **3000**, host mapping **3010** ✅
  `[opraveno proti realitě 2026-06-30: dump si nebyl jistý interním portem; je 3000]`
- **Stack:** `/opt/stacks/nicotrans-palety/`
- **DB:** `nicotrans` (uživatel `nicotrans_user`)
- **Build:** přes SDÍLENÝ builder `nuart-builder` jediným skriptem
  `/opt/stacks/infra/build-module.sh nicotrans-palety` (NE per-projekt builder).
  `[opraveno 2026-06-30: per-projekt `nicotrans-builder` sjednocen do sdíleného `nuart-builder`]`

## Vystavení — ZMĚNA proti dumpu
`[opraveno proti realitě 2026-06-30: dump říkal „jen interně, bez auth, schválně NEvystaveno,
 plán migrace na nico-palety.nuart.cz". REALITA: palety jsou vystaveny veřejně pod
 **crm.nuart.cz/palety** (Caddy handle @palety → nicotrans-palety:3000).]`

- `next.config.ts`: `const basePath = process.env.BASE_PATH ?? "/palety"` — appka běží pod `/palety`,
  Caddy prefix **NEstripuje** (basePath se zapéká do buildu, viz `decisions/006-subdomeny-ne-path.md`).
- Integrace s CRM: `.env` má `IDENTITY_DATABASE_URL` (→ crm_identity) a `CORE_URL` (→ CRM core).
  Palety tedy nově sdílí identitu/práva s CRM Core (login řeší CRM), ne samostatný auth.
- A-záznam `nico-palety.nuart.cz` existuje (→ 178.255.174.235), ale Caddy ho zatím **neobsluhuje**
  (připraveno pro budoucí subdoménovou migraci). `[OVĚŘIT plán]`

## Konfigurace / secrets (umístění)
`.env`: `/opt/stacks/nicotrans-palety/.env` — klíče (bez hodnot):
`DATABASE_URL`, `DIRECT_URL`, `IDENTITY_DATABASE_URL`, `CORE_URL`,
`S3_ENDPOINT`, `S3_REGION`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_FORCE_PATH_STYLE`, `S3_BUCKET`.

## Plán / TODO
- AI pipeline pro mapování sloupců z Excel/CSV (viz `ai/nicotrans-ai.md`).
- Případná migrace na samostatnou subdoménu `nico-palety.nuart.cz`.
