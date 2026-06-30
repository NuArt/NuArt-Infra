# Zálohy (Proxmox + data)

> **Neověřeno 2026-06-30** (z Docker VM není SSH na `pve`). Pravděpodobná díra — řešit.

## Co je
- Windrose herní svět: auto-save každých 60 s (herní mechanika, ne plnohodnotná záloha)
- Git: zdrojový kód aplikací na GitHubu

## Co `[OVĚŘIT]` / pravděpodobně chybí
- **Proxmox vzbackup joby** pro VM 100 / LXC 101 / LXC 102 — `[OVĚŘIT: existují? kam? jak často?]`
- **DB zálohy (Postgres)** — `[OVĚŘIT: pravidelný `pg_dump`? Pravděpodobná díra]`

## ⚠️ Riziko (k vědomí)
Nezálohuje se obsah DB, `.env` soubory, Caddyfile a konfigurace stacků. Při ztrátě VM
by se to nedalo obnovit. Tahle dokumentace je první krok k nápravě dokumentační části;
záložní strategie je samostatný TODO (viz kořenový přehled TODO v `_archiv/knowledge-dump-2026-06-30.md`, sekce K).
