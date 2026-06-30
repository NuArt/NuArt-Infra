# ADR-006: Subdomény, ne path-based (preferováno)

**Rozhodnutí:** Moduly vystavovat na subdoménách (`modul.nuart.cz`), ne path-based (`/modul`).

**Proč:** `BASE_PATH` se u Next.js **zapéká do buildu** → path-based znamená rebuild při
migraci. Subdoména = migrace jen Caddyfile + DNS, bez rebuildu.

**Výjimka v praxi (2026-06-30):** `nicotrans-palety` jede path-based pod `crm.nuart.cz/palety`
(Next `basePath=/palety`), protože sdílí doménu a identitu s CRM Core. Caddy prefix nestripuje.
Viz `stacks/nicotrans-palety.md` — pokud bude potřeba samostatnost, migrace na `nico-palety.nuart.cz`
(A-záznam už existuje) si vyžádá rebuild s prázdným basePath.
