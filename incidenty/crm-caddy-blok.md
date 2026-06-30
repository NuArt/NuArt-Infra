# Incident: CRM nasazení — Caddy blok se „neaplikoval"

## Co se stalo
Při nasazení crm.nuart.cz Caddy znal jen `office`. Blok pro crm „nefungoval" — protože
nebyl reálně vložen do Caddyfile (Claude Code ho jen **navrhl**, nevložil).

## Fix
Blok vložit do `/opt/stacks/caddy/Caddyfile` ručně + reload
(`docker exec caddy caddy reload --config /etc/caddy/Caddyfile`).

## Poučení
Claude Code drží princip „změnu ti ukážu, ty vložíš" — u Caddyfile vždy ověřit, že blok
**skutečně je** v souboru, ne jen navržen. Související: `incidenty/ping-vs-web` (níže).

## Pozn.: ping ≠ web
crm.nuart.cz měl ping 100 % loss → falešný poplach. DNS fungovalo, ICMP byl blokovaný.
**Testovat webem/curl, ne pingem.**
