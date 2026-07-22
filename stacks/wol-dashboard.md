# Stack: wol-dashboard

> Ověřeno 2026-07-22 (`docker ps`, `docker-compose.yml`, `curl localhost:5050`).

## Co to je
Jednoduchý webový Wake-on-LAN dashboard (Flask) se stavovým přehledem počítačů
v LAN — ovládaný z mobilu přes prohlížeč. Buzení přes magic packet + ping status.

- **Kontejner:** `wol-dashboard`, port **5050**
- **Síť:** `network_mode: host` — NE bridge (potřebuje UDP broadcast + ping do LAN)
- **Stack:** `/opt/stacks/wol-dashboard/` (build lokálně, `python:3.11-slim` + Flask)
- **Vystavení:** jen LAN — **http://10.3.20.199:5050**, NENÍ za Caddy, NENÍ veřejně
- **Healthcheck:** žádný → v MOTD se hlásí šedě jako `up` (běží, ale nemonitorováno)
- **Restart:** `unless-stopped`

## Konfigurace
`config.json` — seznam počítačů (`name`, `ip`, `mac`, `broadcast`). Bez secrets.
Změna seznamu = editace `config.json`; čte se za běhu při každém requestu (netřeba rebuild,
stačí, aby soubor byl v kontejneru — je bind přes `COPY . .`, takže po editaci `docker compose up -d --build`).

## Poznámky
- **Cross-subnet WoL:** server je v `10.3.20.0/24`. Buzení do `10.3.20.255` funguje.
  Buzení „Doma PC" jde na broadcast `10.3.10.255` (jiná podsíť) — directed broadcast
  projde jen pokud to router/VLAN mezi 10.3.10 a 10.3.20 propouští (většina L3 to blokuje).
  Pokud „Doma PC" nejde vzbudit, je to tímto, ne aplikací.
- **Proč host network:** v bridge módu kontejner nevidí LAN broadcast doménu ani nepinguje
  LAN zařízení → WoL by nefungoval. Proto `network_mode: host`.
- Přibyl 2026-07-22.
- Nezálohuje se DB (žádná); stav je jen `config.json` v repu stacku.
