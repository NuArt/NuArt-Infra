# CLAUDE.md — proxmox/ (úroveň hypervizoru)

> **Tahle úroveň je KONZULTAČNÍ.** Soubory tady popisují fyzický stroj a Proxmox.
> Nic se odsud neexekvuje automaticky. Když je potřeba zásah na Proxmoxu, navrhni ho
> člověku a počkej na potvrzení — Proxmox je základ, chyba tu shodí všechno nad ní.

## Kontext shora
Patříš do `nuart-infra` (viz kořenový CLAUDE.md). Tahle složka = vrstva pod Docker hostem.
Vše, co běží v `docker-host/` a `stacks/`, stojí fyzicky na téhle vrstvě.

## Co je tady
- `hardware.md` — fyzický stroj (CPU/RAM/GPU/disk), ISP, veřejná IP
- `nodes.md` — Proxmox node `pve`, verze, přístup
- `vms-a-kontejnery.md` — VM 100 (docker), LXC 101 (Ollama), LXC 102 (Kuma)
- `sit.md` — bridge, VLAN, IP rozsahy, port forwardy, double-NAT
- `zalohy.md` — vzbackup joby

## Klíčová fakta
- **Node:** `pve` @ 192.168.0.186:8006, SSH `root@192.168.0.186`
- **Stroj:** ASUS Z10PA-U8, Xeon E5-1650 v4 (6c/12t), GTX 1070 8GB, SSD 1.92TB, 32GB RAM
- **VM/LXC:** 100=docker (4c/20GB), 101=Ollama (4c/8GB, GPU passthrough), 102=Kuma (1c)
- **ISP:** Starnet, veřejná IP 178.255.174.235 (1:1 NAT), gateway UCG-Fiber
- **CPU:** 12 threadů, rozdáno 9 vCPU (overcommit OK, reálný load nízký)

## Pravidla pro tuto úroveň
- **qm/pct příkazy** běží JEN tady (na `pve`), NE na Docker VM. Před každým zkontroluj
  `hostname` → musí být `pve`.
- **Nikdy neměň** síťové/storage/passthrough nastavení bez explicitního pokynu a snapshotu.
- **GPU passthrough** (GTX 1070 → LXC 101) je křehký — Pascal + ovladač. Nesahat bez důvodu.
- Změny zdrojů VM (RAM/CPU) často vyžadují **odstávku VM** — plánovat, ne za běhu.

## Co ověřit (drift)
> ⚠️ **Tato úroveň nebyla 2026-06-30 ověřena.** Z Docker VM (192.168.0.199) není SSH
> přístup na `pve` (`root@192.168.0.186` → Permission denied, publickey/password).
> Všechna Proxmox-fakta níže pocházejí z dumpu a jsou `[OVĚŘIT]` přímo na `pve`.
- Proxmox verze (`pveversion`), zdroje VM/LXC (`qm config 100`, `pct config 101 102`)
- RAM hardware (32 vs plánovaných 64 GB), Ollama IP (DHCP → měla dostat statickou)
- Stav záloh (jsou vzbackup joby?), subnet renumbering (plánováno 10.44.10.0/24)
