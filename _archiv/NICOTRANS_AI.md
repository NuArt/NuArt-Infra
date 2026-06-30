# AI v Nicotrans Palety — kontext pro Claude Code

> Tento dokument vysvětluje, jaká AI je k dispozici, kde běží, jak s ní komunikovat,
> a k čemu se má (a nemá) v Nicotransu používat. Čti před prací na AI funkcích.

---

## Co budujeme

Nicotrans dostává od různých firem **Excel/CSV soubory** s evidencí palet/voucherů.
Každá firma má **jinou strukturu** (jiné názvy sloupců, jiné pořadí), ale **stejný
sémantický obsah** (datum, SPZ, řidič, složeno/naloženo palet, dluhy, vouchery, firmy).

**Cíl:** automatická pipeline, která:
1. přečte nahraný Excel/CSV (libovolná firma, libovolná struktura)
2. pomocí AI **namapuje sloupce** na jednotné cílové schéma Nicotransu
3. pomocí kódu (pandas) **vytáhne data** podle toho mapování
4. zapíše strukturovaná data do systému

**Klíčový princip dělby práce:**
- **AI (qwen3)** dělá JEN to, co kód neumí → **pochopení významu sloupců** (mapování)
- **Kód (pandas)** dělá vše ostatní → čtení souboru, tahání hodnot, validace, zápis do DB
- AI NEzpracovává jednotlivé řádky dat — to je práce pandas (přesné, rychlé, zdarma)

---

## Kde AI běží (infrastruktura)

- **Ollama** běží v samostatném LXC kontejneru na Proxmoxu, s GPU passthrough (GTX 1070, 8 GB VRAM)
- **API endpoint:** `http://192.168.0.154:11434`
  - ⚠️ **POZOR: IP se může změnit** (LXC má DHCP). Když AI přestane odpovídat, ověř aktuální IP
    Ollama LXC v Proxmoxu. Doporučeno: dát LXC statickou IP (zatím není).
- **Komunikace:** REST API (`POST /api/generate`), JSON payload, žádný auth (interní síť)
- Nicotrans (Docker VM, 192.168.0.199) volá Ollamu po lokální síti

### Dostupné modely (jen TEXT modely!)
- **`qwen3:8b`** — hlavní pracant. 8.2B, Q4, context 40960, umí česky, tools, "thinking".
  Běží na GPU rychle (generování ~30 tok/s). Použij pro mapování sloupců a rozhodování.
- **`qwen3:4b`** — menší, ale context **262144** (obří!). Použij když potřebuješ nacpat
  hodně dat do kontextu najednou (velké tabulky, hodně řádků jako ukázka).

### ⚠️ Co NEjde (důležité, ověřeno testováním)
- **Vision/OCR NEFUNGUJE.** GTX 1070 (8 GB) je na vision modely málo:
  - granite3.2-vision: context jen 16k → nevejde se čitelný obrázek, nečte nic
  - qwen2.5-vl:7b: potřebuje 12.5 GB → teče na CPU → timeout (5+ min)
  - minicpm-v: vejde se, ale 46s/obrázek a **halucinuje čísla** (přečetl špatné číslo dokladu)
- **Závěr:** PDF/skeny přes lokální AI NEDĚLÁME. Jen strukturovaná data (Excel/CSV) přes text model.
- Pokud bude potřeba zpracovat i PDF/skeny → samostatné řešení (cloud OCR), ne tato pipeline.

---

## Jak s AI komunikovat (ověřený vzor)

```python
import requests, json

OLLAMA_URL = "http://192.168.0.154:11434/api/generate"

def zeptej_se_ai(prompt, model="qwen3:8b"):
    r = requests.post(OLLAMA_URL, json={
        "model": model,
        "prompt": prompt,
        "stream": False,
        "think": False,          # vypne "thinking" mód = rychlejší, čistší výstup
        "options": {
            "num_ctx": 4096,     # zvedni když posíláš víc dat (qwen3:8b max 40960)
            "temperature": 0.1   # nízká = deterministické, důležité pro mapování
        }
    }, timeout=300)
    return r.json()["response"]
```

