# ADR-001: Proxmox bare-metal, ne Docker na železe

**Rozhodnutí:** Na fyzický server běží Proxmox; aplikace, AI a hry jsou oddělené do VM/LXC.

**Proč:** Oddělit „světy" (produkční appky / AI / hry), aby se navzájem nerušily.
Snapshoty, izolace zátěže, čistý GPU passthrough do izolovaného LXC.

**Důsledek:** Docker host je VM 100; Ollama je LXC 101; monitoring LXC 102.
