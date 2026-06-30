# Stack: caddy (reverse proxy)

> Ověřeno 2026-06-30 (`cat /opt/stacks/caddy/Caddyfile`, `docker ps`).

## Co to je
Reverse proxy + automatické HTTPS (Let's Encrypt, HTTP-01 challenge na portu 80).
Jediný veřejný vstup. Routuje podle domény na **interní port** kontejneru (síť `web`).

- **Kontejner:** `caddy`, porty 80 + 443 (tcp+udp), admin 2019 (interní)
- **Config:** `/opt/stacks/caddy/Caddyfile`
- **Reload:** `docker exec caddy caddy reload --config /etc/caddy/Caddyfile`
- **Logy:** `/var/log/caddy/<sluzba>.log` (uvnitř kontejneru), roll 10mb / keep 5

## Domény (reálný Caddyfile, ověřeno 2026-06-30)
| Doména / cesta | Cíl |
|---|---|
| `office.nuart.cz` | `nuart-backoffice:3000` |
| `crm.nuart.cz/palety`, `/palety/*` | `nicotrans-palety:3000` (basePath, prefix se NEstripuje) |
| `crm.nuart.cz` (vše ostatní) | `nicotrans-crm:3000` (login, portál, admin) |

`[opraveno proti realitě 2026-06-30: dump měl jen „crm.nuart.cz → nicotrans-crm (root, BASE_PATH prázdné)";
 reálně crm.nuart.cz navíc routuje /palety/* na nicotrans-palety (path-based, Next basePath).
 Tím je palety nově VYSTAVENO veřejně — viz stacks/nicotrans-palety.md]`

> ⚠️ Komentář v hlavičce Caddyfile („vystavujeme VÝHRADNĚ office.nuart.cz") je **zastaralý** —
> crm.nuart.cz blok existuje pod ním. `[OVĚŘIT/uklidit komentář]`

## Bezpečnostní hlavičky (každý blok)
HSTS (max-age 1 rok, includeSubDomains), X-Frame-Options SAMEORIGIN, X-Content-Type-Options nosniff,
Referrer-Policy strict-origin-when-cross-origin, `-Server`. Komprese `encode gzip zstd`.

## Pravidla
- Vystavovat přes Caddy **jen služby s vlastním loginem** (viz `decisions/007-vystavovat-jen-s-auth.md`).
- Sanitizovaný vzor: `stacks/_vzory/Caddyfile.example`.
- Nová doména s Better Auth → checklist v `incidenty/better-auth-origin.md`.
- CRM blok historicky nešel „sám" — Claude Code návrh jen ukázal, vložit ručně + reload
  (viz `incidenty/crm-caddy-blok.md`).
