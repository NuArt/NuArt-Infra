# infra-postgres (sdílená DB)

> Ověřeno 2026-06-30 (`docker exec infra-postgres psql -U postgres -c "\l"` a `"\du"`).

- **Kontejner:** `infra-postgres`, image `postgres:16`, port 5432 (interní, síť `web`)
- **Stack:** `/opt/stacks/infra/` (compose.yaml + `initdb/`) — sdílí Postgres i MinIO
- **Připojení z appek:** `infra-postgres:5432` (po síti `web`, jménem)

## Databáze (ověřeno)
| DB | Owner | Modul |
|---|---|---|
| `nuart` | nuart_user | nuart-backoffice |
| `nicotrans` | nicotrans_user | nicotrans-palety |
| `crm_identity` | crm_user | nicotrans-crm (+ palety čte identitu) |

`[opraveno proti realitě 2026-06-30: dump uváděl uživatele postgres + crm_user; reálně má
 každý modul dedikovaného uživatele (nuart_user, nicotrans_user, crm_user) — což odpovídá konvenci]`

## Uživatelé (role)
- `postgres` — superuser (Create role/DB, Replication, Bypass RLS)
- `nuart_user`, `nicotrans_user`, `crm_user` — dedikovaní per modul, bez nadřazených práv

## Bezpečnost / secrets
- Hesla v `.env` jednotlivých stacků (`DATABASE_URL`), NE v repu.
- ⚠️ `[OVĚŘIT/TODO]` Heslo `postgres` historicky uniklo do chatu → doporučeno **rotovat**
  (interní služba, ne katastrofa, ale udělat).
- ⚠️ SPOF: jeden sdílený Postgres pro víc modulů — kompenzováno dedikovanými DB uživateli
  (viz `decisions/003-sdilena-infra.md`).
