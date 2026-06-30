# infra-minio (S3 úložiště)

> Ověřeno 2026-06-30 (`docker ps`, `/opt/stacks/infra/compose.yaml`).

- **Kontejner:** `infra-minio`, image `minio/minio`
- **Stack:** `/opt/stacks/infra/` (sdílený s Postgres)
- **Porty:** 9000 (S3 API) + 9001 (web konzole), host mapping 9000-9001 (LAN)
  `[opraveno proti realitě 2026-06-30: dump měl host mapping „?"; reálně 9000-9001 mapováno na host]`
- **Síť:** `web` — appky volají `infra-minio:9000`

## Použití
- S3 úložiště pro moduly. `nuart-backoffice` a `nicotrans-palety` mají v `.env` S3_* proměnné
  (S3_ENDPOINT, S3_ACCESS_KEY_ID, S3_SECRET_ACCESS_KEY, S3_BUCKET…).
- CRM core ho zatím nepotřebuje.

## Secrets
- Přístupové klíče v `.env` příslušných stacků, NE v repu.
