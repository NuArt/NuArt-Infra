# Ollama (lokální AI)

> Ověřeno 2026-06-30 (ping 10.3.20.154, `GET /api/tags`).

## Kde běží
- LXC 101 na Proxmoxu, **GPU passthrough** (GTX 1070, 8 GB VRAM)
- **API:** `http://10.3.20.154:11434` (REST, bez auth, jen LAN) ✅ odpovídá
- ✅ **IP `.154` zafixována** DHCP rezervací na UniFi (2026-06-30) — historický problém se
  změnou IP (.153 → .154) je vyřešen, už se nebude opakovat. Viz `incidenty/ollama-ip-dhcp.md`.

## Modely (ověřeno — jen TEXT)
- `qwen3:8b` — hlavní pracant (8.2B, Q4, context 40960, umí česky, tools, thinking). ~30 tok/s na GPU.
- `qwen3:4b` — menší, ale context 262144 (obří) — když je potřeba nacpat hodně dat do kontextu.

`[opraveno proti realitě 2026-06-30: dump zmiňoval „qwen3:8b/4b"; ověřeno přesně tyto dva modely]`

## Co NEjde
Vision/OCR na GTX 1070 nefunguje (8 GB Pascal nestačí) — viz `incidenty/vision-ocr-slepa-ulicka.md`
a `decisions/009-ai-jen-text-modely.md`. PDF/skeny → cloud OCR, ne lokální.

## Konzumenti
- Open WebUI (`stacks/openwebui.md`)
- Nicotrans AI pipeline (`ai/nicotrans-ai.md`)
