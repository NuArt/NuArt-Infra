# NuArt Infrastructure — Knowledge Dump

> Kompletní extrakce kontextu o serveru NuArt, vytvořená 2026-06-30 z jediné konverzace,
> kde žila celá historie serveru. Slouží jako vstup pro stavbu repozitáře `nuart-infra`.
>
> **DŮLEŽITÉ:** Tento dump psala AI, která NEMĚLA nejnovější stav serveru. Část věcí
> (CRM deploy, porty, Windrose+) byla řešena přímo na serveru přes Claude Code až po
> posledním plném vědomí AI. Proto jsou v textu značky **[OVĚŘIT]** a celá **sekce M**.
> Než se cokoliv zapíše jako fakt do repa, OVĚŘIT proti realitě serveru.
>
> Pozn.: konverzace prošla kompakcí (shrnutím), nejstarší detaily jsou přes shrnutí.

---

# A. Velký obraz a historie

## Co ten server je
Domácí/firemní homelab pro **NuArt Media s.r.o.** (Daniel Habrda, České Budějovice). Slouží k:
- provozu interních firemních aplikací (NuArt Backoffice, Nicotrans paletový systém, Nicotrans CRM)
- lokální AI (Ollama – text modely pro zpracování dat)
- herním serverům (Windrose – komunitní hraní)
- infrastruktuře pro vývoj (sdílená Postgres/MinIO, reverse proxy, monitoring)

Filozofie: jeden fyzický stroj, na něm Proxmox, a v něm oddělené světy (produkční appky /
AI / hry) tak, aby se navzájem nerušily.

## Časová osa (klíčové milníky)
1. Bare-metal Proxmox na ASUS serveru (Xeon, 32 GB RAM, GTX 1070, SSD 1.92 TB).
2. Docker VM (VM 100) jako hlavní aplikační prostředí s `/opt/stacks/` a Dockge.
3. Sdílená infra – Postgres + MinIO jako základ pro aplikace.
4. Ollama – nejdřív LXC CPU-only (chyběl ovladač), později GPU passthrough zprovozněn
   (GTX 1070, driver 580.95.05/CUDA 13).
5. Caddy jako reverse proxy + automatické HTTPS.
6. NuArt Backoffice vystaven na office.nuart.cz (Better Auth) – řešily se origin problémy.
7. Windrose herní server + Windrose+ mod (dashboard/RCON na :8780).
8. Lokální AI deep-dive – vision/OCR na GTX 1070 nefunguje, text modely (qwen3) ano.
9. Uptime Kuma monitoring (LXC 102).
10. Nicotrans CRM nasazen na crm.nuart.cz (poslední velký milník, přes Claude Code na serveru).

---

# B. Proxmox / hardware (hypervizor)

## Fyzický stroj
- **Deska:** ASUS Z10PA-U8
- **CPU:** Intel Xeon E5-1650 v4 (Broadwell, 6 jader / 12 threadů, AVX+AVX2)
- **RAM:** 32 GB [OVĚŘIT: plánovaná výměna na 64 GB 4×8GB ECC RDIMM 2400 – jestli proběhla; vyžadovala odstávku]
- **GPU:** NVIDIA GTX 1070 8 GB (Pascal) – tvrdý strop pro AI úlohy
- **Disk:** SSD 1.92 TB

## Proxmox
- **Verze:** VE 9.2.3 [OVĚŘIT: mohlo se updatovat]
- **Hostname:** `pve`
- **Management IP:** 192.168.0.186, web na :8006
- **Přístup:** SSH `ssh root@192.168.0.186`; mobilně Termius (ed25519 klíče přes web konzoli)

## VM a kontejnery
| ID | Typ | Název | IP | Role | Zdroje |
|---|---|---|---|---|---|
| 100 | VM | docker | 192.168.0.199 | Docker host (aplikace + sdílená infra) | 4 jádra, 20 GB RAM (8→10→20), disk ~296 GB |
| 101 | LXC | Ollama | 192.168.0.154 | Lokální AI | 4 jádra, 8 GB RAM, GPU passthrough |
| 102 | LXC | Kuma | 192.168.0.158 | Uptime Kuma monitoring | 1 jádro |

