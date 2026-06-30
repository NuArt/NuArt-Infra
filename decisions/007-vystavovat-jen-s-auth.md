# ADR-007: Přes Caddy vystavovat jen služby s loginem (vlastním nebo přes CRM)

**Rozhodnutí:** Veřejně (přes Caddy) vystavit jen službu, která je za loginem — buď má
**vlastní auth**, nebo je **gateovaná CRM Core identitou**.

**Proč:** Bezpečnost — nevystavovat nechráněné interní nástroje na internet.

**Stav 2026-06-30 (potvrzeno):** `nicotrans-palety` nemá vlastní login, ALE je vystaveno
pod `crm.nuart.cz/palety` **za CRM identitou** (`.env`: `IDENTITY_DATABASE_URL` + `CORE_URL`).
Rozhodnuto, že **CRM identita = dostatečný login gate** — princip ADR platí, palety jsou v pořádku.

Služby bez jakéhokoliv loginu se přes Caddy nevystavují (zůstávají interní / jen LAN).
Viz `stacks/nicotrans-palety.md`, `stacks/caddy.md`.
