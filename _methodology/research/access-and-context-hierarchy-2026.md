# Access & Context Hierarchy for Multi-Agent CLI Ecosystems — May 2026

> **Pavel's question:** "How are the levels built? What context is at the top? What skills at the top? What rules at the top? What access to information? Is there access to all research?"
>
> This file answers it for Forge / Muse / Maestro — three CLIs sharing one TypeScript monorepo, one OpenSpec corpus, and one prompt-cache budget.

**Scope.** Research-only — not a spec, not a proposal. Anything that becomes a rule lands in a capability spec or methodology document via the normal OpenSpec change process. The Rapoport-specific sections in Part 3 are recommendations to inform a future change proposal, not decisions.

**Status note.** [`openspec/current/agents/maestro.md`](../../current/agents/maestro.md) already has an `## Access scope` section with explicit DB / file / API / tool allowlists. That spec is the **prior art** this research builds on — Part 3 generalizes the Maestro pattern across all six agents (and three CLIs) rather than inventing something new.

---

## TL;DR

1. **The convergent "on top" pattern across all major AI CLI tools in 2026 is progressive disclosure: descriptions at the top, bodies on demand.** Anthropic Skills frontmatter (≤1024 chars description, loaded into system prompt at startup; SKILL.md body and supporting files loaded only when triggered), AGENTS.md (single Markdown file at repo root, closest wins for nested), Claude Code's CLAUDE.md tier order (managed policy → project → user → local; concatenated walking up the tree; subdirectory files load lazily when Claude reads files in those subdirectories), and Cursor's project-rules `.mdc` system all share one shape: **metadata is hot, body is warm, references are cold**. The token budget for descriptions is ~1% of the model context window in Claude Code (~2,000 tokens on a 200K Sonnet session), capped by a built-in tripwire that warns at session start when descriptions overflow.

2. **The single biggest access-architecture decision Pavel needs to make NOW** is: *capability-based (per-tool allowlist) vs role-based (per-agent role)*. The Maestro spec already chose capability-based (`Access scope` section with explicit DB / path / API / tool allowlists, "Anything not declared is denied", "future tool additions are denied by default unless explicitly added to the allowlist"). **Generalize this convention to a single methodology file (`openspec/_methodology/tools/agent-access-scope.md`) so every per-agent spec inherits the same allowlist shape — before Atlas / Echo / Pulse specs land.** That doc should mandate: (a) explicit DB-table allowlist, (b) explicit path allowlist with read/write distinction, (c) explicit external-API allowlist, (d) explicit tool-name allowlist with "deny by default for future tools" rider, (e) a "Why this list, not more" justification paragraph. This single decision converts a six-row table into a contract that holds up under audit and prompt injection.

