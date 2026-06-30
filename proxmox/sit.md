# Síť, routing, ISP

## ISP a veřejná IP
- **Veřejná IP:** 178.255.174.235 (ISP Starnet, 1:1 NAT) — ✅ ověřeno 2026-06-30
- **Gateway:** UniFi UCG-Fiber (.1)
- **WAN IP na gatewayi:** 10.6.123.174 (vnitřní WAN za ISP NAT → **double-NAT**)
- ⚠️ Double-NAT historicky blokoval OpenVPN setup. `[OVĚŘIT: vyřešeno?]`

## Port forwardy (UCG-Fiber)
Ověřeno (dle dumpu), nepřímo potvrzeno tím, že veřejné HTTPS funguje:
- `caddy-http`  : 80  → 192.168.0.199:80
- `caddy-https` : 443 → 192.168.0.199:443

Caddy = jediný veřejný vstup. HTTP-01 ACME challenge potřebuje port 80 → forward 80 je nutný.

## LAN
- Rozsah: 192.168.0.0/24 (NuArt-Staff VLAN)
- `[OVĚŘIT: plánovaný subnet renumbering 192.168.0.0/24 → 10.44.10.0/24]`

## DNS
- Doména `nuart.cz`, registrátor/správa **Wedos**. ⚠️ Živé MX záznamy — **nerozbít**.
- A-záznamy → 178.255.174.235. Ověřeno 2026-06-30 (`getent hosts`):
  - `office.nuart.cz` → 178.255.174.235 ✅
  - `crm.nuart.cz` → 178.255.174.235 ✅
  - `nico-palety.nuart.cz` → 178.255.174.235 ✅ existuje
    `[opraveno proti realitě 2026-06-30: dump měl nico-palety jen jako plán; A-záznam UŽ existuje,
    ale Caddy ho neobsluhuje — palety jedou pod crm.nuart.cz/palety, viz stacks/nicotrans-palety.md]`
- Mapování domén → služby viz `stacks/caddy.md` a `docker-host/porty.md`.
