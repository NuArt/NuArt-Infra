# ADR-002: Ollama v LXC, ne v Docker VM

**Rozhodnutí:** Lokální AI (Ollama) běží v samostatném LXC 101, ne v Docker VM.

**Proč:** GPU passthrough je čistší do LXC; izolace AI zátěže od produkčních appek.

**Historie:** Začínalo CPU-only (chyběl ovladač pro Pascal GTX 1070), později zprovozněn
GPU passthrough (driver 580.95.05 / CUDA 13).
