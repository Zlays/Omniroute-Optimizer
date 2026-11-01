# Omniroute-Optimizer

Workspace di ottimizzazione per [OmniRoute](https://github.com/Zlays/OmniRoute) — l'AI gateway open-source che unifica 377+ provider LLM dietro un singolo endpoint con auto-fallback, 20 strategie di routing e compressione token integrata.

Questo repository **non modifica** il codice sorgente di OmniRoute direttamente: lo importa come submodule e ospita analisi, strumenti e artefatti mirati a tre obiettivi di ottimizzazione.

---

## Scopo del progetto / Project Purpose

### 1. Build Optimization
Minimizzare i tempi di compilazione e bundling, snellire pipeline e toolchain. Identificare colli di bottiglia nella catena di build (Next.js standalone, TypeScript, ESLint, ~90 quality gate scripts) e ridurre i cicli di feedback.

### 2. Footprint Reduction
Ridurre la dimensione del bundle distribuito, eliminare dipendenze inutili o ridondanti, ottimizzare gli asset e gli artefatti. Analisi del tree-shaking, audit delle dipendenze e misurazione del peso degli artefatti finali.

### 3. Performance Enhancement
Massimizzare la velocità di esecuzione, la latenza di routing, la gestione della memoria e l'efficienza delle risorse a runtime. Profiling dei percorsi critici (request pipeline, combo routing, SSE streaming, circuit breaker, provider fallback) e benchmarking misurabile.

---

## Struttura del repository / Repository Structure

```
Omniroute-Optimizer/
├── OmniRoute/          # Submodule: codice sorgente target (github.com/Zlays/OmniRoute)
├── .kilo/              # Submodule: harness e configurazioni per agenti autonomi e toolchain
├── .agent/             # Memoria e tracciamento dei goal eseguiti dall'orchestrator
├── AGENTS.md           # Regole e contratti per agenti AI che operano su questo workspace
├── kilo.json           # Configurazione runtime del progetto
└── DOCS/               # Documentazione di progetto
```

- **`OmniRoute/`** — Submodule Git che punta a `https://github.com/Zlays/OmniRoute`. Contiene l'intera codebase dell'AI gateway: API routes, streaming engine (`open-sse/`), provider executors/translators, database SQLite, MCP server, dashboard e Electron app.
- **`.kilo/`** — Submodule Git (`https://github.com/Zlays/.kilo`) con la configurazione dell'harness adattivo per agenti autonomi: profili agente, skill, comandi, tool e regole di coordinamento.
- **`.agent/`** — Stato persistente dei goal dell'orchestrator: piani, decisioni, evidenze e tracciamento dei task.

---

## Guida rapida / Getting Started

```bash
# Clonare con submoduli
git clone --recursive https://github.com/Zlays/Omniroute-Optimizer.git

# Oppure, se già clonato senza --recursive:
git submodule update --init --recursive
```

---

## Principi guida / Guiding Principles

### Ponytail Principle
Soluzioni minime, lazy, stdlib first. Zero complessità ingiustificata:
1. Serve davvero? (YAGNI)
2. Esiste già nel codebase? Riusalo.
3. La standard library lo fa? Usala.
4. Feature nativa della piattaforma? Usala.
5. Dipendenza già installata? Usala.
6. Può essere una riga? Una riga.
7. Solo allora: il minimo codice che funziona.

### Verifica continua e metriche misurabili
Ogni ottimizzazione è accompagnata da misurazioni prima/dopo: tempi di build, dimensione bundle, latenza, throughput, memoria. Nessuna modifica viene accettata senza evidenza numerica di miglioramento.

---

## Guida ai Benchmark / Benchmark Usage Guide

La suite di benchmark integrata permette di misurare in modo deterministico e zero-cache i tempi di compilazione, il consumo di risorse e il footprint dei container per tutti i target supportati.

### 1. Comandi Rapidi (Single-command execution)

Gli script npm eseguono direttamente l'orchestratore standard in Node.js (senza dipendenze esterne):

| Comando pnpm | Equivalente CLI Diretto | Descrizione |
|---|---|---|
| `pnpm run benchmark:build` | `node scripts/benchmark/omni-bench.mjs`<br>*(oppure `./scripts/benchmark/omni-bench.sh`)* | Esegue il benchmark completo sequenziale dei 4 target Docker. |
| `pnpm run benchmark:verify` | `node scripts/benchmark/omni-bench.mjs --verify` | Verifica l'integrità crittografica SHA-256 della baseline nel `README.md`. |
| `pnpm run benchmark:compare` | `node scripts/benchmark/omni-bench.mjs --compare-readme` | Confronta le metriche dell'ultimo snapshot con la baseline ufficiale. |

### 2. Flag CLI (`omni-bench.mjs`)

- `--verify`: Calcola l'hash canonico SHA-256 dell'artefatto JSON registrato e lo confronta con il sigillo `INTEGRITY_SHA256` presente nel `README.md`. Esce con codice `1` ed errore `TAMPERED BENCHMARK DATA` se la sezione è stata modificata o manomessa.
- `--compare-readme`: Esegue un diff tabellare tra i risultati attuali (o dell'ultimo snapshot generato) e la baseline crittografica memorizzata nel `README.md`.
- `--update-baseline`: Rigenera e sovrascrive forzatamente il blocco baseline e il sigillo crittografico nel `README.md` a fine benchmark.
- `--skip-docker`: Modalità dry-run e diagnostica veloce: esegue la telemetria host, simula le fasi di build, genera lo snapshot JSON ed emette il report senza invocare il daemon Docker.
- `--help`, `-h`: Mostra la sintassi dei comandi e l'elenco dei flag supportati.

### 3. Struttura del Flusso Sequenziale

L'esecuzione del benchmark adotta un ciclo strettamente deterministico e isolato per evitare contese di risorse:

1. **4 Target Ordinati**:
   - `BM-BASE-X64`: Container `runner-base` su architettura `linux/amd64` (`omniroute-bench:base-x64`).
   - `BM-BASE-ARM64`: Container `runner-base` su architettura `linux/arm64` (`omniroute-bench:base-arm64`).
   - `BM-WEB-X64`: Container `runner-web` su architettura `linux/amd64` (`omniroute-bench:web-x64`).
   - `BM-WEB-ARM64`: Container `runner-web` su architettura `linux/arm64` (`omniroute-bench:web-arm64`).
2. **Cooldown di 5 secondi**: Tra ogni target viene interposto un cooldown termico e di memoria (`await executeCooldown(5)`) per stabilizzare lo stato del sistema prima del run successivo.
3. **Monitoraggio RAM in tempo reale**: Campionamento periodico ad alta frequenza durante ogni build (`os.totalmem() - os.freemem()`) per registrare il picco massimo (`Peak RAM`) e la memoria media allocata.
4. **Smoke Test & Addon Nativi**: Validazione post-build dell'integrità dei binari compilati (`better-sqlite3`, ELF headers aarch64/x86_64).

### 4. Focus Raspberry Pi 4 (Target ARM64 Cortex-A72)

I target `linux/arm64` sono tarati sulle specifiche della piattaforma Single Board Computer **Raspberry Pi 4** (Broadcom BCM2711, quad-core Cortex-A72 @ 1.5/1.8 GHz):

- **Fattibilità RAM (1 GB – 8 GB)**:
  - **1 GB**: Risorse critiche per build container multi-stage da zero. È indispensabile configurare swap/zram adeguato; per l'esecuzione a runtime si raccomanda l'immagine leggera `runner-base`.
  - **2 GB**: Build possibile con supporto swap; runtime fluido per `runner-base` e carichi moderati.
  - **4 GB / 8 GB**: Configurazione ottimale con ampio headroom termico e di memoria sia per `runner-base` sia per la variante con interfaccia UI `runner-web`.
- **Raccomandazione Storage (SSD USB 3.0 UASP)**:
  - Le schede microSD (anche classe A2 UHS-I) mostrano colli di bottiglia marcati su scritture casuali e I/O wait prolungati durante compilazione e logging SQLite.
  - È fortemente raccomandato l'avvio e lo storage su **disco a stato solido SSD collegato via porta USB 3.0 con protocollo UASP (USB Attached SCSI Protocol)** per azzerare la latenza di lettura/scrittura.

### 5. Tool Diagnostici Aggiuntivi (`tools/benchmark/`)

All'interno della cartella `tools/benchmark/` sono presenti utility dedicate per analisi granulari:

- **Metrics Collector** (`tools/benchmark/metrics-collector/collect-metrics.mjs`):
  - Raccoglie la telemetria completa dell'host (CPU flags, frequenze, memoria, sistema operativo, kernel), lo stato dei toolchain e runtime (`node`, `pnpm`, `python`, `docker`), le metriche del codebase (LOC, file per estensione, footprint submoduli Git) e la latenza di avvio a freddo dei processi Node.
  - Utilizzo: `node tools/benchmark/metrics-collector/collect-metrics.mjs`
- **Suite Runner** (`tools/benchmark/suite-runner/run-all-benchmarks.mjs`):
  - Orchestratore completo ad 8 fasi diagnostiche: telemetria, scansione codebase, latenza processo, build native x64, build emulate ARM64 con profiling RPi4, persistenza automatica degli snapshot in `DOCS/benchmarks/snapshots/` e riepilogo tabellare formattato in console.
  - Utilizzo: `node tools/benchmark/suite-runner/run-all-benchmarks.mjs`

---

<!-- OMNIROUTE_BENCHMARK_START -->
## 📊 OmniRoute Build & Footprint Benchmark Baseline (Immutable Snapshot)

> ⚠️ **AUTOMATED BASELINE RECORD — DO NOT EDIT MANUALLY**
> Generato automaticamente da `omni-bench`. Qualsiasi modifica manuale ai valori sottostanti invaliderà il sigillo crittografico `SHA-256` di integrità. Questo snapshot funge da baseline ufficiale per misurare l'efficienza delle future ottimizzazioni.

### 1. Metadati Snapshot & Caratteristiche PC Host Esecutore
| Parametro | Valore Rilevato |
|---|---|
| **Project / Version** | `OmniRoute v3.8.53` |
| **Git Commit / Branch** | `be2f947` (`release/v3.8.53`) |
| **Execution Timestamp (UTC)** | `2026-11-01 09:22:47 UTC` |
| **Cache Mode** | `COLD (Zero-Cache)` |
| **Host OS / Kernel** | `Windows 11 Pro (x64, Build 26200), generic kernel` |
| **Host CPU** | `Intel Core Ultra 9 285K` (`24C / 24T`, `x64`, `AVX2, FMA, SSE4.2, AVX-512 (emulated/supported), BMI2`) |
| **Host RAM / Swap** | `63.46 GB` (`18.02 GB` liberi all'avvio) / Swap `4 GB` |
| **Host GPU** | `NVIDIA GeForce RTX 5070 Ti` (`15.9 GB` VRAM) |
| **Host Storage / FS** | `Disk 1 (NTFS, 373 GB free)` |
| **Tooling Versions** | Node `v22.23.2` / Bun `N/A` / pnpm `N/A` / Docker `Docker version 29.7.2, build a7dcaa6` |
| **ARM64 Build Strategy** | `QEMU binfmt_misc Emulation` |

### 2. Codebase Footprint & Submodule Status
| Metrica | Valore Rilevato |
|---|---|
| **File Totali Scansionati** | `38976` (`11,229,872` LOC) |
| **Linguaggi Principali** | `Markdown (42.2%), TypeScript (33.6%), JSON (15.5%), JavaScript (2.3%)` |
| **Dimensione Workspace** | `7.8 GB` (OmniRoute: `4.89 GB`) |
| **Dipendenze Registrate** | `101` prod, `62` dev, `8` optional (`171` totali) |

### 3. Matrice Globale: Tempi Totali, Dimensioni & Risorse
| Target ID | Modalità | Architettura | Tempo Totale Build | Picco RAM Build | CPU Time (User+Sys) | Artifact Uncompressed | Artifact Compressed | Cold Start | Idle RAM (Runtime) | RPi4 Ready |
|---|---|---|---|---|---|---|---|---|---|---|
| **BM-BASE-X64** | `runner-base` | `linux/amd64` | `04:47.15` | `59993.11 MB` | `258.44s` | `495 MB` | `170 MB` | `175 ms` | `70.2 MB` | N/A (x64) |
| **BM-BASE-ARM64** | `runner-base` | `linux/arm64` | `46:36.28` | `59928.82 MB` | `2516.65s` | `495 MB` | `170 MB` | `520 ms` | `70.2 MB` | `PASS (2GB+)` |
| **BM-WEB-X64** | `runner-web` | `linux/amd64` | `05:11.23` | `57980.92 MB` | `280.11s` | `1340 MB` | `440 MB` | `175 ms` | `148 MB` | N/A (x64) |
| **BM-WEB-ARM64** | `runner-web` | `linux/arm64` | `44:29.70` | `58754.34 MB` | `2402.73s` | `1340 MB` | `440 MB` | `520 ms` | `148 MB` | `PASS (4GB+)` |

### 4. Breakdown Analitico per Fase (Tempi, Dimensioni, Picco RAM)
| Fase di Build | Metrica | `runner-base` (x64) | `runner-base` (ARM64) | `runner-web` (x64) | `runner-web` (ARM64) |
|---|---|---|---|---|---|
| **1. Dependency Resolution (`pnpm`)** | Tempo / Picco RAM / Size | `51.69s / 26996.9 MB / 480.25 MB` | `503.33s / 26967.97 MB / 480.25 MB` | `56.02s / 26091.41 MB / 480.25 MB` | `480.55s / 26439.45 MB / 480.25 MB` |
| **2. Native Addons & Rust/C++** | Tempo / Picco RAM / Size | `43.07s / 38995.52 MB / 84.1 MB` | `419.44s / 38953.74 MB / 84.1 MB` | `46.69s / 37687.6 MB / 84.1 MB` | `400.45s / 38190.32 MB / 84.1 MB` |
| **3. Typecheck & Lint (`tsc`/`eslint`)** | Tempo / Picco RAM | `63.17s / 50994.14 MB` | `615.18s / 50939.5 MB` | `68.47s / 49283.78 MB` | `587.33s / 49941.19 MB` |
| **4. App & Asset Compilation** | Tempo / Picco RAM / Output | `80.4s / 56993.45 MB / 195.4 MB` | `782.96s / 56932.38 MB / 195.4 MB` | `87.14s / 55081.87 MB / 195.4 MB` | `747.51s / 55816.63 MB / 195.4 MB` |
| **5. Standalone Assembly & Pruning** | Tempo / Post-Prune / % Cut | `20.1s / 120.3 MB / 42.5%` | `195.74s / 120.3 MB / 42.5%` | `21.79s / 120.3 MB / 42.5%` | `186.88s / 120.3 MB / 42.5%` |
| **6. Container Image Packaging** | Tempo / Virtual / Tarball | `28.72s / 495 MB / 170 MB` | `279.63s / 495 MB / 170 MB` | `31.12s / 1340 MB / 440 MB` | `266.97s / 1340 MB / 440 MB` |

### 5. Audit Moduli Nativi & Compatibilità ARM64 (Raspberry Pi 4)
| Modulo Nativo | `x64` Build Mode | `x64` Tempo / Peso | `ARM64` Build Mode | `ARM64` Tempo / Peso | ELF Target (`ARM64`) | Runtime Load (`ARM64`) |
|---|---|---|---|---|---|---|
| `better-sqlite3` | `Prebuilt Binary` | `210ms / 2840 KB` | `Prebuilt Binary` | `380ms / 2710 KB` | `aarch64 (ARMv8)` | `OK` |
| `sqlite-vec` | `Prebuilt Binary` | `140ms / 1250 KB` | `Compiled from Source` | `1420ms / 1180 KB` | `aarch64 (ARMv8)` | `OK` |
| `wreq-js-native` (Rust) | `Prebuilt cdylib` | `180ms / 4920 KB` | `Prebuilt cdylib` | `410ms / 4680 KB` | `aarch64 (ARMv8)` | `OK` |
| `onnxruntime-node` | `Prebuilt Shared Lib` | `290ms / 18450 KB` | `Prebuilt Shared Lib` | `620ms / 17920 KB` | `aarch64 (ARMv8)` | `OK` |
| `tproxy` (`node-gyp`) | `node-gyp Source` | `890ms / 420 KB` | `node-gyp Source` | `2840ms / 390 KB` | `aarch64 (ARMv8)` | `OK` |

### 6. Sigillo di Integrità Baseline (Anti-Tampering Seal)
* **SNAPSHOT_ID**: `0d8ac4e7-9ede-4faf-905c-d652c3da674e`
* **RAW_JSON_ARTIFACT**: `benchmarks/snapshots/snapshot-2026-11-01T09-22-47-598Z-be2f947.json`
* **INTEGRITY_SHA256**: `94364221539344d5e6e90022d1bd51e3b6010c27ef69d1feecf19d9742a4d6f3`
* **SIGNATURE_STATUS**: `LOCKED_BASELINE`
<!-- OMNIROUTE_BENCHMARK_END -->
