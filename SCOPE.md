# textmatch — Project Scope

> **Status:** Draft v0.1 · Living document · Update it whenever scope decisions change.

## 1. Vision

**textmatch** is an all-in-one, dependency-free utility library for text matching.
It gives developers one consistent, well-documented API for every kind of "does
this text match that text?" question: exact, normalized, fuzzy, phonetic,
token-based, pattern-based, and ranked search.

Developers today stitch together many single-purpose packages (a Levenshtein
package, a fuzzy search package, a glob package, a phonetic package), each with
its own API, typing, module format, and runtime support. textmatch replaces that
with one library that behaves the same in every JavaScript runtime and package manager.

### Guiding principles

1. **Basics first, depth later.** Ship a small, excellent core, then grow by domain.
2. **Self-explaining.** Names, types, errors, and docs teach the user how to use it.
3. **Zero runtime dependencies.** Nothing to audit, nothing to break.
4. **Runs everywhere.** One codebase, all major runtimes and package managers.
5. **Tree-shakeable.** Users pay only for what they import.
6. **Predictable.** Pure functions, deterministic output, no hidden global state.

## 2. Goals

| # | Goal | Measure of success |
|---|------|--------------------|
| G1 | One unified API for common text-matching needs | Each capability in section 5 is reachable from a single package |
| G2 | First-class support for npm, Yarn, pnpm, Bun, and Deno | Install + import + test pass in CI on all five (section 6) |
| G3 | Handles real business use cases across domains | Documented recipes per domain (section 7) |
| G4 | Self-explaining documentation | A new user can solve a common task from docs alone, without reading source (section 8) |
| G5 | Small and fast | Per-function bundle-size and benchmark budgets enforced in CI |

## 3. Non-goals

To keep the library focused, textmatch will **not**:

- Be a full-text search engine or database (no persistent indexes, no server). It
  may offer in-memory indexes for small/medium datasets.
- Do NLP beyond matching (no POS tagging, sentiment, embeddings, or ML models).
  Pluggable hooks for external embeddings may be considered later.
- Depend on native addons, WASM, or network access.
- Support end-of-life runtimes (see section 6.3).

## 4. Target audience

- **Application developers** who need search boxes, autocomplete, and "did you mean".
- **Data/backend engineers** doing deduplication, record linkage, and data cleaning.
- **Tooling authors** building CLIs, linters, routers, and file matchers.
- **Domain teams** (e-commerce, healthcare, finance, logistics, HR, legal, support) that
  need reliable name, address, and product matching.

## 5. Functional scope

Capabilities are grouped into modules. Each module is independently importable
(e.g. `textmatch/fuzzy`) and also re-exported from the root.

### 5.1 Phase 1 — Core (MVP, "the basics")

| Module | Capabilities |
|--------|--------------|
| `normalize` | Case folding, Unicode normalization (NFC/NFD/NFKC), accent/diacritic stripping, whitespace collapsing, punctuation stripping, configurable pipelines |
| `exact` | Exact, case-insensitive, locale-aware equality; `startsWith`/`endsWith`/`includes` with normalization options |
| `distance` | Levenshtein, Damerau-Levenshtein, Hamming, Longest Common Subsequence |
| `similarity` | Normalized 0–1 scores: Levenshtein ratio, Jaro, Jaro-Winkler, Dice coefficient |
| `fuzzy` | `fuzzyMatch(a, b, { threshold })`, `fuzzyIncludes`, best-match-in-list |
| `search` | Rank a list of strings/objects against a query; multi-field, weighted keys, limit, min score |
| `wildcard` | Glob-style (`*`, `?`, `[a-z]`) matching for strings and paths |
| `highlight` | Return match ranges and wrap matched segments for UI display |

### 5.2 Phase 2 — Intermediate

