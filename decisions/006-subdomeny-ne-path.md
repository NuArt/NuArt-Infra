# ADR-006: Subdomény pro samostatné appky, path-based pro moduly pod CRM

**Rozhodnutí (aktualizováno 2026-06-30):** Dvě pravidla podle typu nasazení:
- **Samostatné aplikace** → vlastní **subdoména** (`modul.nuart.cz`). Např. `office.nuart.cz`.
- **Moduly spadající pod CRM Core** → **záměrně path-based** (`crm.nuart.cz/modul`),
  protože sdílí doménu a identitu s CRM. Např. `crm.nuart.cz/palety`.

**Proč subdomény u samostatných appek:** `BASE_PATH` se u Next.js zapéká do buildu →
path-based by znamenal rebuild při migraci. Subdoména = migrace jen Caddyfile + DNS.

**Proč path-based u CRM modulů:** modul má sdílet doménu i přihlášení s CRM Core
(jeden login, jedna identita). Path-based pod `crm.nuart.cz` to umožňuje; Caddy prefix
nestripuje, Next `basePath` je nastavený napevno (např. `/palety`).

**Stav:** `nicotrans-palety` jede path-based pod `crm.nuart.cz/palety` (Next `basePath=/palety`),
napojeno na CRM identitu (`IDENTITY_DATABASE_URL` + `CORE_URL`). Viz `stacks/nicotrans-palety.md`.
A-záznam `nico-palety.nuart.cz` existuje pro případnou budoucí samostatnost (vyžadovala by
rebuild s prázdným basePath).

`[opraveno proti realitě 2026-06-30: dump měl jen „subdomény, ne path-based"; doplněno
 záměrné path-based pro moduly pod CRM]`
