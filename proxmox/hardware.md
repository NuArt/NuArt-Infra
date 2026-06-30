# Hardware — fyzický stroj

> Zdroj: knowledge-dump. **Tato úroveň nebyla 2026-06-30 ověřena** (z Docker VM není
> SSH na `pve`). Ověř přímo na `pve` (`dmidecode`, `lspci`, `free -h`).

## Sestava
- **Deska:** ASUS Z10PA-U8
- **CPU:** Intel Xeon E5-1650 v4 (Broadwell, 6 jader / 12 threadů, AVX + AVX2)
- **RAM:** 32 GB (dashboard pve: 31.2 GiB) ✅ ověřeno 2026-06-30.
  `[opraveno proti realitě 2026-06-30: plánovaná výměna na 64 GB (4×8GB ECC RDIMM 2400)
   ZATÍM NEPROBĚHLA — stále 32 GB. Zůstává jako TODO (vyžaduje odstávku)]`
- **GPU:** NVIDIA GTX 1070 8 GB (Pascal) — tvrdý strop pro AI úlohy (viz `decisions/009-ai-jen-text-modely.md`)
- **Disk:** SSD 1.92 TB

## ISP / veřejná konektivita
- **Veřejná IP:** 178.255.174.235 (ISP **Starnet**, 1:1 NAT) — ✅ ověřeno 2026-06-30 (office/crm.nuart.cz na ni rezolvují)
- **Gateway:** UniFi UCG-Fiber (.1)
- **WAN IP na gatewayi:** 10.6.123.174 (vnitřní WAN za ISP NAT → **double-NAT**)
- Detaily routingu/portů viz `sit.md`.

## Pozn. k AI stropu
GTX 1070 (8 GB Pascal) je limit pro lokální AI: text modely (qwen3) běží dobře, vision/OCR ne
(viz `incidenty/vision-ocr-slepa-ulicka.md`).