> ⚠️ Ollama IP historicky nestabilní: .153 → .154 (DHCP). [OVĚŘIT: statická IP? Byl to TODO;
> pokud ne, při restartu LXC se zase změní a rozbije napojení Open WebUI.]

## CPU přidělení
- Fyzicky 12 threadů. Rozdáno: Docker 4 + Ollama 4 + Kuma 1 = 9 vCPU (overcommit OK,
  reálný load hostitele ~0.78/12 = velmi nízký). vCPU nejsou napevno, sdílejí se.

## Síť / ISP
- Veřejná IP: 178.255.174.235 (ISP Starnet, 1:1 NAT)
- Gateway: UniFi UCG-Fiber (.1)
- Port forwardy (UCG-Fiber, ověřeno): caddy-http 80→192.168.0.199:80, caddy-https 443→192.168.0.199:443
- WAN IP na gatewayi: 10.6.123.174 (vnitřní WAN za ISP NAT → double-NAT)
- ⚠️ Double-NAT blokoval OpenVPN setup [OVĚŘIT: vyřešeno?]
- DNS: registrátor/správa Wedos (živé MX – nerozbít)

---

# C. Docker host (VM 100)

## Základ
- OS: Debian 12. Hostname `docker`, IP 192.168.0.199
- Docker: 29.6.0, Compose v5.1.4 [OVĚŘIT: verze]
- RAM: navýšena na 20 GB (8→10→20 kvůli Windrose)
- Přístup: `ssh nuart-docker` (alias, endy@docker), nebo `ssh root@192.168.0.199`
- pip: není defaultně – doinstalován python3-pip; balíky s `--break-system-packages`

## Docker prostředí
- Síť: `web` (external, kontejnery se vidí jménem)
- Dockge – správa stacků (:5001), stacky v /opt/stacks/
- buildx (buildkit) – pro buildy
- autoheal – restart nezdravých kontejnerů
- Konvence: každý stack = složka v /opt/stacks/, vlastní compose.yaml + .env

---

# D. Stacky (per kus)

> ⚠️ Porty a názvy se posledních hodin měnily na serveru – [OVĚŘIT].

