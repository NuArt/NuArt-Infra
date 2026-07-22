# Proxmox node `pve`

> Ověřeno přímo na `pve` 2026-06-30 (`pveversion`, dashboard).

- **Verze:** `pve-manager/9.2.3/d0fde103346cf89a`, kernel `6.17.13-13-pve` ✅
- **Hostname:** `pve`
- **Management IP:** 10.3.20.186, web UI na `:8006`
  `[2026-07-22: bylo 192.168.0.186 — renumbering sítě → 10.3.20.0/24, viz `sit.md`]`
- **Hardware (z dashboardu):** 12 jader, 31.2 GiB RAM (= 32 GB, viz `hardware.md`),
  root disk `/` 94 GB (využito ~13 %), teplota sda ~47 °C.
- **Přístup:**
  - SSH: `ssh root@10.3.20.186` (funguje z pracovních strojů Daniela).
    - ⚠️ Z **Docker VM** (10.3.20.199) tento přístup **nefunguje** (Permission denied,
      publickey/password) — klíč na Docker VM není. Proxmox příkazy proto pouští Daniel ručně.
  - Mobilně: Termius (ed25519 klíče deployované přes Proxmox web konzoli)

## Pravidlo
`qm` / `pct` příkazy běží JEN tady. Před exekucí ověř `hostname` → `pve`.
