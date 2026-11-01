# Project Support Tools & Utilities (./tools)

This directory contains project-specific diagnostic, analysis, domain, and helper utilities created specifically for **Omniroute-Optimizer**.

> **Scope Partition**:
> - **Global / Generic Tools**: Reusable across any engineering project (e.g. skills downloader, git sync, generic validators) belong in `.kilo/tools/`.
> - **Project-Specific Tools**: Domain-specific scripts (e.g. routing heuristics, distance matrix processors, VRP/TSP benchmark harnesses, dataset generators) belong here in `./tools/`.

---

## 1. Directory Structure Standard

All project-specific tooling scripts must follow the hierarchical category-based structure:

```
tools/
├── README.md
├── {category}/
│   └── {tool-name}/
│       ├── {script_name}.py (or .ts, .sh, etc.)
│       └── output/ <-- Temporary outputs (gitignored)
```

### Established Categories:
- `routing/`: VRP/TSP route analyzers, heuristics inspectors, distance/duration matrix validators.
- `data/`: Synthetic dataset generators, geojson/OSM loaders, customer demand synthesizers.
- `benchmark/`: Solver throughput/latency benchmarks, memory profiling for route computations.
- `inspection/`: Project stack checks, database/cache state validators, API endpoint probes.

---

## 2. Tools Catalog

| Category | Tool Name | Script | Description |
|---|---|---|---|
| `benchmark` | `metrics-collector` | `tools/benchmark/metrics-collector/collect-metrics.mjs` | Extracts hardware specs, runtime environment, codebase footprint, LOC breakdown, startup benchmark, and Docker runner targets (`runner-base` vs `runner-web`). Supports `--no-cache`, `--platform` (multi-arch `linux/arm64`), `--clean`, and Raspberry Pi 4 ARM64 feasibility audit (`--rpi4`). Zero-dependency Node.js CLI. |
| `benchmark` | `suite-runner` | `tools/benchmark/suite-runner/run-all-benchmarks.mjs` | Sequentially executes complete benchmark suite across native host and ARM64/RPi4 with real-time RAM sampling (200ms interval, no timeout). Generates executive summary console table, full markdown report (`DOCS/benchmarks/benchmark-complete-suite.md`), and JSON snapshot. Supports `--clean`, `--skip-docker`, `--help`. |
| `benchmark` | `omni-bench` | `scripts/benchmark/omni-bench.mjs` | Master benchmark CLI orchestrator executing sequential zero-cache builds across 4 target profiles (`BM-BASE-X64`, `BM-BASE-ARM64`, `BM-WEB-X64`, `BM-WEB-ARM64`) with 5s cooldown. Profiles 6 build phases, runtime smoke tests, and injects cryptographic anti-tampering SHA-256 baseline into `README.md`. Supports `--verify`, `--compare-readme`, `--update-baseline`, `--skip-docker`, `--help`. |

---

## 3. Tooling Guidelines & Rules

1. **Isolation & Modularity**: Each tool resides in its own folder `./tools/{category}/{tool-name}/`.
2. **Output Handling & Cleanup**:
   - Any generated report, dump, or test output must be written to `./tools/{category}/{tool-name}/output/`.
   - The `output/` directories and output artifacts (`*.out`, `*.tmp`, `*.log`) are globally gitignored.
   - Preserve executor raw proof in assigned root `.agent/goals/<goal-id>/artifacts/<task-id>/` before cleaning scratch.
3. **Parameterization & Ponytail Principle**:
   - Tools must accept CLI arguments (flags, input paths, output paths, `--help`) using standard argument parsers (`argparse` in Python, standard flags in shell/Node).
   - Use standard library first; no unapproved third-party dependencies.
4. **Catalog Maintenance**:
   - Whenever a new project-specific tool is added or modified, update this `README.md` catalog so that `tool-engineer` and `goal-orchestrator` can discover and reuse existing tools.
