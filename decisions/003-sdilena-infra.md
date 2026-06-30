# ADR-003: Sdílená infra (jeden Postgres, jeden MinIO)

**Rozhodnutí:** Jeden sdílený `infra-postgres` a `infra-minio` pro všechny moduly.

**Proč:** Úspora zdrojů, jednodušší správa.

**Riziko:** SPOF (jeden Postgres pro víc modulů) → **kompenzováno dedikovanými DB uživateli**
per modul (`nuart_user`, `nicotrans_user`, `crm_user`), oddělené databáze.

Viz `docker-host/infra/postgres.md`.
