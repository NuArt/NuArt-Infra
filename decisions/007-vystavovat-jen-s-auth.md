# ADR-007: Přes Caddy vystavovat jen služby s vlastním auth

**Rozhodnutí:** Veřejně (přes Caddy) vystavit jen službu, která má vlastní login/ochranu.

**Proč:** Bezpečnost — nevystavovat nechráněné interní nástroje na internet.

**Stav 2026-06-30:** `nicotrans-palety` (samo o sobě bez vlastního loginu) je sice vystaveno
pod `crm.nuart.cz/palety`, ALE přístup je gateován **CRM Core identitou** (palety mají v `.env`
`IDENTITY_DATABASE_URL` + `CORE_URL`). Tím je princip zachován — modul stojí za CRM loginem.
Viz `stacks/nicotrans-palety.md`. `[OVĚŘIT, že gate skutečně chrání všechny cesty palet]`
