# OmniRoute - Report Stack Tecnologico

## 1. Executive Summary & Architettura Globale
OmniRoute (v3.8.53) è una piattaforma unificata ad alte prestazioni per il routing, proxying, orchestrazione e ottimizzazione di oltre 377 provider di intelligenza artificiale (LLM, Vision, Audio, Multimodal). Il sistema implementa un'architettura ibrida all-in-one concepita sia per il deployment cloud/self-hosted in container, sia come applicazione desktop locale autonoma (Electron / Bun / Node).

Le capacità core includono:
- **API Gateway & Reverse Proxy**: Interfaccia unificata compatibile con lo standard OpenAI (`/v1/chat/completions`, `/v1/embeddings`, ecc.) con fallback automatico a cascata, circuit breaker e bilanciamento del carico intelligente.
- **Protocolli Agenti Avanzati**: Supporto nativo completo per MCP (Model Context Protocol), ACP (Agent Client Protocol) e protocolli A2A (Agent-to-Agent).
- **Compressione & Ottimizzazione Token**: Pipeline integrate di tokenomics con algoritmi di compressione prompt (LLMLingua-2, RTK e filtri contestuali) per la riduzione della spesa e della latenza.
- **Streaming Robusto**: Workspace specializzato `open-sse` per gestione backpressure, retry resilienti, stream state caching e quota preflight.
- **Ispezione & Man-In-The-Middle (MITM)**: Moduli integrati per intercettazione DNS, generazione certificati TLS on-the-fly e modalità transparent proxy (`tproxy`).

---

## 2. Runtime, Linguaggi ed Ecosistema
- **Linguaggio Principale**: TypeScript 6.0.3 (`typescript@^6.0.3`) con target strict e interoperabilità ESM/CJS completa.
- **Runtimes Supportati**:
  - **Node.js**: Specificato in engine `package.json` come `>=22.22.2 <23 || >=24.0.0 <27`. Target di riferimento nei container: Node v26.12.1 LTS.
  - **Bun**: Engine `bun@1.4.2` pienamente supportato per esecuzione ad altissima velocità e packaging binario nativo (`scripts/release/bun-pack.mjs`).
- **Architettura Monorepo**: Gestione multi-package tramite `pnpm-workspace.yaml`, comprendente:
  - `open-sse`: Workspace dedicato alla pipeline Server-Sent Events e streaming ad alta resilienza.
  - `packages/browser-pool`: Gestore avanzato di pool browser Chromium headless per task di navigazione e web scraping.
  - `packages/ui-kit`: Libreria interna di componenti UI (`@omniroute/ui-kit@0.0.79`).
  - `@omniroute/*`: Plugin e provider di estensione per OpenCode (`opencode-plugin`, `opencode-plugin-v2`, `opencode-provider`).

---

## 3. Frontend Architecture & UI Stack
L'interfaccia utente è una dashboard reattiva moderna fornita da Next.js e integrabile anche all'interno della shell Desktop:

- **Framework Web**: Next.js 16.4.2 (App Router, Server Actions, Standalone Build).
- **Core UI Library**: React 19.2.8 (`react@19.2.8`, `react-dom@19.2.8`) con supporto avanzato per React Compiler (`babel-plugin-react-compiler@^1.0.0`).
- **Styling & CSS Engine**:
  - Tailwind CSS v4 (`@tailwindcss/postcss@^4.3.0`, `tailwindcss@^4.3.0`).
  - Utilità di composizione classi: `clsx@^2.1.1` e `tailwind-merge@^3.6.0`.
- **Component Primitives**:
  - Radix UI (`@radix-ui/react-dialog@1.1.23`, `@radix-ui/react-dropdown-menu@2.1.24`, `@radix-ui/react-popover@1.1.23`, `@radix-ui/react-select@2.3.7`, `@radix-ui/react-slot@1.3.3`, `@radix-ui/react-tooltip@1.2.16`).
  - `@omniroute/ui-kit`: Design system interno fondato sui token Keak Cheaper Inference (`ci-ui.css`, `ci-tokens.css`).
- **Componenti Specialistici & Visualizzazione**:
  - Diagrammi di flusso e grafi agenti: `@xyflow/react@^12.11.5`.
  - Grafici & metriche telemetria: `recharts@^3.8.1`.
  - Editor di codice integrato: `@monaco-editor/react@^4.7.0` e `monaco-editor@^0.56.0`.
  - Drag & Drop: `@dnd-kit/core@^6.3.1` e `@dnd-kit/sortable@^10.0.0`.
  - Documentazione in-app: Fumadocs (`fumadocs-core@^16.15.4`, `fumadocs-ui@^16.15.4`, `fumadocs-mdx@^15.4.0`).
  - Iconografia: `lucide-react@^1.33.0` e `@lobehub/icons@^5.16.0`.
