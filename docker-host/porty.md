# Mapa portů (Docker host 192.168.0.199)

> Ověřeno 2026-06-30 (`docker ps --format "table {{.Names}}\t{{.Ports}}"`).

## Reverse proxy model
Caddy = jediný vstup pro veřejné HTTPS. Routuje podle domény na **INTERNÍ port kontejneru**
(na síti `web`), NE na host mapping. HTTPS automaticky (HTTP-01 na portu 80 → port forward 80 nutný).

## Tabulka portů
| Služba | Interní port | Host mapping | Veřejně | Pozn. |
|---|---|---|---|---|
| caddy | – | 80, 443 (tcp+udp) | ano (proxy) | + 2019 (admin, interní) |
| nuart-backoffice | 3000 | 3020 | office.nuart.cz | ✅ |
| nicotrans-crm | 3000 | — (bez mappingu) | crm.nuart.cz | ✅ |
| nicotrans-palety | 3000 | 3010 | crm.nuart.cz/palety | ✅ viz pozn. níže |
| infra-postgres | 5432 | — | ne | ✅ |
| infra-minio | 9000 / 9001 | 9000 / 9001 | ne (LAN) | ✅ API + konzole |
| openwebui | **8080** | **3000** | ne | ✅ viz pozn. níže |
| dockge | 5001 | 5001 | ne | ✅ |
| windrose | host net | host net | herní porty + 8780 (LAN dashboard) | ✅ |
| Ollama (LXC 101) | 11434 | – | ne (LAN) | @192.168.0.154 |
| Uptime Kuma (LXC 102) | 3001 | – | ne (LAN) | @192.168.0.158 `[OVĚŘIT]` |

## Opravy proti dumpu
- `[opraveno proti realitě 2026-06-30: openwebui — bylo „interní port 3000", JE interní **8080**,
  host mapping **3000** (mapování 3000→8080)]`
- `[opraveno proti realitě 2026-06-30: nicotrans-palety — bylo „interní? / jen interní, nevystaveno",
  JE interní 3000, host 3010, a navíc **vystaveno veřejně** pod crm.nuart.cz/palety přes Caddy
  (Next basePath, prefix se NEstripuje) — detail stacks/caddy.md a stacks/nicotrans-palety.md]`
- `[opraveno proti realitě 2026-06-30: infra-minio má host mapping 9000-9001 (ne jen interní)]`