## caddy
- Reverse proxy + automatické HTTPS (Let's Encrypt, HTTP-01 na portu 80)
- Kontejner `caddy`, porty 80+443
- Config: /opt/stacks/caddy/Caddyfile
- Domény: office.nuart.cz → nuart-backoffice:3000, crm.nuart.cz → nicotrans-crm:3000
- Každý blok: bezpečnostní hlavičky (HSTS, X-Frame-Options, X-Content-Type-Options,
  Referrer-Policy, -Server), encode gzip zstd, logy /var/log/caddy/<sluzba>.log
- Reload: `docker exec caddy caddy reload --config /etc/caddy/Caddyfile`
- Pravidlo: vystavovat jen služby s vlastním loginem

## nuart-backoffice
- Interní firemní backoffice. Doména office.nuart.cz
- Kontejner `nuart-backoffice`, interní port 3000, host mapping 3020
- Next.js 16.2.6 + Better Auth + Prisma + Postgres. DB `nuart`
- env: /opt/stacks/nuart-backoffice/.env. Auth: src/lib/auth/auth.ts (trustedOrigins)
- Úskalí: Better Auth origin – viz J-1. [OVĚŘIT: nepushnutý commit?]

## nicotrans-palety
- Paletový systém. Jen interně (bez auth, schválně nevystaveno)
- Kontejner `nicotrans-palety`, host mapping 3010 [OVĚŘIT interní port]
- Next.js + Prisma (vzor pro další moduly). DB `nicotrans`
- Plán: migrace na nico-palety.nuart.cz, AI pipeline (NICOTRANS_AI.md)
- buildx_buildkit_nicotrans-builder – build kontejner

## nicotrans-crm  ⭐ (nově nasazeno)
- Nicotrans CRM Core – centrální správa uživatelů a práv nad moduly
- Doména crm.nuart.cz (root, BASE_PATH prázdné) – LIVE s HTTPS
- Kontejner `nicotrans-crm`, interní port 3000 (ověřeno 3000/tcp, healthy)
- Next.js + Prisma + auth (login/seed admin), síť web
- DB `crm_identity` + uživatel `crm_user` (dedikovaný)
- Repo github.com/Endyz/Nicotrans-CRM → /opt/stacks/nicotrans-crm
- env: /opt/stacks/nicotrans-crm/.env (DATABASE_URL, SESSION_SECRET, CRM_HOST, BASE_PATH, ADMIN_*)
- Deploy: migrate + seed (--profile tools), pak up -d --build app
- [OVĚŘIT: finální login potvrzen? admin heslo změněno?]

## infra-postgres
- Sdílená DB. Kontejner `infra-postgres`, port 5432 (interní)
- DB: nuart, nicotrans, crm_identity. Uživatelé: postgres, crm_user
- ⚠️ Heslo postgres uniklo do chatu – doporučeno rotovat (interní, ne katastrofa)

## infra-minio
- S3 úložiště (pro budoucí moduly). Kontejner `infra-minio`, porty 9000 (API) + 9001 (konzole)
- CRM core ho zatím nepotřebuje

## openwebui
- Webové UI k Ollama. Kontejner `openwebui`, port 3000 [OVĚŘIT host mapping]
- Napojení na Ollama API http://192.168.0.154:11434
- Úskalí: změna Ollama IP (DHCP) rozbije spojení → ručně přepsat API URL v nastavení

## windrose
- Dedikovaný herní server. Kontejner `windrose`, network_mode: host
- mem_limit: 10g (z 8g), cpus: 2.0 (2 ze 4 jader – produkce má vždy 2 volné)
- env: /opt/stacks/windrose/.env (INVITE_CODE, SERVER_NAME, MAX_PLAYERS=6, WINDROSE_PLUS_ENABLED=true)
- Windrose+ mod v1.3.14 (UE4SS + PAK), dashboard/RCON/live-map na http://192.168.0.199:8780 (LAN)
- Config: /opt/stacks/windrose/server-files/windrose_plus_data/windrose_plus.json (RCON heslo, multiplikátory)
- RAM roste s explorací (lazy loading), ustálila se ~7,2 GB/10 GB. CPU špičky 150-200% (limit 200%), load nízký
- Restart kvůli .env/compose vykopne hráče. Auto-save světa každých 60 s
- Image od indifferentbroccoli (Steam app 4129620)
- Bell limit fast-travel: jde zvednout JEN modem, co musí mít server I každý hráč
  (Windrose validuje client-side, server-only varianta neexistuje). Mod #54 přes Mod Manager.

## dockge
- Webová správa stacků. Kontejner `dockge`, port 5001. Spravuje /opt/stacks/

## autoheal
- Automatický restart nezdravých kontejnerů (healthcheck-based)

---

# E. Síť, routing, domény, porty

## Reverse proxy model
- Caddy = jediný vstup pro veřejné HTTPS. Routuje podle domény na INTERNÍ port kontejneru
  (na síti web), NE na host mapping. HTTPS automaticky (HTTP-01 na portu 80 → port forward 80 nutný).

## DNS (Wedos)
- Doména nuart.cz (+ MX živé – nerozbít)
- A-záznamy → 178.255.174.235: office, crm (a později nico-palety, nico-crm)
- [OVĚŘIT: jaké A-záznamy reálně existují]

## Mapa portů (k poslednímu vědomí)
| Služba | Interní port | Host mapping | Veřejně |
|---|---|---|---|
| caddy | – | 80, 443 | ano (proxy) |
| nuart-backoffice | 3000 | 3020 | office.nuart.cz |
| nicotrans-crm | 3000 | – | crm.nuart.cz |
| nicotrans-palety | ? | 3010 | ne (interní) |
| infra-postgres | 5432 | – | ne |
| infra-minio | 9000/9001 | ? | ne |
| openwebui | 3000 | ? | ne |
| dockge | 5001 | 5001 | ne |
| windrose | host net | host net | herní porty + 8780 |
| Ollama (LXC) | 11434 | – | ne (LAN) |
| Uptime Kuma (LXC) | 3001 | – | ne (LAN) |

> ⚠️ [OVĚŘIT všechny host mappingy – posledních hodin se porty řešily na serveru.]

## Subdomény vs. path-based (rozhodnutí)
- Zvoleny subdomény (crm.nuart.cz), NE path-based (/crm). Důvod: BASE_PATH se zapéká do
  buildu → path-based = rebuild při migraci. Subdoména = migrace jen Caddyfile + DNS.

---

# F. Pojmenování a konvence
- Kontejnery: malá písmena, pomlčky (nuart-backoffice, nicotrans-palety, nicotrans-crm,
  infra-postgres, infra-minio)
- Prefix `infra-` = sdílená infrastruktura
- DB: podle domény/účelu (nuart, nicotrans, crm_identity)
- DB uživatelé: dedikovaní per modul (crm_user), ne sdílení
- Stacky: složka = název v /opt/stacks/
- GitHub: repo pod Endyz/ (osobní), cílově NuArt/ org pro infra repo
- Domény: modul.nuart.cz (subdomény), cílově migrace na nicotrans.cz pro Nicotrans moduly
- Interní port appek: 3000 (Next.js default), host mapping 30xx pro přímý přístup
- Reverse proxy: vždy na interní port (3000)

---

# G. Tajemství a účty (jen umístění, NE hodnoty)

## Kde žijí secrets
| Secret | Umístění |
|---|---|
| DB hesla (postgres, crm_user) | .env v každém stacku (/opt/stacks/<stack>/.env) |
| BETTER_AUTH_SECRET | /opt/stacks/nuart-backoffice/.env |
| SESSION_SECRET (CRM) | /opt/stacks/nicotrans-crm/.env |
| ADMIN heslo (CRM iniciální) | /opt/stacks/nicotrans-crm/.env (dočasné) |
| RCON heslo (Windrose+) | /opt/stacks/windrose/server-files/windrose_plus_data/windrose_plus.json |
| INVITE_CODE (Windrose) | /opt/stacks/windrose/.env |
| SSH klíče (mobil) | ed25519, deployované přes Proxmox web konzoli |
| GitHub SSH klíč (Docker VM) | ~/.ssh/id_ed25519, registrovaný jako "NuArt Docker Server" |

> Pravidlo: .env vždy v .gitignore. ⚠️ Postgres heslo uniklo do chatu → rotovat.

## Účty a služby
- GitHub: osobní `Endyz` (Nicotrans-CRM, Geparim-Arrivals…). Cílově org `NuArt` pro infra.
- Registrátor/DNS: Wedos (nuart.cz, živé MX)
- ISP: Starnet (178.255.174.235, 1:1 NAT, double-NAT problém)
- Let's Encrypt: ACME účet (Caddy)
- Přístup: Daniel (endy@docker, root), mobilně Termius

---

# H. Zálohy a monitoring

## Monitoring
- Uptime Kuma (LXC 102, 192.168.0.158:3001) [OVĚŘIT: co konkrétně monitoruje]
- autoheal – restartuje nezdravé Docker kontejnery

## Zálohy
- Windrose svět: auto-save každých 60 s (herní mechanika)
- Proxmox zálohy VM/CT: [OVĚŘIT: jsou vzbackup joby?]
- DB zálohy (Postgres): [OVĚŘIT: pravidelný dump? Pravděpodobná díra]
- Git: kód aplikací na GitHubu
- ⚠️ NEzálohuje se (k vědomí): DB obsah, .env, Caddyfile, konfigurace stacků – RIZIKO.
  Tahle extrakce do gitu je první krok k nápravě dokumentační části.

---

# I. Rozhodnutí a důvody (ADR)

**ADR-001: Proxmox bare-metal, ne Docker na železe.** Oddělit světy do VM/LXC, snapshoty, GPU passthrough do izolovaného LXC.

**ADR-002: Ollama v LXC, ne v Docker VM.** GPU passthrough čistší do LXC; izolace AI zátěže. (Začínalo CPU-only kvůli chybějícímu Pascal ovladači.)

**ADR-003: Sdílená infra (jeden Postgres, jeden MinIO).** Úspora zdrojů, jednodušší správa. Riziko SPOF → kompenzováno dedikovanými DB uživateli.

**ADR-004: Caddy, ne Traefik.** Jednodušší config (Caddyfile vs labely), automatické HTTPS bez certresolverů. CRM byl původně na Traefik, přepsán.

**ADR-005: Reverse proxy na interní port (3000), ne host mapping.** Caddy je s appkami na síti web, mluví na interní port. Host mapping jen pro debug.

**ADR-006: Subdomény, ne path-based.** BASE_PATH se zapéká do buildu → path-based = rebuild při migraci.

**ADR-007: Vystavovat přes Caddy jen služby s auth.** Bezpečnost. Nicotrans-palety (bez loginu) zůstává interní.

**ADR-008: Windrose 2 jádra / 10 GB RAM.** Izolace herní zátěže – produkce má 2 jádra volná. RAM 10 GB pokrývá plató po exploraci (~7,2 GB).

**ADR-009: Lokální AI jen text modely (qwen3), ne vision/OCR.** GTX 1070 (8 GB Pascal) na vision nestačí (testováno granite/qwen2.5vl/minicpm – selhaly). Architektura: AI mapuje význam sloupců, pandas tahá data.

**ADR-010: Dokumentace v Git markdown, ne dedikovaný nástroj.** Jednoduchost, verzování, čitelnost.

---

# J. Úskalí, incidenty, poučení

**J-1: Better Auth "Invalid origin" na office.nuart.cz.** Login házel "Nesprávný e-mail nebo heslo", reálně log "Invalid origin". Better Auth znal jen IP:port. Fix: BETTER_AUTH_URL=https://office.nuart.cz + trustedOrigins (office+IP+localhost) v src/lib/auth/auth.ts + REBUILD (ne restart). Zdokumentováno v BETTER_AUTH_PROXY.md; návrh číst trustedOrigins z env.

**J-2: Ollama IP změna (DHCP) rozbila Open WebUI.** .153→.154, Open WebUI ztratil modely. Fix: ručně přepsat API URL. Poučení: nastavit statickou IP (opakuje se).

**J-3: Vision OCR na GTX 1070 – slepá ulička.** granite (kontext 16k málo), qwen2.5vl (12.5 GB > 8 GB → CPU → timeout), minicpm (vešel se, 46s/doc + halucinoval). Zdokumentováno v NICOTRANS_AI.md. PDF/skeny → cloud OCR.

**J-4: Windrose RAM na hraně (98%) s Windrose+.** Mod overhead + lazy loading. Fix: bump 8→10 GB. RAM roste s explorací, ustálí se na plató.

**J-5: GitHub HTTPS auth selhal při clone.** GitHub od 2021 nepřijímá heslo. Fix: SSH klíč (registrovaný), clone přes git@github.com:. Na server klonovat přes SSH.

**J-6: Terminál mrší multiline paste.** Příkazy slepené. Posílat po jednom.

**J-7: CRM nasazení – Caddy blok se "neaplikoval".** Caddy znal jen office. Blok nebyl vložen (Claude Code ho jen navrhl). Fix: vložit ručně + reload. Claude Code drží "změnu ti ukážu, ty vložíš".

**J-8: ping ≠ web.** crm.nuart.cz ping 100% loss → falešný poplach. DNS fungovalo, ICMP blokovaný. Testovat webem/curl.

---

# K. Otevřené věci / TODO
1. Ollama statická IP (LXC 101) – opakující se [OVĚŘIT]
2. CRM: první login + změna admin hesla
3. DB zálohy – pravděpodobně chybí (riziko)
4. Proxmox backup joby – ověřit/nastavit
5. Postgres heslo rotovat (uniklo do chatu)
6. RAM upgrade 32→64 GB (odstávka)
7. Double-NAT / OpenVPN – ISP problém
8. nuart-backoffice – nepushnutý commit [OVĚŘIT]
9. nicotrans-palety: AI pipeline, migrace na nico-palety.nuart.cz, login před vystavením
10. Windrose+ dashboard do rozcestníku, GPU passthrough 1070 (pending), live-map wp.mapgen
11. trustedOrigins z env (automatizace)
12. Subnet renumbering NuArt-Staff VLAN 192.168.0.0/24 → 10.44.10.0/24 [OVĚŘIT]

---

# L. Volné konce
- Dokumenty už vytvořené (zahrnout do repa): HOMELAB.md, NICOTRANS_AI.md, BETTER_AUTH_PROXY.md,
  networks.md, Caddyfile, windrose-compose.yaml, windrose.env
- Nicotrans AI: qwen3:8b/4b na 192.168.0.154:11434, column-mapping (AI mapuje, pandas tahá).
  Test file vzorova__tabulka.xlsx
- Související projekty (kontext): Geparim Arrivals (Godot hra), Fest Na tahu app, Na tahu DnD
- Daniel: dyslexie/dysgrafie – krátké odrážky, "příkaz + proč", přímé instrukce.
  Pracuje v rozkouskovaných časech, přepíná Mac↔Windows.

---

# M. Co ověřit (drift)

> Kritické věci, co se mohly změnit. Claude Code je projde proti realitě PŘED zápisem jako fakt.

| # | Co ověřit | Proč | Jak |
|---|---|---|---|
| M-1 | Host port mappingy všech stacků | Porty se řešily na serveru | `docker ps --format "table {{.Names}}\t{{.Ports}}"` |
| M-2 | Interní port nicotrans-palety | Neznám jistě | `docker compose -f /opt/stacks/nicotrans-palety/compose.yaml config \| grep -A2 ports` |
| M-3 | CRM stav (login? heslo změněno?) | Dokončováno po vědomí | crm.nuart.cz, `docker compose ps` |
| M-4 | Ollama IP (.154? statická?) | DHCP | `ping 192.168.0.154`, `pct config 101` |
| M-5 | A-záznamy ve Wedosu | Přidávaly se nové | `dig +short office.nuart.cz crm.nuart.cz` |
| M-6 | RAM Docker VM (20 GB?) + Windrose limit | Měnilo se 8→10→20 | `free -h`, `grep mem_limit /opt/stacks/windrose/compose.yaml` |
| M-7 | Caddyfile obsah (jaké domény) | crm přidán | `cat /opt/stacks/caddy/Caddyfile` |
| M-8 | Seznam běžících kontejnerů | Mohlo přibýt/ubýt | `docker ps -a` |
| M-9 | Proxmox verze, VM/LXC zdroje | Update/změna | `pveversion`, `qm config 100`, `pct config 101 102` |
| M-10 | DB seznam na infra-postgres | crm_identity přibyl | `docker exec infra-postgres psql -U postgres -c "\l"` |
| M-11 | RAM hardware (32 vs 64 GB) | Plánovaná výměna | `free -h` / `dmidecode` na Proxmoxu |
| M-12 | Subnet renumbering | Plánováno | `ip a`, UniFi config |

---

*Konec dumpu. Vstup pro stavbu repozitáře nuart-infra — viz zaváděcí prompt.*
