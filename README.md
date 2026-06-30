# nuart-infra

Znalostní repozitář infrastruktury **NuArt Media s.r.o.** (Daniel Habrda, České Budějovice).

Dokumentuje domácí/firemní homelab: fyzický server, Proxmox hypervizor, Docker host
s produkčními aplikacemi, sdílenou infrastrukturu (Postgres/MinIO), reverse proxy (Caddy),
lokální AI (Ollama) a herní server (Windrose). Včetně rozhodnutí (ADR), incidentů a konvencí.

## Pro koho
Pro člověka i AI agenta, který má bezpečně pochopit a spravovat tuto infrastrukturu.
Začni souborem [CLAUDE.md](CLAUDE.md) — je to mapa celého repa.

## Struktura
| Složka | Obsah |
|---|---|
| `proxmox/` | Fyzický stroj + hypervizor (VM/LXC, síť, ISP, zálohy) |
| `docker-host/` | Docker VM 192.168.0.199 (konvence, mapa portů, sdílená infra) |
| `stacks/` | Jeden soubor na aplikační stack (caddy, backoffice, crm, palety, windrose…) |
| `ai/` | Lokální AI — Ollama, Nicotrans AI pipeline |
| `decisions/` | ADR — architektonická rozhodnutí a jejich důvody |
| `incidenty/` | Co se rozbilo a jak se to vyřešilo |
| `_archiv/` | Syrové původní dumpy a ověřovací report |

## Důležité
- **Žádné secrets v repu.** Hodnoty hesel/tokenů/klíčů žijí v `.env` na serveru
  (`/opt/stacks/<stack>/.env`), tady je jen jejich umístění. Šablony bez hodnot v `stacks/_vzory/`.
- Stav infrastruktury se mění — fakta označená `[OVĚŘIT]` ověř proti serveru, než se na ně spolehneš.

## Stav
Vzniklo a ověřeno **2026-06-30** z jediné velké znalostní extrakce. Viz
[_archiv/OVERENI-2026-06-30.md](_archiv/OVERENI-2026-06-30.md).
