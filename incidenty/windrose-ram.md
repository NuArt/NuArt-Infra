# Incident: Windrose RAM na hraně (98 %) s Windrose+

## Co se stalo
S modem Windrose+ (overhead + lazy loading světa) šplhala RAM kontejneru na ~98 %.

## Fix
Bump `mem_limit` 8 → 10 GB (a postupné navýšení RAM celé Docker VM až na 20 GB).

## Chování
RAM roste s explorací světa (lazy loading), ustálí se na plató ~7,2 GB / 10 GB.
CPU špičky 150–200 % (limit 200 %), load hostitele nízký.

Souvisí: `decisions/008-windrose-limity.md`, `stacks/windrose.md`.
