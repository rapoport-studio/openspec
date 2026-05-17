# Rapoport Studio — Engagement Charter

> **Read this first.** This document is the hand-off artifact that lands in your project's OpenSpec corpus (typically `openspec/_methodology/` or `openspec/_source/`, depending on your local convention) when **Rapoport Studio** is engaged as an external partner. It describes who the studio is, what modes the engagement can take, what the studio commits to in each mode, what it never commits to, and how the studio's OpenSpec artifacts integrate into your repository.
>
> **Public mirror.** The canonical, read-only copy of this charter lives at [github.com/rapoport-studio/openspec](https://github.com/rapoport-studio/openspec/blob/main/_methodology/rapoport-studio-engagement.md). Prospective clients can read it before signing. The version delivered into your project at kickoff is a copy of the same file at that point in time; subsequent edits to the public mirror don't retroactively rewrite your local copy.
>
> Companion to your project's `openspec/AGENTS.md`. Where they conflict, your project's `AGENTS.md` wins on procedural matters (branch naming, PR template, lint rules); this charter wins on the **shape and convention of the OpenSpec artifacts the studio produces**.
>
> Sister document: [`studio-charter.md`](./studio-charter.md) — the bootstrap charter for a *new sub-product under the studio's umbrella*. That charter applies to repositories the studio runs end-to-end. **This** charter applies to **your** repository, when the studio is engaged as an external partner that contributes OpenSpec artifacts (and, in the deeper modes, code) into a codebase your team owns.

---

## 0. How to use this document

If you are an **agent** working in this repository and you encounter an OpenSpec change folder authored by Rapoport Studio (signed in `proposal.md § Authors`, slug arrived via the handoff shape in § 9), read this end-to-end before acting. It tells you:

- What the studio is and how it works.
- What an "OpenSpec change from the studio" looks like in your repo and what guarantees it carries.
- Which conventions the studio follows that may differ from your team's defaults.
- What the studio does *not* commit to (so you don't expect it).

If you are a **human reviewer**, the same applies — plus § 9 (handoff protocol) and § 11 (off-ramp) describe how the engagement attaches and detaches without leaving runtime artifacts in your codebase.

This document is the truth as of its delivery date. If the studio's master copy changes after kickoff, **the version in your repo wins for the duration of the engagement**; updates are negotiated explicitly, never pushed.

---

## 1. What Rapoport Studio is

Rapoport Studio is a Moldovan SRL (in formation, MITP applicant) building digital platforms and AI-powered infrastructure. The studio operates as a **curator of teams**: each engagement gets a hand-picked group of specialists from the studio's network — OpenSpec authors, domain experts, designers, copywriters, security reviewers, clinical advisors — assembled around the shape of the problem. Pavel Rapoport is founder and engineering anchor; the network around him is part of the offering, not a vendor pool.

The studio's thesis: **AI is the orchestration substrate of a platform, not a feature bolted onto it** — *but the substrate is operated by humans, not the other way around*. In practice that means every engagement begins with a domain map (Mirror), is shaped through spec authoring inside the studio's canvas surface (Muse Architect mode **plus** a curated specialist team working in the same conversation), tracked via Linear, and delivered as OpenSpec change proposals — `proposal.md`, `design.md`, `tasks.md` — that the agreed-upon engagement mode then implements (§ 4).

You are reading this because you have engaged Rapoport Studio. The studio operates in one of three modes per engagement (see § 4); the specific mode and specialist roster for **your** engagement are recorded in *Appendix B* below, alongside the rest of the per-engagement decisions.

---

## 2. What this engagement gives you

Six concrete capabilities, in delivery order:

1. **Curated specialist network.** A hand-picked group of humans assembled from the studio's network for *your* engagement. Disciplines are open-ended and chosen from the brief — examples include: OpenSpec authors, business-model validators, product monetization specialists, domain experts (clinical psychology, legal, finance, regulatory), product copywriters, brand/UX designers, security reviewers, accessibility reviewers, frontend specialists, mobile specialists. Each specialist's involvement is bounded (per-change or per-discipline) and recorded in *Appendix B* `D7` — the specialist roster — together with their scope, training status, and bio link.

   The full operational spec of how specialists are sourced, NDA'd, trained, certified, matched to engagements, and compensated lives in [`specialist-network.md`](./specialist-network.md). Below is the contract-level summary that affects you as the client:

   - **All specialists work inside the studio's canvas surface** (`app.rapoport.studio/canvas/[id]`) alongside Muse — not in your repository, not in your Slack, not by direct email. They never receive access to your repo, your secrets, your DB, or your tracker. Your team is invited into the same canvas under the `client` role (chat + spec view, no artifact panel) when live participation is wanted; otherwise drafts ship via the standard handoff in § 9.
   - **NDA is mandatory and signed with the studio entity, not with you.** Every specialist signs an NDA with Rapoport Studio SRL **before** receiving any access to your engagement materials. If your engagement carries additional confidentiality requirements (e.g. clinical data, pre-acquisition due diligence, regulated industry), the studio executes an engagement-specific NDA addendum on top of the base NDA. You sign one NDA — with the studio. The studio in turn flows the obligations through to each specialist.
   - **Specialists are trained, not just sourced.** Every specialist completes the studio's training cycle on (a) prose-style OpenSpec authoring discipline and (b) business-model validation methodology before being routed to an engagement. Training status is tracked per specialist and surfaced in `D7`.
   - **The platform suggests specialists for an engagement; the human picks.** Once Mirror produces the brief's domain map (capability #2 below), the studio's matching layer suggests candidate specialists by discipline + availability + training certifications + past-engagement quality signals. Pavel and you review the shortlist and confirm the roster — the suggestion is a starting point, not an automated decision.
   - **Specialists may attach their own prompts and tools to Muse.** Each specialist maintains a personal prompt set and tool list inside their studio profile. When a specialist joins a canvas, Muse loads those augmentations alongside its base configuration — so the conversation gets the specialist's domain framing without the specialist having to re-explain it each session. Specialist-contributed augmentations are studio-internal IP unless they reach the studio's "promote to public network" threshold.

2. **Domain modeling (Mirror).** A structured discovery output covering the seven canonical steps — actors, resources, states, transactions, integrations, communications, metrics. Built collaboratively across the canvas with you, the relevant specialists, and Muse over 3–4 weeks of discovery. Lands as `openspec/brands/<engagement-slug>/identity.md` (or as an attached artifact, depending on your repo's preference).

3. **OpenSpec change authoring (Muse Architect + specialists).** Each meaningful change is drafted as a `proposal.md` / `design.md` / `tasks.md` triplet inside the canvas. The split of labor between Muse and humans is fluid: Muse drafts the structural prose and citations; specialists supply domain claims, contradicting positions, and tone calibration; Pavel arbitrates and ships. Each triplet is tied to one Linear issue on the studio side (see § 6).

4. **Persona-shaped output.** Drafts can be tuned for five engagement personas (architect / domain / tech / business / ops) so the spec lands at the level of abstraction the reviewer expects. Persona is a soft-routing hint to Muse and a framing cue for the specialist team — not a marketing label.

5. **Citation-grade research.** When a change needs external framing, the studio's research tools (web search, fetch URL, domain-research agent) ground claims in citations rendered as GFM footnotes inside `design.md`. Cost-capped per canvas; no surprises. Specialist findings (e.g. clinical literature, regulator filings) flow through the same citation discipline.

6. **Spec hygiene reports (Forge `audit` and `spec`).** On request, the studio runs `forge audit <module>` and `forge spec <module>` against your `openspec/current/` corpus and reports drift between specs and code. Read-only against your repo. Output is a markdown report, not a PR.

Everything else — implementation, deploys, infra, RLS, secrets, code review of your team's PRs (in deliverables-only mode) — is **out of scope for this engagement** unless added by separate addendum or by escalating the engagement mode (§ 4).

---

## 3. What this engagement does NOT give you (any mode)

Hard out-of-scope list. These rules hold in **every** mode, including Full mode. If you expect any of these, you have the wrong engagement.

- **No edits to `openspec/current/`.** The studio drafts changes in `openspec/changes/<slug>/`; your team performs the live amendment to `current/` at archive time (§ 5.2). The "applied YYYY-MM-DD from change <slug>" pattern is documented for your team to apply, not for the studio to apply.
- **No direct merges to `main`.** Even in Full mode the studio's PRs land on feature branches. Your team merges to `main`. Always.
- **No infrastructure ownership.** Cloud accounts, secrets, DNS, deploys, runbooks — yours. The studio may *recommend* approaches in `design.md`, but never owns them.
- **No secret access.** The studio never receives, holds, or transmits your secrets. Sample env vars in code or specs carry only names, never values.
- **No on-call.** The studio is asynchronous. Response SLA is on business days (Moldova, EET/EEST), not 24/7.
- **No legal or compliance opinion.** GDPR, jurisdiction, retention rules, attorney-client privilege — surfaced in specs as constraints; the legal opinion is your team's.
- **No tracker bridge to your tools.** The studio tracks engagement work in its own Linear (§ 6); mirroring into your Jira / Shortcut / Linear / GitHub Issues is your team's responsibility if you want it. The studio can document a webhook on request, but does not operate one for you.

Mode-specific exclusions (e.g. "no code commits") live in § 4 alongside each mode.

---

## 4. Engagement modes

The studio operates in one of three modes per engagement, picked at kickoff and recorded in *Appendix B* as the engagement's first decision (`D1`). The mode determines what the studio touches in your repo, what bot access it requests, and what kinds of artifacts it produces.

### 4.1 Comparison

| Mode | Studio authors specs | Studio writes code | Default bot access on your repo |
|---|---|---|---|
| **Deliverables-only** *(default)* | yes | no | none |
| **Hybrid** | yes | drafts only, never on `main` | `Pull requests: write` (fork-PR) |
| **Full** | yes | yes, on feature branches | `Contents: write` + `Pull requests: write` (engagement repo only) |

Common to all three modes: § 3 (no `current/` edits, no `main` merges, no infrastructure, no secrets, no on-call, no legal opinion, no tracker bridge), § 5 (OpenSpec convention), § 6 (studio-side Linear), § 10 (hard rules).

### 4.2 Deliverables-only mode *(default)*

The studio authors OpenSpec changes; your team implements them. The studio never commits code into your repository. The boundary is intentionally sharp: the studio's deliverable is **markdown that compiles in your team's heads**, not code. This keeps the engagement composable with any stack and keeps code ownership trivial.

```
Discovery (3–4 weeks, fixed scope)
    ├─ kickoff conversation (canvas opens on app.rapoport.studio)
    ├─ domain mapping → Mirror perception.md
    └─ deliverable: signed-off domain map + first batch of changes drafted

Iteration (ongoing, per change)
    ├─ studio drafts proposal.md / design.md / tasks.md in canvas
    ├─ studio packages the triplet under openspec/changes/<slug>/
    ├─ delivery: tarball / patch (your choice — see § 9)
    └─ your team: review → edit → place in repo → implement → archive

Handoff (per release / per quarter)
    ├─ studio runs forge audit on agreed modules (if requested)
    └─ deliverable: drift report (markdown)
```

You own implementation, review, merge, deploy. The studio owns spec quality and domain modeling.

### 4.3 Hybrid mode

Same authoring loop as deliverables-only, plus the studio may **draft code** for selected tasks via fork-PR onto a `studio/<slug>` branch in your repo. Your team reviews, edits, and merges (or rejects); the studio's bot never targets `main` directly, never opens or modifies CI workflows, and never installs new top-level dependencies without prior `design.md` approval.

```
Authoring loop (same as Deliverables-only)
    ↓
Draft code (selective, by task)
    ├─ studio bot opens PR from rapoport-studio-bot
    ├─ base branch: studio/<slug>  (never main)
    ├─ touches: only files explicitly listed in tasks.md
    └─ your team: review → edit → merge to main yourselves
```

Hybrid is appropriate when your team wants the studio's authoring discipline plus implementation acceleration on bounded tasks (e.g. a new capability's first scaffold, a research-driven prototype) without giving up code ownership.

### 4.4 Full mode

Same authoring loop, plus the studio writes code on `forge/<change-slug>/<task>` branches and opens PRs to `main` (subject to your normal review). The studio bot's access scope is `Contents: write` + `Pull requests: write` on **the engaged repository only** — never org-level, never `Administration`, never `Workflow`. Your team's review remains the final gate; final merges to `main` are always yours.

```
Authoring loop (same as Deliverables-only)
    ↓
Implementation loop (per task in tasks.md)
    ├─ studio creates branch forge/<change-slug>/<task-slug>
    ├─ studio commits implementing tasks.md, with hard scope discipline
    ├─ studio opens PR titled "<LINEAR-ID>: <task title>"
    ├─ your team: review → request edits → approve → merge
    └─ studio archives the change once all tasks are merged
```

Full mode is appropriate when the studio is the primary engineering bandwidth on a project (the relationship is closest to "studio runs the build, your team owns the product"). The studio's runtime, deploy, secret, infra, and compliance boundaries from § 3 still hold absolutely — Full mode adds code velocity, not authority.

### 4.5 Mode change during engagement

The mode is declared at kickoff. Changing mode mid-engagement is allowed but requires (1) an explicit decision row in *Appendix B* superseding the original `D1` and (2) a corresponding access change on your side (granting or revoking bot scopes). The studio does not unilaterally escalate mode.

---

## 5. OpenSpec convention the studio follows

Your team has confirmed alignment with the prose-style OpenSpec convention. The studio operates under exactly this convention. For completeness:

### 5.1 Three-file contract per change

Every change folder the studio delivers has exactly three files:

```
openspec/changes/<slug>/
├── proposal.md   # Why + What changes
├── design.md     # How (interfaces, data shapes, boundaries)
└── tasks.md      # Phased checklist for the implementer
```

There is **no `specs/` directory**, no per-capability delta files, no `Requirement` blocks. The studio's house style is uniform across every engagement and matches what your team already practices.

### 5.2 Archive pattern (your team applies)

When your team merges the implementation of a change, they amend the relevant `openspec/current/<capability>.md` with a dated section in this exact form:

```
## <Title> (applied YYYY-MM-DD from change <slug>)
```

Then they rename `openspec/changes/<slug>/` → `openspec/archive/YYYY-MM-DD-<slug>/`. **The studio does not perform this step** — it lives entirely on your side. The studio re-reads your `current/` at the start of every new change to stay aligned with what your team has accepted.

### 5.3 `openspec validate` is expected to fail

The strict-deltas validator built into the OpenSpec CLI exits 1 on every change the studio delivers, with the error *"Change must have at least one delta… ensure your change has a `specs/` directory."* This is by design — the studio's prose convention deliberately doesn't use that structure. **Skip the command.** Nothing in the studio's pipeline runs it; nothing in your CI should run it on studio-authored changes either.

### 5.4 Folder slug naming

Change slugs the studio ships are bare (no date prefix) until your team archives them. Format: `<verb>-<concise-noun>` — examples: `add-session-lifecycle`, `enrich-onboarding-flow`, `sync-billing-to-stripe`. Linear-issue keys are not embedded in the slug; the linkage lives in `proposal.md § Linked tracking`.

The studio's own openspec corpus uses the same convention and is published at [github.com/rapoport-studio/openspec](https://github.com/rapoport-studio/openspec) as a living example — the charters and templates the studio operates under are visible there as plain markdown, the same shape that lands in your repo at kickoff.

### 5.5 No template folder

There is intentionally no `TEMPLATE/` reference. The newest archived change is the canonical example. If your team needs a copy of the studio's house style as a starting point, the studio will share its most recent representative archived change on request.

### 5.6 Convention adapter — which OpenSpec dialect your engagement uses

When the studio runs Forge against your repo, it routes every read and write through a **convention adapter** — a small TS module that captures the shape of your `openspec/` layout. Two adapters ship with Forge today:

- **`studio`** — prose-style minimal, exactly as described in §§ 5.1–5.5. Three files per change (`proposal.md` / `design.md` / `tasks.md`), no YAML frontmatter, no cross-cutting registries, edits to `current/` applied alongside the change folder. This is the default and matches what your team practices unless agreed otherwise at kickoff.
- **`unbind`** — prose plus four extensions: YAML frontmatter on every capability spec, four cross-cutting registries at the corpus root (`decisions.md` / `dependencies.md` / `maturity.md` / `open-questions.md`), capability-prefixed decision IDs (e.g. `D-SE-3`), and **diff-at-archive** — `current/` edits described inside `tasks.md` rather than applied inside the change folder, then mechanically applied at archive time. Used by the Unbind engagement; documented as a reference for engagements that want a richer corpus shape.

If your engagement needs a third convention, the studio writes one new adapter (`packages/forge/src/conventions/<your-engagement>.ts`) at kickoff. The shape is documented in `openspec/_methodology/templates/convention-adapter-example.md` on the public mirror — you can read it before signing to understand exactly what plumbing the studio commits to maintaining.

The adapter is invisible to your team in everyday operation: branch naming, PR shape, change-folder layout, archive ritual all stay as described in §§ 5.1–5.5 from your side. The adapter is the studio's machinery for serving multiple engagements without per-project conditionals leaking into the build agent.

---

## 6. Tracking — studio-side Linear

All engagement work tracks in **the studio's own Linear workspace** under a dedicated team prefix (e.g. `<YOUR-PREFIX>-`, agreed at kickoff and recorded in `proposal.md § Linked tracking` of every change). Each OpenSpec change carries one Linear issue ID; that ID is the canonical reference both sides use.

### 6.1 Why studio Linear, not yours

- **Single source of truth for spec authoring.** The studio's authoring loop (canvas → spec draft → review → ship) is wired to Linear deeply: status changes, automation, the Forge CLI's `estimate` and `audit` commands all read from Linear. Reproducing this on your tracker would degrade the engagement.
- **Asymmetric work tracking.** Your team's implementation work happens on your tracker; the studio's authoring work happens on its tracker. Each side owns its own backlog.
- **Privacy.** Studio-Linear is studio-internal. You see issue IDs and titles in `proposal.md`; you do not need accounts in our workspace.

### 6.2 Mirroring into your tracker (optional)

If your team wants engagement work visible on your own board:

- A simple n8n bridge or webhook can push studio-Linear updates into your system. The studio can document a recipe on request.
- Your team operates the bridge; the studio does not.
- Status sync is one-way (studio → you). Round-trip sync is out of scope.

The studio does not need access to your tracker.

---

## 7. Identities & access

The studio operates programmatic actors under dedicated bot identities, distinct from any individual's personal credentials. The access footprint scales with engagement mode (§ 4).

| Touch point | Studio identity | Permissions on your side, per mode |
|---|---|---|
| GitHub — your repo (Deliverables-only) | `rapoport-studio-bot` | **None.** Tarball / patch handoff; the bot has no account on your repo. |
| GitHub — your repo (Hybrid) | `rapoport-studio-bot` | `Contents: read` + `Pull requests: write`, scoped to **the engagement repo only**. No `Administration`, no `Workflow`, no org-level scopes. |
| GitHub — your repo (Full) | `rapoport-studio-bot` | `Contents: write` + `Pull requests: write` + `Issues: write`, scoped to **the engagement repo only**. No `Administration`, no `Workflow`, no org-level scopes. **PRs target feature branches; the bot never merges to `main`.** |
| Linear | `hello@rapoport.studio` (studio-internal) | None on your Linear. The studio's Linear is studio-internal regardless of mode. |
| Supabase / your DB | — | None. The studio never receives DB credentials. |
| Secrets / Infisical | — | None. The studio never receives or holds your secrets. |
| CI / GitHub Actions | — | None. The studio cannot trigger, modify, or read your CI configuration in any mode. |

**Off-ramp from access** is one click: revoke the bot from your repo settings (or the entire org if it was added at org level). The studio holds no other foothold to revoke. See § 11.

---

## 8. Communication channels

Default: **email** (`hello@rapoport.studio`) for handoffs and decisions. This is the floor — it always works, it leaves a paper trail, it requires no setup on your side.

Optional addition: a per-engagement Telegram group named `Rapoport Studio · {Your Name}`, operating with `@rapoport_studio_bot`. The bot listens for explicit triggers only — `/note`, `/task @user`, `/done <ID>`, plus 📝 and ✅ reactions on messages — and never auto-logs free-text traffic. Messages without explicit triggers stay in-chat and never reach the studio's persistent stores.

| Trigger | Action (on the studio's Linear) |
|---|---|
| `/note <text>` | Creates an issue in the engagement's Linear team, status = backlog, no assignee. |
| `/task @user <text>` | Creates an issue with assignee. |
| `/done <ID>` | Closes the referenced issue. |
| `/status` | Bot replies with engagement summary. |
| 📝 reaction on a message | Same as `/note`, using the message text. |
| ✅ reaction on a message linked to an issue | Marks issue as done. |

If your team prefers Slack, Signal, WhatsApp, or no chat layer at all, opt out at kickoff and the engagement runs over email only. The Telegram offering is **not required** to receive deliverables; it exists because per-engagement groups have proven faster than email threads for live shaping. Either choice is fine.

---

## 9. Handoff protocol

### 9.1 Spec handoff (all modes)

Three accepted shapes for each delivered OpenSpec change. Your team picks one at kickoff; the studio sticks to it for the duration of the engagement.

| Shape | What you receive | When to use |
|---|---|---|
| **Tarball drop** *(default)* | `<slug>.tar.gz` over email or shared drive, containing `openspec/changes/<slug>/{proposal,design,tasks}.md` | Lowest friction. Zero repo access required. |
| **Git patch** | `git format-patch` output, applied by your team to a branch named `studio/openspec-<slug>` | Single commit on a named branch, without granting repo access to the studio. |
| **Fork PR** | A pull request from `rapoport-studio-bot` against `studio/openspec-<slug>` (never `main`) | GitHub review tooling on the spec itself. Requires Hybrid or Full mode access per § 7. |

Whichever shape you pick, the studio's contract is:

1. The change touches **only `openspec/changes/<slug>/`**. No other files. No drive-by edits, no "while I'm here" tweaks elsewhere.
2. The triplet is internally consistent: `proposal.md` declares scope, `design.md` matches that scope, `tasks.md` is an actionable phased checklist for the implementing party (you in deliverables-only / hybrid; the studio itself in full mode).
3. Each change references its Linear issue in `proposal.md § Linked tracking` and lists explicit dependencies on prior changes (if any).
4. Markdown is GFM, no proprietary fences, no embedded binaries beyond inline-base64-free image links.
5. A single change ships with a single slug. Multi-change shipments arrive as N tarballs / N patches / N PRs — never as one folder containing many changes.

Your team's review of a delivered change has three valid outcomes:

- **Accept** — place in repo, run the tasks, archive at the end.
- **Request revision** — the studio re-ships the same slug with a `-revN` suffix on the Linear issue (slug stays stable across revisions).
- **Reject** — the studio closes the canvas internally and adjusts. The engagement Linear issue moves to `cancelled`.

### 9.2 Code handoff (Hybrid and Full modes only)

When the studio writes code, code arrives via PRs from `rapoport-studio-bot`. The PR contract is:

| Field | Convention |
|---|---|
| **Source branch** | `forge/<change-slug>/<task-slug>` — one branch per task in `tasks.md`. Never reused across tasks. |
| **Base branch** | The change's spec branch `studio/openspec-<slug>` (Hybrid) or your `main` (Full). Never a third-party branch you didn't authorize. |
| **PR title** | `<LINEAR-ID>: <task title>` — taken verbatim from the Linear issue and the task list. |
| **PR body** | Three sections: `## Summary` (one paragraph), `## Linked spec` (path + Linear ID), `## Checkpoint verified` (which tasks.md items this PR closes). |
| **Files touched** | Only the files listed in the corresponding `tasks.md` item. Drive-by edits are a regression and the PR is rejected. |
| **Dependencies** | The studio does not add new top-level npm/pip/etc. dependencies without prior approval recorded in `design.md § Dependencies added`. |
| **Tests** | If `tasks.md` checkpoint says "X works", the PR carries verification (test, screenshot, log output). |
| **CI** | The studio does not modify CI configuration (`.github/workflows/`, `.gitlab-ci.yml`, etc.) in any mode. CI changes go through your team. |

The studio bot **never merges its own PRs**. Even in Full mode, your team reviews and merges. If your team wants the bot to auto-merge labeled PRs, that's a separate decision recorded in *Appendix B* and is implemented via a workflow your team controls — not via studio bot escalation.

---

## 10. Hard rules the studio follows

These are non-negotiable on the studio side, in every mode. They should match your team's expectations.

1. **No silent assumptions.** If your brief contradicts a claim in `openspec/current/` or in this charter, the studio surfaces the conflict in `proposal.md § Conflicts surfaced` rather than choosing.
2. **No edits outside agreed scope.** Spec PRs touch `openspec/changes/<slug>/` and nothing else. Code PRs touch only files listed in the corresponding `tasks.md` item. Typo fixes to `current/`, drive-by refactors, and "while I'm here" tweaks are never bundled into shipments.
3. **No hidden dependencies.** Every external citation in `design.md` carries a footnote with URL and access date. New runtime dependencies are declared and rationalized in `design.md § Dependencies added` before any code PR introduces them.
4. **No secrets in deliverables.** Sample env vars carry only names, never values.
5. **No `main` merges from the studio bot.** In every mode. Your team holds the merge button.
6. **No CI changes from the studio.** `.github/workflows/`, `.gitlab-ci.yml`, deploy configs, and similar files belong to your team, not to the studio.
7. **No proprietary lock-in.** All deliverables (specs and code) remain plain text under your team's ownership. Specs are GFM markdown; code matches the language and conventions already present in your repo. The studio does not ship binaries, generated artifacts requiring studio tooling to read, or tracker integrations that bind you to the studio's stack.

---

## 11. Off-ramp

Ending the engagement is a single conversation. Concretely:

- The studio archives the canvas on its side. **Your historical specs and any merged code remain in your repo — they are yours.**
- If a GitHub bot was granted access (Hybrid or Full mode), your team revokes it from the repo / org settings. The studio holds no other foothold.
- Open studio-authored PRs at off-ramp time stay open or are closed at your discretion. The studio does not re-open or reattempt them after off-ramp.
- If a Telegram group exists, the studio leaves the group at your request; the bot is removed.
- All Linear issues on the studio side carrying your engagement prefix are marked `cancelled` or `delivered` per their final status.
- No runtime artifact from the studio remains in your repo that wasn't already yours. The OpenSpec markdown and merged code are yours and stay.

There is no licensing tail on delivered specs or code: both are work-product owned by your team. The studio retains the right to reference *the existence* of the engagement in case studies (with your written approval per the SOW), but never the substance of any spec or any merged code.

### 11.1 Spec-merge gating

An engagement does not enter the `closed` state until the OpenSpec triplet
(`proposal.md`, `design.md`, `tasks.md`) authored during the engagement is
merged into the `main` branch of the client's primary repository.

If the client elects not to merge a portion of the triplet (e.g., declines
to ship a specific phase), the change archive records the declined portion
explicitly. Closure does not block on declined portions; closure does block
on undecided portions.

This gating is the structural mechanism behind Pillar C
(Portability) of the studio's homepage trust pillars — see
`archive/2026-05-06-expand-home-with-services-and-community/research/muse-trust-and-ownership.md § 2.3`.

> **Applicability.** This clause applies to engagements commencing on or after
> 2026-05-12. In-flight engagements continue under the prior § 11 prose unless
> the client explicitly opts in (record opt-in as a new `D11` row in *Appendix B*).

---

## 12. Glossary

| Term | Meaning |
|---|---|
| **Rapoport Studio** | The Moldovan SRL providing this engagement. |
| **Engagement** | A bounded scope of work between the studio and your team, governed by this charter plus a SOW. |
| **Canvas** | The studio's authoring surface at `app.rapoport.studio/canvas/[id]`. Conversations between Pavel (or a curated specialist) and Muse that produce spec drafts. |
| **Muse** | The studio's AI substrate. In v1 it operates in Architect mode — drafting OpenSpec proposal/design/tasks triplets via tool-augmented Claude. |
| **Mirror** | The studio's domain-modeling layer. Produces a `perception.md` per engagement that anchors all subsequent specs. |
| **Forge** | The studio's CLI. The studio runs Forge against your spec corpus and (in Hybrid/Full mode) against your code; you do not need to install or run Forge yourself. Its `audit` and `spec` reports may be delivered to you on request in any mode. |
| **Mode** | One of `deliverables-only` / `hybrid` / `full` — the engagement-specific shape recorded in *Appendix B* `D1`. See § 4. |
| **Change** | A folder in `openspec/changes/<slug>/` containing exactly `proposal.md`, `design.md`, `tasks.md`. The atomic unit of studio delivery. |
| **Capability** | A first-class concern with its own `openspec/current/<name>.md`. The studio's changes target one or more capabilities. |
| **Curation** | The studio's operating philosophy: hand-picked specialists per engagement, not a fixed staff. |
| **Network** | The studio's curated roster of humans available for engagements — OpenSpec authors, business-model validators, domain experts, designers, copywriters, reviewers. Operational spec lives in [`specialist-network.md`](./specialist-network.md); public-facing roster at `rapoport.studio/network`; per-engagement routing in *Appendix B* `D7`. |
| **Specialist** | One member of the network engaged for a specific scope inside an engagement. Specialists are NDA-bound to the studio, trained on OpenSpec authoring + business-model validation, and join the canvas as `collaborator` role. They do not receive access to your repo, secrets, or tracker. |
| **AI orchestration** | The studio's thesis that AI is the architecture layer of a platform, not a feature bolted on — operated by humans, not the other way around. Engagements are platforms with AI as orchestration substrate, not products with AI features. |

---

## 13. Responsibility levels — entity, person, role

Rapoport Studio engagements involve **at least four distinct legal/operational actors** that look superficially similar but carry different liability, IP rights, equity, and tax implications. Conflating them is the single most common source of investor concern, IP drift, and personal-liability surprise. This section is the canonical map.

The four actors are:

1. **Pavel-the-citizen** — natural person, dual citizen of Moldova and Israel, individual taxpayer, signatory on personal contracts.
2. **Pavel-as-administrator-of-Rapoport-Studio-SRL** — Pavel acting on behalf of the studio entity. Signs studio contracts, hires/manages specialists, executes the engagement contract.
3. **Pavel-as-cofounder/CTO-of-the-client** — Pavel personally engaged in your company per a separate founders agreement (when the engagement is structured as the brief's "Hybrid" model). Equity-bearing labor; reports to your CEO; accountable to your board/shareholders.
4. **Rapoport Studio SRL** — the legal entity. Moldovan SRL, Moldova IT Park resident, independent of Pavel-the-person for liability purposes. Employer of specialists. Holder of methodology IP.

### 13.1 Allocation by act

Every act in an engagement is attributable to **exactly one** of the four actors. The table below is the default allocation; *Appendix B* `D8` may override per engagement.

| Act | Default attribution | IP outcome | Equity-bearing? | Liability falls on |
|---|---|---|---|---|
| Code committed to your repo from `rapoport-studio-bot` (Hybrid/Full mode) | Studio SRL | Mutual license: deliverable to you, methodology to studio | No | Studio SRL |
| Code committed to your repo from Pavel's personal GitHub identity (cofounder mode) | Pavel-as-cofounder | Standard work-for-hire under your founders agreement | Yes | Pavel personally + your company |
| OpenSpec change (`proposal.md` / `design.md` / `tasks.md`) drafted in canvas | Studio SRL | Mutual license: deliverable to you, methodology to studio | No | Studio SRL |
| Forge-generated code in your repo (any mode) | Studio SRL (tooling) operating on Pavel-cofounder's behalf if applicable | Deliverable to you; Forge engine remains studio IP under license | Depends on the underlying actor | Studio SRL for tooling defects; the underlying actor for choice to use it |
| Specialist work delivered through canvas | Studio SRL | Deliverable to you; specialist IP remains studio-internal | No | Studio SRL (specialist has no privity with you) |
| Investor pitch tech-section, due diligence answers, code review on your behalf | Pavel-as-cofounder | N/A | Yes | Pavel personally + your company |
| Studio-Linear backlog, MITP-compliant accounting, specialist payroll | Studio SRL | N/A | No | Studio SRL |
| Personal tax return, dual-citizenship elections | Pavel-the-citizen | N/A | N/A | Pavel personally |

### 13.2 Boundary rules (non-negotiable)

These rules hold in every engagement mode and every commercial structure:

1. **No mixing of identities in a single commit.** Pavel-as-cofounder commits use Pavel's personal GitHub email and personal GitHub account. Studio commits use `rapoport-studio-bot`. A single commit never claims both.
2. **No undisclosed dual interest.** When Pavel-as-cofounder makes a decision that affects the studio's commercial interest in the engagement (e.g. extending studio's specialist scope, renewing the engagement, raising studio rates), the decision is disclosed to your CEO before execution. Same in reverse: when the studio entity makes a decision affecting your company's interests, Pavel-as-cofounder is excluded from the studio's side of that decision.
3. **No personal liability shield for studio acts.** Pavel-the-administrator's signatures bind Rapoport Studio SRL, not Pavel personally. Pavel-as-cofounder's signatures bind Pavel personally and your company per the founders agreement, not the studio.
4. **No equity for studio work.** If a piece of work is attributed to Studio SRL per § 13.1, no Pavel-personal equity claim arises from it. Pavel earns equity only for Pavel-as-cofounder labor that is documented in the founders agreement.
5. **No methodology surprise.** What constitutes "methodology" (and therefore stays with the studio under mutual license) versus "deliverable" (and therefore vests in your company) is enumerated in `D8` of *Appendix B* at kickoff — not improvised at delivery time. Default carve-out: Forge engine, OpenSpec prose conventions, specialist-network registry, Mirror domain wizard config, training materials, and reusable prompt scaffolds remain studio IP. Engagement-specific specs, your domain map, your code, your prompts authored against your domain remain yours.
6. **NDAs flow through the studio.** Specialists do not sign NDAs with you; they sign with Rapoport Studio SRL, and the studio's master NDA with you ties everything together. If your engagement requires per-specialist NDAs with you directly (rare — typically only for regulated due-diligence contexts), `D8` records that override.

### 13.3 Conflict-of-interest discipline

Pavel-as-cofounder of your company and Pavel-as-administrator of the studio that contracts with your company is a **structural dual interest**. It is not concealed; it is documented and managed:

- The founders agreement (drafted by i-avocat.md — Andrei & Dmitry — per the kickoff brief) names the dual interest explicitly.
- Studio rate changes, specialist hires onto your engagement, scope changes to studio deliverables, and engagement renewal are discussed openly between Pavel and your CEO. Decisions are recorded in *Appendix B* with both names.
- Studio invoices to your company are reviewed by your CEO before payment, even when Pavel-as-cofounder is responsible for tech delivery on the receiving side.
- An eventual exit (acquisition, IPO, wind-down) requires explicit treatment of the studio's residual rights — this is contemplated in the founders agreement, not improvised post-hoc.

### 13.4 Quick test for ambiguous acts

When an act feels ambiguous, ask three questions in order:

1. **Whose tool is doing the work?** Forge / Muse / specialist-network → studio side. Pavel's hands writing code in his editor against the founders agreement → Pavel-as-cofounder.
2. **Who is invoiced or compensated?** Studio invoices for the work → studio side. Your payroll or equity vesting accrues to Pavel → Pavel-as-cofounder.
3. **Where does liability land if the work goes wrong?** Your repo's broken code from a bot account → studio. Your repo's broken code from Pavel's account, while wearing the cofounder hat → Pavel personally + your company per founders agreement.

If two of three answers contradict, the act needs a quick clarification with i-avocat.md before proceeding. Disagreements after the fact cost vastly more than 30 minutes of legal review before.

---

## Appendix A — How to recognize a studio-authored change in your repo

Mark of authenticity in `proposal.md`:

```markdown
## Authors

- Rapoport Studio (engagement: <your-engagement-slug>)
- Linear: <STUDIO-PREFIX>-<NNN>
```

If both lines are present, the change is authored by the studio under this charter and the rules in §§ 4–10 apply. If either line is missing or contradicted, treat it as your team's own change and disregard this charter for that change.

---

## Appendix B — Decisions captured (per-engagement)

This section is filled in at kickoff and stays append-only for the duration of the engagement. Every deviation from the charter's defaults goes here. Decisions are immutable: a row is replaced by issuing a new row that supersedes it (the older row's status moves to `Superseded`), never by editing in place.

| # | Decision | Rationale | Date |
|---|---|---|---|
| D1 | Engagement mode: *(deliverables-only / hybrid / full)* | *(rationale)* | *(kickoff)* |
| D2 | Linear team prefix: `<TBD>-` | Single source of truth for spec authoring (and code, in Full mode) | *(kickoff)* |
| D3 | Spec handoff shape: *(tarball / patch / fork-PR — see § 9.1)* | *(rationale)* | *(kickoff)* |
| D4 | Code handoff *(Hybrid/Full only)*: branch shape `forge/<change>/<task>`, base branch `<studio/openspec-... | main>` | *(rationale per § 9.2)* | *(kickoff)* |
| D5 | Communication: email floor + *(Telegram opt-in / opt-out / Slack / etc.)* | *(rationale)* | *(kickoff)* |
| D6 | Research contribution: *(none / scoped subfolder / all subfolders)* — applies if your repo has an `openspec/_research/` discipline | *(rationale)* | *(kickoff)* |
| D7 | Specialist roster: *(list of network members + role + scope per § 2 capability #1)*. Names may be redacted in the public version of this charter copy if your engagement has confidentiality constraints. | *(rationale: which disciplines the brief calls for)* | *(kickoff)* |
| D8 | Responsibility split per § 13: Pavel acts as *(personal cofounder of your company / studio administrator only / hybrid of both)*; studio entity acts as *(strategic partner with infrastructure license / vendor under fee / both)*. Methodology vs deliverable carve-out: *(default per § 13.2 rule 5 / engagement-specific list attached as Appendix D)*. | *(rationale: matches founders agreement model A/B/C)* | *(kickoff)* |
| D9 | … | … | … |

### D11 — TG canvas surface opt-in (added 2026-05-09 by `add-tg-canvas-surface` / RAP-121)

**Question**: Does this engagement allow per-canvas Telegram groups via `@rs_canvas_bot`?

**Default**: No.

**If yes**:
- Set `projects.tg_surface_enabled = true` on the engagement's project.
- Record opt-in date and signing party in this Appendix entry.
- Acknowledge in writing: TG conversations may include incidental personal data;
  GDPR special-category data (health, religion, ethnicity, biometric, sexual
  orientation per Art. 9) is BANNED on this surface; runtime classification
  enforces the ban (matched messages NOT processed, NOT stored as transcript).
- Acknowledge in writing: Voice notes are transcribed via Whisper; audio retained
  in R2 for 30 days then deleted.
- Acknowledge in writing: All canvas mutations from TG (Mirror writes, GitHub
  PRs, Architect-emit) execute as proposals — owner approval required.

**If no**: `projects.tg_surface_enabled` stays `false`. Worker rejects any TG
chat_id whose canvas's project has the flag false. Bot remains accessible for
public-channel/discussion-group flows (separate scope).

**Reversibility**: Yes — flip flag to `false`; in-flight turns finish (D-TCS-13);
next turn rejected. Audio in R2 retained per existing 30d lifecycle.

**Source**: `add-tg-canvas-surface` (RAP-121), capability spec at
`openspec/current/telegram.md`. Decisions D-TCS-1..13 in
`openspec/archive/2026-05-09-add-tg-canvas-surface/proposal.md` (archived 2026-05-09).

---

## Appendix C — Closure checklist (client-facing)

This is the checklist the client receives by email at engagement off-ramp.
It is also embedded in the engagement-closure email template at
`_methodology/templates/closure-email-template.md`. The studio is
responsible for confirming items the studio controls; the client is
responsible for confirming the rest.

### C.1 Spec & artifacts (4)
- [ ] OpenSpec triplet merged into client `main` branch — link: <URL>
- [ ] Linear issues exported to client-owned destination — format: <CSV | JSON>
- [ ] Canvas conversation history snapshot offered — format: <Markdown export>
- [ ] Mirror perception document in client repo — path: <repo path>

### C.2 Access revocation (4)
- [ ] Studio-member access to client repo revoked — confirmed by: <client-side reviewer>
- [ ] Specialist canvas access revoked — confirmed by: <studio>
- [ ] Bot tokens / OAuth grants revoked — confirmed by: <client>
- [ ] Telegram / WhatsApp surface archived or transferred — disposition: <archived | transferred>

### C.3 Knowledge transfer (2)
- [ ] Final review meeting completed — date: <YYYY-MM-DD>
- [ ] Postmortem published or scheduled — link: <URL> | scheduled-for: <YYYY-MM-DD>

### C.4 Financial & consent (2)
- [ ] Final invoice settled
- [ ] Case study consent confirmed (or declined or deferred) — disposition: <consent | declined | deferred>

---

*This charter is versioned in the studio's repository at `openspec/_methodology/rapoport-studio-engagement.md`. The version delivered to your project is a copy at the time of kickoff.*

---

## § Discovery — deliverable contract (applied 2026-05-13 from change `lock-positioning-trajectory`)

> **Q-POS-3 → D-POS-DISCOVERY-1.** Ratified deliverable contract for the Discovery tier. This section extends § 4.2's discovery description with the explicit per-item scope, resolving the ambiguity between the route structure (established in `archive/2026-05-04-add-client-onboarding/`) and the Stage 1 deliverable contract.

**Fixed scope: €5k, 3–4 weeks.**

The standard Discovery delivers exactly three artifacts:

1. **OpenSpec capability corpus** — `openspec/current/` files covering the domain's first-class capabilities (platform.md at minimum; additional capability stubs where the domain warrants it). Authored in prose-style, not formal-delta style. Status: `Stub` or `Active` per the honest assessment of what discovery produced.

2. **Mirror domain map** — `openspec/brands/<engagement-slug>/identity.md` covering the seven canonical Mirror steps: actors, resources, states, transactions, integrations, communications, metrics. This is the durable domain artifact; it anchors all subsequent OpenSpec changes.

3. **First-pass render** — a rendered preview output (e.g., `forge render` or equivalent) demonstrating that the spec compiles cleanly into the studio's reference architecture. Not a production deploy; a coherence check.

**Not included in standard Discovery scope:**

- Sprint plan or Phase 1 implementation proposal
- First-sprint-week of execution
- Recommendation document or options analysis

A sprint plan, if wanted, is the first task of a subsequent full engagement (D-POS-ENGAGEMENT-1). Discovery is bounded; extensions are new work, not Discovery scope creep.

**Conflict resolution note:** `archive/2026-05-04-add-client-onboarding/` ratified the Discovery route's commercial structure (€5k, 3–4 weeks, "Validation Atelier" name). This section ratifies the Stage 1 deliverable contract within that structure. If the two descriptions diverge in future, this section is the normative one (it is later and more specific).

---

## § Engagement model — Stage 1 (applied 2026-05-13 from change `lock-positioning-trajectory`)

> **Q-POS-7 → D-POS-ENGAGEMENT-1; Q-POS-2 → pending D-POS-PRICE.** Ratified engagement model for Stage 1. Extends § 4 by giving the sprint-based billing model its authoritative home.

### Engagement model

**Sprint-based: €10–15k per 2-week sprint × N sprints.**

Each sprint is a fixed-scope, fixed-time unit. The scope is a subset of tasks from an agreed OpenSpec change (one `tasks.md`). Sprint N cannot start before Sprint N-1 delivers and is reviewed. Sprint count is proposed at discovery close and revised at the end of each sprint.

**Optional retainer:** ongoing advisory (not delivery) at €2–3k/month for clients who want async access between sprints. Retainer is not a sprint; it does not produce a deliverable. It covers async answers, spec reviews, and blocker calls. Retainer activates after a minimum of two delivery sprints.

**Founder Track equity variant:** deferred to Stage 2. Q-POS-1 resolved to MD/RO Client Track first; IL Founder Track is warm but not actively driven in 2026 without an inside-partner. When Stage 2 activates and IL Founder Track is live, a cash + small equity variant may be ratified via `amend-positioning-stage-2`. Until then, Founder Track clients in MD/RO engage under the standard sprint model above.

### Pricing floor (pending D-POS-PRICE — operator decision required)

> **⚠️ Operator-blocked.** The absolute pricing floor for a full engagement is gated by `add-funding-strategy` archive. The sprint unit price above (€10–15k / sprint) is the current operative figure. The floor (minimum engagement size = N sprints minimum) is ratified when `add-funding-strategy` produces D-FUND-* on bootstrapped vs investor-targeted posture.
>
> - Bootstrapped scenario → floor likely €30–50k (3–4 sprints minimum).
> - Investor-targeted scenario → floor likely €50–80k (4–6 sprints minimum).
>
> Close trigger: `add-funding-strategy` archives. Off-cycle review if gap between funding archive and next quarterly review exceeds 6 weeks.

Until D-POS-PRICE-N is ratified, Discovery (€5k fixed) is the only hard-priced tier. Full engagements are quoted per-sprint from the range above.

### Mode selection

Engagement mode (deliverables-only / hybrid / full — § 4) is independent of the sprint model. The sprint model is the **billing and pacing structure**; mode determines what the studio touches in the client's repo. Both dimensions are locked in Appendix B at kickoff.

---

## § Engagement tracks (applied 2026-05-17 from change `add-migration-track-methodology`)

> Names a third, orthogonal dimension by which an engagement is described.

An engagement is described along three independent dimensions:

- **Access mode** (§ 4) — `deliverables-only` / `hybrid` / `full`. What the studio touches in the client's repo.
- **Billing structure** (§ Engagement model — Stage 1) — sprint-based pacing. How the work is priced and paced.
- **Engagement track** — *this section* — the kind of work the engagement does, inside the frame the other two dimensions set.

A **track** is a defined, repeatable process for one kind of engagement work. It runs after Discovery, within the engagement frame established by charter and kickoff. A track does not change access mode or billing structure — those are locked separately in Appendix B.

**Migration Track** is the studio's defined process for engagements where a founder migrates an existing product. Other tracks (Feature, Audit) are not yet defined.

The Migration Track methodology — its phases, gates, and care principles — is documented at `openspec/_methodology/migration-track/migration-track.md`. This charter governs how the studio enters and exits the engagement around a track; the methodology governs what happens between the track's first and last phase. They are different scopes.