- **State Management & Client Context**:
  - Store globale: `zustand@^5.0.15` in `src/store/`.
  - Internazionalizzazione: `next-intl@^4.14.0` in `src/i18n/`.
  - Gestione Temi (Dark/Light/System): `next-themes@^0.4.6`.

---

## 4. Backend & Gateway Routing Engine
Il livello di routing e proxying è un'infrastruttura ibrida scalabile progettata per basse latenze ed elevato throughput:

- **Server HTTP Ibrido**:
  - Express 5.2.1 (`express@^5.2.1`) come runtime server per middleware complessi e proxying ad alte prestazioni.
  - Hono 4.13.13 (`hono@^4.13.13`) per router ultraleggeri ed endpoint edge-compatible.
  - Next.js App Router API Routes (`src/app/api/`) suddivise in 105 domini applicativi distinti.
- **Reverse Proxy & Connettività**:
  - `http-proxy-middleware@^4.0.0`.
  - Client HTTP di livello enterprise: `undici@^8.11.2`, `axios@^1.20.0`.
  - Client nativo in Rust: `wreq-js@^3.2.0` (con binding binari compilati `wreq-js-native` per bypass di colli di bottiglia TLS e socket Node.js).
- **Realtime & Streaming**:
  - WebSocket: `ws@^8.21.3` con implementazione `src/server/ws/liveServer.ts` e bridge per multiplexing socket.
  - Pipeline SSE (`open-sse`): backpressure controller, buffer circolare per prevenzione OOM, chunking dinamico e preflight quota checks.
- **Sicurezza, Middleware & Route Guarding**:
  - Gestione Token & Crittografia: `jose@^6.2.10` (JWT/JWE), `bcryptjs@^3.0.3`.
  - Throttling & Rate Limiting: `bottleneck@^2.19.5`.
  - Guardie di sistema in `src/server/authz/`: CSRF protection, Route Guards multilivello, modalità trust-proxy, policy CORS modulari e Single Sign-On aziendale con Microsoft Entra ID (`src/server/authz/entra/`).

---

## 5. AI & LLM Provider Integration
OmniRoute integra un routing fabric in grado di interfacciare 377 provider:

- **Provider Supportati**:
  - Major Commercial Cloud: OpenAI, Anthropic Claude, Google Gemini / Vertex AI, AWS Bedrock (`@aws-sdk/client-bedrock-runtime@^3.1120.0`), Azure OpenAI, Mistral AI, Cohere, xAI Grok.
  - High-Speed / Open Weights: Groq, Together AI, Fireworks, DeepSeek, Cerebras, Sambanova.
  - Local & Self-Hosted: Ollama, vLLM, LM Studio, LocalAI.
- **Protocolli Standardizzati**:
  - MCP (Model Context Protocol): `@modelcontextprotocol/sdk@^1.31.0`.
  - ACP (Agent Client Protocol): `@agentclientprotocol/sdk@1.4.0`.
- **Tokenomics & Calcolo Token**:
  - `tiktoken@^1.0.22` e `js-tiktoken@^1.0.20` per calcolo esatto su famiglie GPT, Claude e Llama.
- **Inference Locale & Compressione Contestuale**:
  - Modelli ONNX locali: `onnxruntime-node@^1.30.0`.
  - Modelli Hugging Face in-process: `@huggingface/transformers@^4.2.0`.
  - Compressione semantica prompt: `@atjsh/llmlingua-2@^3.0.0`.
- **Politiche di Routing & Resilienza**:
  - Algoritmi di fallback intelligenti: switch su provider secondari o modelli equivalenti in caso di rate limit (HTTP 429), timeout o errore 5xx.
  - Concurrency management, health-check periodici e tracciamento quote per utente/chiave.

---

## 6. Storage, Database & Caching
La persistenza adotta una strategia multi-tier per garantire portabilità totale sia in desktop che in cluster enterprise:

- **Database Principale**:
  - `better-sqlite3@^13.0.3`: SQLite nativo ad alte prestazioni su disco locale per configurazioni, log, cronologia prompt e routing tables.
  - `sql.js@^1.14.2`: Fallback WebAssembly di SQLite in ambienti privi di toolchain C++ nativa.
