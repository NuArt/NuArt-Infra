# ADR-008: Windrose limity — 2 jádra / 10 GB RAM

**Rozhodnutí:** Herní server Windrose má `cpus: 2.0` a `mem_limit: 10g`.

**Proč:** Izolace herní zátěže — produkce (NuArt/office) má vždy 2 jádra volná a chráněnou RAM
(po navýšení VM na 20 GB zůstává ~12 GB mimo Windrose). RAM 10 GB pokrývá plató po exploraci
(~7,2 GB), protože RAM roste s lazy loadingem světa.

> **Doplněno 2026-07-01 (viz ADR-009):** samotné `cpus`/`mem_limit` nechrání před scheduling
> latencí ani I/O — když na hostu běžel build nicotransu, Windrose se sekal. Řešení: VM bumpnuta
> 4→6 jader + Windrose pinnutý přes `cpuset: "0-1"`, builder izolovaný na 2-5 se stropem.
> Detaily: `decisions/009-cpu-izolace-cpuset.md`, `incidenty/windrose-io-contention-2026-07-01.md`.

Viz `stacks/windrose.md`, `incidenty/windrose-ram.md`, `decisions/009-cpu-izolace-cpuset.md`.
