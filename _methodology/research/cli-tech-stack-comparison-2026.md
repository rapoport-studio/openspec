# CLI tech-stack comparison — Rapoport Studio orchestra

> Deep tech-stack research for the Forge / Muse / Maestro three-CLI orchestra and the shared `packages/cli-core/` extraction.
> Cutoff: May 2026. Concrete claims verified against vendor docs, GitHub repos, npm trends, and benchmark posts; anything I could not pin down is marked `[unverified]`.
> Companion to: `ai-cli-landscape-2026-05.md`, `orchestra-training-architecture-2026.md`.

---

## How to read this document

- **"Role in the orchestra"** = what this component does for Forge, Muse, and Maestro specifically — not a generic description.
- **Comparison tables** use criteria specific to Rapoport's constraints: CF Workers deploy target, pnpm monorepo, Anthropic-first, Infisical secrets, the Claude API-only inference path.
- **Risk register** = the four risk lenses we always apply: operational (does it break in prod?), security (does it widen attack surface?), lock-in (how trapped are we?), maintenance (will it still exist in 24 months?).
- **Recommendation** is opinionated and constrained: each pick must justify a place in `packages/cli-core/` or in one CLI's private dependency graph.

The CLIs already share three primitives (`GitHubClient`, `LinearClient`, `loadForgeIdentity`) per the rule-of-three carve-out in CLAUDE.md § Package boundaries. This document defines what *else* is a candidate when `packages/cli-core/` lands.

---

## 1. Argument parsing framework

**Role in the orchestra:** Every CLI surface (`forge`, `muse`, `maestro`) needs sub-commands, typed flags, generated `--help`, and good DX for adding new commands. The parser sits underneath every entry point and ships in every binary.

### Top options 2026

| Option | Lang | Weekly downloads | GitHub stars | Last release | Notes |
|---|---|---|---|---|---|
| commander | TS | ~367M/wk | ~28k | active | Industry default, zero deps, used by everyone |
| yargs | JS | ~173M/wk | ~11k | active | Mature, ~7 transitive deps, heavier |
| oclif | TS | ~257k/wk | ~9.5k | active | Salesforce, plugin model, ~30 deps, slow startup |
| citty | TS | published ~1 mo ago (v0.2.2) | UnJS family | ESM-only v0.2 | Built on `node:util.parseArgs`, used by Nuxt/Nitro |
| cac | TS | ~15M/wk | ~2.8k | active | Tiny, fast, but maintainer cadence has slowed |
| clipanion | TS | ~3M/wk | Yarn's parser | v4.0.0-rc.4, no npm release in 12 mo | Class-based, powers Yarn Berry |

Sources: npm-compare, npmtrends, GitHub release pages (all verified May 2026).

### Comparison on Rapoport's criteria

| Criterion | commander | citty | oclif | cac | clipanion |
|---|---|---|---|---|---|
| TypeScript inference quality | OK | Excellent (parseArgs-backed) | Decorator-based | Good | Excellent (class-based) |
| Plugin model | None | Hooks (`setup`/`cleanup`) | Built-in | None | None |
| Bundle/startup cost | ~25 ms cold | <20 ms (parseArgs) | ~135 ms cold | <20 ms | ~30 ms |
| Sub-command nesting | Manual | Native + lazy import | Native | Native | Native |
| Ecosystem health (2026) | Very active | Very active (UnJS) | Active (Salesforce) | Slowing | Stale (no releases in 12 mo) |
| Boilerplate per command | Medium | Low | High (file-per-command) | Lowest | Medium |
| ESM-only viability | Dual | ESM-only v0.2+ | Dual | Dual | Dual |

### Risks per option

- **commander**
  - Lock-in: ~zero — most ubiquitous parser, migrate-out is trivial.
  - Maintenance: very low — TJ Holowaychuk historical, but corporate maintainers active.
  - Operational: no native lazy-load → every command loaded eagerly, hurts cold start as command count grows.
  - Security: zero-dep means zero CVE blast radius from transitive deps.
- **citty**
  - Maintenance: tied to UnJS organization health — broad, but bus-factor concentrated on a few maintainers (Pooya, Anthony).
  - Lock-in: ESM-only since v0.2 means consumers must be ESM; downgrade path painful.
  - Operational: uses `node:util.parseArgs` which is stable in Node 22+ but quirky around boolean negation.
  - Security: minimal surface; one transitive dep (`consola` peer).
- **oclif**
  - Maintenance: Salesforce-backed but Salesforce-shaped — generators assume their workflow.
  - Operational: 135 ms cold-start kills the `forge cli` snappiness ceiling; ~30 transitive deps inflate install.
  - Lock-in: file-per-command convention deeply structures repo; migrating away is real work.
  - Security: 30 deps = 30 supply-chain surfaces.
- **cac**
  - Maintenance: maintainer cadence visibly slowed in 2025-2026 [unverified single-maintainer status].
  - Lock-in: minimal API surface, migrate-out is easy.
  - Operational: no first-class plugin model — three-CLI sharing must build its own abstraction layer.
- **clipanion**
  - Maintenance: **red flag** — no npm release in past 12 months (per libraries.io), no PR activity in past month. Powers Yarn so it won't die, but feature velocity is dead.
  - Lock-in: class-based + decorator-adjacent style is the most "framework-shaped" option.

### Recommendation for Rapoport: **citty**

Forge already lives in the UnJS ecosystem-adjacent world (Nitro for Workers, h3 patterns elsewhere). Citty's lazy sub-command loading is the single biggest unlock for a three-CLI shared core: when `cli-core/` exposes the `GitHubClient`/`LinearClient`/auth primitives, citty lets Muse + Maestro share command *registration* shapes without paying for each other's command code at startup. The ESM-only constraint matches where the rest of the stack is already heading (Vercel AI SDK v6, Anthropic SDK, `node:util.parseArgs`). The bus-factor risk is real but mitigated by the small API surface — a fork is ~1 engineer-week.

**Defer-back option:** if citty's pace falters, commander is a one-day migration. Don't pick oclif (boot cost), cac (maintenance pace), or clipanion (already coasting).

---

## 2. Config loader

**Role in the orchestra:** Forge config (`forge.config.mjs`, roles, model tiers), Muse config (when it lands), Maestro config (calendar/email surfaces) all need the same layering: CLI flags > env vars (Infisical-injected) > project file > user file (`~/.forge/`) > defaults. The loader is shared across all three CLIs and the studio UI.

### Top options 2026

| Option | Lang | Weekly downloads | Last commit | Notes |
|---|---|---|---|---|
| cosmiconfig | JS | ~70M/wk | active | The de facto standard for "find config file up the tree" |
| c12 | TS | ~15M/wk | active (UnJS) | Layered + extends + remote sources |
| conf | TS | ~3M/wk | Sindre | User-data dir style (Electron-friendly) |
| rc | JS | ~50M/wk | mostly dormant | Old node-rc convention; ini/json/env |

### Comparison on Rapoport's criteria

| Criterion | c12 | cosmiconfig | conf | rc |
|---|---|---|---|---|
| Layering precedence (flags > env > file > defaults) | Native (`defu`-merged layers exposed) | Hand-roll on top | Hand-roll | Partial |
| File format (JSON / JSONC / JSON5 / YAML / TOML / JS / TS) | All via confbox | All via custom loaders | JSON/JSON5 only | INI / JSON / env |
| Env var resolution into config | Native (`$ENV{...}`) | Hand-roll | Hand-roll | Native |
| Type safety | Strong (TS generics on layers) | Generic but external types | Strong | Weak |
| Watch + reload | Native (chokidar peer) | External | None | None |
| Remote/extends from git URL | Native | None | None | None |

### Risks per option

- **c12**
  - Lock-in: medium — its layering object shape is opinionated; switching means rewriting how layers are surfaced to the orchestrator.
  - Maintenance: UnJS-aligned, healthy. Same bus-factor caveat as citty.
  - Operational: chokidar as peer dep means CF Workers builds need to externalize it; Forge already does this for `chokidar` so the pattern is known.
  - Security: the `extends: github:...` auto-clone feature is a footgun if config can be attacker-influenced — disable in shared loader.
