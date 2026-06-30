# VM a kontejnery (Proxmox)

> Zdroje VM/LXC **neověřeny 2026-06-30** (z Docker VM není SSH na `pve`).
> IP a role jsou ale nepřímo potvrzené z Docker VM (Ollama .154 odpovídá).

| ID | Typ | Název | IP | Role | Zdroje |
|---|---|---|---|---|---|
| 100 | VM | docker | 192.168.0.199 | Docker host (aplikace + sdílená infra) | 4 jádra, ~20 GB RAM (8→10→20), disk ~296 GB |
| 101 | LXC | Ollama | 192.168.0.154 | Lokální AI | 4 jádra, 8 GB RAM, GPU passthrough |
| 102 | LXC | Kuma | 192.168.0.158 | Uptime Kuma monitoring | 1 jádro |

- VM 100 RAM ~20 GB ✅ ověřeno na hostu 2026-06-30 (`free -h` → 19Gi). Detail `qm config 100` `[OVĚŘIT na pve]`.
- LXC 101 Ollama @ **192.168.0.154** ✅ ověřeno 2026-06-30 (ping ok; .153 už nereaguje).
  - ⚠️ IP historicky nestabilní (DHCP): .153 → .154. `[OVĚŘIT: dostala statickou IP? Pokud ne,
    restart LXC ji zase změní a rozbije napojení Open WebUI — viz `incidenty/ollama-ip-dhcp.md`]`
- LXC 102 Kuma @ 192.168.0.158 `[OVĚŘIT zdroje + co monitoruje]`

## CPU přidělení
Fyzicky 12 threadů. Rozdáno: Docker 4 + Ollama 4 + Kuma 1 = 9 vCPU (overcommit OK,
reálný load hostitele velmi nízký). vCPU nejsou napevno, sdílejí se.
