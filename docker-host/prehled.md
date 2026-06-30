# Docker host — přehled

> Ověřeno na hostu 2026-06-30 (`hostname`, `docker --version`, `docker ps -a`, `free -h`).

## Základ
- **OS:** Debian 12. Hostname `docker`, IP 192.168.0.199
- **Docker:** 29.6.0, Compose v5.1.4 ✅
- **RAM:** ~20 GB (navyšováno 8→10→20 kvůli Windrose) ✅ (`free -h` → 19Gi total)
- **Přístup:** `ssh nuart-docker` (alias, endy@docker) nebo `ssh root@192.168.0.199`
- **pip:** není defaultně — doinstalován `python3-pip`; balíky s `--break-system-packages`

## Docker prostředí
- **Síť `web`** (external) — kontejnery se vidí jménem (definovaná ve stacku `infra`)
- **Dockge** — správa stacků (:5001). Běží z `/opt/dockge`, spravuje `/opt/stacks/`
  `[opraveno proti realitě 2026-06-30: dump řadil dockge mezi stacky; reálně to není
   složka v /opt/stacks/, ale samostatně v /opt/dockge]`
- **buildx (buildkit)** — pro buildy (`buildx_buildkit_nicotrans-builder0`)
- **autoheal** — restart nezdravých kontejnerů (healthcheck-based)
- **Konvence:** každý stack = složka v `/opt/stacks/`, vlastní `compose.yaml` + `.env`

## Běžící kontejnery (snapshot 2026-06-30)
| Kontejner | Stav | Pozn. |
|---|---|---|
| caddy | healthy | reverse proxy 80/443 |
| nuart-backoffice | healthy | 3020→3000 |
| nicotrans-crm | healthy | 3000 (bez host mappingu) |
| nicotrans-palety | healthy | 3010→3000 |
| windrose | healthy | network_mode host |
| infra-postgres | healthy | 5432 |
| infra-minio | healthy | 9000-9001 |
| openwebui | healthy | 3000→8080 |
| dockge | healthy | 5001 |
| autoheal | healthy | — |
| buildx_buildkit_nicotrans-builder0 | up | build helper |
| valheim | Exited (7 dní) | starý/odstavený herní stack `[OVĚŘIT: zrušit?]` |

`[opraveno proti realitě 2026-06-30: dump neuváděl kontejner `valheim` (odstavený)
 ani `buildx_buildkit_nicotrans-builder0`]`
