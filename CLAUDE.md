# CLAUDE.md — nuart-infra

> Znalostní repozitář infrastruktury NuArt Media. Tohle je **mapa** — řekne ti,
> co je kde a jak repo číst. Detaily jsou v jednotlivých souborech.

## Co to je
Kompletní kontext domácího/firemního serveru NuArt: hardware, Proxmox, Docker host,
aplikační stacky, síť, rozhodnutí a jejich důvody, incidenty a poučení. Cílem je, aby
kdokoliv (člověk i AI agent) měl po přečtení úplný obraz a mohl bezpečně zasahovat.

## Úrovně (čti shora dolů podle potřeby)
1. **proxmox/** — fyzický stroj + hypervizor. VM a kontejnery, síť, ISP, zálohy.
2. **docker-host/** — Docker VM (10.3.20.199), sdílená infra, konvence, mapa portů.
3. **stacks/** — jednotlivé aplikace (jeden soubor na stack).
4. **ai/** — lokální AI (Ollama, Nicotrans AI pipeline).
5. **decisions/** — ADR: PROČ jsou věci tak, jak jsou.
6. **incidenty/** — co se rozbilo a jak se to vyřešilo (ať se chyby neopakují).
7. **_archiv/** — syrové původní dumpy.

## Jak repo číst podle situace
- **„Co kde běží?"** → `docker-host/porty.md` + `stacks/`
- **„Proč je to tak?"** → `decisions/`
- **„Něco se rozbilo, už to bylo?"** → `incidenty/`
- **„Jak se připojit / kde jsou secrets?"** → `docker-host/CLAUDE.md` (umístění, NE hodnoty)
- **„Síť / domény / porty"** → `proxmox/sit.md` + `docker-host/porty.md` + `stacks/caddy.md`

## Klíčová fakta (rychlý přehled, ověřeno 2026-06-30; IP dorovnány 2026-07-22)
- **Síť:** LAN **10.3.20.0/24**, GW/DNS `10.3.20.1` `[2026-07-22: renumbering z 192.168.0.0/24
  → 10.3.20.0/24, poslední oktety zachovány. Detaily a důvod viz `proxmox/sit.md`.]`
- **Proxmox:** `pve` @ 10.3.20.186:8006
- **Docker host:** `docker` @ 10.3.20.199 (`ssh nuart-docker`), stacky v `/opt/stacks/`
- **Reverse proxy:** Caddy, automatické HTTPS, routuje na INTERNÍ port kontejneru (3000)
- **Veřejná IP:** 178.255.174.235 (Starnet), DNS na Wedos, domény *.nuart.cz
- **Sdílená infra:** infra-postgres, infra-minio (síť `web`)

## Pravidla pro práci s tímto repem
- **Secrets:** NIKDY sem nepiš hodnoty hesel/tokenů/klíčů. Jen kde žijí.
- **Drift:** stav se mění. Když konáš na serveru, ověř realitu (`docker ps`, `cat`)
  PŘED tím, než budeš věřit tomu, co je tu napsané. Co ověříš a opravíš, commitni.
- **Konvence:** drž stávající (viz `docker-host/konvence.md`) — nová appka = stejný vzor.
- **NESAHAT bez rozmyslu:** infra-postgres, caddy, jiné běžící stacky, DNS, MX (živé).

## Stav dokumentace
Tohle repo vzniklo z jedné velké znalostní extrakce (viz `_archiv/knowledge-dump-2026-06-30.md`)
a bylo **ověřeno proti realitě serveru 2026-06-30**. Místa opravená proti dumpu nesou
poznámku `[opraveno proti realitě 2026-06-30: bylo X, je Y]`. Co nešlo ověřit (hlavně
Proxmox úroveň — z Docker VM není SSH přístup na `pve`) zůstává označeno `[OVĚŘIT]`
s důvodem. Přehled viz `_archiv/OVERENI-2026-06-30.md`.
