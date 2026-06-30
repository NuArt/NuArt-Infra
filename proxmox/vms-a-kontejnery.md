# VM a kontejnery (Proxmox)

> Ověřeno přímo na `pve` 2026-06-30 (`qm config 100`, `pct config 101 102`).

| ID | Typ | Název (hostname) | IP | Role | Zdroje |
|---|---|---|---|---|---|
| 100 | VM | docker | 192.168.0.199 | Docker host (aplikace + sdílená infra) | 4 jádra, 20 GB RAM (20480), disk 300 GB (scsi0, local-lvm) |
| 101 | LXC | ollama | 192.168.0.154 (DHCP) | Lokální AI | 4 jádra, 8 GB RAM (8192), GPU passthrough |
| 102 | LXC | uptimekuma | 192.168.0.158 (DHCP) | Uptime Kuma monitoring | 1 jádro, 1 GB RAM (1024) |

- **VM 100** ✅ cores 4, memory 20480 (20 GB), scsi0 300G (local-lvm, discard, ssd), bridge vmbr0,
  net0 virtio (MAC 02:30:8F:A9:E3:A1), nameserver 1.1.1.1 / 8.8.8.8.
  `[opraveno proti realitě 2026-06-30: disk je 300 GB (ne ~296)]`
- **LXC 101 (ollama)** ✅ cores 4, memory 8192 (8 GB), `features: nesting=1,mknod=1`.
  - **GPU passthrough** ověřen: dev0–dev3 = `/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-uvm`,
    `/dev/nvidia-uvm-tools` (gid=44).
  - ⚠️ **net0 `ip=dhcp`** — statická IP STÁLE NENÍ. `[OVĚŘIT/TODO trvá: restart LXC změní IP a rozbije
    Open WebUI + Nicotrans AI pipeline — viz `incidenty/ollama-ip-dhcp.md`]`
- **LXC 102 (uptimekuma)** ✅ cores 1, memory 1024 (1 GB), net0 `ip=dhcp`.
  `[opraveno proti realitě 2026-06-30: hostname je `uptimekuma` (ne „Kuma"); ověřena RAM 1 GB]`

## CPU přidělení
Fyzicky 12 threadů. Rozdáno: Docker 4 + Ollama 4 + Kuma 1 = 9 vCPU (overcommit OK,
reálný load hostitele velmi nízký). vCPU nejsou napevno, sdílejí se.

## Monitoring (Uptime Kuma na LXC 102)
> Ověřeno z UI 2026-06-30 (http://192.168.0.158:3001). 4 monitory, interval 60 s, všechny „Běží".

| # | Monitor | Typ | Cíl | Uptime (1 r) |
|---|---|---|---|---|
| 1 | Docker VM | Ping | 192.168.0.199 | 99.93 % |
| 2 | Proxmox host | Ping | 192.168.0.186 | 100 % |
| 3 | NuArt Backoffice | HTTP | http://192.168.0.199:3020/login | 99.93 % |
| 4 | Nicotrans palety | HTTP | http://192.168.0.199:3010/ | 99.92 % |

`[opraveno proti realitě 2026-06-30: dump měl u Kumy „[OVĚŘIT co monitoruje]" — výše je realita]`

### ⚠️ Díry v monitoringu (TODO — zvážit doplnění)
- **nicotrans-crm / crm.nuart.cz se NEmonitoruje** (živá produkce bez dohledu).
- Monitory míří na **interní IP:porty, ne na veřejné domény přes Caddy** → nehlídá se
  veřejná dostupnost, **TLS certifikát** ani samotný Caddy (office.nuart.cz / crm.nuart.cz zvenku).
- Nemonitoruje se **Ollama** (192.168.0.154:11434), **infra-postgres**, **infra-minio**,
  **Windrose**, **Dockge**.
- U monitoru #4 (palety) UI ukazuje „Expirace Cert. 2026-09-27 (89 dní)", ač cíl je `http://…:3010`
  (ne HTTPS) — `[OVĚŘIT konfiguraci monitoru]`.
- Notifikace (e-mail/telegram apod.) — `[OVĚŘIT: jsou nastavené?]`
