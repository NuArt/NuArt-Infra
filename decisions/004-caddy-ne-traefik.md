# ADR-004: Caddy, ne Traefik

**Rozhodnutí:** Reverse proxy je Caddy.

**Proč:** Jednodušší config (Caddyfile vs. labely u Traefiku), automatické HTTPS bez
certresolverů. 

**Historie:** CRM byl původně na Traefiku, přepsán na Caddy kvůli jednotnosti.

Viz `stacks/caddy.md`.
