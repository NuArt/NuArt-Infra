# Incident: Windrose se sekal při buildu/deployi nicotransu (2026-07-01)

## Co se stalo
2026-07-01 cca **18:55–19:20 CEST** hráči hlásili sekání (cukání) herního serveru Windrose.
Ve stejném okně probíhal **build + deploy nicotrans stacku** (crm, palety, nabor, hr) na stejném
Docker hostu. Windrose se **NEzhroutil** — soupeření o zdroje způsobilo přechodné záseky herní smyčky.

## Důkazy (čas serveru je UTC; okno 18:55–19:20 CEST = 16:55–17:20 UTC)
- `windrose`: `restarts=0`, `OOMKilled=false`, běžel v kuse 3 dny → **žádný crash ani OOM**, jen stall.
- Žádný OOM v `dmesg`, žádné WARN v system journalu v okně.
- Časy image nicotrans modulů (kdy se dobuildily) — **sériově během okna**:
  - `nicotrans-crm` 19:01:30, `nicotrans-palety` 19:03:44, `nicotrans-nabor` 19:06:41, `nicotrans-hr` 19:08:53 CEST
- Všechny 4 kontejnery **znovuvytvořeny naráz 19:10:16–19:10:21 CEST** (`up -d` = finální CPU špička).
- Sdílený builder `nuart-builder` byl v tu dobu **bez jakýchkoli limitů** (cpuset prázdný, mem 0)
  a odvedl obří I/O (`BlockIO ~21 GB read / ~39 GB write`).
- Host v tu dobu: **4 jádra**, 20 GB RAM VM, **swap 0 B**, volné jen ~430 MB.

## Proč to Windrose odnesl, i když měl limity (`cpus: 2.0`, `mem_limit: 10g`)
Limity chrání jen CPU kvótu a RAM strop, **NE** to, co reálně sekalo hru:
1. **CPU scheduling / paměťová propustnost + L3 cache** (nejpravděpodobnější hlavní příčina) —
   paralelní Next.js build (webpack/TS) sytil zbývající jádra; UE dedikovaný server je extrémně
   citlivý na scheduling latency → tik zadrhne i uvnitř své kvóty. Disk je SSD (`sda` ROTA=0,
   scheduler `none`), takže čistá disková propustnost je **nejméně pravděpodobný** hlavní viník.
2. **RAM tlak jako zesilovač** — 0 swap + host skoro plný → jádro vyhazovalo page cache včetně
   mmapovaných souborů světa Windrose → nutné re-čtení z disku.
3. **Finále 19:10** — start 4 Node containerů současně = synchronní CPU špička (ocas incidentu).

> Poznámka: nemáme historická per-resource metrika (sysstat/sar není nainstalován, PSI countery
> jsou kumulativní od bootu), takže řazení příčin je **kvalifikovaný odhad z topologie a symptomu**,
> ne změřený graf. Symptom (cukání, ne trvalý propad FPS) odhad podporuje.

## Fix (řešeno hned týž den)
Zvolena „Levná" varianta: izolovat jen Windrose, zastropovat builder, produkci nechat plovoucí.
1. **Proxmox: VM 100 `cores` 4 → 6** (`qm set 100 --cores 6` + `qm reboot 100`). Fyzicky 12 threadů,
   po bumpu rozdáno 11 → OK. Teprve 6 jader umožní čisté rozdělení hra / build+deploy / produkce.
2. **Windrose pin na jádra `0-1`** — `docker update --cpuset-cpus 0-1 windrose` (za běhu, bez výkopu
   hráčů) + trvale `cpuset: "0-1"` v `compose.yaml`.
3. **Builder zastropován** — `nuart-builder` znovuvytvořen s `cpuset-cpus=2-5`, `cpu-quota=300000`
   / `cpu-period=100000` (strop 3 jádra), `memory=6g`. Trvale zapsáno do `setup-buildx.sh`.
   (POZOR na `--driver-opt`: čárka je oddělovač k=v → cpuset psát rozsahem `2-5`, ne `2,3,4,5`.)
4. Produkce (nicotrans, caddy, postgres, minio…) zůstává plovoucí — je skoro pořád ~0 %, na
   6-jádrovém stroji má vždy vzduch (builder max 3 ze 4 sdílených jader).

`memory=6g` na builderu drží RAM osu → **swap není potřeba** (a na herním hostu ho ani nechceme).

## Prevence / co si odnést
- Sdílený builder už nikdy nemůže vyhladovět živou hru (izolovaný na jiná jádra + strop).
- Provozně: velké buildy/deploye nicotransu ideálně mimo večerní hraní.
- Kdyby se přece jen ukázal disk, cgroup v2 tu je (`io` controller) → lze doplnit `io.weight` pro Windrose.

Souvisí: `decisions/009-cpu-izolace-cpuset.md`, `decisions/008-windrose-limity.md`,
`incidenty/windrose-ram.md`, `stacks/windrose.md`, `proxmox/vms-a-kontejnery.md`.