| Module | Capabilities |
|--------|--------------|
| `token` | Token set/sort ratio, Jaccard, cosine, n-gram / shingle similarity, stop-word handling, stemming hooks |
| `phonetic` | Soundex, Metaphone / Double Metaphone, NYSIIS, Cologne (extensible per language) |
| `regex` | Safe regex helpers: escape, build from word lists, ReDoS-guarded execution with timeouts |
| `dedupe` | Find duplicates and cluster near-duplicates in a list |
| `suggest` | "Did you mean", autocomplete, prefix and typo-tolerant lookup |
| `diff` | Text and word-level diffs with match/mismatch ranges |
| `index` | Optional in-memory index (BK-tree, n-gram) for fast lookup on larger datasets |

### 5.3 Phase 3 — Advanced / domain-specific

| Module | Capabilities |
|--------|--------------|
| `entity` | Person/company name matching (titles, suffixes, nicknames, transliteration, name order) |
| `address` | Address normalization and matching (abbreviations, unit numbers, locale formats) |
| `product` | Product/SKU title matching (units, sizes, model numbers, brand aliases) |
| `record` | Multi-field record linkage with weighted scoring and explainable results |
| `i18n` | Script-aware matching (CJK, Arabic, Indic, Cyrillic), transliteration, locale collation |
| `pii` | Pattern helpers for emails, phone numbers, IDs (matching only, no validation guarantees) |
| `stream` | Match against large or streamed input without loading everything into memory |
| `plugins` | Public extension API: custom algorithms, scorers, normalizers, dictionaries |

Phase contents are a starting point. Items may move between phases as feedback arrives.

### 5.4 Cross-cutting API requirements

- **Consistent shape:** every matcher accepts `(input, target, options?)` and returns
  typed, documented results.
- **Explainable results:** matching functions can return `{ score, matched, reason, ranges }`
  so users can see *why* something matched.
- **Configurable pipelines:** normalization, tokenization, and scoring are composable.
- **Sync by default;** async variants only where I/O or streaming requires them.
- **Deterministic and pure:** same input, same output. Tie-breaking is documented.
- **Safe by default:** bounded input sizes, ReDoS-safe patterns, no `eval`.
- **Actionable errors:** typed error classes with messages that state the cause and the fix.

## 6. Platform and ecosystem scope

### 6.1 Package managers and registries