3. **Top 3 access-control mistakes to avoid based on 2025-2026 incidents:**
   - **Confused-deputy via shared OAuth/PAT.** The Meta March 2026 Sev-1 (rogue agent inherited elevated forum-admin scope from a credential it was meant to share, exposed two hours of data to hundreds of engineers before detection), the April 2026 Vercel incident (compromised third-party AI tool's Google Workspace OAuth app), and the "Comment and Control" attacks on Claude Code / Gemini CLI Action / Copilot Agent (GitHub Issues used as command-and-control to exfiltrate the runner's GITHUB_TOKEN scope) all share one shape: **one token, shared by all agents, scoped to the union of what any of them might need.** Forge's `GITHUB_BOT_TOKEN` and `LINEAR_BOT_API_KEY` are exactly this shape today. The fix is per-agent identity with audience-bound tokens (RFC 8707 resource indicators) — not "Maestro reuses Forge's clients" except as a documented carve-out with a tripwire.
   - **Tool poisoning via MCP description.** Postmark-mcp npm rug-pull (September 2025), WhatsApp-MCP rug-pull (2025), GitHub MCP toxic agent flow (2025), CVE-2025-54136 (Cursor MCP filename case-sensitivity bypass), CVE-2025-59536 (Claude Code RCE via project config on untrusted directory open), CVE-2026-21852 (Claude Code project-load info disclosure including API keys), the "Claudy Day" trio (March 2026, Oasis Security) — every one of these attacks lands in **content the agent reads from a path the agent treats as trusted**. For Rapoport, the equivalent attack surface is `openspec/_methodology/research/` (anyone can drop a markdown file with hidden instructions) and any future MCP server bundled into a CLI. Mitigation: treat all repo-root config as untrusted-by-default on first open (Claude Code's "External imports" approval dialog is the pattern); never load supporting Markdown into the system prompt — load it as a user-role message labeled as content; sign or hash-pin any MCP server bundled into a CLI.
   - **Cache poisoning via shared L3 corpus.** [unverified-as-of-2026-05-12] Anthropic switched prompt-cache isolation from organization-level to workspace-level on 2026-02-05, and the cache TTL silently regressed from 1h to 5m in early March 2026 (issue #46829 on `anthropics/claude-code`). If Forge / Muse / Maestro share a single prompt prefix (corpus block), a poisoned methodology file is the input for all three; if they share a workspace, all three pay the cache-miss cost on rotation. The mitigation is to keep the shared corpus block append-only and reviewed (anything in `_methodology/research/` is reviewable through the OpenSpec change process before it lands) and to put role-specific system prompts at the *top* of the prompt (per Anthropic's `tools → system → messages` cache hierarchy) so role-specific differences invalidate only the role-specific suffix, not the shared corpus prefix.

---

## Part 1: Access control patterns

### 1.1 Role-based (RBAC) vs capability-based (CapBAC)

**RBAC** attaches permissions to a role; the identity carries the role; the system maps role → permissions at check time. **CapBAC** makes the capability the token itself — possession is permission, attenuation is delegation. The two are not exclusive (real systems usually do both) but they suit different scopes.

| Property | RBAC | CapBAC |
|---|---|---|
| Permission lookup | Indirect (identity → role → permissions) | Direct (token = permission) |
| Delegation | Awkward (re-assign role) | Native (attenuate token, hand off) |
| Revocation | Easy (remove role) | Harder (need short TTL or revocation list) |
| Fit for AI agents | Poor — agent's effective role shifts per turn | Strong — capability scoped to current task |
| Audit trail | Role grants are durable, low-volume | Each delegation is recorded, high-volume |
| Industry examples | Linux groups, AWS IAM, Linear roles | macaroons, OAuth scopes, Linux capabilities(7), Object capabilities (E, Pony) |

**Why CapBAC fits multi-agent CLI better in 2026.** RBAC was designed for humans with stable job functions ([TianPan blog, April 2026](https://tianpan.co/blog/2026-04-20-rbac-ai-agents-authorization)). An agent's effective role shifts mid-conversation — read-only research → write-capable code execution → external API calls — and static roles cannot express this. The industry response is to issue the agent its own OAuth client ID, scope its access to what the current task requires, and make it expire when the task ends ([GitGuardian on AI agent authentication, 2026](https://blog.gitguardian.com/ai-agents-authentication-how-autonomous-systems-prove-identity/)).

**Macaroons** ([Birgisson et al., Google Research, 2014](https://research.google.com/pubs/archive/41892.pdf); [libmacaroons](https://github.com/rescrv/libmacaroons)) are the canonical CapBAC primitive for delegated AI agent authority in 2026. A macaroon is a token plus a chain of caveats (HMAC-bound conditions: "expires in 5 minutes", "only for resource X", "only operation read", "and third party Y must certify Z"). A parent agent receives a macaroon, appends a tighter caveat ("ttl < 60s", "audience = subagent-Y") before handing it to a subagent. The receiver cannot remove caveats — only add more. Google DeepMind's "Intelligent AI Delegation" paper (February 2026) lands on macaroons as the answer for safe agent task decomposition ([summary via DEV Community](https://dev.to/mattdeangit/what-google-deepmind-gets-right-about-agent-delegation-and-what-satgate-already-built-3e0a)).

**OAuth scopes** are RBAC-shaped (the scope set is fixed at token issuance), but the new MCP authorization spec (`2026-03-15`, [modelcontextprotocol.io](https://modelcontextprotocol.io/specification/draft/basic/authorization)) bolts on macaroon-like properties via RFC 8707 Resource Indicators: every token must declare its `resource=` (audience), and MCP servers MUST reject tokens that aren't bound to them. This is **audience-bound scope attenuation** in the OAuth idiom. The "Step-Up Authorization Flow" (returns 403 + `WWW-Authenticate: scope="required_scope"`) is **runtime scope escalation** — the OAuth equivalent of attenuating-then-re-minting a macaroon.

**For Rapoport.** Maestro's `Access scope` allowlist (`openspec/current/agents/maestro.md § Access scope`) is already CapBAC: explicit tool list, deny-future-by-default. The convention should be lifted into a methodology doc and applied to every agent (see Part 3.1).

### 1.2 Layered context loading — what's "on top" in 2026 practice

The four major conventions, with their precedence rules:

| Convention | Top-level discovery | Precedence rule | Token cost |
|---|---|---|---|
| **Anthropic Skills** ([best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)) | YAML frontmatter `name` + `description` (≤1024 chars) injected into system prompt at startup | Closest-relevant-description wins at trigger time; Claude reads SKILL.md only when one matches | Pre-load ≈ N × ~150 tokens per Skill metadata; body 0 tokens until triggered, then ≤500 lines budget |
| **CLAUDE.md** ([Claude Code memory docs](https://code.claude.com/docs/en/memory)) | Concatenated walking from filesystem root down to cwd; subdirectory files load lazily when Claude reads files there | Order: filesystem-root → cwd (closer = later = effectively higher priority because LLMs weight recency); `CLAUDE.local.md` appended after `CLAUDE.md` at same level; managed policy file cannot be excluded | All ancestor `CLAUDE.md` files load in full at launch (no body lazy-loading except for subdirectory descendants) |
| **AGENTS.md** ([agents.md](https://agents.md/), [GitHub blog on 2500 repos](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)) | Single Markdown file at repo root; nested files in subdirs allowed | Closest AGENTS.md to the edited file wins; explicit user chat prompts override everything | Loaded at session start by Codex CLI / Cursor / Continue / Aider / OpenHands / Windsurf; Claude Code reads `CLAUDE.md`, not `AGENTS.md` — convention is to `@AGENTS.md`-import from `CLAUDE.md` |
| **Cursor rules** ([Cursor docs](https://cursor.com/docs/rules), [Medium guide, April 2026](https://medium.com/@vibecodingdirectory/how-to-structure-cursor-rules-in-2026-the-5-level-system-cursor-rules-eaf0df16e8e7)) | `.cursor/rules/*.mdc` files with YAML frontmatter (globs, `alwaysApply`, etc.) | User rules → project rules → auto-attached (by glob) → manually-referenced (`@ruleName`); project rules override user rules; auto-attached files load only when their globs match | All `alwaysApply: true` rules in system prompt; glob-matched rules load on file-read |
| **llms.txt** ([codersera guide May 2026](https://codersera.com/blog/llms-txt-complete-guide-2026/)) | Root-level `/llms.txt` with curated links to highest-value content | Read-time only — not auto-loaded by any major CLI; IDE agents (Cursor, Continue, Cline, Aider) consult it when pointed at a docs site | ~10% adoption across ~300k domains audited by SERanking; OpenAI / Google / Anthropic crawlers don't fetch it in measurable volume |

**The convergent pattern**: every modern AI CLI has settled on **progressive disclosure**. Description fields, frontmatter, and ancestor files load at startup (hot tier); bodies and rules-by-glob load on demand (warm tier); reference docs and supporting files load only when explicitly accessed via filesystem tools (cold tier). The Anthropic Skills documentation calls this out explicitly: *"At startup, only the metadata (name and description) from all Skills is pre-loaded. Claude reads SKILL.md only when the Skill becomes relevant, and reads additional files only as needed."* ([Skill best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)).

### 1.3 Filesystem-level access for agent CLIs

The 2026 landscape of sandboxing primitives, ranked from heaviest to lightest:

| Mechanism | Layer | Where used | Trust assumption |
|---|---|---|---|
| **microVM** (Firecracker, Cloud Hypervisor) | Hypervisor | Claude Code for web; Cloudflare / Vercel / Modal / E2B sandboxes | Strongest isolation; cold-start latency, no host FS access |
| **Container** (Docker, gVisor) | Kernel-namespaces | Codex CLI cloud sandbox; Northflank, Firecrawl | Strong; cgroups + namespaces |
| **bubblewrap / namespaces+seccomp** | Kernel | Anthropic `sandbox-runtime` on Linux, Flatpak, nsjail | Unprivileged sandbox; no root needed |
| **Landlock** (Linux ≥5.13) | Kernel-LSM | Codex CLI default on Linux | Per-process FS allowlist, no namespaces required |
| **macOS Seatbelt (`sandbox-exec`)** | XNU mac_policy | Claude Code's bash tool, SandVault, `bx`, `nono` | Deprecated-but-functional; "irrevocable once applied" |
| **Path-allowlist (in-process)** | Application | Gemini CLI's "trusted folders", Cursor sandbox settings | Trust the agent code; bypassable via subprocess |
| **chroot / bind-mounts (FUSE)** | Kernel-VFS | Older patterns | Brittle for AI agents; symlink escape, /proc leakage |

**The 2026 status quo**: AI CLIs converge on **per-process FS+network sandbox via OS primitives**. Codex CLI ships sandbox-by-default (Seatbelt on macOS, bubblewrap+seccomp on Linux). Claude Code wraps its bash tool in the same primitives via `anthropic-experimental/sandbox-runtime`. Cursor opt-in. Gemini CLI uses application-level trusted folders. References: [Cursor sandboxing](https://cursor.com/blog/agent-sandboxing), [a field guide to sandboxes for AI](https://www.luiscardoso.dev/blog/sandboxes-for-ai), [Codex sandboxing](https://developers.openai.com/codex/concepts/sandboxing).

**For Rapoport.** Forge runs `git`, `pnpm`, `gh`, opens files anywhere under repo root, and dispatches subprocesses (Builder mode). Muse runs in two places — `apps/studio` (Cloudflare Worker, network-restricted) and locally via `runMuseHeadless`. Maestro is read-only by spec. The right move is: lean on the spec-declared allowlist (Maestro's `Access scope` model) for the *what*, and add per-CLI sandbox-runtime config for the *enforcement* — but enforcement is a v1.5+ concern, not blocking. **Specs first, sandboxes second.**

### 1.4 Secret access hierarchies

Three patterns in 2026, ranked by least-trust:

| Pattern | What | Examples | Trust |
|---|---|---|---|
| **Just-in-time injection via proxy** | Agent makes HTTP request, local proxy injects credential at the network layer; agent never sees the secret | Infisical Agent Vault ([blog](https://infisical.com/blog/agent-vault-the-open-source-credential-proxy-and-vault-for-agents)), 1Password Unified Access ([March 2026 launch](https://1password.com/press/2026/mar/1password-unified-access)) | Strongest |
| **Just-in-time injection via env-var** | Wrapper resolves secret at process-start, exports to env, child sees plaintext for lifetime | `infisical run --env=prod --path=/`, `op run` | Medium — process can leak via logs/dumps |
| **Ambient env-var** | Secret in shell `~/.zshrc` / `.env` / system keychain | `ANTHROPIC_API_KEY` set in shell forever | Weakest |

**Path-based hierarchy in Infisical** (Rapoport's chosen provider): projects → environments → folders → keys. Permissions follow the CASL pattern: a token can be granted access to `prod/forge/*` but not `prod/web/*`. Cross-environment dynamic references via `${SECRET_NAME}` syntax allow one project to alias another. ([Infisical secrets-management best-practices, 2026](https://infisical.com/blog/secrets-management-best-practices))

**For Rapoport.** Today: `infisical run --env=prod --path=/` injects everything at the root of the prod workspace into Forge's env — Forge gets `ANTHROPIC_API_KEY`, `GITHUB_BOT_TOKEN`, `LINEAR_BOT_API_KEY` ambient (per [`MEMORY.md` infisical_workspace_state](https://example.invalid/infisical_workspace_state.md)). When Muse and Maestro come online, that root-level grant is too coarse — Maestro should not need GitHub credentials at all. The path hierarchy is **the** mechanism: `/forge/*`, `/muse/*`, `/maestro/*` paths with three different Infisical tokens, each injected only by the matching CLI. **This is the secret-access analogue of Maestro's `Access scope` allowlist.**

### 1.5 Inter-CLI communication trust model

| Pattern | Mechanism | Trust assumption | Auditability |
|---|---|---|---|
| **Filesystem hand-off** | Write artifact, the other CLI reads | Both CLIs trust the path; both can write; race conditions possible; the path is the contract | Whatever git/audit tooling tracks |
| **Append-only journal (JSONL)** | One writes, many read; never edit history | All readers trust the journal's integrity; a corrupted line affects all readers | Native — every event is a line |
| **Named pipe / Unix socket** | Local, ephemeral, one-process-to-another | Only the process can authenticate the caller | Transient |
| **Local HTTP with token** | One CLI runs a server on `127.0.0.1:port` | Token is the auth; localhost is the perimeter | HTTP access logs |
| **MCP local server** | Stdio-based MCP, no auth (per spec, "retrieve credentials from the environment") | Process-level; same-user filesystem trust | Caller logs only |
| **A2A protocol** | HTTP-over-network with signed Agent Cards (v1.0) | Cryptographic — signed AgentCard verifies identity; payload schema is JSON-RPC over HTTP | Native — protocol-level signing |

**Filesystem hand-off is the dominant 2026 pattern for local multi-agent.** [agentic-stack](https://toolhunter.cc/tools/agentic-stack) uses `.agent/` directory as source of truth with harness-specific adapters. [Beads memory system](https://devengoratela.com/2026/02/the-beads-memory-system-technical-architecture-and-integration-with-gemini-cli-for-agentic/) uses `.beads/issues.jsonl` as the canonical store, hydrated by a background daemon into SQLite. The [MEMTIER architecture](https://arxiv.org/html/2604.23280) uses tripartite memory with a JSONL episodic store. The pattern recurs because: git-friendly, replayable, debuggable, no daemon required.

**MCP authorization spec (2026-03)** ([spec](https://modelcontextprotocol.io/specification/draft/basic/authorization)) mandates RFC 8707 Resource Indicators for HTTP-based transports — the `resource=` parameter binds tokens to a specific MCP server's canonical URI. Stdio-based MCP is explicitly carved out: *"Implementations using a STDIO transport SHOULD NOT follow this specification, and instead retrieve credentials from the environment."* This is correct for the threat model (local subprocess of the same user) but **means stdio MCP cannot enforce inter-CLI access boundaries** — that has to be done elsewhere.

**A2A protocol** ([Google A2A](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)) reached 150+ organizations by April 2026 and shipped v1.0 with cryptographically-signed Agent Cards in Q1 2026. It's overkill for three CLIs sharing a filesystem; it's the right answer for cross-org agent-to-agent.

**For Rapoport.** Forge's `~/.forge/` directory is already the filesystem hand-off model (build state, lessons-learned, lineage). Maestro's spec adds `~/.maestro/journal/` (append-only Markdown mirror of `orchestra_journal` table). The DB table `orchestra_journal` (provisioned in Layer 2) is the canonical journal — JSONL on disk is a secondary mirror. **This is the right pattern; the only thing missing is appending to the same shared journal from all three CLIs vs three separate journals.** (Recommendation in Part 3.4.)

---

## Part 2: Information hierarchy — what "on top" actually means

### 2.1 Progressive disclosure architectures in production

Three concrete examples:

**Anthropic Skills (Claude Code)** ([Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), [Skills budget setting](https://claudefa.st/blog/guide/mechanics/skill-listing-budget))

```
System prompt @ session start:
├─ Core system prompt (always)
├─ Tool definitions (always)
├─ Skill metadata, all installed Skills (name + ≤1024-char description)
│   └─ Budget: ≤ ~1% context window = ~2,000 tokens on 200K Sonnet
│   └─ Overflow: least-used Skills' descriptions get dropped first
├─ User CLAUDE.md, project CLAUDE.md, managed policy CLAUDE.md (full text)
└─ Auto-memory MEMORY.md (first 200 lines or 25KB)

Lazy-loaded (on demand):
├─ Triggered Skill: full SKILL.md body (≤500-line target)
│   └─ Skill-referenced files: load via filesystem tools, one level deep
├─ Subdirectory CLAUDE.md (when Claude reads files in that subdirectory)
└─ Path-scoped rules (when matching files are read)
```

**Source numbers:** The 1% budget figure and v2.1.129 tripwire are from [claudefa.st on skill listing budget, May 2026](https://claudefa.st/blog/guide/mechanics/skill-listing-budget). The 200-line/25KB MEMORY.md cap is from the [Claude Code memory docs](https://code.claude.com/docs/en/memory).

**Cursor rules (`.cursor/rules/*.mdc`)** ([Cursor docs](https://cursor.com/docs/rules))

```
System prompt @ session start:
├─ Cursor's core agent prompt
├─ User Rules (global, ~/.cursor/rules.json) — always applied
├─ Project Rules with alwaysApply: true — always applied
└─ Manually-referenced rules (@ruleName) — present this session only

Lazy-loaded:
├─ Auto-attached rules (matching glob patterns from current file)
└─ Manually-attached rules (next @ruleName invocation)
```

**llms.txt + llms-full.txt** ([codersera May 2026](https://codersera.com/blog/llms-txt-complete-guide-2026/))

```
Web-fetch on demand (NOT auto-loaded by any major CLI):
├─ /llms.txt — curated index, Markdown, links to high-value content
└─ /llms-full.txt — concatenated full content (less common)
```

Both are read-time-only, fetched when an IDE agent is pointed at a docs site. Crawler adoption (OpenAI / Google / Anthropic) is unmeasurable per [SERanking 2026 audit of ~300k domains](https://seranking.com/blog/llms-txt/).

### 2.2 What "context is at the top" actually means in practice

Three tiers:

| Tier | Anthropic cache binding | Token cost | What lives here |
|---|---|---|---|
| **Hot — system prompt** | `tools → system → messages` cache hierarchy ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)) | Pays full per-token cost on every request unless cached | Role identity, tool definitions, Skills metadata, ancestor CLAUDE.md, always-apply rules |
| **Warm — first-page memory** | Loaded into messages tier; subject to compaction | Cheaper-than-cold (cache-aware), but counts against context | MEMORY.md first 200 lines / 25KB, project-rules, triggered-Skill body |
| **Cold — filesystem reference** | Not loaded until tool-read | Zero until accessed; full price when accessed | Skill-supporting files, subdirectory CLAUDE.md, `_methodology/research/*`, archive |

**Real numbers (Claude Code, May 2026):**

- **Skills metadata**: ~150 tokens × N installed Skills, capped at 1% of context window = **~2,000 tokens** on a 200K Sonnet session ([claudefa.st budget guide](https://claudefa.st/blog/guide/mechanics/skill-listing-budget)).
- **MEMORY.md**: capped at **first 200 lines or 25KB**, whichever comes first ([Claude Code memory docs](https://code.claude.com/docs/en/memory)).
- **CLAUDE.md ancestors**: no cap, loaded in full; target **<200 lines per file**.
- **Triggered Skill body**: target **<500 lines**.
- **Auto-memory topic files** (`debugging.md`, etc.): **0 tokens until read by Claude on demand.**

The cache hierarchy matters because Anthropic builds the cache prefix as `tools → system → messages` ([prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)). Anything that changes per-CLI (role identity, tool definitions) must live at the top of *its* tier; anything shared across CLIs (corpus, methodology) should be at the bottom of its tier so the longer prefix still hits cache across CLIs.

### 2.3 Skills / rules ordering

**Three competing ordering rules in 2026 practice:**

| Rule | Where it applies | Mechanism |
|---|---|---|
| **Closest wins** | AGENTS.md (`agents.md`); CLAUDE.md (effective via concatenation order — closer is read last, so LLMs weight it heaviest); Cursor project rules over user rules | Filesystem-proximity to the file being edited |
| **Explicit override** | User chat prompts override everything in every system; manually-referenced (`@ruleName`) overrides auto-attached | Most-recent-explicit-mention wins |
| **Specificity wins** | Path-scoped rules (Cursor's globs, Claude Code's `paths` frontmatter) load only when matching | Glob matches → rule activates; non-matching → rule absent |

The Claude Code memory docs are explicit: *"Across the directory tree, content is ordered from the filesystem root down to your working directory. For the `foo/bar/` example, `foo/CLAUDE.md` appears in context before `foo/bar/CLAUDE.md`, so instructions closer to where you launched Claude are read last."* ([memory docs](https://code.claude.com/docs/en/memory#how-claude-md-files-load))

**Predictability heuristic.** Designing ordering so agents are predictable means **make the dimension of override visible in the filename or location**. Cursor's `paths:` frontmatter, Claude Code's `paths:` in `.claude/rules/`, AGENTS.md proximity — every working pattern in 2026 ties precedence to the structure on disk, not to runtime negotiation.

### 2.4 Cross-CLI knowledge propagation

The two designs:

**Shared corpus (one file, many readers).** All three CLIs read `openspec/_methodology/` from the same path. New finding → one PR → all three CLIs pick it up on the next session. The risk: change-impact is global; a poisoned methodology file is the input for all three CLIs simultaneously (cache-poisoning angle in §1.5 / TL;DR mistake #3).

**CLI-specific corpus (per-CLI doc folder).** Forge reads `openspec/_methodology/forge/`, Muse reads `openspec/_methodology/muse/`, Maestro reads `openspec/_methodology/maestro/`. Update fanout is manual.

**The 2026 convergent design is shared corpus + per-CLI rules.** Anthropic Skills work this way (Skills shared, agent-specific system prompt distinct). agentic-stack ([toolhunter.cc](https://toolhunter.cc/tools/agentic-stack)) uses `.agent/` as the shared brain and per-harness adapters. OpenSpec itself works this way (specs are shared; tool-specific adapters consume them). Beads ([devengoratela.com](https://devengoratela.com/2026/02/the-beads-memory-system-technical-architecture-and-integration-with-gemini-cli-for-agentic/)) shares `.beads/issues.jsonl` across Gemini CLI and any other harness.

**For Rapoport.** OpenSpec is the corpus; per-CLI access scope is the variation. The Maestro spec gets this right: Maestro reads `openspec/current/**`, `_methodology/**`, `archive/**`, `changes/**` (research corpus is shared), but its **tool allowlist** differs from Forge's. Same source, different read/write profiles.

---

## Part 3: Rapoport's three-CLI specifics

### 3.1 Per-CLI access matrix (recommended)

This table generalizes [`openspec/current/agents/maestro.md § Access scope`](../../current/agents/maestro.md) to all three CLIs. **Not yet a spec — research output for review.**

| Resource | Forge | Muse (Architect) | Muse (Builder) | Maestro |
|---|---|---|---|---|
| **`openspec/current/**`** | read | read | read | read |
| **`openspec/changes/**`** | read + write (Builder applies diffs at archive); read only otherwise | read + write (Architect authors proposal.md / design.md / tasks.md via emit tools) | read; write only via Architect emit tools (not direct fs) | read |
| **`openspec/_methodology/**`** (research corpus) | read | read; write via emit tools when promoting research → spec | read | read |
| **`openspec/archive/**`** | read | read | read | read |
| **Source code (`apps/`, `packages/`)** | read + write (all) | read | read + write (within issue scope) | none |
| **`events.jsonl` / DB `orchestra_journal`** | emit (build events) | emit (mode changes) | emit (build events) | emit (route decisions, audit observations) + read (replays) |
| **`agent_status` DB table** | write own row | write own row | write own row | read all rows |
| **`~/.forge/`** | read + write | none | none | none |
| **`~/.maestro/journal/`** | none | none | none | append-only write |
| **`~/.muse/`** [if needed] | none | read + write | read + write | none |
| **Anthropic API** | yes (Builder + audit roles) | yes (Architect, Builder) | yes (Builder) | yes (single role) |
| **GitHub API** | read + write (PR creation, branch ops via `forge/core/github-client.ts`) | none | none | none |
| **Linear API** | read + write (issue comments, status transitions) | read (issue context for Architect) | read | read only (routing context) |
| **Supabase** | read (event emission); write via service role only in CI | read + write (project_state, conversations) | read | read (orchestra_journal, agent_status, dialog_registry); no PII tables |
| **Infisical (secrets)** | `infisical run --path=/forge --env=prod` | `infisical run --path=/muse --env=prod` | inherits Forge's env when spawned as subprocess | `infisical run --path=/maestro --env=prod` |
| **Cloudflare API** | none (deploy via GH Actions, not local) | none | none | none |
| **Tool allowlist** | (full Forge tool set: bash, write_file, edit_file, git, gh, anthropic, linear, openspec) | Architect emit tools + read-only research tools (per [`muse.md § Research tools + persona system`](../../current/muse.md)) | Builder tool set (executes Architect-authored plans) | read-only allowlist (per existing Maestro spec) |
| **Sandbox profile** [v1.5+] | Seatbelt/bubblewrap, repo-root only | Worker sandbox (Cloudflare) for studio mode; Seatbelt for headless | Seatbelt with build worktree only | Seatbelt read-only profile |

**Key carve-outs already documented in CLAUDE.md:**

- `apps/studio` may import `@rapoport-studio/forge/core` for `GitHubClient`, `LinearClient`, `loadForgeIdentity` — but only the server-side clients. The rest of forge is CLI-only.
- `packages/muse` may import `@rapoport-studio/forge/core` for auth + Anthropic + Linear + GitHub client primitives, as a pre-extract carve-out until rule-of-three triggers a `packages/cli-core/` extraction.

These carve-outs are **shared-code boundaries**, not shared-permission boundaries. Muse importing Forge's `GitHubClient` constructor does not give Muse a GitHub token — the token still comes from Muse's own Infisical-injected env. The capability is "Muse can construct a GitHub-talking object"; the token is what authorizes specific calls. **This separation is the right CapBAC pattern.**

### 3.2 Context-loading order per CLI (recommended)

Each CLI loads three blocks at startup, in this order (matches Anthropic's `tools → system → messages` cache hierarchy):

```
┌────────────────────────────────────────────────────────────────┐
│ TIER 1 — TOOL DEFINITIONS (cached, per-CLI)                    │
│ Per-CLI tool registry. Maestro: 9 read tools. Forge: ~25       │
│ tools. Muse: ~15 tools per mode.                               │
├────────────────────────────────────────────────────────────────┤
│ TIER 2 — SYSTEM PROMPT (cached, per-CLI)                       │
│ a) Identity: which CLI, which mode (Architect / Builder / etc) │
│ b) Persona system instructions                                 │
│ c) Capability spec link (e.g. `openspec/current/agents/...`)  │
│ d) Access-scope summary (allowlist of paths / APIs / tools)    │
│ e) Always-apply rules from per-CLI methodology                 │
├────────────────────────────────────────────────────────────────┤
│ TIER 3 — METHODOLOGY CORPUS (cached, SHARED across all 3 CLIs) │
│ a) Skills metadata (≤1024-char descriptions, all installed)    │
│ b) openspec/_methodology/manifesto.md (when it lands, Phase B) │
│ c) openspec/_methodology/studio-charter.md                     │
│ d) openspec/_methodology/tools/* (operating conventions)       │
│ e) Per-CLI index of relevant `_methodology/research/*` files,  │
│    NOT the bodies (loaded on demand via file tools)            │
└────────────────────────────────────────────────────────────────┘

LAZY-LOADED (zero token cost until accessed):
├─ openspec/current/<capability>.md (when a specific capability surfaces)
├─ openspec/_methodology/research/<topic>.md (when topic surfaces)
├─ openspec/archive/<date>-<change>/* (when historical context needed)
├─ Per-CLI subdirectory rules (when Claude reads files in those subdirs)
└─ Triggered Skill body + supporting files
```

**Why this order matches the cache hierarchy.** Tools and system prompt are role-specific — they invalidate per-CLI. The methodology corpus is shared — it's the longest cacheable prefix that survives across all three CLIs **only if it sits below the role-specific block**, which is exactly what `tools → system → messages` gives you when the methodology corpus lives in the messages tier as a high-frequency user-role message at the start of every conversation. [unverified-as-of-2026-05-12 — needs prototype to validate cache-hit rate]

**What's "on top" for each CLI:**

- **Forge:** Builder identity + per-issue worktree context + Linear issue body + Architect's `tasks.md`. The methodology corpus is below — Forge cares more about the specific task than the operating philosophy.
- **Muse Architect:** Persona system (Architect-voice) + emit-tool reminders + project state + conversation history. Methodology corpus matters more here (research → spec promotion benefits from broad context).
- **Muse Builder:** Same Architect block + builder-specific gates (verification, lessons-learned).
- **Maestro:** Routing-context (who owns what, current `orchestra_journal` tail) + audit prompts (drift signals to surface) + minimal everything-else. Maestro is read-only; its system prompt is heaviest on **non-responsibilities** (the negative list in `agents/maestro.md § Function`).

### 3.3 Cache hierarchy alignment

Anthropic's prompt cache (May 2026) has two TTLs — 5-minute default and 1-hour at additional cost (1.25× write vs 2× write per `cache_control: { ttl: "1h" }`). Per the [Anthropic prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching), and the Spring AI integration's [coverage of 1h caching](https://spring.io/blog/2025/10/27/spring-ai-anthropic-prompt-caching-blog/), cache prefixes are built in order `tools → system → messages`.

The four-layer prompt cache Pavel referenced maps to:

| Layer | TTL | Cost vs base | Content | Shared across CLIs? |
|---|---|---|---|---|
| **L1 — system prompt (role)** | 5m | 1.25× write | Per-CLI role + identity + persona | No |
| **L2 — tools + skills metadata** | 5m | 1.25× write | Per-CLI tool registry + Skill descriptions | Partly (shared Skills are cross-CLI) |
| **L3 — corpus (shared)** | 1h | 2× write | `_methodology/*`, agent-overview, manifesto, studio-charter | **Yes — this is the win** |
| **L4 — live state (uncached)** | n/a | 1× | Conversation messages, in-flight Linear issue, current build state | No |

**The 2026-02-05 workspace-isolation change** ([Anthropic prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)) means: caches are isolated per workspace within an org. Three CLIs running under the same Anthropic workspace will share L3 corpus cache. Three CLIs running under separate workspaces will not. **For Rapoport, keep all three CLIs in one workspace** unless the orgs-of-record split (which would force separate workspaces anyway).

**The TTL silent-regression incident** ([GitHub issue #46829, March 2026](https://github.com/anthropics/claude-code/issues/46829)) is the cautionary note: cache behavior can change without API surface change. Treat 1h-TTL as a cost optimization, not a correctness guarantee.

### 3.4 Permission escalation paths

Three named scenarios:

**Scenario A — Muse Architect drafts a red-line spec (RLS/auth).** Architect mode authors a change proposal that touches authentication or row-level security. The proposal lands in `openspec/changes/<change-id>/proposal.md` via the emit-tool sink (no fs side effects on emission per [`muse.md § Sink pattern`](../../current/muse.md)). The proposal is staged; a human reviews; Forge picks up the change once Linear status flips to ready-for-build.

- **No automatic escalation.** Architect cannot promote its own draft.
- **Maestro can surface that the proposal exists** in its routing log (`orchestra_journal` entry: "Architect drafted RLS change, awaiting Pavel review").
- **Forge picks up only after Pavel transitions the Linear issue** (the `forge build` command checks `issueState === "ready-for-build"`).

This is the right pattern. Don't change it.

**Scenario B — Forge fails verification.** Builder runs the verification gate, the gate fails. Per [`forge.md § Build outcome`](../../current/forge.md), Builder sets `BuildState.outcome = "failed"` and stops. The orchestrator's contract is to:
- Emit a `build_failed` event to `~/.forge/events.jsonl`.
- Append to lessons-learned (`~/.forge/lessons-learned.md`) per [`forge.md § Lessons-learned mechanism`](../../current/forge.md).
- Update the Linear issue with a comment.

**Who's notified?** Today: Pavel-via-Linear. Tomorrow: Maestro reads the `build_failed` event (or `orchestra_journal` entry, after Layer 2) and can route follow-up. Maestro **does not retry** — it surfaces. The retry decision is Pavel's or Architect's.

**Scenario C — Maestro detects drift.** Maestro reads `orchestra_journal`, sees an agent stuck on work that crosses its declared scope (e.g., Forge editing files under `_methodology/` when it's only supposed to edit code and changes/). Per [`maestro.md § Function`](../../current/agents/maestro.md), Maestro:
- Records an audit observation in `orchestra_journal` (append-only).
- Surfaces to Pavel — likely via a `_audit/observations.md` file (Layer 2) or a Linear comment (no — Maestro doesn't write Linear).
- **Does not act.** The decision is Pavel's.

**What authority does Maestro have to enforce?** None. Maestro is a router and an auditor. The enforcement layer is downstream: when sandbox profiles ship (v1.5+), the OS-level access-scope allowlist is what blocks Forge from editing `_methodology/` — not Maestro's audit. **This is the right separation.** Maestro is a smoke alarm, not a fire suppression system.

---

## Synthesis: the access architecture in one diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 4 — ENFORCEMENT (v1.5+, OS-level)                         │
│ • sandbox-exec / bubblewrap profile per CLI                     │
│ • Infisical path-scoped tokens per CLI (/forge, /muse, /maestro)│
│ • MCP audience binding (RFC 8707) for any MCP server bundled    │
└──────────────────────────────────▲──────────────────────────────┘
                                   │ profiles derived from
┌──────────────────────────────────┴──────────────────────────────┐
│ LAYER 3 — DECLARATION (spec, today)                             │
│ Per-agent capability spec § Access scope                        │
│ Explicit allowlists: DB tables, paths, APIs, tool names         │
│ Deny-by-default for future tools                                │
│ Lives in: openspec/current/agents/<name>.md                     │
└──────────────────────────────────▲──────────────────────────────┘
                                   │ inherits shape from
┌──────────────────────────────────┴──────────────────────────────┐
│ LAYER 2 — CONVENTION (methodology, recommended new doc)         │
│ openspec/_methodology/tools/agent-access-scope.md               │
│ The shape every per-agent spec § Access scope must follow       │
│ Five required subsections (DB / paths / APIs / tools / why)     │
└──────────────────────────────────▲──────────────────────────────┘
                                   │ frames itself by
┌──────────────────────────────────┴──────────────────────────────┐
│ LAYER 1 — PHILOSOPHY (manifesto, Phase B of philosophy-foundn)  │
│ "Agents are bounded by negation, not assertion"                 │
│ "The spec is the contract; the runtime adapts to it"            │
│ Lives in: openspec/_methodology/manifesto.md (pending)          │
└─────────────────────────────────────────────────────────────────┘

  CONTEXT LOADED INTO EACH AGENT (per-session):
  ──────────────────────────────────────────────
   HOT  ▶  Tool registry (per-CLI)        ─┐
        ▶  System prompt (per-CLI)         ├ L1+L2 cache, 5m TTL
        ▶  Skills metadata (descriptions)  ─┘
  WARM  ▶  Methodology corpus index       ─┐
        ▶  Manifesto + studio-charter      ├ L3 cache, 1h TTL
        ▶  Agent-overview roster            ─┘
  COLD  ▶  Specific capability specs       ─┐
        ▶  Research files (by reference)   ├ Loaded on demand
        ▶  Archive folders                  │ via file-read tools
        ▶  Triggered Skill bodies          ─┘
```

**One-sentence summary**: *Every agent declares its allowlist in its capability spec (Layer 3), the spec inherits its shape from one methodology document (Layer 2), and the runtime enforces it via OS primitives (Layer 4) — context loaded at session start follows progressive disclosure (hot/warm/cold) matched to Anthropic's `tools → system → messages` cache hierarchy.*

---

## Open questions

1. **Where does the access-scope methodology doc live?**
   `openspec/_methodology/tools/agent-access-scope.md`? Or `openspec/_methodology/conventions/access-scope.md`? Per CLAUDE.md's reference to `openspec/_methodology/tools/cli-standards.md § Auth conventions`, the `tools/` directory already exists. Recommend `tools/` for consistency. **Pavel decides.**

2. **Should research-corpus access be tagged or all-or-nothing?**
   Today: Maestro reads `openspec/_methodology/**` wholesale. Tomorrow: 50+ research files including some that contain sensitive client info or internal-only philosophy. Two options:
   - **(a) Tag-based.** Frontmatter `audience: [forge, muse, maestro]` per research file. Loader filters at session start.
   - **(b) Path-based.** `_methodology/research/public/`, `_methodology/research/internal/`, `_methodology/research/per-cli/<cli>/`. Path is the contract.
   - Industry pattern in 2026 leans path-based (Cursor globs, Claude Code path-scoped rules) because the dimension of override is visible on disk. **Recommend path-based.**

3. **Is `events.jsonl` shared or per-CLI?**
   Today: Forge writes `~/.forge/events.jsonl`. Maestro plans `~/.maestro/journal/`. If we want cross-CLI replay (Maestro auditing Forge's build outcomes), the events need to live in one place. Two options:
   - **(a) Per-CLI files, Maestro reads all three.** Simpler, no concurrent-write conflicts.
   - **(b) One shared `~/.rapoport/events.jsonl`, append-only.** Simpler for Maestro. Risk of write contention if multiple CLIs run simultaneously.
   - The DB `orchestra_journal` table (provisioned in Layer 2 per Maestro spec) sidesteps both options for the canonical journal — DB serializes the writes. JSONL on disk remains a debug mirror. **Lean canonical-DB, per-CLI mirrors.**

4. **Should Infisical tokens be process-scoped or session-scoped?**
   Today: `infisical run --env=prod --path=/` injects everything at root for the whole process lifetime. If Forge spawns a Builder subprocess for a 30-minute build, the GitHub token sits in that subprocess's env for 30 minutes. Just-in-time injection via Agent Vault ([Infisical Agent Vault blog](https://infisical.com/blog/agent-vault-the-open-source-credential-proxy-and-vault-for-agents)) eliminates this — credentials are injected at the network layer per-request and never enter the agent's env. **Worth piloting for the GitHub token specifically**, which is the highest-blast-radius credential in the studio.

5. **Does the existing Maestro `Access scope` shape need to be applied retrospectively to `forge.md` and `muse.md`?**
   Today, Forge's spec describes capabilities throughout the body (Anthropic API integration, GitHub API integration, etc.) but doesn't have a single `## Access scope` section with the explicit allowlist format. Muse same. This is a **convention-drift** problem — the new convention lands in Maestro because it was written under the new methodology, while older specs use the older convention. Two options:
   - **(a) Backfill.** Add `## Access scope` sections to `forge.md` and `muse.md` retroactively. High-touch on two large actively-maintained files.
   - **(b) Roll-forward.** Land the methodology doc (`agent-access-scope.md`), let new amendments comply, accept that older sections don't until next major edit.
   - The CLAUDE.md note "OpenSpec archive folders are frozen" doesn't apply (these are `current/`, not `archive/`). The agents-overview spec's Phase A handling of muse.md/forge.md ("remain at parent level... layout asymmetry is intentional") supports option (b). **Lean roll-forward.**

6. **The MCP authorization spec (2026-03-15) — does Rapoport need it now?**
   Today: no MCP servers shipped from Rapoport. If a future CLI ships an HTTP-transport MCP server (e.g., Maestro exposing routing decisions to external orchestrators), RFC 8707 audience binding becomes mandatory. **Not blocking; flag for whoever ships the first MCP server.**

7. **A2A protocol — Rapoport is single-org, so it's overkill. But if Maestro ever needs to coordinate with a client's agent stack, A2A is the protocol.** Defer. ([Google A2A v1.0 docs](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/))

---

## Sources

### Access control patterns
- [RBAC Is Not Enough for AI Agents — TianPan, April 2026](https://tianpan.co/blog/2026-04-20-rbac-ai-agents-authorization)
- [Agentic AI Security — BeyondTrust](https://www.beyondtrust.com/blog/entry/agentic-ai-security)
- [AI Agents Authentication — GitGuardian](https://blog.gitguardian.com/ai-agents-authentication-how-autonomous-systems-prove-identity/)
- [Macaroons: Cookies with Contextual Caveats — Birgisson et al., Google Research, 2014 (PDF)](https://research.google.com/pubs/archive/41892.pdf)
- [libmacaroons reference implementation](https://github.com/rescrv/libmacaroons)
- [Macaroons Escalated Quickly — fly.io blog](https://fly.io/blog/macaroons-escalated-quickly/)
- [What Google DeepMind Gets Right About Agent Delegation — DEV Community, 2026](https://dev.to/mattdeangit/what-google-deepmind-gets-right-about-agent-delegation-and-what-satgate-already-built-3e0a)
- [The Confused Deputy Problem Just Hit AI Agents — DEV Community, 2026](https://dev.to/claude-go/the-confused-deputy-problem-just-hit-ai-agents-and-nobodys-scanning-for-it-384f)
- [Confused Deputy Attacks on Autonomous AI Agents — Cloud Security Alliance Labs](https://labs.cloudsecurityalliance.org/research/csa-research-note-ai-agent-confused-deputy-prompt-injection/)
- [Agent Authentication & Delegated Access — Zylos Research, April 2026](https://zylos.ai/research/2026-04-11-agent-authentication-delegated-access-oauth-scoped-tokens)

### MCP authorization
- [Model Context Protocol — Authorization specification (2026 draft)](https://modelcontextprotocol.io/specification/draft/basic/authorization)
- [The New MCP Authorization Specification — dasroot.net, April 2026](https://dasroot.net/posts/2026/04/mcp-authorization-specification-oauth-2-1-resource-indicators/)
- [Is that allowed? AuthZ in Model Context Protocol — Stack Overflow Blog, January 2026](https://stackoverflow.blog/2026/01/21/is-that-allowed-authentication-and-authorization-in-model-context-protocol/)
- [MCP OAuth 2.1, PKCE — Aembit, 2026](https://aembit.io/blog/mcp-oauth-2-1-pkce-and-the-future-of-ai-authorization/)

### Skills / context architecture
- [Agent Skills overview — Anthropic Claude API docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Skill authoring best practices — Anthropic Claude API docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Equipping agents for the real world with Agent Skills — Anthropic Engineering blog](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Claude Code's Hidden Skill Budget Setting — claudefa.st, May 2026](https://claudefa.st/blog/guide/mechanics/skill-listing-budget)
- [How Claude remembers your project — Claude Code Docs (memory)](https://code.claude.com/docs/en/memory)
- [The Complete Guide to CLAUDE.md — Medium, May 2026](https://medium.com/@bijit211987/the-complete-guide-to-claude-md-memory-rules-loading-and-cross-tool-compression-97cc12ed037b)
- [Claude Code's Secret Weapon — sidsaladi substack](https://sidsaladi.substack.com/p/claude-codes-secret-weapon-the-complete)
- [AGENTS.md — agents.md homepage](https://agents.md/)
- [agentsmd/agents.md — GitHub](https://github.com/agentsmd/agents.md)
- [How to write a great agents.md — GitHub Blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
- [Cursor Rules documentation](https://cursor.com/docs/rules)
- [How to Structure Cursor Rules in 2026 — Medium, April 2026](https://medium.com/@vibecodingdirectory/how-to-structure-cursor-rules-in-2026-the-5-level-system-cursor-rules-eaf0df16e8e7)
- [llms.txt Explained (May 2026) — codersera](https://codersera.com/blog/llms-txt-complete-guide-2026/)
- [Is llms.txt Dead? — llms-txt.io](https://llms-txt.io/blog/is-llms-txt-dead)
- [LLMs.txt audit — SERanking, 2026](https://seranking.com/blog/llms-txt/)

### Prompt caching
- [Prompt caching — Claude API docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Prompt caching for faster model inference — Amazon Bedrock docs](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html)
- [Cache TTL silently regressed from 1h to 5m around early March 2026 — claude-code issue #46829](https://github.com/anthropics/claude-code/issues/46829)
- [Anthropic Silently Dropped Prompt Cache TTL — DEV Community](https://dev.to/whoffagents/anthropic-silently-dropped-prompt-cache-ttl-from-1-hour-to-5-minutes-16ao)
- [How prompt caching actually works — mager.co, April 2026](https://www.mager.co/blog/2026-04-29-claude-prompt-caching/)
- [Spring AI Anthropic prompt caching](https://spring.io/blog/2025/10/27/spring-ai-anthropic-prompt-caching-blog/)

### Sandboxing
- [Implementing a secure sandbox for local agents — Cursor blog](https://cursor.com/blog/agent-sandboxing)
- [anthropic-experimental/sandbox-runtime — GitHub](https://github.com/anthropic-experimental/sandbox-runtime)
- [List of coding agent sandboxes 2026-05 — wincent gist](https://gist.github.com/wincent/2752d8d97727577050c043e4ff9e386e)
- [A field guide to sandboxes for AI — Luis Cardoso](https://www.luiscardoso.dev/blog/sandboxes-for-ai)
- [Codex CLI sandbox — OpenAI Developers](https://developers.openai.com/codex/concepts/sandboxing)
- [Sandboxing in Gemini CLI — geminicli.com](https://geminicli.com/docs/cli/sandbox/)
- [nsjail sandbox for AI — Morph LLM](https://www.morphllm.com/nsjail-sandbox)
- [Processes Are All You Need for AI Sandboxing — Multikernel](https://multikernel.io/2026/03/14/introducing-sandlock/)

### Secret hierarchies
- [infisical secrets CLI commands](https://infisical.com/docs/cli/commands/secrets)
- [Secrets Management Best Practices — Infisical 2026](https://infisical.com/blog/secrets-management-best-practices)
- [Agent Vault — Infisical](https://infisical.com/blog/agent-vault-the-open-source-credential-proxy-and-vault-for-agents)
- [Infisical/agent-vault — GitHub](https://github.com/Infisical/agent-vault)
- [1Password Unified Access press release, March 2026](https://1password.com/press/2026/mar/1password-unified-access)
- [Secure Agentic AI: Authentication & Access Control — 1Password](https://1password.com/solutions/agentic-ai)

### Inter-CLI / multi-agent
- [Announcing the Agent2Agent Protocol — Google Developers Blog](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [Agent Interoperability Protocols 2026 — Zylos Research](https://zylos.ai/research/2026-03-26-agent-interoperability-protocols-mcp-a2a-acp-convergence)
- [Google's A2A and Anthropic's MCP — Gravitee](https://www.gravitee.io/blog/googles-agent-to-agent-a2a-and-anthropics-model-context-protocol-mcp)
- [agentic-stack — toolhunter.cc](https://toolhunter.cc/tools/agentic-stack)
- [The Beads Memory System — devengoratela.com, February 2026](https://devengoratela.com/2026/02/the-beads-memory-system-technical-architecture-and-integration-with-gemini-cli-for-agentic/)
- [Building Multi-Agent Systems with Shared Memory — Hindsight](https://hindsight.vectorize.io/guides/2026/04/21/guide-building-multi-agent-systems-with-shared-memory)

### Incidents (2025–2026)
- [Caught in the Hook: RCE and API Token Exfiltration Through Claude Code Project Files — Check Point Research, CVE-2025-59536 + CVE-2026-21852](https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/)
- [Claude Code Flaws Allow RCE and API Key Exfiltration — The Hacker News, February 2026](https://thehackernews.com/2026/02/claude-code-flaws-allow-remote-code.html)
- [Claudy Day trio of flaws — Dark Reading, March 2026](https://www.darkreading.com/vulnerabilities-threats/claudy-day-flaws-claude-users-data-theft)
- [Claude.ai Prompt Injection Vulnerability — Oasis Security](https://www.oasis.security/blog/claude-ai-prompt-injection-data-exfiltration-vulnerability)
- [Comment and Control: Prompt Injection to Credential Theft — Aonan Guan](https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/)
- [Cursor IDE's MCP Vulnerability — Check Point Research](https://research.checkpoint.com/2025/cursor-vulnerability-mcpoison/)
- [CVE-2025-59944 case-sensitivity bug in Cursor — Lakera](https://www.lakera.ai/blog/cursor-vulnerability-cve-2025-59944)
- [MCP Tool Poisoning (CVE-2025-54136) — Truefoundry](https://www.truefoundry.com/blog/blog-mcp-tool-poisoning-gateway-defense)
- [Malicious MCP Server on npm postmark-mcp — Snyk](https://snyk.io/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/)
- [MCP Supply Chain Advisory — OX Security](https://www.ox.security/blog/mcp-supply-chain-advisory-rce-vulnerabilities-across-the-ai-ecosystem/)
- [Anthropic MCP Design Vulnerability Enables RCE — The Hacker News, April 2026](https://thehackernews.com/2026/04/anthropic-mcp-design-vulnerability.html)

### OpenSpec
- [OpenSpec — openspec.dev](https://openspec.dev/)
- [Fission-AI/OpenSpec — GitHub](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec concepts](https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md)

### Rapoport repo references (internal)
- [`openspec/current/platform.md`](../../current/platform.md)
- [`openspec/current/agents-overview.md`](../../current/agents-overview.md)
- [`openspec/current/agents/maestro.md`](../../current/agents/maestro.md) — prior art for the Access scope allowlist pattern
- [`openspec/current/forge.md`](../../current/forge.md)
- [`openspec/current/muse.md`](../../current/muse.md)
- [`CLAUDE.md`](../../../CLAUDE.md) — hard rules, package boundaries, environment gotchas
