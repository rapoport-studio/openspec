# Multi-audience documentation patterns (2026)

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — web research across Diataxis, Stripe/Twilio/GitHub docs analyses, agency client-portal practices, AI-readable docs (Skills/AGENTS.md/llms.txt), NN/g progressive-disclosure literature, ~23 search queries.
> **Method:** Single-pass web synthesis. Sources: Diataxis canonical site, Stripe/Twilio/Mintlify docs analyses, IBM Carbon / Shopify Polaris source repos, Anthropic Skills documentation, GitHub blog on AGENTS.md (2500-repo study), NN/g UX articles, arxiv survey on LLM hallucination triggers, GitBook permissions docs. No first-party empirical measurement.
> **Reusable for:** `bridge-vocabulary-and-restructure` design; `introduce-indexing-and-llms-standards` design (Epic 4); `expand-onboarding-cheat-sheets` (Epic 6). Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 1 (audience tag in frontmatter, narrative.md split from spec.md), Epic 4 (Skills-pattern INDEX.md, llms.txt), Epic 6 (per-role onboarding one-pagers).

---

## TL;DR

Documentation for software products in 2026 splits along **reader intent** (Diataxis: tutorial / how-to / reference / explanation), not topic. The strongest pattern for multi-audience repos: one source of truth per capability + audience tag in YAML frontmatter + build-time projections (founder one-pager, client extract, AI bundle). The studio-relevant gaps are (a) progressive disclosure for AI readers (Skills pattern), (b) namespace separation for client-facing content, (c) age/literacy adaptation through layered TL;DR.

---

## Diataxis — the dominant framework

