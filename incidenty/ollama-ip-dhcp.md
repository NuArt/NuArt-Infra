# Incident: Ollama IP změna (DHCP) rozbila Open WebUI

## Co se stalo
LXC 101 (Ollama) má DHCP. IP se posunula `.153 → .154`. Open WebUI byl nastavený na starou
IP → „ztratil modely" (ztratil spojení s Ollama API).

## Fix
Ručně přepsat API URL v nastavení Open WebUI na aktuální IP (`http://10.3.20.154:11434`).

## Stav 2026-06-30
`.154` aktivní (ping ok), `.153` mrtvá. Spojení funguje. Ověřeno na `pve`: LXC 101 má
v `net0` stále `ip=dhcp` (MAC `BC:24:11:46:DE:91`) — fix tedy NEbyl proveden.

## Řešení — VYŘEŠENO 2026-06-30 ✅: DHCP rezervace na UniFi
Zvolen a **proveden** přístup **DHCP rezervace** (ne statika v configu LXC) — nemění config
kontejneru, nevyžadovalo odstávku Ollama:
- Na UniFi přiřazena MAC `BC:24:11:46:DE:91` napevno IP `10.3.20.154` (zamčeno).
- LXC 101 má dál `net0 ip=dhcp`, ale gateway mu už vždy přidělí stejnou `.154` → problém
  se změnou IP se **nebude opakovat**.
- Kuma monitor „Ollama (LXC 101)" (viz `proxmox/vms-a-kontejnery.md`) hlídá dostupnost.

> Alternativa (nepoužita): statická IP přímo v `pct config 101` — vyžadovala by restart LXC.
