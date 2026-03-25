# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

MET4MORFOSES Web Edition — a Next.js 16 App Router site that presents an MFA thesis as an immersive multi-mode web experience. The thesis content (Ovid's Metamorphoses reinterpretation) is stored as canonical PDFs in `public/mirror/` and ingested through a build pipeline into markdown and JSON data files.

## Commands

```bash
npm run dev              # Start dev server (Turbopack)
npm run build            # Full content pipeline + Next.js production build
npm run lint             # ESLint (next/core-web-vitals + TypeScript)
npm run typecheck        # tsc --noEmit (strict mode)
npm test                 # Vitest unit + integration tests
npm run test:unit        # Unit tests only
npm run test:integration # Integration tests only
npm run test:e2e         # Playwright E2E (desktop Chrome + mobile iPhone 13, port 3007)
npm run content:build    # Run full content pipeline (ingest → mirror → nodes → qa → integrity)
npm run build:analyze    # Production build with bundle analyzer
```

Run a single test file: `npx vitest run tests/unit/node-map-data.test.ts`

Run a single E2E test: `npx playwright test tests/e2e/site.spec.ts`

The content pipeline (`content:build`) uses `tsx` to run scripts that require `pdftotext` (poppler-utils) on the system. Install via `brew install poppler` (macOS) or `sudo apt-get install poppler-utils` (Linux).

## Architecture

### Content Pipeline (build-time, `scripts/`)

Source PDFs in `public/mirror/2018-03-20 - met4 - sixth draft/` flow through five sequential scripts:

1. **ingest-canonical** → extracts PDF text → `src/content/*.md` markdown files
2. **build-mirror-manifest** → catalogs all `public/mirror/` files with SHA-256 → `src/data/mirror-manifest.json`
3. **build-node-map-data** + **mock-trending-nodes** → generates narrative graph + feed data → `src/data/node-map.json`, `src/data/feed-items.json`
4. **qa-canonical-fidelity** → validates text extraction quality, enforces ≤8% delta → `src/data/canonical-fidelity-report.json`
5. **verify-mirror-integrity** → checks data invariants → `src/data/data-integrity-report.json`

Additional content scripts (not part of `content:build`): `analyze-style-dna`, `ingest-evolution`, `generate-cycle-4`, `generate-full-corpus`, `map-visual-artifacts`, `compare-fidelity`, `export-publishable`, `broadcast-signal`.

`npm run build` automatically runs `content:build` before `next build`.

### Generated Data (`src/data/`)

All JSON files in `src/data/` are pipeline-generated artifacts — do not edit manually. Key files: `node-map.json`, `feed-items.json`, `mirror-manifest.json`, `canonical-fidelity-report.json`, `data-integrity-report.json`, `processed-content.json`, `evolution-map.json`, `style-dna.json`, `ontology.json`, `artifacts.json`.

### Viewing Modes (routes)

| Route | Mode | Component |
|---|---|---|
| `/` | Mythic Node Map | `node-map-experience.tsx`, `node-map-2d.tsx` |
| `/feed` | Faux Social Feed | `feed-view.tsx` |
| `/scroll` | Three-Cycle Scroll | `scroll-view.tsx`, `scroll-memory.tsx` |
| `/read/[docSlug]` | Canonical Reader | `reader-sections.tsx`, `reader-nav.tsx`, `reader-progress.tsx` |
| `/oracle` | AI Oracle | `oracle-experience.tsx` |
| `/evolution/[slug]` | Draft Evolution | `evolution-view.tsx` |
| `/archive` | Mirror Downloads | `archive-browser.tsx` |
| `/about` | Portfolio Context | static page |

Mode navigation (Node Map, Feed, Scroll, Oracle) is handled by `mode-nav.tsx` in the layout. Archive and About are in a secondary nav.

### Server vs Client Components

All page routes are server components. Almost all components in `src/components/` are client components (`"use client"`) — they handle interactivity, animations, and browser APIs. Server components load data from `src/lib/` (which reads JSON/markdown via `fs` or direct imports) and pass it as props to client components.

### Domain Libraries (`src/lib/`)

- **content.ts** — loads canonical docs from `processed-content.json` and evolution data from `evolution-map.json` (server-side)
- **nodes.ts** — loads node map and feed items from generated JSON
- **mirror.ts** — loads mirror manifest, filtering, byte formatting
- **analytics.ts** — client-side PostHog init + event tracking via `/api/analytics` (uses `sendBeacon`)
- **content-engine/** — text normalization, section parsing, slug generation (used by build scripts and server lib)
- **codex/** — AI text generation engine: `analyzer.ts` (StyleDNA extraction), `generator.ts`, `ai-engine.ts` (SmartGenerator with ontology-aware entity transforms)
- **audio/synth.ts** — Web Audio API generative soundscape (`GlitchSynth` class, singleton `globalSynth`)

### Hooks (`src/hooks/`)

- **use-glitch-synth.ts** — React wrapper for the audio synth
- **use-mvs-agent.ts** — MVS system agent state
- **use-node-navigation.ts** — node map interaction logic

### Type System (`src/types/`)

- **content.ts** — `CanonicalDoc`, `CanonicalSection`, `VersionData`, `EvolutionaryDoc`, `CommentaryEntry`, `NarrativeNode`, `FeedItem`, `MirrorAsset`, `AnalyticsEvent`
- **api.ts** — API response types for analytics and manifest endpoints

### APIs

- `POST /api/analytics` — proxies analytics events to PostHog server-side
- `GET /api/manifest/canonical` — returns canonical document manifest
- `GET /api/manifest/mirror` — returns mirror asset manifest

### 3D / Immersive

The node map uses React Three Fiber (`@react-three/fiber`, `@react-three/drei`) with optional XR support (`@react-three/xr`). The `sensorium/chrono-helix.tsx` component renders 3D content. Tests mock WebGL and R3F via `vitest.setup.tsx` (ResizeObserver polyfill, WebGL2 mock, component stubs).

### Styling

Tailwind CSS v4 with `@tailwindcss/postcss`. Global styles in `globals.css` use CSS custom properties for theming (dark palette: `--bg`, `--ink`, `--accent-1/2/3`, `--card`, `--panel`). Three Google Fonts loaded via CSS variables: `--font-headline` (Barlow Condensed), `--font-body` (Space Grotesk), `--font-literary` (Cormorant Garamond).

## Conventions

- Path alias: `@/*` → `src/*`
- 2-space indentation, semicolons, double quotes
- Kebab-case filenames, PascalCase component names
- Conventional Commits (`feat:`, `fix:`, `chore:`)
- Tests live in `tests/unit/`, `tests/integration/`, `tests/e2e/` (not colocated with source)
- Vitest uses jsdom environment with `next/link` and `next/navigation` mocked in `vitest.setup.tsx`
- Playwright runs on port 3007 with desktop Chrome and mobile iPhone 13 projects
- `public/mirror/**` files are canonical input artifacts — never edit directly
- `src/data/*.json` files are pipeline-generated — never edit directly
- Analytics gracefully degrades when PostHog keys are absent

## Environment

Copy `.env.example` to `.env.local`: `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`, `POSTHOG_KEY`, `POSTHOG_HOST`. All optional — analytics is a no-op without them.

## CI

`.github/workflows/ci.yml` runs lint, typecheck, unit-integration, and build on push to main and PRs (Node 20, installs poppler-utils for build). E2E runs separately via `.github/workflows/e2e.yml` when PR has `e2e` label. Required status checks: `lint`, `typecheck`, `unit-integration`, `build`.

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** ORGAN-II (Art) | **Tier:** standard | **Status:** GRADUATED
**Org:** `organvm-ii-poiesis` | **Repo:** `ivi374ivi027-05`

### Edges
- **Produces** → `unspecified`: creative-artifact
- **Consumes** ← `ORGAN-I`: theory-artifact

### Siblings in Art
`core-engine`, `performance-sdk`, `example-generative-music`, `metasystem-master`, `example-choreographic-interface`, `showcase-portfolio`, `archive-past-works`, `case-studies-methodology`, `learning-resources`, `example-generative-visual`, `example-interactive-installation`, `example-ai-collaboration`, `docs`, `a-mavs-olevm`, `a-i-council--coliseum` ... and 16 more

### Governance
- Consumes Theory (I) concepts, produces artifacts for Commerce (III).

*Last synced: 2026-03-25T22:27:09Z*

## Session Review Protocol

At the end of each session that produces or modifies files:
1. Run `organvm session review --latest` to get a session summary
2. Check for unimplemented plans: `organvm session plans --project .`
3. Export significant sessions: `organvm session export <id> --slug <slug>`
4. Run `organvm prompts distill --dry-run` to detect uncovered operational patterns

Transcripts are on-demand (never committed):
- `organvm session transcript <id>` — conversation summary
- `organvm session transcript <id> --unabridged` — full audit trail
- `organvm session prompts <id>` — human prompts only


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | prompting-standards | Prompting Standards |
| system | any | research-standards-bibliography | APPENDIX: Research Standards Bibliography |
| system | any | phase-closing-and-forward-plan | METADOC: Phase-Closing Commemoration & Forward Attack Plan |
| system | any | research-standards | METADOC: Architectural Typology & Research Standards |
| system | any | sop-ecosystem | METADOC: SOP Ecosystem — Taxonomy, Inventory & Coverage |
| system | any | autonomous-content-syndication | SOP: Autonomous Content Syndication (The Broadcast Protocol) |
| system | any | autopoietic-systems-diagnostics | SOP: Autopoietic Systems Diagnostics (The Mirror of Eternity) |
| system | any | background-task-resilience | background-task-resilience |
| system | any | cicd-resilience-and-recovery | SOP: CI/CD Pipeline Resilience & Recovery |
| system | any | community-event-facilitation | SOP: Community Event Facilitation (The Dialectic Crucible) |
| system | any | context-window-conservation | context-window-conservation |
| system | any | conversation-to-content-pipeline | SOP — Conversation-to-Content Pipeline |
| system | any | cross-agent-handoff | SOP: Cross-Agent Session Handoff |
| system | any | cross-channel-publishing-metrics | SOP: Cross-Channel Publishing Metrics (The Echo Protocol) |
| system | any | data-migration-and-backup | SOP: Data Migration and Backup Protocol (The Memory Vault) |
| system | any | document-audit-feature-extraction | SOP: Document Audit & Feature Extraction |
| system | any | dynamic-lens-assembly | SOP: Dynamic Lens Assembly |
| system | any | essay-publishing-and-distribution | SOP: Essay Publishing & Distribution |
| system | any | formal-methods-applied-protocols | SOP: Formal Methods Applied Protocols |
| system | any | formal-methods-master-taxonomy | SOP: Formal Methods Master Taxonomy (The Blueprint of Proof) |
| system | any | formal-methods-tla-pluscal | SOP: Formal Methods — TLA+ and PlusCal Verification (The Blueprint Verifier) |
| system | any | generative-art-deployment | SOP: Generative Art Deployment (The Gallery Protocol) |
| system | any | market-gap-analysis | SOP: Full-Breath Market-Gap Analysis & Defensive Parrying |
| system | any | mcp-server-fleet-management | SOP: MCP Server Fleet Management (The Server Protocol) |
| system | any | multi-agent-swarm-orchestration | SOP: Multi-Agent Swarm Orchestration (The Polymorphic Swarm) |
| system | any | network-testament-protocol | SOP: Network Testament Protocol (The Mirror Protocol) |
| system | any | open-source-licensing-and-ip | SOP: Open Source Licensing and IP (The Commons Protocol) |
| system | any | performance-interface-design | SOP: Performance Interface Design (The Stage Protocol) |
| system | any | pitch-deck-rollout | SOP: Pitch Deck Generation & Rollout |
| system | any | polymorphic-agent-testing | SOP: Polymorphic Agent Testing (The Adversarial Protocol) |
| system | any | promotion-and-state-transitions | SOP: Promotion & State Transitions |
| system | any | recursive-study-feedback | SOP: Recursive Study & Feedback Loop (The Ouroboros) |
| system | any | repo-onboarding-and-habitat-creation | SOP: Repo Onboarding & Habitat Creation |
| system | any | research-to-implementation-pipeline | SOP: Research-to-Implementation Pipeline (The Gold Path) |
| system | any | security-and-accessibility-audit | SOP: Security & Accessibility Audit |
| system | any | session-self-critique | session-self-critique |
| system | any | smart-contract-audit-and-legal-wrap | SOP: Smart Contract Audit and Legal Wrap (The Ledger Protocol) |
| system | any | source-evaluation-and-bibliography | SOP: Source Evaluation & Annotated Bibliography (The Refinery) |
| system | any | stranger-test-protocol | SOP: Stranger Test Protocol |
| system | any | strategic-foresight-and-futures | SOP: Strategic Foresight & Futures (The Telescope) |
| system | any | styx-pipeline-traversal | SOP: Styx Pipeline Traversal (The 7-Organ Transmutation) |
| system | any | system-dashboard-telemetry | SOP: System Dashboard Telemetry (The Panopticon Protocol) |
| system | any | the-descent-protocol | the-descent-protocol |
| system | any | the-membrane-protocol | the-membrane-protocol |
| system | any | theoretical-concept-versioning | SOP: Theoretical Concept Versioning (The Epistemic Protocol) |
| system | any | theory-to-concrete-gate | theory-to-concrete-gate |
| system | any | typological-hermeneutic-analysis | SOP: Typological & Hermeneutic Analysis (The Archaeology) |

Linked skills: cicd-resilience-and-recovery, continuous-learning-agent, evaluation-to-growth, genesis-dna, multi-agent-workforce-planner, promotion-and-state-transitions, quality-gate-baseline-calibration, repo-onboarding-and-habitat-creation, structural-integrity-audit


**Prompting (Anthropic)**: context 200K tokens, format: XML tags, thinking: extended thinking (budget_tokens)


## Ecosystem Status

- **delivery**: 0/1 live, 0 planned
- **content**: 1/2 live, 0 planned
- **marketing**: 0/1 live, 0 planned

Run: `organvm ecosystem show ivi374ivi027-05` | `organvm ecosystem validate --organ II`


## External Mirrors (Network Testament)

- **technical** (6): vercel/next.js, facebook/react, eslint/eslint, tailwindlabs/tailwindcss, microsoft/TypeScript +1 more

Convergences: 20 | Run: `organvm network map --repo ivi374ivi027-05` | `organvm network suggest`


## Entity Identity (Ontologia)

**UID:** `ent_repo_01KKKX3RVMM4K4VGY01RTTPNR5` | **Matched by:** primary_name

Resolve: `organvm ontologia resolve ivi374ivi027-05` | History: `organvm ontologia history ent_repo_01KKKX3RVMM4K4VGY01RTTPNR5`


## Live System Variables (Ontologia)

| Variable | Value | Scope | Updated |
|----------|-------|-------|---------|
| `active_repos` | 64 | global | 2026-03-25 |
| `archived_repos` | 54 | global | 2026-03-25 |
| `ci_workflows` | 106 | global | 2026-03-25 |
| `code_files` | 0 | global | 2026-03-25 |
| `dependency_edges` | 60 | global | 2026-03-25 |
| `operational_organs` | 8 | global | 2026-03-25 |
| `published_essays` | 29 | global | 2026-03-25 |
| `repos_with_tests` | 0 | global | 2026-03-25 |
| `sprints_completed` | 33 | global | 2026-03-25 |
| `test_files` | 0 | global | 2026-03-25 |
| `total_organs` | 8 | global | 2026-03-25 |
| `total_repos` | 127 | global | 2026-03-25 |
| `total_words_formatted` | 0 | global | 2026-03-25 |
| `total_words_numeric` | 0 | global | 2026-03-25 |
| `total_words_short` | 0K+ | global | 2026-03-25 |

Metrics: 9 registered | Observations: 15536 recorded
Resolve: `organvm ontologia status` | Refresh: `organvm refresh`


## System Density (auto-generated)

AMMOI: 56% | Edges: 41 | Tensions: 33 | Clusters: 5 | Adv: 7 | Events(24h): 23754
Structure: 8 organs / 127 repos / 1654 components (depth 17) | Inference: 98% | Organs: META-ORGANVM:64%, ORGAN-I:55%, ORGAN-II:47%, ORGAN-III:55% +4 more
Last pulse: 2026-03-25T22:27:04 | Δ24h: +3.5% | Δ7d: n/a


## Dialect Identity (Trivium)

**Dialect:** AESTHETIC_FORM | **Classical Parallel:** Music | **Translation Role:** The Poetry — proves formal structures have sensory form

Strongest translations: III (structural), V (analogical), VI (analogical)

Scan: `organvm trivium scan II <OTHER>` | Matrix: `organvm trivium matrix` | Synthesize: `organvm trivium synthesize`

<!-- ORGANVM:AUTO:END -->


## ⚡ Conductor OS Integration
This repository is a managed component of the ORGANVM meta-workspace.
- **Orchestration:** Use `conductor patch` for system status and work queue.
- **Lifecycle:** Follow the `FRAME -> SHAPE -> BUILD -> PROVE` workflow.
- **Governance:** Promotions are managed via `conductor wip promote`.
- **Intelligence:** Conductor MCP tools are available for routing and mission synthesis.