[diataxis.fr/start-here](https://diataxis.fr/start-here/) splits docs by **reader's mode** rather than topic:

| Type | Reader mode | Studio file analogue |
|---|---|---|
| Tutorial | learning by doing | `methodology/tutorials/<name>.md` (not yet) |
| How-to guide | solving a specific task | `methodology/templates/*.md` |
| Reference | looking up facts | `openspec/current/<cap>/spec.md` |
| Explanation | building understanding | `openspec/current/<cap>/narrative.md` (planned) |

Key claim: mixing types in one document is the #1 cause of confusing documentation. Adopted by Stripe, Twilio, GitBook, Mintlify; recommended by Anthropic Skills authoring guide.

## Industry gold standard — Stripe and Twilio

[apidog Stripe analysis](https://apidog.com/blog/stripe-docs/) + [Mintlify recommendations](https://www.mintlify.com/library/our-recommendations-for-creating-api-documentation-with-examples):
- Three-column layout (nav left, prose center, executable code right).
- Auto-inject test-keys into code samples (Stripe's signature).
- Group by **developer goal**, not by endpoint.
- Differentiate audience: developers get code, founders get case studies — different content under the same brand.

## Designer-readable specs — Carbon, Polaris, Primer

Common pattern from [Shopify Polaris repo](https://github.com/shopify/polaris) and IBM Carbon:
- **Visual-first** (anatomy before description).
- **Anatomy diagrams** labeling component parts.
- **Do / Don't pairs** with images.
- **Token names**, not CSS properties (designers write `color-text-primary`, devs map to RGB).
- **Use-case context** — "when to use Modal vs Drawer", not prop signatures.

## Agency client-facing docs

[Clinked best practices](https://www.clinked.com/blog/best-client-portals-for-marketing-agencies) + [Softr](https://www.softr.io/blog/what-is-a-client-portal):

> "Keep client-facing spaces separate from internal workspaces — the moment a client sees your internal notes or draft versions, trust erodes."
> "Clients don't want documentation. They want to find their files."

What clients see: deliverables, timeline, approvals, brand assets.
What clients don't see: drafts, internal critique, ADRs, technical implementation, agent prompts.

Dominant pattern: separate namespaces (`client/` vs `internal/`), not file-level permissions.

## Specialist onboarding (2-week freelancer pattern)

Consensus across ~10 onboarding guides ([Worksuite contractor onboarding](https://worksuite.com/resources/insights/contractor-onboarding-checklist), [Useme freelancer guide](https://useme.com/en/blog/freelancer-onboarding-checklist-download-pdf/)): **target ≤30 minutes to first productive action**.

Structure of a brief:
1. Project one-pager (what, why, who owns).
2. Scope (my piece, boundaries).
3. Access (repo, Figma, Slack channel, who answers).
4. Definition of done (checkpoint).
5. Glossary (5–10 studio terms).

Antipattern: handing a new freelancer the full spec library on day one. **Just-in-time docs** — link depth, don't embed it.

## AI-readable specs

Three canonical conventions, all converging on **frontmatter as routing signal**:

- **Anthropic Skills** ([code.claude.com/docs/skills](https://code.claude.com/docs/en/skills)): YAML frontmatter (`name`, `description` — ≤1024 chars), SKILL.md body ≤500 lines, overflow to `references/`. Name in gerund form, lowercase-hyphens. Description is the only thing Claude reads on session start.
- **AGENTS.md** ([agents.md](https://agents.md/), [GitHub blog on 2500 repos](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)): root-level instructions, ≤150 lines. Hierarchical: nested AGENTS.md extends the root. Six core sections — commands, testing, project structure, code style, git workflow, boundaries.
- **llms.txt** ([gitbook.com on llms.txt](https://www.gitbook.com/blog/what-is-llms-txt)): root-level index with one-line per doc + optional `llms-full.txt` (≤200K tokens). Adopted by Anthropic, Cursor, Vercel, Cloudflare; 844K sites by Oct 2025 (BuiltWith).

Critical for AI readers:
- **Frontmatter description specific enough** to route relevance without reading body.
- **Shallow > deep** — several 500-line files beat one 5000-line monolith.
- **Cross-links by stable paths**, not by name.
- **Single glossary** — otherwise the model hallucinates definitions.

[ASDLC research](https://asdlc.io/practices/agents-md-spec/) notes: LLM-generated context files reduced task success by ~3% and raised cost by 20% vs human-curated. **Curation matters.**

## Risk patterns by audience

From [Frontiers in Psychology — information overload review](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2023.1122200/full), [arxiv LLM hallucination survey 2509.18970](https://arxiv.org/html/2509.18970v1), [Decision Lab on choice overload](https://thedecisionlab.com/biases/choice-overload-bias):

| Risk | Hits first | Where it bites |
|---|---|---|
| Information overload | Specialist, client | Working memory 7±2 unit; novices scan chaotically vs experts |
| Staleness | Engineer, AI | Engineers reverse-engineer from code; AI hallucinates with confident tone |
| Leak (internal→client) | Client trust + reputation | One glance at drafts erodes trust |
| Ambiguity / undefined terms | AI agent, novice | Top hallucination trigger per arxiv 2509.18970 |
| Decision paralysis | Founder, designer | Iyengar jam study: 24 options → 3% buy vs 6 options → 30% |

Key observation: **AI and the novice suffer from the same defects** (ambiguity, overload). Founder and client suffer from different ones (leak, paralysis).

## Age and literacy

[NN/g — Usability for Older Adults](https://www.nngroup.com/articles/usability-for-senior-citizens/), [NN/g — Lower-literacy users](https://www.nngroup.com/articles/writing-for-lower-literacy-users/):
- No age gap in *understanding* in tech-literate cohorts; there is a **patience gap**: younger readers scan and jump to code; older readers will read linearly but require structure.
- Both audiences want **progressive disclosure** — but they enter from different sides. TL;DR for the young, linear narrative for the older.
- Short sentences, active voice, define jargon inline. Same advice for non-tech clients (35–55) as for AI agents — both penalize rhetorical filler.

## Permissions and access (future)

[Confluence role-based access (beta)](https://community.atlassian.com/forums/Confluence-articles/Beta-Simplify-space-access-in-Confluence-with-roles/ba-p/3044550), [Notion 3.0 granular DB permissions](https://matthiasfrank.de/en/granular-database-permissions-in-notion/), [GitBook permissions cascade](https://gitbook.com/docs/account-management/member-management/permissions-and-inheritance):
- Notion 3.0 uses property-based rules (person-property match → permission level).
- Markdown-in-Git uses repo-level permissions; per-file is workaround through audience-tag frontmatter + build-time filter.
- Studio recommendation: structural separation (multi-repo or namespaces) is safer than ACL config; less chance of accidental leak.

## Ten consolidated principles for multi-audience spec docs

1. One source of truth per capability; views are projections, not duplicates.
2. Intent defines type (Diataxis). Never mix tutorial and reference.
3. Progressive disclosure ≤ 3 levels (TL;DR → narrative → deep dive).
4. Frontmatter is the routing primitive for both AI and CI. Description ≤ 1KB. Audience-list mandatory.
5. Files shallow, not deep. ≤500 lines for capability spec; ≤150 lines for AGENTS-style root context.
6. Internal vs client = different namespaces, not different permissions.
7. Glossary is one. Inline definitions for rare terms.
8. Status lifecycle mandatory (proposed / stable / deprecated / superseded).
9. Audience tag in frontmatter enables filtered views without content duplication.
10. Bridges, not walls. Each doc type cross-links to its neighbours but does not duplicate.

## Implications for Rapoport Studio

- **Mandatory frontmatter** on every capability spec: `title / status / owner / audience / tldr / related` (Epic 4).
- **Split `<cap>.md` into `<cap>/{narrative,spec,flows,entities}.md`** for the 4 monoliths (Epic 2). Each file ≤500 lines.
- **`_root/INDEX.md`** as Skills-pattern routing — ~100-token description per capability, full body loaded on demand (Epic 4).
- **`apps/web/public/llms.txt`** to be readable by external AI crawlers (Epic 4).
- **`methodology/onboarding/for-<role>.md`** as 30-minute one-pagers (Epic 6, lazy).
- **No file-level permissions**. Client-facing content namespaced under `brands/<b>/engagements/<eng>/` (Epic 1).

---

## Sources

- [Diataxis framework](https://diataxis.fr/start-here/)
- [apidog Stripe docs analysis](https://apidog.com/blog/stripe-docs/)
- [Mintlify API docs recommendations](https://www.mintlify.com/library/our-recommendations-for-creating-api-documentation-with-examples)
- [Twilio docs](https://www.twilio.com/docs)
- [IBM Carbon Design System](https://carbondesignsystem.com/)
- [Shopify Polaris repo](https://github.com/shopify/polaris)
- [Figma Dev Mode handoff guide](https://www.figma.com/best-practices/guide-to-developer-handoff/)
- [Pentagram on systemising brand](https://the-brandidentity.com/interview/presented-by-brandpad-how-to-systemise-a-brand-featuring-pentagram-how-how-and-studio-blackburn)
- [Worksuite contractor onboarding](https://worksuite.com/resources/insights/contractor-onboarding-checklist)
- [Useme freelancer onboarding](https://useme.com/en/blog/freelancer-onboarding-checklist-download-pdf/)
- [Anthropic Skills docs](https://code.claude.com/docs/en/skills)
- [AGENTS.md spec](https://agents.md/)
- [GitHub blog: AGENTS.md from 2500 repos](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
- [ASDLC research on AGENTS.md](https://asdlc.io/practices/agents-md-spec/)
- [GitBook on llms.txt](https://www.gitbook.com/blog/what-is-llms-txt)
- [NN/g Progressive Disclosure](https://www.nngroup.com/articles/progressive-disclosure/)
- [NN/g Usability for Older Adults](https://www.nngroup.com/articles/usability-for-senior-citizens/)
- [NN/g Writing for lower-literacy users](https://www.nngroup.com/articles/writing-for-lower-literacy-users/)
- [Frontiers in Psychology — Information overload review](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2023.1122200/full)
- [Decision Lab — Choice overload](https://thedecisionlab.com/biases/choice-overload-bias)
- [arxiv 2509.18970 — LLM hallucination survey](https://arxiv.org/html/2509.18970v1)
- [Clinked — Client portal best practices](https://www.clinked.com/blog/best-client-portals-for-marketing-agencies)
- [Softr — What is a client portal](https://www.softr.io/blog/what-is-a-client-portal)
- [Confluence role-based access beta](https://community.atlassian.com/forums/Confluence-articles/Beta-Simplify-space-access-in-Confluence-with-roles/ba-p/3044550)
- [Notion 3.0 granular DB permissions](https://matthiasfrank.de/en/granular-database-permissions-in-notion/)
- [GitBook permissions](https://gitbook.com/docs/account-management/member-management/permissions-and-inheritance)
