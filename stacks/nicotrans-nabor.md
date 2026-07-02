# Stack: nicotrans-nabor

> Ověřeno 2026-07-02 (`docker ps`, `compose.yaml`, Caddyfile, `.env` klíče).

## Co to je
Náborový modul Nicotrans (Next.js + Prisma). Stejný vzor jako `nicotrans-palety`/`nicotrans-hr` —
modul napojený na CRM Core (sdílená identita), vystavený jako cesta pod `crm.nuart.cz`.

- **Kontejner:** `nicotrans-nabor`, interní port **3000**, host mapping **3040**
- **Stack:** `/opt/stacks/nicotrans-nabor/`
- **DB:** `crm_nabor` (na sdíleném `infra-postgres`) + identita v `crm_identity`
- **Vystavení:** veřejně pod **crm.nuart.cz/nabor** (Caddy `handle @nabor` → `nicotrans-nabor:3000`,
  path-based, Next basePath se NEstripuje — viz `decisions/006-subdomeny-ne-path.md`)
- **Healthcheck:** HTTP GET na `127.0.0.1:3000/` (autoheal label → restart při unhealthy)
- **Build:** přes SDÍLENÝ builder — `/opt/stacks/infra/build-module.sh nicotrans-nabor`

## Integrace s CRM
`.env`: `DATABASE_URL` (→ `crm_nabor`), `IDENTITY_DATABASE_URL` (→ `crm_identity`),
`CORE_URL="https://crm.nuart.cz"`. Login/práva řeší CRM Core, ne samostatný auth.

## Konfigurace / secrets (umístění)
`.env`: `/opt/stacks/nicotrans-nabor/.env` (chmod 600), `.npmtoken` (read:packages PAT, 600).

## Poznámky
- Přibyl 2026-07-01 (spolu s `nicotrans-hr`) — dokumentace dorovnána při auditu 2026-07-02.
- Zálohuje se: DB `crm_nabor` (dynamický seznam v `/opt/nuart-backup.sh`), konfigy stacku.
- Pozn.: DB má prefix `crm_` (ne `nicotrans_`) — odpovídá napojení na CRM náborový proces.
