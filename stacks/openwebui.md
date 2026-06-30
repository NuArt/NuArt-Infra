# Stack: openwebui

> Ověřeno 2026-06-30 (`docker ps`).

## Co to je
Webové UI k lokální Ollama (LXC 101).

- **Kontejner:** `openwebui`, interní port **8080**, host mapping **3000** (3000→8080)
  `[opraveno proti realitě 2026-06-30: dump měl „interní port 3000, host mapping ?";
   reálně interní 8080, host 3000]`
- **Stack:** `/opt/stacks/openwebui/`
- **Napojení na Ollama:** `http://192.168.0.154:11434`
- Nevystaveno veřejně (jen LAN).

## Úskalí
- Změna Ollama IP (LXC DHCP: .153 → .154) rozbije spojení → ručně přepsat API URL v nastavení
  Open WebUI. Viz `incidenty/ollama-ip-dhcp.md`. Dlouhodobý fix: statická IP pro LXC 101.