- **cosmiconfig**
  - Lock-in: lowest. The output is a plain object; you build your own layering.
  - Maintenance: extremely stable, decade old, conservative changes.
  - Operational: you write the precedence machinery yourself = more code to maintain across three CLIs.
  - Security: no remote sources, no surprises.
- **conf**
  - Operational: user-data-dir semantics (`~/Library/Application Support/...`) are Electron-shaped; for a CLI, awkward.
  - Lock-in: store API is opinionated and hard to escape.
- **rc**
  - Maintenance: effectively unmaintained.

### Recommendation for Rapoport: **c12**

The layered model (`{ config, layers }` with `defu` merge) is exactly the four-level precedence Forge already simulates by hand. Lifting it into `cli-core/` removes ~200 LOC of bespoke merging. Disable the `extends: github:...` feature in the shared wrapper. cosmiconfig is the rational fallback if c12's UnJS bus factor materializes.

---

## 3. Process orchestration

**Role in the orchestra:** Forge spawns `pnpm`, `git`, `gh`, `vitest`, OpenSpec CLI, the verifier subprocess, sometimes `infisical`. Muse and Maestro will do less of this but still need git + gh. Streaming stdout to disk (for `~/.forge/runs/<id>/build.log`) and to terminal in real time is the hot path.

### Top options 2026

| Option | Maintainer | Weekly downloads | Notes |
|---|---|---|---|
| execa | Sindre | ~150M/wk | Industry default, v9 (May 2025) brought scripts API |
| tinyexec | tinylibs | ~5M/wk | Minimal wrapper over `child_process`, async iteration |
| zx | Google | ~700k/wk | Shell-like template literals, batteries included |
| dax | denoland | ~50k/wk | Deno-first, Node-compatible, cross-shell |

### Comparison on Rapoport's criteria

| Criterion | execa | tinyexec | zx | dax |
|---|---|---|---|---|
| Streaming output to multiple sinks | Excellent (web streams + node streams + Uint8Array) | Good (`for await` line iterator) | OK | Good |
| Cross-platform shell quoting safety | Excellent (no shell by default) | Good (no shell by default) | Risky (template literals invoke shell) | Excellent (own shell parser) |
| Sync + async API | Both | Async only | Async only | Async only |
| Dependency weight | Heavy (12+ transitive) | Tiny (0-1 dep) | Heavy | Medium |
| TypeScript ergonomics | Excellent (v9) | Good | OK | Excellent |
| Bun/Workers compatibility | Good (Node-compat path) | Excellent | OK | OK |

### Risks per option

- **execa**
  - Operational: v9 is ESM-only; combined with our other ESM picks this is fine, but mid-migration to ESM it bites.
  - Lock-in: API surface is wide; tests pin against execa-specific stream shapes.
  - Maintenance: Sindre + active contributors — best-in-class.
  - Security: shell-injection avoided by design (no shell unless you ask).
- **tinyexec**
  - Operational: missing convenience features (no `pipe()`, no template literal sugar) — fine for an orchestrator that already has its own helpers, painful for ad-hoc scripts.
  - Maintenance: tinylibs is healthy but small; rule-of-three risk.
  - Lock-in: tiny API, migration is trivial.
- **zx**
  - Security: **major red flag for an agent runtime** — template-literal-as-shell-command means LLM-generated strings flowing into `$\`...\`` is a shell-injection vector. Wrong tool for Forge.
  - Operational: zx assumes "this is a script", not "this is a daemon" — error semantics get weird inside long-running orchestrators.
- **dax**
  - Operational: Deno-first means Node-mode is second-class; fine today, vulnerable to Deno-side priorities.
  - Lock-in: own shell parser is a useful behavior model but introduces a third behavior to think about (sh, cmd.exe, dax-shell).

### Recommendation for Rapoport: **execa** (keep) + **tinyexec** for hot inner loops if profiling demands it

Forge already uses execa. Don't move. The v9 streaming API is exactly what `forge build` needs for `log.tee(stdout, file)`. Reject zx outright on the security ground — LLM-generated command strings must never enter a shell-coercing template literal. Hold tinyexec in reserve for the verifier subprocess if startup cost ever becomes a measurable issue (it isn't today).

---

## 4. File locking

**Role in the orchestra:** Forge's global build lock at `~/.forge/locks/build.lock` (RAP-224) serializes builds. Muse will need a similar lock for its session state. Maestro will need locking around calendar mutex regions. The lock library is reused identically in all three.

### Top options 2026

| Option | Weekly downloads | Last release | Style |
|---|---|---|---|
| proper-lockfile | ~10M/wk | v4.1.2, last published 5 years ago | Directory-based lockfile |
| fs-ext | ~1M/wk | Maintained, native | `flock(2)` bindings |
| lockfile | ~5M/wk | Old isaacs lib | File-based |
| Custom flock-based | — | — | Wrap `node:fs` + `O_EXCL` |

### Comparison on Rapoport's criteria

