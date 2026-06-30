# Incident: Ollama IP změna (DHCP) rozbila Open WebUI

## Co se stalo
LXC 101 (Ollama) má DHCP. IP se posunula `.153 → .154`. Open WebUI byl nastavený na starou
IP → „ztratil modely" (ztratil spojení s Ollama API).

## Fix
Ručně přepsat API URL v nastavení Open WebUI na aktuální IP (`http://192.168.0.154:11434`).

## Stav 2026-06-30
`.154` aktivní (ping ok), `.153` mrtvá. Spojení funguje. Ověřeno na `pve`: LXC 101 má
v `net0` stále `ip=dhcp` (MAC `BC:24:11:46:DE:91`) — fix tedy NEbyl proveden.

## Řešení (rozhodnuto 2026-06-30): DHCP rezervace na UniFi
Zvolen přístup **DHCP rezervace** (ne statika v configu LXC) — nemění config kontejneru,
nevyžaduje odstávku Ollama:
- Na UniFi gateway (UCG-Fiber) přiřadit MAC `BC:24:11:46:DE:91` napevno IP `192.168.0.154`.
- `[TODO: provést na UniFi — k 2026-06-30 ještě neuděláno]`
- Po rezervaci ověřit, že Open WebUI (`stacks/openwebui.md`) má v nastavení API URL
  `http://192.168.0.154:11434` a že přežije restart LXC.

> Alternativa (nepoužita): statická IP přímo v `pct config 101` (`net0 ip=192.168.0.154/24,gw=...`),
> ale vyžaduje restart LXC.
