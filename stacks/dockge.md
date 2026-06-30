# Stack: dockge

> Ověřeno 2026-06-30 (`docker ps`, `/opt/dockge`).

Webová správa Docker stacků.

- **Kontejner:** `dockge`, port **5001** (host 5001→5001)
- **Umístění:** běží z `/opt/dockge` (NE jako složka v `/opt/stacks/`)
  `[opraveno proti realitě 2026-06-30: dump řadil dockge mezi stacky v /opt/stacks/;
   reálně žije v /opt/dockge]`
- Spravuje stacky v `/opt/stacks/`.
- Jen LAN, nevystaveno veřejně.