- **Vector Search & Embedding Storage**:
  - `sqlite-vec@^0.1.9`: Estensione SQLite nativa per memorizzazione vettoriale e similarità semantica (k-NN / cosine distance) integrata nel database locale.
  - Supporto per Qdrant containerizzato come sidecar esterno per storage vettoriale su larga scala.
- **Caching Distribuito & State Storage**:
  - `ioredis@^5.10.1`: Per istanze cluster multi-nodo, lock distribuiti, sessioni condivise e rate-limit cache.
  - `lowdb@^7.0.1`: Persistenza lightweight JSON-based per preferenze applicative e state flags veloci.

---

## 7. Testing, Linting & QA
OmniRoute adotta una suite rigorosa di qualità a livelli multipli:

- **Test Runners**:
  - **Node.js Test Runner Nativo**: `node --test` eseguito in parallelo con loader TypeScript `tsx/esm` (`package.json`).
  - **Vitest**: `vitest@^5.0.0` con configurazioni modulari:
    - `vitest.config.ts`: test unitari frontend/servizi.
    - `vitest.mcp.config.ts`: suite di validazione protocollo Model Context Protocol.
    - `vitest.e2e-live.config.ts`: test di integrazione live verso gateway.
    - `vitest.quarantine.config.ts`: isolamento test flaky.
  - **Bun Test**: per validazione istantanea driver SQLite nativi.
- **Property-Based Testing**: `fast-check@^4.10.0` per verifica su input randomizzati e test di invarianti logici.
- **Component & UI Testing**: `@testing-library/react@^16.3.3`, `@testing-library/jest-dom@^7.0.1`, `jsdom@^30.0.1`.
- **E2E Testing**: `playwright@^1.62.1` per test completi end-to-end su browser Chromium, Firefox e WebKit.
- **Mutation Testing**: Stryker Mutator `10.0.0` (`@stryker-mutator/core`, `@stryker-mutator/tap-runner`) per audit di solidità delle asserzioni.
- **Coverage & Reporting**: `c8@^12.0.0`, `ctrf@^0.4.0`, `playwright-ctrf-json-reporter@^0.1.0`.
- **Code Quality & Linter**:
  - ESLint 10 (`eslint@^10.10.0`) con `typescript-eslint@^8.70.0`.
  - Regole AST personalizzate per interdizione pattern vietati o memory leaks.

---

## 8. Tooling, Build & Packaging
- **Package Manager**: `pnpm` (supporto pnpm workspace).
- **Frontend Bundler**: Turbopack & Next.js compiler per la dashboard web; Vite `^8.3.3` per tooling ausiliario e test.
- **Desktop Packaging**:
  - Electron 45 (`electron@^45.0.0`) configurato in `electron/`.
  - `electron-builder@^26.15.3` per la creazione di installer nativi Windows (.exe/NSIS), macOS (.dmg/.zip) e Linux (.AppImage/.deb).
- **Standalone Assembly**:
  - Script personalizzati `scripts/build/assembleStandalone.mjs` e `scripts/build/build-next-isolated.mjs` per la generazione di bundle server autonomi a zero dipendenze esterne.
  - Packaging binario Bun con `bun scripts/release/bun-pack.mjs`.
- **Compilazione Nativa**: `node-gyp` per build dei moduli transparent proxy (`src/mitm/tproxy/native`).

---

## 9. Infrastruttura & Deployment
- **Containerization**:
  - `Dockerfile`: Multi-stage build basato sull'immagine ufficiale `node:26.12.1-trixie-slim`.
  - `Dockerfile.bun`: Immagine ultra-ottimizzata su base Bun per container a footprint minimo.
- **Profili Docker Compose**:
  - `docker-compose.yml`: Configurazione standard di produzione.
  - `docker-compose.selfhost.yml`: Profilo self-hosted con volume persistente e SQLite.
  - `docker-compose.prod.yml`: Profilo ad alta disponibilità enterprise con Redis e monitoring.
  - `docker-compose.ollama-local.yml`: Stack completo integrato con istanza Ollama locale per inferenza su GPU/CPU.
  - Servizi sidecar: `docker/devin-bridge/` e `docker/vnc-browser/` (browser isolato accessibile via VNC).
