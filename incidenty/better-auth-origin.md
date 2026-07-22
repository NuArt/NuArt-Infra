# Incident: Better Auth „Invalid origin" (office.nuart.cz)

> Plný checklist: [`_archiv/BETTER_AUTH_PROXY.md`](../_archiv/BETTER_AUTH_PROXY.md)
> (zkopírováno ze serveru `/opt/stacks/nuart-backoffice/docs/`).

## Co se stalo
office.nuart.cz vystaveno přes Caddy → nuart-backoffice (Next.js + Better Auth).
Login házel „Nesprávný e-mail nebo heslo", reálná příčina v logu:
`ERROR [Better Auth]: Invalid origin: https://office.nuart.cz`. Přes IP fungovalo, přes doménu ne.

## Fix
1. `.env`: `BETTER_AUTH_URL="https://office.nuart.cz"` (bez portu, Caddy řeší 443)
2. `src/lib/auth/auth.ts`: `trustedOrigins` musí obsahovat všechny adresy
   (`https://office.nuart.cz` + `http://10.3.20.199:3020` + `http://localhost:3020`)
3. **REBUILD** (`docker compose up -d --build`), ne jen restart — Next.js se buildí.
4. Ověřit log: žádné „Invalid origin".

## Poučení
Pro každou novou službu s auth za proxy → checklist v archivu. Možná automatizace:
`trustedOrigins` číst z env (`TRUSTED_ORIGINS`), pak nasazení domény = jen `.env`, bez rebuildu kódu.
