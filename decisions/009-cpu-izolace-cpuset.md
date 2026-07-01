# ADR-009: CPU izolace přes cpuset — 6 vCPU, Windrose pinned, builder zastropován

**Kontext:** 2026-07-01 se Windrose sekal, když na stejném hostu běžel build+deploy nicotransu
(viz `incidenty/windrose-io-contention-2026-07-01.md`). `cpus`/`mem_limit` nechrání před scheduling
latencí ani I/O — na 4-jádrovém hostu si hra a build lezly do zelí.

**Rozhodnutí:**
1. **Docker VM (100) má 6 vCPU** (bylo 4). Fyzicky 12 threadů; rozdáno Docker 6 + Ollama 4 + Kuma 1 = 11.
2. **Windrose pinned na jádra `0-1`** (`cpuset: "0-1"` v `compose.yaml`) — izolovaný, 2 jádra jen pro sebe.
   Naměřená zátěž je bursty: klid ~0,5–1 jádro, špička ~2 jádra (150–205 %). **1 jádro nestačí** —
   uškrtilo by hru přesně ve špičce (boj / víc hráčů).
3. **Builder `nuart-builder` zastropován** (`setup-buildx.sh`): `cpuset-cpus=2-5`, strop 3 jádra
   (`cpu-quota=300000`/`cpu-period=100000`), `memory=6g`. Nikdy se nedotkne jader `0-1`.
4. **Produkce plovoucí** (nedotčená) — „Levná" varianta. Nicotrans/caddy/postgres/minio jsou skoro
   pořád ~0 %, na 6 jádrech mají vždy vzduch (builder max 3 ze 4 sdílených jader).

**Proč cpuset a ne jen vyšší `cpus`:** cpuset dělá izolaci deterministickou (hra a build se fyzicky
nepotkají na jednom jádře), ne jen pravděpodobnostní. Použit **překrývající se** cpuset: exkluzivní
je jen Windrose (`0-1`); builder + produkce sdílejí `2-5`, takže když builder nejede, jeho výkon
volně bere produkce → nic neleží ladem.

**Proč „Levná" (produkce nepinnutá):** produkce je téměř pořád idle, takže její plovoucí na `0-1`
Windrose reálně skoro neruší; ušetří se `cpuset:` řádek v každém prod stacku. Zbylé riziko: deploy
špička (start N containerů naráz) může na vteřiny sáhnout i na `0-1`. Kdyby vadilo → připnout produkci
do `2-5` (airtight varianta).

**Proč `memory=6g` na builderu:** host má 0 swap; strop brání tomu, aby build tlačil host do
vyhazování page cache (příčina č. 2 incidentu). Tím pádem **swap netřeba** (a na herním hostu ho ani
nechceme kvůli latenci).

**Pozor (past):** `docker buildx --driver-opt` bere čárku jako oddělovač `k=v` → cpuset se MUSÍ psát
rozsahem (`2-5`), ne výčtem (`2,3,4,5`), jinak `ERROR: invalid value "3", expecting k=v`.

**Rollback:** `qm set 100 --cores 4` + reboot; builder zpět na `cpuset-cpus=2-3`/`cpu-quota=150000`;
`docker update --cpuset-cpus 0-5 windrose` + odebrat `cpuset` z compose.

Viz `incidenty/windrose-io-contention-2026-07-01.md`, `decisions/008-windrose-limity.md`,
`stacks/windrose.md`, `proxmox/vms-a-kontejnery.md`, `/opt/stacks/infra/setup-buildx.sh`.
