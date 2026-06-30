# Konvence pojmenování a struktury

## Kontejnery a stacky
- Kontejnery: malá písmena, pomlčky (`nuart-backoffice`, `nicotrans-palety`, `nicotrans-crm`,
  `infra-postgres`, `infra-minio`).
- Prefix `infra-` = sdílená infrastruktura.
- Stack = složka v `/opt/stacks/<nazev>/` s vlastním `compose.yaml` + `.env`.

## Databáze
- DB pojmenované podle domény/účelu: `nuart`, `nicotrans`, `crm_identity`.
- **DB uživatelé: dedikovaní per modul** — ne sdílení. Ověřeno 2026-06-30:
  `nuart_user`, `nicotrans_user`, `crm_user` (+ superuser `postgres`).
  `[opraveno proti realitě 2026-06-30: dump uváděl jen uživatele postgres + crm_user;
   reálně má dedikovaného uživatele i nuart (nuart_user) a nicotrans (nicotrans_user)]`

## Porty
- Interní port appek: **3000** (Next.js default).
- Host mapping **30xx** pro přímý/debug přístup (např. backoffice 3020, palety 3010).
- Reverse proxy (Caddy) jde **vždy na interní port (3000)**, NE na host mapping
  (Caddy je s appkami na síti `web`). Viz `decisions/005-proxy-interni-port.md`.

## Domény
- `modul.nuart.cz` (subdomény) — preferováno (viz `decisions/006-subdomeny-ne-path.md`).
- Cílově migrace Nicotrans modulů na `nicotrans.cz`.
- ⚠️ Výjimka v praxi: `nicotrans-palety` jede path-based pod `crm.nuart.cz/palety`
  (Next `basePath`) — viz `stacks/nicotrans-palety.md`.

## GitHub
- Kód aplikací: repo pod `Endyz/` (osobní) — Nicotrans-CRM, Geparim-Arrivals…
- Infra dokumentace (tohle repo): cílově org `NuArt`.