**Důležité parametry:**
- `temperature: 0.1` — pro mapování/extrakci chceš deterministické výstupy, ne kreativitu
- `think: False` — qwen3 má "thinking" mód; pro strukturované úlohy ho vypni (rychlejší)
- `num_ctx` — default nízký; zvedni podle objemu dat (ale pozor na VRAM)
- vždy validuj, že odpověď je platný JSON (model občas přidá text okolo → ošetři parsování)

---

## Ověřený funkční vzor: mapování sloupců

Toto bylo OTESTOVÁNO a funguje (~3s generování na GPU, 7/9 polí trefeno přesně):

```python
# 1. Kód (pandas) přečte názvy sloupců z nahraného Excelu
sloupce_firmy = list(df.columns)  # ['datum','SPZ','řidič','složeno','naloženo',...]

# 2. AI namapuje na cílové schéma Nicotransu
prompt = f"""Jsi expert na mapování datových sloupců.
Tabulka od firmy má sloupce: {json.dumps(sloupce_firmy, ensure_ascii=False)}
Namapuj je na cílové schéma: {json.dumps(CILOVE_SCHEMA, ensure_ascii=False)}
Vrať POUZE JSON: {{"cilovy_nazev":"nazev_sloupce_firmy_nebo_null", ...}}"""
mapping = json.loads(zeptej_se_ai(prompt))

# 3. Kód (pandas) vytáhne data podle mapování — ŽÁDNÁ AI tady
for _, row in df.iterrows():
    zaznam = {cil: row[mapping[cil]] for cil in CILOVE_SCHEMA if mapping.get(cil)}
    # ... validace + zápis do Nicotransu
```

**Proč takto:** AI mapuje sloupce JEDNOU za soubor/formát (levné, rychlé).
Data samotná tahá pandas (100% přesné). AI se nedotýká hodnot → žádné halucinace v datech.

### Tipy pro lepší mapování (k doladění)
- Dej AI nejen názvy sloupců, ale i **pár řádků ukázkových dat** → líp pochopí význam
- Dej AI **popis cílových polí** (co přesně "voucher_pocet" znamená) → trefí i sporné sloupce
- U nejednoznačných sloupců (víc "voucher" sloupců) zvaž **lidskou kontrolu** mapování
  než se to zapíše (alespoň poprvé pro každou novou firmu)
- Mapování pro známou firmu si **ulož** (cache) → příště neptej AI znovu, použij uložené

---

## Cílové schéma Nicotransu (DOPLNIT)

> Toto je na tobě/Danielovi doplnit — jaká jsou skutečná cílová pole v Nicotransu,
> jejich názvy v DB, typy, a co znamenají. Příklad struktury:

```python
CILOVE_SCHEMA = {
    "datum":          {"typ": "date",    "popis": "datum transakce/dodání"},
    "spz_vozidla":    {"typ": "string",  "popis": "SPZ vozidla"},
    "jmeno_ridice":   {"typ": "string",  "popis": "jméno řidiče"},
    "palety_slozeno": {"typ": "int",     "popis": "počet složených palet"},
    "palety_nalozeno":{"typ": "int",     "popis": "počet naložených palet"},
    "dluh_palet":     {"typ": "int",     "popis": "saldo/dluh palet"},
    "voucher_pocet":  {"typ": "int",     "popis": "počet voucherů"},
    "voucher_firma":  {"typ": "string",  "popis": "firma voucheru (Albert/Lidl/...)"},
    "poznamka":       {"typ": "string",  "popis": "volná poznámka"},
}
```

---

## Co řešit s Claude Code dál
1. Definovat skutečné cílové schéma (napojení na existující DB Nicotransu)
2. Postavit pipeline: upload → pandas → AI mapping → pandas extrakce → validace → zápis
3. Cache mapování per-firma (ať se AI neptá pořád dokola)
4. Lidská kontrola/schválení mapování pro novou firmu (než se zapíše do ostrých dat)
5. Ošetření chyb: nevalidní JSON z AI, chybějící sloupce, Ollama nedostupná (IP změna)
