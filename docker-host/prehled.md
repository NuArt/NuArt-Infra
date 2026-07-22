# Docker host — přehled

> Ověřeno na hostu 2026-06-30 (`hostname`, `docker --version`, `docker ps -a`, `free -h`).

## Základ
- **OS:** Debian 12. Hostname `docker`, IP 10.3.20.199
- **Docker:** 29.6.0, Compose v5.1.4 ✅
- **RAM:** ~20 GB (navyšováno 8→10→20 kvůli Windrose) ✅ (`free -h` → 19Gi total)
- **Přístup:** `ssh nuart-docker` (alias, endy@docker) nebo `ssh root@10.3.20.199`
- **pip:** není defaultně — doinstalován `python3-pip`; balíky s `--break-system-packages`

## Docker prostředí
- **Síť `web`** (external) — kontejnery se vidí jménem (definovaná ve stacku `infra`)
- **Dockge** — správa stacků (:5001). Běží z `/opt/dockge`, spravuje `/opt/stacks/`
  `[opraveno proti realitě 2026-06-30: dump řadil dockge mezi stacky; reálně to není
   složka v /opt/stacks/, ale samostatně v /opt/dockge]`
- **buildx (buildkit)** — SDÍLENÝ perzistentní builder `nuart-builder` na síti `web`
  (kontejner `buildx_buildkit_nuart-builder0`). Vytváří/startuje ho idempotentní
  `/opt/stacks/infra/setup-buildx.sh` (boot přes systemd `nuart-buildx.service`).
  Moduly se buildí jediným skriptem `/opt/stacks/infra/build-module.sh <modul>`
  (sám doplní secrets). Žádné per-projekt buildery. Detaily v `/opt/stacks/infra/README.md`.
  `[2026-07-01: builder ZASTROPOVÁN — cpuset 2-5, strop 3 jádra, mem 6g — aby build nevyhladověl
   živý Windrose (jádra 0-1). Viz decisions/009 + incidenty/windrose-io-contention-2026-07-01.md.]`
  `[2026-07-02: build cache dostala GC STROP maxUsedSpace=15GB (buildkitd.toml přes --config
   v setup-buildx.sh) — dřív rostla bez limitu (audit našel ~35 GB). Buildkit maže nejstarší
   vrstvy nad strop. Jednorázově uvolněno ~22 GB (image prune + rekreace builderu).]`
- **autoheal** — restart nezdravých kontejnerů (healthcheck-based)
- **Konvence:** každý stack = složka v `/opt/stacks/`, vlastní `compose.yaml` + `.env`

## Běžící kontejnery (snapshot 2026-07-02)
| Kontejner | Stav | Pozn. |
|---|---|---|
| caddy | healthy | reverse proxy 80/443 |
| nuart-backoffice | healthy | 3020→3000 |
| nicotrans-crm | healthy | 3000 (bez host mappingu) |
| nicotrans-palety | healthy | 3010→3000, crm.nuart.cz/palety |
| nicotrans-hr | healthy | 3030→3000, crm.nuart.cz/hr ➕ 2026-07-01 |
| nicotrans-nabor | healthy | 3040→3000, crm.nuart.cz/nabor ➕ 2026-07-01 |
| windrose | healthy | network_mode host |
| infra-postgres | healthy | 5432 |
| infra-minio | healthy | 9000-9001 |
| openwebui | healthy | 3000→8080 |
| dockge | healthy | 5001 |
| autoheal | healthy | — |
| buildx_buildkit_nuart-builder0 | up | sdílený build helper (nuart-builder); v MOTD jako `nuart-builder` |
| valheim | Exited (záměrně) | herní stack, ZÁMĚRNĚ vyplý kvůli úspoře energie (jiná hra než windrose, hraje se jen jedna; startuje se on-demand). NEMAZAT. |

`[opraveno proti realitě 2026-06-30: dump neuváděl kontejner `valheim` (odstavený)
 ani buildx buildkit kontejner]`
`[2026-06-30: per-projekt buildery (nicotrans-builder/nuartbuilder) sjednoceny do
 jednoho sdíleného `nuart-builder` — viz /opt/stacks/infra/README.md]`
