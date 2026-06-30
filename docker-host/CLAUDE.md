# CLAUDE.md — docker-host/ (Docker VM 192.168.0.199)

> Úroveň aplikačního hostu. Tady běží produkční stacky + sdílená infra.
> Na rozdíl od `proxmox/` se tu reálně pracuje (deploy, build, restart kontejnerů),
> ALE vždy opatrně — běží tu produkce a sdílené služby.

## Kontext shora
Patříš do `nuart-infra` (kořenový CLAUDE.md). Sedíš na VM 100, která běží na Proxmoxu
(viz `proxmox/`). Detaily jednotlivých appek jsou v `stacks/`.

## Co je tady
- `prehled.md` — OS, Docker, síť web, Dockge, buildx, autoheal
- `konvence.md` — pojmenování, reverse-proxy pravidlo, struktura stacků
- `porty.md` — kompletní mapa portů (interní vs host)
- `infra/postgres.md`, `infra/minio.md` — sdílená infra

## Klíčová fakta (ověřeno 2026-06-30)
- **Host:** `docker` @ 192.168.0.199, Debian 12, `ssh nuart-docker` (endy) / `ssh root@...`
- **Docker:** 29.6.0, Compose v5.1.4. RAM VM ~20 GB.
- **Stacky:** `/opt/stacks/<nazev>/` (compose.yaml + .env). Dockge běží z `/opt/dockge` (:5001).
- **Síť:** `web` (external) — kontejnery se vidí jménem (např. `infra-postgres:5432`)
- **Sdílená infra:** infra-postgres (DB: nuart, nicotrans, crm_identity), infra-minio (9000/9001)
- **Reverse proxy:** Caddy (viz `stacks/caddy.md`) — routuje na INTERNÍ port (3000)

## Konvence (shrnutí, detail v konvence.md)
- Kontejnery: malá písmena, pomlčky. Prefix `infra-` = sdílená infra.
- DB uživatelé: dedikovaní per modul (nuart_user, nicotrans_user, crm_user) — ne sdílení.
- Appky: interní port 3000, host mapping 30xx pro přímý přístup.
- Caddy vždy na interní port, NE host mapping.

## Pravidla pro práci
- **qm/pct NEPOUŽÍVAT** tady (to je Proxmox, jiný stroj).
- **Caddyfile** smíš editovat + reload (`docker exec caddy caddy reload --config /etc/caddy/Caddyfile`),
  ALE změnu nejdřív ukázat člověku. NErestartovat celý caddy stack zbytečně.
- **Vystavovat přes Caddy jen služby s vlastním auth.**
- **Secrets:** `.env` v každém stacku, vždy v `.gitignore`. Nikdy necommitovat hodnoty.
- **pip:** používat `--break-system-packages`.
- **Build appek:** Next.js se buildí → změna kódu/configu vyžaduje `up -d --build`, ne restart.
  Buildy NuArt modulů jdou přes SDÍLENÝ builder `nuart-builder` (`--builder nuart-builder`,
  síť `web` + secrets `appenv`/`npmtoken`) — NEvytvářej per-projekt buildery.
  Kanonický příkaz a setup: `/opt/stacks/infra/README.md` + `setup-buildx.sh`.
- **Terminál mrší multiline paste** → posílat příkazy po jednom, když záleží na pořadí.

## Kde jsou secrets (umístění, NE hodnoty)
- DB hesla, SESSION_SECRET, AUTH secrety → `.env` v příslušném stacku
- RCON heslo Windrose+ → `server-files/windrose_plus_data/` (viz `stacks/windrose.md`)
- GitHub přístup → SSH klíč `~/.ssh/id_ed25519` (registrovaný jako „NuArt Docker Server")

## Co ověřit (drift)
- Host port mappingy (`docker ps`), seznam kontejnerů (`docker ps -a`)
- Caddyfile obsah (jaké domény) — viz `stacks/caddy.md`
- DB seznam na infra-postgres (`docker exec infra-postgres psql -U postgres -c "\l"`)
