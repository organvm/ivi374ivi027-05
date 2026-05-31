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

*Last synced: 2026-05-23T00:26:31Z*

## Active Handoff Protocol

If `.conductor/active-handoff.md` exists, **READ IT FIRST** before doing any work.
It contains constraints, locked files, conventions, and completed work from the
originating agent. You MUST honor all constraints listed there.

If the handoff says "CROSS-VERIFICATION REQUIRED", your self-assessment will
NOT be trusted. A different agent will verify your output against these constraints.

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


## System Library

Plans: 269 indexed | Chains: 5 available | SOPs: 8 active
Discover: `organvm plans search <query>` | `organvm chains list` | `organvm sop lifecycle`
Library: `/Users/4jp/Code/organvm/praxis-perpetua/library`


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | atomic-clock | The Atomic Clock |
| system | any | execution-sequence | Execution Sequence |
| system | any | multi-agent-dispatch | Multi-Agent Dispatch |
| system | any | session-handoff-avalanche | Session Handoff Avalanche |
| system | any | system-loops | System Loops |
| system | any | prompting-standards | Prompting Standards |
| system | any | background-task-resilience | background-task-resilience |
| system | any | context-window-conservation | context-window-conservation |
| system | any | session-self-critique | session-self-critique |
| system | any | the-descent-protocol | the-descent-protocol |
| system | any | the-membrane-protocol | the-membrane-protocol |
| system | any | theory-to-concrete-gate | theory-to-concrete-gate |
| system | any | triangulation-protocol | triangulation-protocol |

Linked skills: SOP-TRIADIC-REVIEW-PROTOCOL, cicd-resilience-and-recovery, continuous-learning-agent, evaluation-to-growth, genesis-dna, multi-agent-workforce-planner, promotion-and-state-transitions, quality-gate-baseline-calibration, repo-onboarding-and-habitat-creation, session-self-critique, structural-integrity-audit, the-membrane-protocol, triple-reference


**Prompting (Anthropic)**: context 200K tokens, format: XML tags, thinking: extended thinking (budget_tokens)


## Atomization Pipeline

Run `organvm atoms pipeline --write && organvm atoms fanout --write` to generate task queue.


## System Density (auto-generated)

AMMOI: 25% | Edges: 0 | Tensions: 0 | Clusters: 0 | Adv: 27 | Events(24h): 37975
Structure: 8 organs / 148 repos / 1654 components (depth 17) | Inference: 0% | Organs: META-ORGANVM:63%, ORGAN-I:53%, ORGAN-II:48%, ORGAN-III:54% +5 more
Last pulse: 2026-05-23T00:26:28 | Δ24h: n/a | Δ7d: n/a


## Dialect Identity (Trivium)

**Dialect:** AESTHETIC_FORM | **Classical Parallel:** Music | **Translation Role:** The Poetry — proves formal structures have sensory form

Strongest translations: III (structural), V (analogical), VI (analogical)

Scan: `organvm trivium scan II <OTHER>` | Matrix: `organvm trivium matrix` | Synthesize: `organvm trivium synthesize`


## Logos Documentation Layer

**Status:** ACTIVE | **Symmetry:** 0.5 (DREAM)

Nature demands a documentation counterpart. This formation maintains its narrative record in `docs/logos/`.

### The Tetradic Counterpart
- **[Telos (Idealized Form)](../docs/logos/telos.md)** — The dream and theoretical grounding.
- **[Pragma (Concrete State)](../docs/logos/pragma.md)** — The honest account of what exists.
- **[Praxis (Remediation Plan)](../docs/logos/praxis.md)** — The attack vectors for evolution.
- **[Receptio (Reception)](../docs/logos/receptio.md)** — The account of the constructed polis.

### Alchemical I/O
- **[Source & Transmutation](../docs/logos/alchemical-io.md)** — Narrative of inputs, process, and returns.



*Compliance: Record exists without implementation.*

<!-- ORGANVM:AUTO:END -->












## ⚡ Conductor OS Integration
This repository is a managed component of the ORGANVM meta-workspace.
- **Orchestration:** Use `conductor patch` for system status and work queue.
- **Lifecycle:** Follow the `FRAME -> SHAPE -> BUILD -> PROVE` workflow.
- **Governance:** Promotions are managed via `conductor wip promote`.
- **Intelligence:** Conductor MCP tools are available for routing and mission synthesis.