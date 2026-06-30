# Stack: windrose (herní server)

> Ověřeno 2026-06-30 (`docker ps`, `grep` v `/opt/stacks/windrose/compose.yaml`, `.env` klíče).

## Co to je
Dedikovaný herní server Windrose + mod **Windrose+** (komunitní hraní). Izolovaná zátěž.

- **Kontejner:** `windrose`, `network_mode: host` ✅ (NAT punch-through pro připojení přes invite kód)
- **Limity:** `cpus: 2.0` (2 ze 4 jader — produkce má vždy 2 volné), `mem_limit: 10g` ✅
  (komentář v compose: „produkce má chráněných 12 GB"). Viz `decisions/008-windrose-limity.md`.
- **Stack:** `/opt/stacks/windrose/`
- **Image:** indifferentbroccoli (Steam app 4129620)

## Windrose+ mod
- Verze **v1.3.14** ✅ (UE4SS + PAK) — ověřeno `server-files/.windroseplus_version`
- Dashboard / RCON / live-map na `http://192.168.0.199:8780` (jen LAN)
- Config: **`server-files/windrose_plus.json`** (klíče `rcon` = RCON heslo, `multipliers`)
  `[opraveno proti realitě 2026-06-30: cesta je server-files/windrose_plus.json (ne
   server-files/windrose_plus_data/windrose_plus.json — windrose_plus_data je adresář)]`
- Default config šablony: `server-files/config/windrose_plus.*.default.ini` (food/entities/gear/harvest/weapons)
- Bell limit fast-travel: jde zvednout JEN modem, co musí mít server I každý hráč
  (Windrose validuje client-side, server-only varianta neexistuje). Mod #54 přes Mod Manager.

## Konfigurace / secrets (umístění)
`.env`: `/opt/stacks/windrose/.env` — klíče (bez hodnot):
`PUID`, `PGID`, `UPDATE_ON_START`, `INVITE_CODE`, `SERVER_NAME`, `SERVER_PASSWORD`,
`MAX_PLAYERS`, `P2P_PROXY_ADDRESS`, `GENERATE_SETTINGS`, `WINDROSE_PLUS_ENABLED`.

## Provozní poznámky
- RAM roste s explorací (lazy loading), ustálí se na plató (~7,2 GB / 10 GB). Viz `incidenty/windrose-ram.md`.
- CPU špičky 150–200 % (limit 200 %), load nízký.
- **Restart kvůli .env/compose vykopne hráče.** Auto-save světa každých 60 s.
