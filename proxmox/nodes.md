# Proxmox node `pve`

> **Neověřeno 2026-06-30** (z Docker VM není SSH na `pve`). Ověř na `pve`.

- **Verze:** Proxmox VE 9.2.3 `[OVĚŘIT: mohlo se updatovat — `pveversion`]`
- **Hostname:** `pve`
- **Management IP:** 192.168.0.186, web UI na `:8006`
- **Přístup:**
  - SSH: `ssh root@192.168.0.186`
    - ⚠️ `[OVĚŘIT klíče]` Z Docker VM (192.168.0.199) tento přístup **nefunguje**
      (2026-06-30: Permission denied, publickey/password). Klíč je nejspíš jen na
      pracovních strojích Daniela, ne na Docker VM.
  - Mobilně: Termius (ed25519 klíče deployované přes Proxmox web konzoli)

## Pravidlo
`qm` / `pct` příkazy běží JEN tady. Před exekucí ověř `hostname` → `pve`.
