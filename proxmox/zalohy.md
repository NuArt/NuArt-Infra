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
| Postgres | `pg_dump --clean --if-exists` DB **nuart**, **nicotrans** | ⚠️ viz díra níže |
| MinIO | `cp -a` z volume `infra_miniodata` (`/var/lib/docker/volumes/infra_miniodata/_data`) | data malá (~752 KB) |
| Konfigy | `compose.yaml` + `.env` všech stacků + `/opt/motd` + sám skript | kód je na GitHubu, nezálohuje se |

Každý běh má `MANIFEST.txt` (co je uvnitř). Ověřeno: poslední běh `run-20260630-030002`
obsahuje `postgres/{nuart.sql, nicotrans.sql}`, `minio/`, `konfigy/stacks/*`.

### ⚠️ Díra (TODO)
- **`crm_identity` (DB CRM Core) se NEzálohuje** — `PG_DBS="nuart nicotrans"`, CRM chybí.
  CRM je živá produkce → doplnit `crm_identity` do `PG_DBS` v `/opt/nuart-backup.sh`.
  `[TODO: přidat crm_identity do zálohy]`

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

## 📋 Plán rozšíření zálohy (TODO — dokončit po projití všeho)
Klíčové věci k doplnění do `/opt/nuart-backup.sh` (vše malé, vejde se do 5 GB):
- [ ] **`crm_identity`** do `PG_DBS` (CRM Core DB — teď úplně mimo zálohu) — **priorita 1**
- [ ] **Caddyfile** (`/opt/stacks/caddy/Caddyfile`) — **NEzálohuje se** (skript bere jen
      `compose.yaml` + `.env` + `*.env`, ne `Caddyfile`). Routing/TLS config mimo zálohu — **priorita 1**
- [ ] (zvážit) `infra/initdb/` (init SQL pro Postgres) — taky se nebere
- [ ] (zvážit) Windrose `windrose_plus.json` / server config, pokud je důležitý
- [ ] `.env` nových stacků se berou OK (`find -maxdepth 2` je pokrývá)
> Po projití zbytku TODO sem doplnit finální seznam a případně upravit skript.

## Ostatní
- Windrose svět: auto-save každých 60 s (herní mechanika) + konfig je v zálohovaných stacích.
- Git: kód aplikací na GitHubu.