| Criterion | proper-lockfile | fs-ext (flock) | lockfile | Custom (O_EXCL) |
|---|---|---|---|---|
| PID detection | Native (stale lock heuristic via `update`) | None (OS-managed) | None | Yes if you write it |
| Stale lock recovery | Native (configurable `stale` timeout) | OS-managed (process death releases) | Manual | Yes if you write it |
| Cross-platform (macOS + Linux) | Excellent (no native deps) | Native binding — needs prebuilt or build-from-source | Good | Excellent |
| NFS support | Yes (directory-based works) | **No** (flock doesn't lock over NFS) | Spotty | Yes |
| Busy-wait / queueing API | Yes (`onCompromised`, retries) | No | Yes | Yes if you write it |
| Bundle into single binary (SEA) | Easy (pure JS) | Hard (native binding) | Easy | Easy |

### Risks per option

- **proper-lockfile**
  - Maintenance: **yellow flag** — v4.1.2 last published 5 years ago. Still works because the API surface is tiny and the implementation is stable, but no active development.
  - Operational: directory-based locks accumulate stale `.lock` directories if cleanup is interrupted; the `stale` parameter handles this but must be set.
  - Lock-in: very low — the API is `lock()`, `unlock()`, `check()`.
  - Security: none specific.
- **fs-ext**
  - Operational: native module → must rebuild for Linux x64 (Workers compatibility N/A, this is CLI-side only), Linux arm64, macOS x64, macOS arm64. Real distribution pain.
  - Operational: `flock()` doesn't work over NFS. Not relevant for `~/.forge/` (always local) but is for any shared-drive use case.
  - Maintenance: maintained but quiet.
- **lockfile** (isaacs)
  - Maintenance: effectively done. Works but no future.
- **Custom**
  - Maintenance: you own all the corner cases — PID detection, stale recovery, retry backoff.
  - Lock-in: high (your bugs are your problem).
  - Operational: easy to get wrong (e.g., the `O_EXCL` + delete race on POSIX).

### Recommendation for Rapoport: **proper-lockfile** (with our own `stale` policy)

Forge already uses it implicitly through RAP-224's lock pattern. The "5 years no release" risk is offset by: (a) tiny surface, (b) battle-tested in 1290+ npm packages per their README, (c) easy migration to a custom implementation if it ever breaks. Set `stale: 60000` and `update: 10000` and move on. Don't pick fs-ext — the native-binding cost defeats any SEA/single-binary distribution plan in §12.

---

## 5. Pre-commit hooks

**Role in the orchestra:** This is studio-repo concern only (CLIs ship as packages, not consume hooks). Hooks gate Forge PRs, the apps' lint/typecheck, OpenSpec bundle regen, and the Vale prose lint we add per §8.

### Top options 2026

| Option | Lang | Weekly downloads | Notes |
|---|---|---|---|
| husky | JS | ~15M/wk | Node-runtime, prepare-script-installed |
| lefthook | Go | ~400k–1M/wk (sources vary) | Go binary, parallel execution |
| simple-git-hooks | JS | ~1.5M/wk | Minimal, no parallelism |

### Comparison on Rapoport's criteria

| Criterion | husky | lefthook | simple-git-hooks |
|---|---|---|---|
| Install footprint | Node deps + `.husky/` | Go binary + `lefthook.yml` | Tiny |
| Pre-commit speed (10-pkg monorepo) | ~1.2s | ~0.9s | similar to husky |
| Pre-commit speed (100+-pkg monorepo) | baseline | ~2.7x faster than husky | unknown / not designed for |
| Language-agnostic | Implicit (you run any command) | Native (designed for polyglot) | Yes |
| Monorepo `root: <path>` filter | Manual | Native | Manual |
| Parallel execution by default | No (sequential) | Yes | No |

### Risks per option

- **husky**
  - Operational: Node startup on every commit. In small repos fine; in 100+ workspace monorepos the startup dominates.
  - Maintenance: very stable, mature.
  - Lock-in: zero — hooks are just scripts.
- **lefthook**
  - Operational: Go binary needs to be installable cross-platform; `npm i @evilmartians/lefthook` handles this but adds a binary install on every `pnpm install`.
  - Lock-in: `lefthook.yml` schema is specific; migrating away means rewriting the parallelism manually.
  - Maintenance: actively developed (v2.1.6 published ~3 weeks ago per npm).
  - Security: trusted vendor (Evil Martians), but a Go binary in postinstall is a higher trust surface than pure-JS husky.
- **simple-git-hooks**
  - Operational: no parallelism = slowest on big monorepos.
  - Lock-in: lowest.

### Recommendation for Rapoport: **lefthook**

This repo is already a multi-workspace pnpm monorepo with web + studio + 3 CLIs + multiple packages. The husky 1.2s vs lefthook 0.9s gap on small repos is irrelevant; the lefthook ~2.7x gap on real-monorepo parallel pre-commit is. The Evil Martians Go-binary trust surface is acceptable for an internal repo. The OpenSpec bundle-regen hook is a perfect candidate for lefthook's `root: openspec/` glob trigger.

---

## 6. Runtime schema validation

**Role in the orchestra:** Validates Forge config, role/model tier definitions, OpenSpec frontmatter parsing, MCP tool schemas, plan-bundle shapes between roles. Used in every CLI and every app boundary that takes structured input.

### Top options 2026

| Option | Bundle (login form, esbuild) | Notes |
|---|---|---|
| Zod (full) | ~17.7 kB | Industry default, v4 (Aug 2025) launched Zod Mini |
| Zod Mini | ~6.88 kB | Functional/tree-shakable variant of v4 |
| Valibot | ~1.37 kB | Modular pipeline functions, smallest |
| ArkType | ~42 kB (39.8 kB tree-shaken) | JIT-compiled validators, fastest runtime |
| Effect Schema | ~30+ kB [unverified] | Tied to Effect ecosystem |

### Comparison on Rapoport's criteria

| Criterion | Zod (full) | Zod Mini | Valibot | ArkType |
|---|---|---|---|---|
| Bundle size | Large | Medium | Smallest | Medium-large |
| Type inference quality | Excellent | Excellent | Excellent | Best in class (type-level) |
| Transformation support | Native | Limited | Native | Native |
| JSON Schema export | First-class (`zod-to-json-schema`, official in v4) | Same | Plugin (`@valibot/to-json-schema`) | Plugin |
| Vercel AI SDK v6 integration | First-class | First-class | Supported | Supported |
| Anthropic Skills frontmatter validation ergonomics | Best | Best | Good | Good |
| Runtime speed | Baseline | Baseline | Baseline | 5-100x faster (JIT) |

### Risks per option

- **Zod (full)**
  - Operational: large bundle in apps — only matters for `apps/studio` client bundle, not for CLI/Worker which already ship Anthropic SDK (heavier).
  - Maintenance: extremely active, Colin McDonnell.
  - Lock-in: medium — schema definitions are Zod-shaped; migrating to Valibot is a sed-script away in simple cases, painful in cases with `.refine`, `.transform`, `.preprocess`.
  - Security: zero specific concerns.
- **Zod Mini**
  - Operational: functional API forks the developer experience inside one ecosystem — pick one variant per package.
  - Lock-in: still Zod, so compatible.
- **Valibot**
  - Maintenance: very active, 2025-2026 saw the v1.0 line stabilize.
  - Operational: pipeline-style (`v.pipe(v.string(), v.email())`) reads differently than Zod's chains; team needs to switch mental model.
  - Lock-in: schema definitions don't trivially roundtrip to Zod.
  - Security: no concerns.
- **ArkType**
  - Operational: JIT means more memory and more startup cost; trade-off for hot validation paths.
  - Lock-in: TypeScript-template-literal type syntax is unique to ArkType — migrating away is painful.
  - Maintenance: smaller team, faster movement.

### Recommendation for Rapoport: **Zod (full) in CLIs and Workers, no migration**

Vercel AI SDK v6 is Zod-native. The Anthropic SDK ecosystem (including official Skills tooling) gravitates to Zod. The bundle-size argument is moot for our deploy targets — Workers care about KV/asset size and the OpenSpec bundle dwarfs any Zod schema. The discipline of "one schema validator across packages/cli-core/" is more valuable than the byte savings of Valibot. Revisit only if `apps/studio` client-bundle hits a hard ceiling, in which case adopt Zod Mini in the apps and keep Zod in the CLI.

---

## 7. LLM-output schema constraint

**Role in the orchestra:** Forge roles emit structured JSON between checkpoints (plan/build/verify/etc.). Muse emits structured task graphs. Maestro emits structured calendar/email actions. We need *bulletproof* round-tripping from "Claude generates text" to "validated typed object".

### Top options 2026

| Option | Language | Approach | Notes |
|---|---|---|---|
| Vercel AI SDK + Zod (`generateObject`) | TS | Provider-level structured output + retry | Industry default for TS |
| BAML | DSL → codegen TS | `.baml` files, schema-aligned parsing | Best output reliability per benchmarks |
| TypeChat | TS | Microsoft, schema-as-comments prompt | Effectively abandoned |
| Instructor | Python primary, TS port | Function-calling wrapper | Less of a fit for TS-first stack |
| Anthropic tool-use (native) | TS | Provider primitive | Lowest level; useful via AI SDK |

### Comparison on Rapoport's criteria

| Criterion | AI SDK + Zod | BAML | TypeChat | Instructor (TS port) |
|---|---|---|---|---|
| Token efficiency (schema in prompt) | Baseline (Zod → JSON Schema, large) | 50-80% smaller schema strings ([BAML claim, see SAP blog](https://boundaryml.com/blog/schema-aligned-parsing)) | OK | OK |
| Error recovery on bad JSON | Built-in retry, can repair w/ a second call | **Schema-aligned parsing** handles markdown-wrapped, chain-of-thought, trailing commas natively | None | Pydantic-level only |
| Provider portability | 25+ providers in v6 | TS / Python / Ruby / Go / Java / C# / Rust generated | OpenAI mainly | OpenAI mainly |
| TS-native ergonomics | Best | Code-gen step adds friction | OK | Port quality varies |
| Token cost of streaming partials | Native (`streamObject`) | Native | None | Limited |

### Risks per option

- **AI SDK + Zod**
  - Operational: token cost of sending full Zod-derived JSON Schema in every prompt is real — measurable on Forge today.
  - Lock-in: Vercel AI SDK abstraction layer; if Vercel changes pricing/license, switch cost is the AI SDK API, not the schema.
  - Security: nothing specific.
  - Maintenance: Vercel AI SDK is heavily resourced, v6 stable.
- **BAML**
  - Operational: build-time codegen step adds friction in monorepo (CI must run `baml-cli generate` before TS checks). pnpm + Turbo handle this but it's another dance.
  - Lock-in: medium-high — `.baml` files are a learned format; migrating away means re-writing every function definition.
  - Maintenance: BoundaryML is YC-backed, 2024-2026 cadence solid.
  - Security: codegen output runs in your build, trust the vendor binary.
  - Operational upside: SAP genuinely solves the "Claude wrapped my JSON in markdown" pain — Pavel's memory file flags this as a recurring annoyance.
- **TypeChat**
  - Maintenance: Microsoft has effectively left this alone; commit cadence stalled.
- **Instructor (TS port)**
  - Maintenance: the Python project is excellent; the TS port is a third party effort with smaller community.

### Recommendation for Rapoport: **Vercel AI SDK + Zod for v1; pilot BAML for the structured-output hot paths (Forge plan-bundle, Muse task-graph)**

The AI SDK + Zod path is the safe default and lines up with Pavel's locked v6 choice. But BAML's SAP claims are validated on benchmarks (Berkeley Function Calling Leaderboard, 98% type-correct without post-processing) and address a real Forge pain (markdown-wrapped JSON, the `// CLAUDE wraps JSON in markdown fences sometimes` note in CLAUDE.md). Spike BAML for *one* role's output schema in a single PR. If 2026-Q3 numbers show 20%+ token reduction and parsing reliability gains, expand. Otherwise keep AI SDK + Zod and revisit in 12 months.

---

## 8. Prose linter

**Role in the orchestra:** Lints `openspec/current/*.md`, `openspec/changes/*/proposal.md`, `proposal.md` / `design.md` / `tasks.md` files in the studio, SKILL.md files in skills, and the studio marketing copy. Catches voice drift between Pavel and the agents.

### Top options 2026

| Option | Lang | Notes |
|---|---|---|
| Vale | Go | Markup-aware, YAML rules, custom packages, fastest |
| textlint | JS | Plugin ecosystem, AST-based |
| alex | JS | Inclusive-language focus only |
| retext | JS | Plugin set over unified.js |
| markdownlint | JS | Markdown-structure rules only |
| write-good | JS | Pure prose, very old, single-pass |

### Comparison on Rapoport's criteria

| Criterion | Vale | textlint | alex | markdownlint |
|---|---|---|---|---|
| Speed on 1000+ files | Excellent (Go binary, parallel) | OK | Fast (narrow scope) | Fast |
| Rule customization | YAML-based, very flexible | JS plugins, hardest barrier | Fixed | Limited |
| Terminology dictionary support | Native (Substitution check, Spelling check) | Plugin-only | None | None |
| Markup-aware (code-block exclusion) | Native | Plugin | None | Native (own scope) |
| CI integration | First-class (GH Action, JSON output) | Good | Good | Good |
| Cross-tool reuse (Cursor/VSCode/CI) | Excellent (Vale extension is standard) | OK | Limited | Excellent |

### Risks per option

- **Vale**
  - Operational: binary install in CI; Go means no Node dep but does mean cross-platform binary management. Use `@vvago/vale` or download in CI step.
  - Lock-in: rule files are Vale-specific YAML. Effort spent writing the Rapoport style guide is non-portable.
  - Maintenance: active (`vale-cli/vale` GH repo, regular releases).
  - Security: no concerns.
- **textlint**
  - Operational: AST-based but slower; plugin universe is large but quality varies.
  - Maintenance: active core, plugin maintenance is bus-factored.
- **alex**
  - Operational: too narrow on its own; would have to be combined with Vale anyway.
- **markdownlint**
  - Operational: structure-only — doesn't catch "we say X here and Y there".

### Recommendation for Rapoport: **Vale** with a custom `RapoportStudio` style package

A studio that publishes OpenSpec specs publicly needs a single voice across `current/*.md`. Vale's terminology dictionary lets us encode "Forge / Muse / Maestro are proper nouns (capitalized once, lowercased after)", "OpenSpec is one word, capital O capital S", "n8n is always lowercase", etc. The Go-binary install cost is paid once per dev machine; lefthook (§5) parallelizes it on pre-commit. Skip textlint/markdownlint as primary tools; add markdownlint as a *complement* for structural rules where it doesn't overlap.

---

## 9. Event logging

**Role in the orchestra:** Forge writes `~/.forge/runs/<id>/build.log`, `state.json`, and per-role outputs. Muse and Maestro will need their own per-run logs. The CLI also emits "user-pretty" output to TTY. One logger, two reporters.

### Top options 2026

| Option | Lang | Weekly downloads | Notes |
|---|---|---|---|
| pino | JS | Major | ~5x Winston throughput, worker-thread transports |
| winston | JS | Major | Transport flexibility, slower |
| consola | TS | ~8M/wk | UnJS, pretty-in-TTY + JSON-in-CI auto-detect |
| Custom JSONL | — | — | Just `fs.appendFile` + `JSON.stringify` |

### Comparison on Rapoport's criteria

| Criterion | pino | consola | winston | Custom JSONL |
|---|---|---|---|---|
| Throughput | ~222k ops/sec | High but unbenchmarked vs pino | ~36k ops/sec | ~unbounded (no formatting) |
| Structured-first | Yes (JSON-default) | Yes + pretty | Yes (configurable) | Yes (you write it) |
| Transport flexibility | Excellent (worker-thread) | Good (reporter interface) | Excellent (transports built-in) | Whatever you write |
| Bundle size in single binary | Medium | Small | Large | Smallest |
| Pretty TTY out-of-box | Plugin (`pino-pretty`) | Native | Plugin | Manual |
| Forge integration cost | Medium (replace consola) | Zero (consola is the UnJS default + already in Forge orbit) | Higher | Highest |

### Risks per option

- **pino**
  - Operational: requires a separate `pino-pretty` process or transport for human-readable TTY output, which complicates the "one logger" story.
  - Maintenance: NearForm-backed, very active (Matteo Collina).
  - Security: zero.
- **consola**
  - Operational: less raw throughput than pino but Forge log volume is orders below the threshold where that matters (we write per-checkpoint, not per-request).
  - Maintenance: UnJS bus factor (same as citty/c12).
  - Lock-in: small surface, easy to migrate.
- **winston**
  - Operational: slow enough that with 1000+ logs/sec it backpressures.
  - Maintenance: very stable, very mature.
- **Custom JSONL**
  - Operational: you reinvent rotation, transport, levels, formatting.
  - Lock-in: zero.

### Recommendation for Rapoport: **consola**

For an *agent runtime* (not a high-RPS API), consola's automatic "pretty in TTY, JSON in CI" detection is exactly the developer experience we want. Throughput is not the binding constraint — checkpoint-rate logging tops out at ~100 events per build run. Pair with a thin custom JSONL appender for the structured `state.json` payload (which deserves to be *not* a free-form log, but a deliberate record). Don't pick pino unless throughput becomes a measurable problem; we already wrap consola in Forge today.

---

## 10. LLM observability

**Role in the orchestra:** Records every Claude API call across all three CLIs — token counts, costs, latency, cache hit rate, tool use, retries. Feeds the eval harness (§11) and gives us the data to write the next FPI/ECM report.

### Top options 2026

| Option | Self-host | TS-native client | Notes |
|---|---|---|---|
| Langfuse | Yes (MIT, ClickHouse-acquired Jan 2026) | Yes | Strongest OSS option |
| Helicone | Yes (proxy mode) | Yes | Simplest install, proxy-based |
| Braintrust | No (SaaS-only) | Yes | Strongest eval workflow |
| Phoenix (Arize) | Yes (Postgres + Kubernetes) | Yes | OpenInference, strongest ML rigor |
| LangSmith | No (SaaS) | Yes | LangChain-coupled |
| OpenTelemetry GenAI | N/A (standard) | Yes (SDK) | Experimental, dual-emission for stable transition |

### Comparison on Rapoport's criteria

| Criterion | Langfuse | Helicone | Braintrust | Phoenix | OTel GenAI |
|---|---|---|---|---|---|
| Self-hostable | Yes (Docker compose, ClickHouse) | Yes (Helm) | No | Yes (Postgres + K8s) | N/A (you self-host the backend) |
| TS-native client | Yes | Yes | Yes | Yes (OpenInference) | Yes (`@opentelemetry/instrumentation-anthropic`) |
| Cost tracking granularity | Per-call + cache breakdown | Per-call | Per-call + per-eval | Per-call | Per-call (semconv-defined) |
| Eval integration | Native | Plugin | First-class | Native | Backend-dependent |
| Anthropic prompt-cache visibility | Yes (Langfuse v3+) | Yes | Yes | Yes | Yes (semconv `gen_ai.usage.input_tokens.cache_read`) |
| Cost at our scale (~10k Claude calls/mo) | $0 self-hosted | $0 OSS / $20+ cloud | $0 free tier (1M spans) | $0 self-hosted (infra cost) | $0 (you pay backend) |

### Risks per option

- **Langfuse**
  - Operational: ClickHouse is heavy; for 10k events/mo it's overkill — but the Docker compose path is well-trodden.
  - Lock-in: medium — `langfuse.trace()` decorations on every Claude call, but they're shaped roughly like OTel.
  - Maintenance: Jan 2026 ClickHouse acquisition. Acquisitions historically mean either acceleration or absorption; we don't know which yet.
  - Security: self-hosted = your data stays in Rapoport infra. Good for client-confidential prompts.
- **Helicone**
  - Operational: drop-in proxy = lowest install effort, but inserting a proxy in front of Anthropic means another point of failure for every CLI call.
  - Lock-in: minimal — change proxy URL to roll back.
  - Maintenance: well-funded, active.
- **Braintrust**
  - Lock-in: hard. SaaS-only means a vendor outage = no telemetry.
  - Operational: best DX for eval-gated CI workflows.
  - Security: prompt content leaves Rapoport infra. Acceptable for non-client work; not acceptable for client-confidential.
- **Phoenix**
  - Operational: heavy infra (Postgres + K8s); overkill for our scale.
  - Lock-in: OpenInference is OTel-shaped → easy migration.
  - Maintenance: Arize-backed, OSS portion (Phoenix) is healthy.
- **OTel GenAI**
  - Operational: standard is *experimental* — dual-emission required during stability transition. Pin Anthropic and instrumentation libs to versions you've tested.
  - Lock-in: lowest — by definition, picking the standard means picking the most portable shape.
  - Maintenance: vendor-neutral, less risky than any single vendor.

### Recommendation for Rapoport: **Langfuse (self-hosted) as primary, with OpenInference/OTel-shaped instrumentation under it**

Langfuse self-hosted gives us: (a) no per-event pricing as we scale Forge from 50 builds/week to 500, (b) prompts and tool-use payloads stay in Rapoport-controlled infra (matters for client work), (c) ClickHouse storage scales past the point we'd care, (d) prompt-cache visibility we already need for the FPI/ECM measurement work. Instrument with OpenInference-style spans so a future swap to Phoenix or pure-OTel is a backend change, not a code change. Helicone is the fallback if self-hosting bandwidth runs out. Avoid Braintrust as a *primary* — but consider it for the eval harness layer (§11) where its hosted DX is genuinely better than the alternatives.

---

## 11. Eval harness for LLMs

**Role in the orchestra:** Regression-gates Forge role outputs (plan / build / verify), Muse outputs (task graph), Maestro outputs (action confirmations). Runs nightly against golden sets and pre-merge against PRs that touch prompts/skills.

### Top options 2026

| Option | Lang | TS-native | Self-host | Notes |
|---|---|---|---|---|
| promptfoo | Node + Python | Yes (Node-first) | Yes (OSS) | Industry default for JS evals |
| DeepEval | Python | No (Python-only) | Yes | Pytest-shaped, requires Python in CI |
| Braintrust | Multi | Yes | No | SaaS, strongest eval-CI workflow |
| OpenAI Evals | Python | No | Yes | OpenAI-centric, less active community |
| ragas | Python | No | Yes | RAG-specific |

### Comparison on Rapoport's criteria

| Criterion | promptfoo | DeepEval | Braintrust | OpenAI Evals |
|---|---|---|---|---|
| TypeScript-native | Yes (primary surface) | No | Yes | No |
| Golden-set workflow | Yes (YAML test sets, CSV inputs) | Yes (pytest) | Yes (web UI for non-tech reviewers) | Yes (JSONL) |
| Regression cadence | CI + cron | CI | CI + persistent DB | CI |
| LLM-judge support | Native (`llm-rubric`, multi-judge) | Native (G-Eval, QAG, DAG) | Native | Manual |
| Red-team / adversarial | Built-in | Plugin | Plugin | None |
| Anthropic provider support | First-class | Yes | Yes | OpenAI-centric |

### Risks per option

- **promptfoo**
  - Operational: ~21k stars, 924k monthly downloads = healthy. YAML-driven test sets are easy to write but get unwieldy past ~500 cases.
  - Lock-in: tests are YAML/JS; migrating to Braintrust is doable but the assertion DSL is different.
  - Maintenance: very active, well-resourced.
  - Security: built-in red-team is a real feature, not marketing.
- **DeepEval**
  - Operational: **Python-only is a hard blocker** for a Node + TS monorepo. Adds Python to CI for one purpose. No.
  - Maintenance: Confident AI, active.
- **Braintrust**
  - Operational: SaaS means another vendor in the dependency chain; prompts leave Rapoport infra.
  - Lock-in: highest — datasets and eval history are in Braintrust's DB.
  - Maintenance: well-funded.
- **OpenAI Evals**
  - Maintenance: anemic since 2024; community has moved to promptfoo and Braintrust.

### Recommendation for Rapoport: **promptfoo** in CI, with Langfuse (§10) holding the production telemetry side

promptfoo is the only mature TS-native option that doesn't require leaving Rapoport infra. The two-tool pattern noted in industry — "lightweight in-repo evals + heavy telemetry/annotation platform" — maps cleanly to promptfoo + Langfuse. Skip DeepEval (Python-only). Reserve Braintrust for "if we ever do non-confidential client work where the SaaS DX is worth the lock-in".

---

## 12. CLI distribution

**Role in the orchestra:** How do Forge / Muse / Maestro reach a developer's machine? Today: `pnpm install -g @rapoport-studio/forge` then `forge`. The question is whether to keep that, ship single binaries, or hybrid.

### Top options 2026

| Option | Cold start | Install friction | Cross-arch | File size | Notes |
|---|---|---|---|---|---|
| npm + npx | ~Node startup (~60–120 ms) | High (npm install) | Trivial | Small | Industry default |
| Bun build `--compile` | ~8–15 ms (Bun runtime baked) | Low (single download) | Cross-compile supported | ~50–80 MB | Bun is fastest cold-start |
| Node SEA (`node:sea` + `--build-sea`) | ~milliseconds (no JIT warmup) | Low | Per-host build | ~70 MB | Native since v25.5, single-step in 2026 |
| Deno compile | ~40–60 ms | Low | Cross-compile native | ~60 MB | Mature, security-default |
| pkg / yao-pkg | ~Node startup | Low | Cross-compile | ~40 MB | pkg deprecated, yao-pkg fork active |
| nexe | — | — | — | — | Unmaintained, no releases since 2017 |

### Comparison on Rapoport's criteria

| Criterion | npm + npx | Bun compile | Node SEA | Deno compile |
|---|---|---|---|---|
| Cold start (initial CLI invocation) | Slowest | Fastest | Very fast | Fast |
| Install friction for a new dev | High | Lowest (download + chmod) | Low | Low |
| Cross-arch publishing pipeline | Trivial (single tarball) | Cross-target Bun build | One artifact per arch | One artifact per arch |
| File size for distribution | Smallest (defer to npm) | Largest (Bun runtime ~50 MB) | ~70 MB | ~60 MB |
| Auto-update story | npm-native (`pnpm up -g`) | Custom | Custom | Custom |
| Compatibility with native deps (proper-lockfile, etc.) | Best | Good | Good | Tricky |

### Risks per option

- **npm + npx**
  - Operational: install friction = adoption tax. Every new contractor needs Node, pnpm, then the CLI.
  - Maintenance: trivial.
  - Lock-in: zero.
- **Bun compile**
  - Operational: Bun runtime baked into the binary = if Bun has a Node-compat bug, Forge inherits it.
  - Lock-in: medium — `Bun.*` APIs differ from Node and pull users into Bun-shaped code patterns.
  - Maintenance: Bun's pace is fast but breaking; Forge has to track.
  - Security: Bun is auditable but newer than Node.
- **Node SEA**
  - Operational: needs Node 25.5+ as the build host; the `--build-sea` flag is brand-new (Jan 2026).
  - Lock-in: lowest — your code is plain Node; SEA is just a packaging step.
  - Maintenance: this is Node core. As future-proof as we get.
  - Security: smaller attack surface than running shell-installed Node.
- **Deno compile**
  - Operational: Deno permission model is a security win but requires adapting Node-style filesystem assumptions.
  - Lock-in: medium — Deno-isms creep in.
- **pkg / yao-pkg**
  - Maintenance: deprecated upstream; yao-pkg is community-maintained but bus-factor concentrated.
  - Operational: virtual-FS hack is fragile across Node versions.

### Recommendation for Rapoport: **npm + npx for v1, Node SEA for v2 once `--build-sea` matures**

For a CLI used inside Rapoport (Pavel + small team + AI agents), npm install is fine — the audience already has Node. For the eventual public release of Muse (per the founder-track ICP positioning), the install-friction tax is real and SEA's "download one file, chmod, run" beats npx for first-time users. Bun compile is a fast follow if Anthropic SDK + AI SDK v6 prove Bun-compatible at the depth we use them; today it's a bet against Node-compat edges. Don't pick pkg or nexe — both are running on borrowed time.

---

## 13. Skills authoring/validation

**Role in the orchestra:** Forge, Muse, Maestro all ship SKILL.md files. We need to validate frontmatter, cross-tool compatibility, and the "no orphan files in scripts/ references/ assets/" rules.

### Top options 2026

| Option | Source | Notes |
|---|---|---|
| Anthropic official skill-creator | Anthropic | Skill-creator now includes eval mode (Q1 2026) |
| `agent-ecosystem/skill-validator` | Community | Validates structure + content density |
| Custom Rapoport validator | — | Roll our own atop Zod + frontmatter parser |

### Comparison on Rapoport's criteria

| Criterion | Anthropic skill-creator | skill-validator | Custom |
|---|---|---|---|
| Schema enforcement (frontmatter) | Yes | Yes | Yes if you write it |
| Cross-tool compat (Claude Code / Cursor / Codex / Gemini CLI) | Best (vendor-aligned) | Good (open-standard-aligned) | Up to you |
| Density / quality heuristics | Yes (skill-creator eval mode) | Yes | None unless you write them |
| Integration into Forge CI | Manual | Manual | Native |
| Maintenance | Anthropic-owned | Community | You |

### Risks per option

- **Anthropic skill-creator**
  - Operational: requires an Anthropic API call to run eval mode = $ in CI.
  - Lock-in: lowest in terms of format (open standard); medium in tooling.
  - Maintenance: best — vendor-owned.
- **skill-validator**
  - Maintenance: community-maintained, agent-ecosystem org bus factor.
  - Operational: validates structure only — content quality is on Anthropic skill-creator or you.
- **Custom**
  - Operational: you own all the corner cases.
  - Lock-in: zero.
  - Maintenance: yours.

### Recommendation for Rapoport: **`agent-ecosystem/skill-validator` for structure in pre-commit + lefthook + CI, Anthropic skill-creator for density/quality on a slower cadence (PR, not pre-commit)**

The 32-tool cross-compat standard published Dec 2025 means SKILL.md is now a real interop format. Validate structure cheaply (skill-validator, no API call) and quality expensively (skill-creator eval mode, in CI not pre-commit). Roll a tiny Rapoport-specific check on top to enforce our internal conventions (`description` must reference "Rapoport Studio" if it's a studio-internal skill, naming format for `scripts/`, etc.).

---

## 14. GitHub integration

**Role in the orchestra:** Forge creates PRs, reads PR state, posts comments, fetches diffs. Muse will do similar for "self-improvement PRs" against `_methodology/`. Maestro probably touches GitHub less but needs to read repo state for context.

### Top options 2026

| Option | Auth | Rate-limit handling | GraphQL | Notes |
|---|---|---|---|---|
| Octokit (full SDK) | PAT / App / OIDC | `@octokit/plugin-throttling` | First-class | Industry default |
| gh CLI (subprocess) | gh's own (browser flow + token) | gh handles internally | `gh api graphql` | Already on dev machines |
| graphql-request | PAT | Manual | First-class | Tiny |

### Comparison on Rapoport's criteria

| Criterion | Octokit | gh CLI subprocess | graphql-request |
|---|---|---|---|
| Auth model | Native PAT / App / OIDC | gh's flow (browser or env token) | PAT only |
| Rate-limit handling | Throttling + retry plugins | gh's built-in | Manual |
| GraphQL access | First-class | `gh api graphql` works fine | First-class |
| Bundle size in SEA binary | Medium (modular packages) | Zero (subprocess) | Tiny |
| Streaming + pagination | Native (`paginate` plugin) | Manual | Manual |
| Mocking in tests | Easy (msw / nock) | Hard (subprocess stubs) | Easy |

### Risks per option

- **Octokit**
  - Operational: ~30 transitive packages across the org, but you only install what you use.
  - Lock-in: low — REST + GraphQL endpoints are stable, easy to migrate.
  - Maintenance: GitHub-official.
- **gh CLI subprocess**
  - Operational: requires gh installed on every host. Fine for local dev; needs install step in CI Linux containers.
  - Lock-in: gh's CLI surface, not its API surface — gh changes its commands occasionally.
  - Security: gh's token storage (`~/.config/gh/hosts.yml`) is its own attack surface.
- **graphql-request**
  - Operational: no auth handling, no rate-limit handling — you build all of it.

### Recommendation for Rapoport: **Octokit (already in `forge/core`)**

CLAUDE.md already mandates `apps/studio` import `GitHubClient` from `@rapoport-studio/forge/core`. That's Octokit-shaped. Keep it. Use `@octokit/plugin-throttling` + `@octokit/plugin-retry` (already mentioned in Forge audit). Reserve `gh` for *interactive shell flows* where its auth refresh + browser-open are genuinely better than an in-process flow.

---

## 15. Linear integration

**Role in the orchestra:** Forge reads issues, transitions states, posts comments, attaches PRs. Muse will do the same. Maestro probably reads only (calendar-of-work view).

### Top options 2026

| Option | Lang | Auth | Webhook helpers | Notes |
|---|---|---|---|---|
| Linear SDK (`@linear/sdk`) | TS | API key / OAuth | Webhook payloads not typed in SDK (open feature request) | The only official option |
| GraphQL direct (graphql-request + Linear's schema) | TS | API key | Manual | Smaller bundle |

### Comparison on Rapoport's criteria

| Criterion | Linear SDK | GraphQL direct |
|---|---|---|
| TS types | Generated from schema | Manual or codegen yourself |
| Webhook signature verification helper | None — DIY | DIY |
| Attachment management | Native (`linear.attachments.create`) | Manual mutation |
| Cycle / project semantics | Native | Manual |
| Bundle size | Medium-large (full GraphQL schema) | Smaller |

### Risks per option

- **Linear SDK**
  - Operational: SDK pins to a specific schema snapshot; lags behind GraphQL API features by days-to-weeks.
  - Lock-in: low — it's just GraphQL underneath.
  - Maintenance: Linear-official.
  - Known gotcha (from MEMORY): `issueSearch()` is deprecated — use `linear.issues({ filter })`. Don't trust copilot suggestions.
- **GraphQL direct**
  - Operational: you write every mutation.
  - Lock-in: zero.

### Recommendation for Rapoport: **Linear SDK** (keep)

There's no real choice here — the SDK is the only sanctioned path. Wrap it in `cli-core/` so the deprecated-method gotchas have one place to live. Add a webhook-signature verifier ourselves; that gap in the SDK is on the [open issue #596](https://github.com/linear/linear/issues/596).

---

## 16. MCP server framework

**Role in the orchestra:** Each CLI will eventually expose its primitives as MCP tools — Forge's `build`/`plan`/`verify`, Muse's `propose`/`research`, Maestro's `schedule`/`reply`. Studio UI consumes those over MCP.

### Top options 2026

| Option | Maintainer | Transport options | Notes |
|---|---|---|---|
| `@modelcontextprotocol/sdk` | Official Anthropic + community | stdio + Streamable HTTP (+ legacy SSE) | The reference implementation |
| fast-mcp | Community | stdio | Smaller, narrower |
| mcp-framework | Community | stdio | Decorator-style |

### Comparison on Rapoport's criteria

| Criterion | Official MCP SDK | fast-mcp | mcp-framework |
|---|---|---|---|
| Spec compliance | Reference | Behind | Behind |
| Transport options | stdio + Streamable HTTP + (legacy SSE) | stdio | stdio |
| Schema definitions | Zod-native | Custom | Custom |
| Maintenance cadence | Anthropic + active community | Smaller team | Smaller team |
| Cloudflare Workers compat | Streamable HTTP works in Workers | N/A | N/A |

### Risks per option

- **Official MCP SDK**
  - Operational: spec is evolving (Nov 2025 spec introduced Streamable HTTP, replacing legacy SSE). Pin versions.
  - Lock-in: this *is* the standard; lowest possible lock-in.
  - Maintenance: best — Anthropic-owned + community.
- **fast-mcp**
  - Maintenance: lags official spec by definition.
- **mcp-framework**
  - Maintenance: small community.

### Recommendation for Rapoport: **`@modelcontextprotocol/sdk`** (no contest)

The official SDK is the only sane choice. Streamable HTTP is the transport for the studio UI ↔ CLI bridge. Stdio is the transport for Claude Code / Cursor / Codex integrations. Zod-native schema = matches §6 pick exactly.

---

## 17. Vector store (deferred)

**Role in the orchestra:** Not on the roadmap until OpenSpec memory grows past ripgrep range. Forge audit (RAP-358) and the agent training architecture (orchestra-training-architecture-2026.md) both hint at "memory" as a future axis.

### Top options 2026 (short)

| Option | Topology | Hybrid search | Notes |
|---|---|---|---|
| pgvector | Postgres extension | Postgres FTS + ANN | Default if you already have Postgres (we do — Supabase) |
| sqlite-vec | SQLite extension | Limited, no ANN as of early 2026 | Embedded, zero server |
| LanceDB | Embedded columnar | Native | Zero-copy Arrow access |
| Turso libSQL vector | Hosted/Edge SQLite | Limited | Edge-first |
| Chroma | Server | Yes | Local-friendly |

### Recommendation for Rapoport: **pgvector on Supabase** when the time comes

Supabase already runs in `nifagnmgwoqkplegsicy` and exposes pgvector. The "use what you have" answer dominates until embedding-volume justifies a dedicated store. LanceDB is the second pick for an *embedded-in-CLI* memory cache (e.g., per-machine search index over `openspec/`). Defer everything else.

---

## 18. Code-aware embeddings (deferred)

**Role in the orchestra:** Same as §17 — not on roadmap. When needed, it'll be for "find the spec/architecture/research note relevant to *this* prompt" inside agent runs.

### Top options 2026 (short)

| Model | Strength | Price | Notes |
|---|---|---|---|
| voyage-code-3 | Code-specific | $0.18 / M tokens | Best for code retrieval per 2026 benchmarks |
| voyage-3-large | General + code | $0.18 / M tokens | General-purpose state of the art |
| OpenAI text-embedding-3-large | General | $0.13 / M tokens | Strong general retrieval |
| BGE-M3 | Open-source | Self-hosted | Best OSS, comparable to commercial APIs |
| Gemini Embedding 2 | General + code (MTEB Code 84.0) | Google pricing | Strong code variant |

### Recommendation for Rapoport: **voyage-code-3 for code-domain queries, BGE-M3 self-hosted if data-residency wins**

For our content shape (OpenSpec markdown + agent prompts + code references), voyage-code-3 has the best documented head-room over OpenAI. BGE-M3 is the residency-conscious fallback. Don't bother with proprietary-only providers (Cohere, etc.) when Voyage's code-tuning is the best in class.

---

## Findings

> Structured index added 2026-05-29 as part of
> [`inquiry-drift-coverage-retrieval-spike`](./inquiry-drift-coverage-retrieval-spike.md)
> F-7 fix extension. Grade vocabulary per `add-inquiry-entity` design.md §3
> `InquirySource.credibility` enum.
>
> **Capability:** cli-core, forge

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-1 | citty is the right CLI runtime pick over commander / oclif / yargs / cac / clipanion based on UnJS ecosystem fit + Rapoport's existing dep alignment. Bundle: ~typed sub-command shapes that all three CLIs (Forge, Muse, Maestro) can share via `cli-core/`. | engineering_claim | A 6-month maintenance pause on the UnJS org OR citty bundling >2× commander's overhead would justify migration off the pick. |
| F-2 | c12 is the canonical config loader (over cosmiconfig / conf / rc) for its four-level precedence + remote-extends gate. `cli-core/` wrapper pre-configures remote-extends DISABLED — supply-chain hardening. | engineering_claim | A documented c12 vulnerability tied to the remote-extends mechanism that the pre-configured disable doesn't mitigate would refute the supply-chain claim. |
| F-3 | execa v9 is already in use; tinyexec is a smaller alternative but tinyexec lacks the `tee(stdout, file)` pattern Forge already relies on for structured-error capture. Don't migrate. | engineering_claim | A new tinyexec release shipping `tee` semantics + zero-config error capture would re-open the comparison. |
| F-4 | proper-lockfile with `stale` policy is the locking pick over custom `O_EXCL` because of orphan-cleanup. Custom `O_EXCL` leaves stale locks on Ctrl-C without explicit SIGINT handler. (Memory `forge_global_build_lock` documents this pattern.) | engineering_claim | A custom-`O_EXCL` implementation reliably cleaning up orphans under SIGKILL (which proper-lockfile also can't recover from) would tie the comparison. |
| F-5 | lefthook beats husky on benchmarks AND configuration ergonomics (single YAML vs multi-file). Picks: lefthook for pre-commit hooks. simple-git-hooks deferred — no measured advantage at studio scale. | engineering_claim | A husky release closing the benchmark gap to <10% AND offering single-config-file semantics would reopen. |
| F-6 | Zod (full) is the runtime schema pick over Valibot / ArkType / Effect Schema. Bundle size discussion is moot for CLI context (no payload-size constraint); Zod's ecosystem maturity + LLM-output integration via AI SDK is decisive. Apps may prefer Zod Mini. | engineering_claim | A measurable Zod-bundle penalty in the studio CLI distribution (e.g., npx cold-start >500ms attributable to Zod) would reopen. |
| F-7 | Schema-aligned LLM-output: AI SDK + Zod v1 covers the v1 needs. BAML is worth a pilot on one hot path (Forge plan-builder) — schema-aligned parsing claims measurably fewer rejections than retry-on-fail. | engineering_claim | A 4-week BAML pilot on Forge plan-builder showing <5% reduction in plan-validation errors vs Zod-only baseline refutes the pilot's value. |
| F-8 | Vale + custom RapoportStudio package is the prose linter pick. textlint deferred — Vale's vendor-bundled `Microsoft` and `write-good` packages already cover ~80% of the prose-discipline gap. | engineering_claim | A Vale rollout producing FPR >20% on the studio's research corpus would force a textlint reevaluation. |
| F-9 | Langfuse self-hosted + OpenInference spans for LLM observability beats Helicone / Braintrust because of data-residency control. Phoenix (Arize) deferred — useful only when production traffic exists. | engineering_claim | A Langfuse self-hosted operating cost exceeding 2 hours/month of Pavel's time at solo cadence would push toward Braintrust (managed). |
| F-10 | promptfoo is the eval harness pick over Braintrust (SaaS), DeepEval (Python), OpenAI Evals. promptfoo's local-first + git-versionable suite matches the studio's spec-driven discipline. | engineering_claim | promptfoo failing to scale past 500 eval cases (e.g., test run >10min) would justify Braintrust pull-forward. |
| F-11 | Distribution v1: npm + npx. v2: Node SEA (when `--build-sea` lands in stable Node, per Joyee Cheung Jan 2026). Bun compile and Deno compile deferred — both add a runtime-divergence risk for marginal cold-start savings. | engineering_claim | npm + npx cold-start consistently >2s on user machines (worse than Bun compile's claimed <500ms) would reopen the distribution-runtime comparison. |
| F-12 | voyage-code-3 is the embedding pick over OpenAI text-embedding-3 + BGE-M3. Voyage's code-tuning has best-documented head-room on OpenSpec markdown + code-reference content shape. BGE-M3 is the residency-conscious fallback (self-hosted). | engineering_claim | A voyage-code-3 retrieval F1 score within ±2pp of OpenAI on the studio's actual research corpus would tie the comparison and reopen on cost. |

## Stack recommendation summary

| Layer | Component | Pick | Alternative | Defer |
|---|---|---|---|---|
| CLI runtime | Argument parsing | **citty** | commander | oclif, cac, clipanion |
| CLI runtime | Config loader | **c12** | cosmiconfig | conf, rc |
| CLI runtime | Process orchestration | **execa** (already) | tinyexec | zx (security), dax |
| CLI runtime | File locking | **proper-lockfile** (with `stale` policy) | custom O_EXCL | fs-ext (native dep) |
| CLI runtime | Pre-commit hooks | **lefthook** | husky | simple-git-hooks |
| Schema | Runtime validation | **Zod (full)** | Zod Mini in apps | Valibot, ArkType, Effect Schema |
| Schema | LLM-output constraint | **AI SDK + Zod v1**, BAML pilot for one hot path | TypeChat, Instructor TS | DSPy-style |
| Schema | Prose linter | **Vale** + custom RapoportStudio package | textlint | alex, write-good |
| Observability | Event logging | **consola** (already) + custom JSONL for state.json | pino | winston, custom JSONL only |
| Observability | LLM observability | **Langfuse self-hosted** + OpenInference spans | Helicone | Braintrust as primary, LangSmith |
| Observability | Eval harness | **promptfoo** | Braintrust if SaaS becomes acceptable | DeepEval (Python), OpenAI Evals |
| Distribution | CLI packaging | **npm + npx v1**, Node SEA v2 | Bun compile | pkg, nexe, Deno compile |
| Distribution | Skills tooling | **agent-ecosystem/skill-validator** + Anthropic skill-creator on PR | custom | DIY-only |
| Integrations | GitHub | **Octokit** (already in `forge/core`) | gh subprocess for interactive | graphql-request |
| Integrations | Linear | **Linear SDK** (only option) | GraphQL direct | — |
| Integrations | MCP framework | **`@modelcontextprotocol/sdk`** | — | fast-mcp, mcp-framework |
| Memory (deferred) | Vector store | **pgvector on Supabase** when needed | LanceDB embedded | sqlite-vec, Chroma, Turso |
| Memory (deferred) | Embeddings | **voyage-code-3** | BGE-M3 self-hosted | OpenAI text-embedding-3 |

---

## Notes for the `packages/cli-core/` extraction

When the rule-of-three trigger fires (Muse CLI lands, giving us a third consumer alongside Forge and the studio carve-out), the candidate set for `cli-core/` is:

- citty wrappers (typed sub-command shapes that all three CLIs share)
- c12 wrapper (the four-level precedence loader, pre-configured to disable remote `extends`)
- execa wrapper (the `tee(stdout, file)` + structured-error pattern Forge already uses)
- proper-lockfile wrapper (Rapoport-typed locks with our `stale` policy)
- consola wrapper (with the pretty/JSON auto-detection + correlation IDs)
- Octokit / Linear clients (already mandated by CLAUDE.md)
- MCP SDK wrappers (shared Zod schemas for tool inputs/outputs)
- Skill validator + skill-creator orchestration

What does *not* go into `cli-core/`:

- Vale, lefthook, promptfoo, Langfuse → these are repo-level concerns, not library-level.
- BAML codegen → CLI-specific if/when piloted.
- pgvector + embeddings → service-level concerns when they arrive.

---

## Sources verified May 2026

- citty: [github.com/unjs/citty](https://github.com/unjs/citty), [npm](https://www.npmjs.com/package/citty)
- commander vs oclif vs yargs vs cac: [npmtrends](https://npmtrends.com/cac-vs-commander-vs-meow-vs-minimist-vs-oclif-vs-optionator-vs-yargs)
- clipanion maintenance: [libraries.io](https://libraries.io/npm/clipanion)
- c12: [github.com/unjs/c12](https://github.com/unjs/c12)
- execa v9: [execa docs](https://github.com/sindresorhus/execa)
- tinyexec: [github.com/tinylibs/tinyexec](https://github.com/tinylibs/tinyexec)
- proper-lockfile: [github.com/moxystudio/node-proper-lockfile](https://github.com/moxystudio/node-proper-lockfile)
- lefthook benchmarks: [pkgpulse.com husky-vs-lefthook-2026](https://www.pkgpulse.com/blog/husky-vs-lefthook-vs-lint-staged-git-hooks-nodejs-2026), [edopedia 2026](https://www.edopedia.com/blog/lefthook-vs-husky/)
- Zod / Valibot / ArkType bundle sizes: [pkgpulse zod-vs-arktype-2026](https://www.pkgpulse.com/guides/zod-vs-arktype-2026), [valibot.dev/guides/comparison](https://valibot.dev/guides/comparison/)
- BAML schema-aligned parsing: [boundaryml.com/blog/schema-aligned-parsing](https://boundaryml.com/blog/schema-aligned-parsing)
- Vale: [vale.sh](https://vale.sh/), [github.com/vale-cli/vale](https://github.com/vale-cli/vale)
- Pino throughput vs Winston: [betterstack pino-vs-winston](https://betterstack.com/community/guides/scaling-nodejs/pino-vs-winston/), [signoz pino guide](https://signoz.io/guides/pino-logger/)
- Langfuse / Helicone / Braintrust / Phoenix landscape: [laminar langfuse-alternatives-2026](https://laminar.sh/article/langfuse-alternatives-2026), [braintrust deepeval-alternatives-2026](https://www.braintrust.dev/articles/deepeval-alternatives-2026)
- OTel GenAI semconv status: [opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- promptfoo stats: [Braintrust best-promptfoo-alternatives-2026](https://www.braintrust.dev/articles/best-promptfoo-alternatives-2026)
- Bun / Deno / Node runtime cold starts: [bolderapps node-bun-deno performance](https://www.bolderapps.com/blog-posts/node-js-vs-bun-vs-deno-the-ultimate-runtime-performance-showdown)
- Node SEA + `--build-sea`: [Joyee Cheung blog Jan 2026](https://joyeecheung.github.io/blog/2026/01/26/improving-single-executable-application-building-for-node-js/), [nodejs.org/api/single-executable-applications](https://nodejs.org/api/single-executable-applications.html)
- Anthropic Skills open standard adoption: [paperclipped agent-skills-open-standard](https://www.paperclipped.de/en/blog/agent-skills-open-standard-interoperability/), [github.com/agent-ecosystem/skill-validator](https://github.com/agent-ecosystem/skill-validator)
- MCP SDK transports: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- Linear SDK webhook gap: [github.com/linear/linear issue 596](https://github.com/linear/linear/issues/596)
- pgvector / sqlite-vec / LanceDB: [callsphere vector-db-benchmarks-2026](https://callsphere.ai/blog/vector-database-benchmarks-2026-pgvector-qdrant-weaviate-milvus-lancedb), [grokipedia sqlite-vec-pgvector](https://grokipedia.com/page/Comparison_of_sqlite-vec_and_pgvector)
- voyage-code-3 / BGE-M3: [pecollective best-embedding-models](https://pecollective.com/tools/best-embedding-models/), [aimultiple embedding-models](https://aimultiple.com/embedding-models)

Anything marked `[unverified]` in the body is a claim I could not pin to a primary source within this pass — flag for re-verification on the 12-month refresh.
