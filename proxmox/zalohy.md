# Zálohy

> Ověřeno 2026-06-30, rozšířeno 2026-07-02 (skript `/opt/nuart-backup.sh`, systemd timer, běhy v `/opt/backups/`).

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
| Postgres | `pg_dump --clean --if-exists` **všech DB dynamicky** (dotaz na `pg_database`, kromě šablon a `postgres`) | ✅ 2026-07-02: dřív pevný seznam → nové DB (nicotrans_hr, crm_nabor) se nezálohovaly |
| MinIO | `cp -a` z volume `infra_miniodata` (`/var/lib/docker/volumes/infra_miniodata/_data`) | data malá (~900 KB) |
| **Herní světy** | 1 komprimovaný tar `hry/game-saves-*.tar.gz` (**každý běh**) | ✅ 2026-07-02, viz níže |
| Konfigy | `compose.yaml` + `.env` všech stacků + **`Caddyfile`** + `/opt/motd` + sám skript | ✅ Caddyfile doplněn 2026-06-30 |

Každý běh má `MANIFEST.txt` (co je uvnitř).

**Herní světy — jen save/mod data, NE herní instalace.** Windrose `server-files` má 3,4 GB,
ale 2,7 GB z toho jsou stažitelné herní soubory (`R5/Content`, `Binaries`, engine) — ty **NEbereme**
(jako kód na GitHubu; znovu se stáhnou přes Steam). Do tarru jde jen perzistentní stav:
- **Windrose:** `R5/Saved` (save profily), `R5/Plugins`, `windrose_plus` + `windrose_plus_data`
  (mapy, heightmapy, POI, rcon, terrain) — logy vynechány.
- **Valheim:** celý `config` (světy `worlds_local`, prefs, listy).

Výsledek ~193 MB/běh → 7 běhů na Wedosu ≈ 1,35 GB (vejde se do 5GB limitu; **celé 3,4 GB by 7× ≈ 24 GB
limit prorazilo** — proto lean varianta). Tar běží za chodu windrose → GNU tar může vrátit rc=1
(soubor se změnil za běhu), skript to bere jako OK (archiv je platný); rc=2 = skutečná chyba.

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

### ✅ Vyřešeno 2026-07-02 (audit — rozšíření skriptu)
- **Dynamický seznam DB** — `PG_DBS` se získává dotazem na `pg_database` místo pevného seznamu.
  Ověřeno testovacím během: zálohují se všechny DB (crm_identity, crm_nabor, nicotrans,
  nicotrans_hr, nuart), rsync na Wedos exit 0.
- **Herní světy** — nová sekce 2b (windrose save + valheim config jako tar, ~193 MB/běh).
  Ověřeno: archiv validní, obsahuje R5/Saved + windrose_plus_data + valheim/config.
- Změny se projeví každý běh (`nuart-backup.timer`, ~03:00), nebo ručně
  `sudo systemctl start nuart-backup.service`.

## 📋 Zbývající náměty (nice-to-have, v rámci 5 GB)
- [x] ~~`crm_identity` do `PG_DBS`~~ ✅ hotovo 2026-06-30
- [x] ~~`Caddyfile` do zálohy konfigů~~ ✅ hotovo 2026-06-30
- [x] ~~Dynamický seznam DB (pokryje i nové moduly)~~ ✅ hotovo 2026-07-02
- [x] ~~Windrose/Valheim herní světy~~ ✅ hotovo 2026-07-02
- [ ] (zvážit) `infra/initdb/` (init SQL pro Postgres) — zatím se nebere
- [ ] (zvážit) napojení na Uptime Kuma (push monitor) — ať tichý pád zálohy nezůstane bez alertu
- [ ] (zvážit) Proxmox `vzdump` job pro celé image VM/LXC (pozor na místo na `/`)

## Ostatní
- Windrose svět: auto-save každých 60 s (herní mechanika); od 2026-07-02 se save data
  navíc zálohují off-site (viz tabulka „Co se zálohuje").
- Git: kód aplikací na GitHubu.
