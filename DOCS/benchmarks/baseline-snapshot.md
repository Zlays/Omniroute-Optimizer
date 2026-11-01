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