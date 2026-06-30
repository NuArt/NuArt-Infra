# Incident: Ollama IP změna (DHCP) rozbila Open WebUI

## Co se stalo
LXC 101 (Ollama) má DHCP. IP se posunula `.153 → .154`. Open WebUI byl nastavený na starou
IP → „ztratil modely" (ztratil spojení s Ollama API).

## Fix
Ručně přepsat API URL v nastavení Open WebUI na aktuální IP (`http://192.168.0.154:11434`).

## Stav 2026-06-30
`.154` aktivní (ping ok), `.153` mrtvá. Spojení funguje.

## Poučení / TODO
Nastavit LXC 101 **statickou IP** — jinak se to při každém restartu LXC zopakuje a rozbije
napojení Open WebUI i Nicotrans AI pipeline. `[OVĚŘIT: zatím neuděláno?]`
