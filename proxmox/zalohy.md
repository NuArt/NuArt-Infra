# Zálohy

> Ověřeno 2026-06-30 (skript `/opt/nuart-backup.sh`, systemd timer, běhy v `/opt/backups/`).

## Off-site záloha dat (Docker VM → Wedos) — FUNGUJE ✅
`[opraveno proti realitě 2026-06-30: dump tvrdil „DB zálohy pravděpodobně chybí — RIZIKO".
 Realita: běží plnohodnotná denní off-site záloha, o které dump nevěděl.]`

- **Skript:** `/opt/nuart-backup.sh` (na Docker VM)
- **Spouštění:** systemd `nuart-backup.timer` → `nuart-backup.service`, **denně ~03:00**
- **Log:** `/var/log/nuart-backup.log`
- **Cíl:** off-site rsync daemon na **Wedos** (`rsync://<účet>@<...>.s54.wedos.net/<účet>/backups/`, port 873, IPv4)
- **Heslo:** `/root/.wedos-pass` (chmod 600) — NE v repu
- **Retence:** 3 běhy lokálně (`/opt/backups/run-*`), 7 na Wedosu

### Co se zálohuje
| Část | Jak | Pozn. |
|---|---|---|
| Postgres | `pg_dump --clean --if-exists` DB **nuart**, **nicotrans**, **crm_identity** | ✅ crm doplněno 2026-06-30 |
| MinIO | `cp -a` z volume `infra_miniodata` (`/var/lib/docker/volumes/infra_miniodata/_data`) | data malá (~752 KB) |
| Konfigy | `compose.yaml` + `.env` všech stacků + **`Caddyfile`** + `/opt/motd` + sám skript | ✅ Caddyfile doplněn 2026-06-30 |

Každý běh má `MANIFEST.txt` (co je uvnitř).

### ✅ Vyřešeno 2026-06-30 (rozšíření skriptu)
- `PG_DBS` rozšířeno na `nuart nicotrans crm_identity` — CRM DB se nyní zálohuje.
- `find` v sekci konfigy doplněn o `-name "Caddyfile"` — routing config se nyní zálohuje.
- Ověřeno: `pg_dump crm_identity` projde (24K), `find` bere `./caddy/Caddyfile`.
- Záloha originálu skriptu: `/opt/nuart-backup.sh.bak-20260630`.
- Změny se projeví v nejbližším běhu (`nuart-backup.timer`, ~03:00), nebo lze spustit ručně
  `sudo /opt/nuart-backup.sh`.

## Proxmox VM/LXC zálohy (vzdump) — NEEXISTUJÍ ⚠️
Ověřeno na `pve` 2026-06-30: `/etc/pve/jobs.cfg` prázdné (žádné backup joby),
`/var/lib/vz/dump/` prázdné. Storage `local` sice umí `content backup`, ale nic se nezálohuje.
- **Celé image VM 100 / LXC 101 / 102 nejsou nijak zálohované.** Při ztrátě disku by se
  obnovovalo ručně z dat+konfigů (Wedos) + GitHubu. `[TODO: zvážit vzdump job, ale pozor na místo
  — root `/` na pve má jen 94 GB, využito ~13 %]`

## ⚠️ Stav skriptu nuart-backup.sh a Wedos limit
- **Wedos úložiště je limitované na 5 GB** celkem. Aktuální běh je drobný (nuart.sql ~874 KB,
  nicotrans.sql ~55 KB, MinIO ~752 KB) → do 5 GB se klíčové věci v pohodě vejdou.
- Skript je **starší než část současného stavu** — pravděpodobně nikdo nedoplnil nové věci
  (proto chybí `crm_identity`). Plán rozšíření (v rámci 5 GB) níže.

## 📋 Zbývající náměty (nice-to-have, v rámci 5 GB)
- [x] ~~`crm_identity` do `PG_DBS`~~ ✅ hotovo 2026-06-30
- [x] ~~`Caddyfile` do zálohy konfigů~~ ✅ hotovo 2026-06-30
- [ ] (zvážit) `infra/initdb/` (init SQL pro Postgres) — zatím se nebere
- [ ] (zvážit) Windrose `windrose_plus.json` / server config, pokud je důležitý
- [ ] (zvážit) Proxmox `vzdump` job pro celé image VM/LXC (pozor na místo na `/`)

## Ostatní
- Windrose svět: auto-save každých 60 s (herní mechanika) + konfig je v zálohovaných stacích.
- Git: kód aplikací na GitHubu.
