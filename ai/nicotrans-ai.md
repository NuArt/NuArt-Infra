# Nicotrans AI — column-mapping pipeline

> Shrnutí. Plné znění (ověřený kód, prompty, cílové schéma) viz
> [`_archiv/NICOTRANS_AI.md`](../_archiv/NICOTRANS_AI.md) — zkopírováno ze serveru
> (`/opt/stacks/nicotrans-palety/NICOTRANS_AI.md`) 2026-06-30.

## Cíl
Nicotrans dostává Excel/CSV od různých firem — **různá struktura, stejný sémantický obsah**
(datum, SPZ, řidič, palety složeno/naloženo, dluhy, vouchery, firmy). Pipeline:
1. pandas přečte nahraný soubor,
2. **AI (qwen3) namapuje sloupce** na jednotné cílové schéma,
3. **pandas vytáhne data** podle mapování,
4. zápis strukturovaných dat do systému.

## Klíčový princip dělby práce
- **AI dělá JEN pochopení významu sloupců** (mapování) — jednou za soubor/formát.
- **Kód (pandas) dělá vše ostatní** (čtení, tahání hodnot, validace, zápis).
- AI se **nedotýká hodnot dat** → žádné halucinace v datech.

## Infrastruktura
- Ollama `http://10.3.20.154:11434` (viz `ai/ollama.md`), modely `qwen3:8b` / `qwen3:4b`.
- Volání: `POST /api/generate`, `think: False`, `temperature: 0.1` (deterministické), `num_ctx` dle objemu.

## Důležité limity
- **Vision/OCR NEDĚLÁME** lokálně (GTX 1070 nestačí) — viz `incidenty/vision-ocr-slepa-ulicka.md`.
- Vždy validovat, že odpověď AI je platný JSON (model občas přidá text okolo).

## TODO (z archivu)
Definovat skutečné cílové schéma, postavit pipeline, cache mapování per-firma,
lidská kontrola mapování pro novou firmu, ošetření chyb (nevalidní JSON, Ollama nedostupná).
