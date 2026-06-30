# Nasazování služby s Better Auth za reverse proxy — checklist

> Ponaučení z office.nuart.cz (26.6.2026). Při vystavení Better Auth aplikace
> přes Caddy/reverse proxy na veřejnou doménu se VŽDY musí přidat origin,
> jinak login skončí chybou "Invalid origin" (navenek se tváří jako "Nesprávný e-mail nebo heslo").

## Co se stalo
- office.nuart.cz vystaveno přes Caddy → nuart-backoffice (Next.js + Better Auth)
- Login házel "Nesprávný e-mail nebo heslo", ale reálná příčina v logu:
  `ERROR [Better Auth]: Invalid origin: https://office.nuart.cz`
- Přihlášení přes IP (192.168.0.199:3020) fungovalo, přes doménu ne

## Dvě věci, které je potřeba nastavit

### 1. BETTER_AUTH_URL (.env)
```
BETTER_AUTH_URL="https://office.nuart.cz"
```
- veřejná HTTPS doména, BEZ portu (Caddy řeší 443)

### 2. trustedOrigins (src/lib/auth/auth.ts) — TO SE ZAPOMÍNÁ
```typescript
export const auth = betterAuth({
  baseURL: process.env.BETTER_AUTH_URL,
  trustedOrigins: [
    "https://office.nuart.cz",       // veřejná přes Caddy
    "http://192.168.0.199:3020",     // lokální IP přístup
    "http://localhost:3020",         // lokální dev
  ],
  ...
})
```
- Better Auth odmítá requesty z originů, které nezná (ochrana proti CSRF)
- musí obsahovat VŠECHNY adresy, ze kterých se na appku přistupuje

### 3. Po úpravě kódu — REBUILD, ne jen restart
Next.js se buildí, takže změna v .ts vyžaduje:
```
cd /opt/stacks/nuart-backoffice
docker compose up -d --build
```
Ověření, že chyba zmizela:
```
docker logs nuart-backoffice --tail 20   # už žádné "Invalid origin"
```

## Checklist pro PŘÍŠTÍ službu s auth za proxy
1. [ ] přidat doménu do Caddyfile (reverse_proxy)
2. [ ] Wedos A-record doména → 178.255.174.235
3. [ ] BETTER_AUTH_URL = https://nova-domena.cz (.env)
4. [ ] trustedOrigins += "https://nova-domena.cz" (auth config)
5. [ ] docker compose up -d --build (rebuild!)
6. [ ] ověřit log: žádné "Invalid origin"
7. [ ] otestovat login z veřejné domény

## Možná automatizace (do budoucna)
- trustedOrigins číst z env proměnné (např. TRUSTED_ORIGINS="https://a.cz,https://b.cz")
  místo hardcode v kódu → pak stačí .env, žádný rebuild kódu
  ```typescript
  trustedOrigins: process.env.TRUSTED_ORIGINS?.split(",") ?? ["http://localhost:3020"],
  ```
- tím se z toho stane konfigurace (.env), ne změna kódu → nasazení nové domény = jen .env + restart
