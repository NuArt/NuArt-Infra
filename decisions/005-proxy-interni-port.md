# ADR-005: Reverse proxy na interní port (3000), ne host mapping

**Rozhodnutí:** Caddy routuje na **interní port kontejneru** (3000), ne na host mapping.

**Proč:** Caddy je s appkami na společné síti `web`, mluví s nimi jménem na interní port.
Host mapping (30xx) slouží jen pro přímý debug přístup.

Viz `docker-host/porty.md`, `docker-host/konvence.md`.
