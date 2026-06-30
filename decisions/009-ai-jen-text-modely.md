# ADR-009: Lokální AI jen text modely (qwen3), ne vision/OCR

**Rozhodnutí:** Lokálně provozovat jen textové modely (qwen3). Vision/OCR ne.

**Proč:** GTX 1070 (8 GB Pascal) na vision nestačí — testováno granite3.2-vision /
qwen2.5-vl:7b / minicpm-v, všechny selhaly (kontext/VRAM/halucinace). Detaily v
`incidenty/vision-ocr-slepa-ulicka.md`.

**Architektura:** AI mapuje význam sloupců, pandas tahá data (viz `ai/nicotrans-ai.md`).
PDF/skeny → cloud OCR (samostatné řešení), ne lokální pipeline.
