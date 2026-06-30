# ADR-008: Windrose limity — 2 jádra / 10 GB RAM

**Rozhodnutí:** Herní server Windrose má `cpus: 2.0` a `mem_limit: 10g`.

**Proč:** Izolace herní zátěže — produkce (NuArt/office) má vždy 2 jádra volná a chráněnou RAM
(po navýšení VM na 20 GB zůstává ~12 GB mimo Windrose). RAM 10 GB pokrývá plató po exploraci
(~7,2 GB), protože RAM roste s lazy loadingem světa.

Viz `stacks/windrose.md`, `incidenty/windrose-ram.md`.