| Ecosystem | Requirement |
|-----------|-------------|
| **npm** | Published to the npm registry; `npm install textmatch` |
| **Yarn** | Works with Yarn Classic (v1) and Yarn Berry (v2+), including Plug'n'Play (PnP) and zero-installs |
| **pnpm** | Works with strict `node_modules` layout; no phantom dependencies |
| **Bun** | `bun add textmatch`; runs natively under Bun runtime |
| **Deno** | Importable via `npm:textmatch` **and** published to [JSR](https://jsr.io) as `@textmatch/textmatch` (`deno add jsr:@textmatch/textmatch`) with no Node-only APIs |

### 6.2 Package format requirements

- **TypeScript source**, shipped with bundled `.d.ts` type declarations.
- **Dual ESM + CJS** builds, with ESM as the primary format.
- Correct `package.json` `exports` map with `types`, `import`, `require`, and `default`
  conditions, plus per-module subpath exports.
- `"sideEffects": false` so bundlers can tree-shake.
- **No Node built-ins** in the core (only standard Web/ECMAScript APIs), so it also works
  in browsers, Cloudflare Workers, and edge runtimes.
- Validated with `publint` and `@arethetypeswrong/cli` in CI.
- Published with npm provenance; reproducible builds.
- Single lockfile-agnostic setup: no install scripts, no postinstall hooks.

### 6.3 Runtime support matrix

| Runtime | Minimum target |
|---------|----------------|
| Node.js | Active LTS and Maintenance LTS versions |
| Bun | Current stable and previous minor |
| Deno | Current stable (v2+) |
| Browsers | Evergreen (last 2 versions of Chrome, Firefox, Safari, Edge) |
| Edge/workers | Cloudflare Workers, Vercel Edge, Deno Deploy (best effort) |

The exact version floors are set at first release and reviewed each year.

### 6.4 Compatibility CI

A CI matrix installs the *packed tarball* (not the source tree) with each of
npm, Yarn Classic, Yarn Berry (PnP), pnpm, Bun, and Deno on Linux, macOS, and Windows,
then runs an import + smoke test suite (ESM, CJS, and TypeScript consumers).

## 7. Domain use cases ("advanced business cases")

Start with generic primitives, then add recipes and specialized modules per domain.
Each recipe ships as a documented, tested example.

| Domain | Example use cases | Primary modules |
|--------|-------------------|-----------------|
| E-commerce / retail | Typo-tolerant product search, SKU and title matching, catalog dedupe | `search`, `product`, `dedupe` |
| CRM / sales | Duplicate contact and company detection, lead merging | `entity`, `record`, `phonetic` |
| Finance / banking | Payee and counterparty matching, sanction-list name screening, reconciliation | `entity`, `record`, `i18n` |
| Healthcare | Patient record linkage, drug name lookup (look-alike/sound-alike) | `phonetic`, `record` |
| Logistics | Address matching and normalization, carrier/location name matching | `address`, `fuzzy` |
| HR / recruiting | Skill and job-title normalization, candidate dedupe | `token`, `entity` |
| Legal / compliance | Clause and document similarity, party name matching | `token`, `diff` |
| Customer support | FAQ / intent lookup, duplicate ticket detection, "did you mean" | `suggest`, `token` |
| Developer tooling | Command palettes, file and path globbing, CLI typo suggestions | `wildcard`, `fuzzy`, `suggest` |
| Data engineering | Data cleaning, entity resolution, ETL key matching | `dedupe`, `record`, `stream` |

> Regulated domains (finance, healthcare) get matching *tools*, not compliance guarantees.
> Docs must state accuracy limits and recommend human review where stakes are high.

## 8. Documentation scope ("self-explaining")

Documentation is a product feature, held to the same quality bar as code.

1. **Self-documenting code:** descriptive names, strict TypeScript types, and complete
   TSDoc on every public export (description, params, returns, `@example`, `@see`).
2. **README:** what it is, install snippets for all five ecosystems, a 30-second quick start.
3. **Docs site** (generated from source, versioned):
   - *Getting started* per ecosystem (npm, Yarn, pnpm, Bun, Deno)
   - *Concepts* (normalization, scoring, thresholds, tokenization)
   - *Guides / how-to* per task ("match names", "build a search box")
   - *Domain recipes* (section 7)
   - *API reference* auto-generated from TSDoc (e.g. TypeDoc)
   - *Algorithm chooser:* a decision guide that recommends the right function for a scenario
   - *Comparison and benchmarks* against popular alternatives
   - *Migration guides* from common libraries
4. **Runnable examples:** every code sample in docs is executed in CI so it can't rot.
5. **Interactive playground** in the docs site (later phase).
6. **Errors that teach:** messages link to a doc anchor explaining the fix.
7. **Repository documents:** `README.md`, `SCOPE.md` (this file), `ROADMAP.md`,
   `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `CHANGELOG.md`, `LICENSE`.
8. **Architecture Decision Records** (`docs/adr/`) for significant design choices.

## 9. Quality and non-functional requirements

| Area | Requirement |
|------|-------------|
| Correctness | Each algorithm tested against published reference vectors and known edge cases |
| Testing | Unit + property-based tests; coverage target ≥ 95% for core modules |
| Unicode | Grapheme-, code-point-, and locale-aware where relevant; emoji and combining marks handled |
| Performance | Benchmarks in CI; budgets for time and memory; no regression without justification |
| Bundle size | Size budget per module, tracked via size-limit; core import stays small |
| Security | No dependencies, ReDoS protection, input-length guards, supply-chain hardening (provenance, 2FA, pinned CI actions) |
| Accessibility of docs | Readable, searchable, dark/light mode, mobile friendly |
| Stability | Semantic Versioning; documented deprecation policy; public API is only what's exported and documented |
| Licensing | Permissive open source license (MIT proposed) |

## 10. Proposed technical approach

*(Recommendations; confirm before implementation.)*

- **Language:** TypeScript (strict mode).
- **Repo layout:** single package first; move to a monorepo only if domain modules
  need separate versioning.
- **Build:** `tsup` or `tsdown` producing ESM + CJS + `.d.ts`; JSR publish from the same source.
- **Test runner:** Vitest for Node, plus smoke tests executed natively in Bun and Deno.
- **Lint/format:** Biome or ESLint + Prettier.
- **Docs:** TypeDoc + a static site generator (e.g. VitePress or Starlight).
- **Release:** Changesets + GitHub Actions, publishing to npm (with provenance) and JSR.
- **Directory sketch:**

```text
textmatch/
├── src/
│   ├── index.ts            # root re-exports
│   ├── normalize/
│   ├── distance/
│   ├── similarity/
│   ├── fuzzy/
│   ├── search/
│   ├── wildcard/
│   └── ...                 # one folder per module
├── test/
├── docs/
│   ├── guides/
│   ├── recipes/
│   └── adr/
├── examples/               # runnable per-ecosystem examples
├── SCOPE.md
├── ROADMAP.md
└── README.md
```

## 11. Roadmap and milestones

| Milestone | Deliverables | Exit criteria |
|-----------|--------------|---------------|
| **M0 — Foundation** | Repo scaffolding, build (ESM+CJS+types), lint, test, CI skeleton, README, this scope | CI green; empty package installs on all 5 ecosystems |
| **M1 — Core MVP (v0.1)** | Phase 1 modules, TSDoc on all exports, quick-start docs | All Phase 1 functions tested; compat CI matrix passing; docs published |
| **M2 — Intermediate (v0.5)** | Phase 2 modules, docs site, algorithm chooser, benchmarks | Recipes for 3+ domains; size and perf budgets enforced |
| **M3 — Stable (v1.0)** | API freeze, migration guides, security review | Public API reviewed; semver commitment begins |
| **M4 — Advanced (v1.x)** | Phase 3 domain modules, plugins API, playground | Domain recipes for all sections in 7; community plugins possible |

## 12. Success metrics

- Compat CI passes on 5 ecosystems × 3 operating systems for every release.
- 100% of public exports have TSDoc with at least one tested example.
- Core (Phase 1) bundle stays under an agreed size budget (to be set at M0).
- Time-to-first-match for a new user under 5 minutes, measured via docs walkthroughs.
- Issue response time and contributor onboarding tracked after public release.

## 13. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Scope creep from "all in one" and "all domains" | Phased roadmap; new modules need a written proposal against this document |
| Cross-runtime inconsistencies | Web-standard APIs only; native CI on every runtime |
| Fuzzy-match quality disputes | Explainable scores, documented algorithms, reference test vectors |
| Performance on large datasets | Optional indexes, streaming API, benchmark budgets |
| Locale and Unicode complexity | Start with well-tested normalization; expand per-language support incrementally |
| Misuse in high-stakes domains | Prominent accuracy disclaimers; explainable output |
| Maintainer bandwidth | Small core, plugin API, strong contributor docs |

## 14. Open questions

1. Final package name and npm scope availability (`textmatch` vs `@textmatch/*`).
2. License choice (MIT vs Apache-2.0).
3. Exact minimum runtime versions.
4. Single package with subpath exports vs. monorepo of scoped packages.
5. Should Phase 3 domain data (nicknames, address abbreviations) ship in-package or as opt-in data packages?
6. Do we need a CLI (`npx textmatch ...`) in scope, or leave it as a later add-on?

## 15. Change process

Changes to this document are made by pull request. A new module or ecosystem
requires: (1) a short proposal describing the use case, (2) API sketch,
(3) impact on bundle size and docs, and (4) maintainer approval.
