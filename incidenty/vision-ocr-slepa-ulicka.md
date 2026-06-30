# Incident: Vision/OCR na GTX 1070 — slepá ulička

## Co se zkoušelo a proč to nešlo
Cíl: lokální OCR/vision pro čtení PDF/skenů. GTX 1070 (8 GB Pascal) nestačí:
- **granite3.2-vision** — context jen 16k → nevejde se čitelný obrázek, nečte nic
- **qwen2.5-vl:7b** — potřebuje ~12,5 GB > 8 GB → teče na CPU → timeout (5+ min)
- **minicpm-v** — vejde se, ale 46 s/obrázek a **halucinuje čísla** (přečetl špatné číslo dokladu)

## Závěr
PDF/skeny přes lokální AI **NEDĚLÁME**. Jen strukturovaná data (Excel/CSV) přes textový model.
Pro PDF/skeny → samostatné řešení (cloud OCR).

Souvisí: `decisions/009-ai-jen-text-modely.md`, `ai/nicotrans-ai.md`, `_archiv/NICOTRANS_AI.md`.
