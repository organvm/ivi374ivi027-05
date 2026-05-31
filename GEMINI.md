# MET4MORFOSES Web Edition - Project Context

## Project Overview
MET4MORFOSES Web Edition is a digital representation of Anthony James Padavano's MFA thesis project. It is implemented as an immersive, multi-mode web experience using Next.js 16 and TypeScript. The site serves as a creative archive and exploration of a thesis reimagining Ovid's *Metamorphoses*.

### Core Technologies
- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript (Strict mode)
- **Styling:** Tailwind CSS 4
- **Validation:** Zod
- **Analytics:** PostHog (Client-side tracking with server-side proxy)
- **Testing:** Vitest (Unit/Integration), Playwright (E2E)
- **Content:** Markdown (Generated from source PDFs)

### Architectural Modes
The application features several viewing modes:
- `/`: **Mythic Node Map** - An interactive constellation of narrative nodes.
- `/feed`: **Faux Social Feed** - A simulated social media experience reflecting thesis motifs.
- `/scroll`: **Three-Cycle Scroll** - A continuous scroll experience of the three thesis cycles.
- `/read/[docSlug]`: **Canonical Reader** - An inline reader for the primary thesis texts.
- `/archive`: **Mirror Downloads** - A directory of original thesis artifacts (PDFs, Pages, etc.).
- `/about`: **Portfolio Context** - Contextual information about the project.

## Content Pipeline
The project features a custom build-time pipeline that ingests source PDFs and generates the data required for the web experience.

### Source Data
Canonical PDFs are stored in `public/mirror/2018-03-20 - met4 - sixth draft/`. These are treated as immutable source artifacts.

### Ingestion Scripts (`scripts/*.ts`)
1.  **`ingest-canonical`**: Extracts text from PDFs using `pdftotext` (requires Poppler installed on the system) and writes to `src/content/*.md`.
2.  **`build-mirror-manifest`**: Catalogs all files in `public/mirror` with metadata and SHA-256 hashes.
3.  **`build-node-map-data`**: Generates the narrative graph and feed items.
4.  **`qa-canonical-fidelity`**: Validates extraction quality, enforcing a ≤8% character delta between PDF and Markdown.
5.  **`verify-mirror-integrity`**: Checks data invariants and links.

The full pipeline is run via `npm run content:build`.

## Key Commands

### Development
- `npm run dev`: Starts the development server (using Turbopack).
- `npm run start`: Serves the production build.

### Building
- `npm run build`: Runs the content pipeline followed by the Next.js production build.
- `npm run content:build`: Runs the data ingestion and verification scripts only.
- `npm run build:analyze`: Build with bundle analyzer.

### Testing and Linting
- `npm run lint`: Runs ESLint.
- `npm run typecheck`: Runs strict TypeScript checks.
- `npm test`: Runs Vitest unit and integration tests.
- `npm run test:e2e`: Runs Playwright E2E tests (default port 3007).

## Development Conventions

### Coding Style
- **TypeScript:** Strict typing is mandatory. Use `@/*` path aliases for `src/*`.
- **Formatting:** 2-space indentation, semicolons, double quotes.
- **Naming:** kebab-case for filenames, PascalCase for React components.
- **React:** Functional components with standard Next.js file conventions (`page.tsx`, `layout.tsx`).

### Testing Practices
- **Layers:** Unit tests for logic, Integration tests for routes/components, E2E for critical user flows.
- **Mocks:** `next/link` and `next/navigation` are mocked in `vitest.setup.tsx`.
- **Coverage:** Meaningful coverage is expected for new logic or transforms.

### Git Conventions
- **Commits:** Use Conventional Commits (`feat:`, `fix:`, `chore:`, etc.).
- **Workflow:** PRs should include summaries and screenshots for UI changes.

## File Structure Highlights
- `src/app`: Routes and API handlers.
- `src/components`: UI components.
- `src/lib`: Domain logic (content loading, node map helpers, mirror utilities).
- `src/data`: Generated JSON manifests and reports.
- `src/content`: Generated Markdown files.
- `public/mirror`: Canonical thesis artifacts.
- `tests/`: Organized by `unit`, `integration`, and `e2e`.

## Environment Configuration
Set the following in `.env.local` (refer to `.env.example`):
- `NEXT_PUBLIC_POSTHOG_KEY`
- `NEXT_PUBLIC_POSTHOG_HOST`
- `POSTHOG_KEY`
- `POSTHOG_HOST`

If analytics keys are missing, the application will function correctly but events will not be forwarded.

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


**Prompting (Google)**: context 1M tokens (Gemini 1.5 Pro), format: markdown, thinking: thinking mode (thinkingConfig)


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