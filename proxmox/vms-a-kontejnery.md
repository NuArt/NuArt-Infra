# VM a kontejnery (Proxmox)

> Ověřeno přímo na `pve` 2026-06-30 (`qm config 100`, `pct config 101 102`).

| ID | Typ | Název (hostname) | IP | Role | Zdroje |
|---|---|---|---|---|---|
| 100 | VM | docker | 192.168.0.199 | Docker host (aplikace + sdílená infra) | **6 jader**, 20 GB RAM (20480), disk 300 GB (scsi0, local-lvm) |
| 101 | LXC | ollama | 192.168.0.154 (DHCP) | Lokální AI | 4 jádra, 8 GB RAM (8192), GPU passthrough |
| 102 | LXC | uptimekuma | 192.168.0.158 (DHCP) | Uptime Kuma monitoring | 1 jádro, 1 GB RAM (1024) |

- **VM 100** ✅ cores **6**, memory 20480 (20 GB), scsi0 300G (local-lvm, discard, ssd), bridge vmbr0,
  net0 virtio (MAC 02:30:8F:A9:E3:A1), nameserver 1.1.1.1 / 8.8.8.8.
  `[opraveno proti realitě 2026-06-30: disk je 300 GB (ne ~296)]`
  `[2026-07-01: cores 4 → 6 kvůli CPU izolaci Windrose vs. build — viz decisions/009 a incident I/O contention]`
- **LXC 101 (ollama)** ✅ cores 4, memory 8192 (8 GB), `features: nesting=1,mknod=1`.
  - **GPU passthrough** ověřen: dev0–dev3 = `/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-uvm`,
    `/dev/nvidia-uvm-tools` (gid=44).
  - **net0 `ip=dhcp`**, ale ✅ **IP `.154` zamčena DHCP rezervací na UniFi** (2026-06-30,
    MAC `BC:24:11:46:DE:91`) → IP se už nebude měnit. Viz `incidenty/ollama-ip-dhcp.md`.
- **LXC 102 (uptimekuma)** ✅ cores 1, memory 1024 (1 GB), net0 `ip=dhcp`.
  `[opraveno proti realitě 2026-06-30: hostname je `uptimekuma` (ne „Kuma"); ověřena RAM 1 GB]`

## CPU přidělení
Fyzicky 12 threadů. Rozdáno: Docker **6** + Ollama 4 + Kuma 1 = **11 vCPU** (overcommit OK,
reálný load hostitele velmi nízký). vCPU nejsou napevno, sdílejí se.
`[2026-07-01: Docker 4 → 6 vCPU. Uvnitř VM je Windrose pinnutý na jádra 0-1 a builder na 2-5
 (cpuset izolace) — viz decisions/009-cpu-izolace-cpuset.md.]`

## Monitoring (Uptime Kuma na LXC 102)
> Ověřeno z UI 2026-06-30 (http://192.168.0.158:3001). **7 monitorů**, interval 60 s, všechny „Běží".

| # | Monitor | Typ | Cíl | Pozn. |
|---|---|---|---|---|
| 1 | Docker VM | Ping | 192.168.0.199 | |
| 2 | Proxmox host | Ping | 192.168.0.186 | |
| 3 | NuArt Backoffice (Interní IP) | HTTP | http://192.168.0.199:3020/login | |
| 4 | NuArt Backoffice (Doména) | HTTP(s) | https://office.nuart.cz | ➕ 2026-06-30 (Caddy + TLS) |
| 5 | Nicotrans CRM (crm.nuart.cz) | HTTP(s) | https://crm.nuart.cz | ➕ 2026-06-30 (Caddy + TLS; pokrývá i /palety) |
| 6 | Nicotrans palety | HTTP | http://192.168.0.199:3010/ | interní check modulu |
| 7 | Ollama (LXC 101) | HTTP keyword „Ollama" | http://192.168.0.154:11434 | ➕ 2026-06-30 (chytí i změnu DHCP IP) |

`[opraveno proti realitě 2026-06-30: dump měl „[OVĚŘIT co monitoruje]". Nejdřív byly 4 monitory
 (interní), poté doplněny #4/#5/#7 — CRM, veřejné domény přes Caddy (TLS) a Ollama.]`

### Zbývající mezery (volitelné)
- **infra-postgres (5432)** nelze monitorovat z Kumy — nemá host mapping, je dosažitelný jen
  uvnitř docker sítě (a ven ho nechceme vystavovat).
- **infra-minio / Windrose / Dockge** — nemonitorovány (interní, nízká priorita).
- Notifikace (e-mail/telegram apod.) — `[OVĚŘIT: jsou nastavené?]`