- **PaaS & Serverless**: Configurazione pronta per il deploy Fly.io (`fly.toml`).
- **CI/CD Pipelines**: 36 workflow GitHub Actions sotto `.github/workflows/` (test matriciali, build multipiattaforma, container publishing, security scan e audit licenze).

---

## 10. Matrice delle Dipendenze Principali

| Categoria | Pacchetto | Versione Rilevata | Scopo Tecnologico |
|---|---|---|---|
| **Core Framework** | `next` | `16.4.2` | Framework full-stack per la dashboard e API routes |
| **UI Library** | `react` / `react-dom` | `19.2.8` | Interfaccia grafica reattiva |
| **Server Engine** | `express` | `^5.2.1` | HTTP Server e reverse proxy middleware engine |
| **Server Engine** | `hono` | `^4.13.13` | Router HTTP leggero ed edge-ready |
| **Language** | `typescript` | `^6.0.3` | Tipizzazione statica su tutto il monorepo |
| **Styling** | `tailwindcss` | `^4.3.0` | Framework CSS utility-first (v4 con PostCSS) |
| **UI Primitives** | `@radix-ui/*` | varie (1.1 - 2.3) | Componenti accessibili headless (dialog, select, dropdown) |
| **UI Kit Interno** | `@omniroute/ui-kit` | `0.0.79` | Design system aziendale e token UI |
| **State Management** | `zustand` | `^5.0.15` | Gestione stato globale frontend reattiva |
| **WebSocket** | `ws` | `^8.21.3` | Gestione comunicazioni bidirezionali realtime |
| **Reverse Proxy** | `http-proxy-middleware` | `^4.0.0` | Instradamento dinamico richieste HTTP |
| **Native HTTP** | `wreq-js` | `^3.2.0` | Client HTTP nativo ad altissime performance basato su Rust |
| **AI Protocols** | `@modelcontextprotocol/sdk` | `^1.31.0` | Supporto ufficiale al protocollo Model Context Protocol (MCP) |
| **AI Protocols** | `@agentclientprotocol/sdk` | `1.4.0` | Supporto al protocollo Agent Client Protocol (ACP) |
| **AWS Integration** | `@aws-sdk/client-bedrock-runtime` | `^3.1120.0` | Integrazione con modelli AWS Bedrock |
| **Tokenomics** | `tiktoken` | `^1.0.22` | Tokenizzatore BPE per calcolo esatto token OpenAI/LLM |
| **Local Inference** | `onnxruntime-node` | `^1.30.0` | Runtime ONNX per inferenza locale in Node |
| **Local Inference** | `@huggingface/transformers` | `^4.2.0` | Libreria Transformers per modelli Hugging Face in-process |
| **Prompt Optimization** | `@atjsh/llmlingua-2` | `^3.0.0` | Compressione semantica prompt per risparmio token |
| **Database** | `better-sqlite3` | `^13.0.3` | Driver SQLite embedded primario ad alta velocità |
| **Database Fallback** | `sql.js` | `^1.14.2` | Engine SQLite compilato in WebAssembly |
| **Vector Search** | `sqlite-vec` | `^0.1.9` | Estensione di ricerca vettoriale locale per SQLite |
| **Cache & Locks** | `ioredis` | `^5.10.1` | Client Redis per cluster distributed caching e rate limit |
| **Lightweight DB** | `lowdb` | `^7.0.1` | Persistenza dati locale leggera in formato JSON |
| **Security / Auth** | `jose` | `^6.2.10` | Firme crittografiche, JWT e token validation |
| **Rate Limiter** | `bottleneck` | `^2.19.5` | Throttling, task scheduling e token-bucket queues |
| **Desktop App** | `electron` | `^45.0.0` | Shell desktop multipiattaforma per l'applicazione |
| **Desktop Packaging** | `electron-builder` | `^26.15.3` | Costruzione installer eseguibili Windows, Mac, Linux |
| **Unit Testing** | `vitest` | `^5.0.0` | Framework di unit e integration testing ad alte prestazioni |
| **E2E Testing** | `playwright` | `^1.62.1` | Suite di automazione e testing end-to-end multi-browser |
| **Property Testing** | `fast-check` | `^4.10.0` | Framework di testing generativo basato su proprietà |
| **Mutation Testing** | `stryker-mutator` | `10.0.0` | Strumento di mutation testing per valutazione qualità test |
| **Linter** | `eslint` | `^10.10.0` | Analisi statica e garanzia standard di codice |
| **Bundler Ausiliario** | `vite` | `^8.3.3` | Bundler veloce per tooling, componenti e test |
