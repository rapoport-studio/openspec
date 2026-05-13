---
title: Cofounder Engagement Contract
status: proposed
owner: pavel
audience: [pavel, ai, cofounder, future-operator, legal-counsel]
tldr: Companion to rapoport-studio-engagement.md. Lifts § 13's four-actor model into a full economic contract for the cofounder-engagement Contract subtype. Three-way party shape (Pavel-as-cofounder + other cofounder + joint Product), equity vesting language, dual-interest discipline, commit attribution rules, founders-agreement integration with i-avocat-drafted legal templates.
---

# Cofounder Engagement Contract

> **Companion** to [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md). The engagement charter governs how Studio SRL sells services into external repositories. This document governs what happens economically when Pavel is *also* a cofounder of the product the studio is building.
>
> **When to use this doc.** If your engagement is `cofounder-engagement` per [`archive/2026-05-13-establish-studio-economy/design.md § D-ECN-1`](../archive/2026-05-13-establish-studio-economy/design.md), read this end-to-end. If your engagement is `client-engagement`, this doc does not apply — read the engagement charter alone.
>
> **Status.** Proposed as part of `establish-studio-economy` (P0 capture). Ratification per cofounder engagement happens at that engagement's kickoff in `<slug>.yaml § engagement.cofounder` block.

---

## 1. What a cofounder engagement is

A `cofounder-engagement` Contract exists when **Pavel personally holds equity in the product the studio is building**. The product is jointly owned by Pavel-as-cofounder and one or more other cofounders; the studio is engaged as the technical vendor by that joint product.

Two cofounder engagements are active or pre-kickoff as of this document's authoring:

- **Unbind** — joint product with Olya. Olya's role: administrative. Pavel's role: development + spec authoring. Equity split: 50/50 (subject to confirmation conversation per PF-1 in [`archive/2026-05-13-establish-studio-economy/tasks.md`](../archive/2026-05-13-establish-studio-economy/tasks.md)).
- **Vivod** — joint product with Misha Ganevich. Misha's role: management, contractors, Israeli sales, in-market network. Pavel's role: development. Equity split: 50/50 (explicitly agreed).

This document is the canonical reference for how those engagements are economically structured. Per-engagement specifics (vesting schedule, cliff, exit clauses, IP carve-outs) ratify in `<slug>.yaml § engagement.cofounder` at the engagement's kickoff and in the founders agreement legal document.

---

## 2. The three-way party shape

Where a `client-engagement` Contract has two parties (buyer + seller), a `cofounder-engagement` Contract has **three parties**:

1. **Pavel-as-cofounder** (Person Actor #2 per [engagement charter § 13](./rapoport-studio-engagement.md)) — the equity-bearing labor contributor. NOT Pavel-as-administrator-of-Studio. NOT Studio SRL. NOT Pavel-citizen.
2. **The other cofounder** (Person Actor; Olya for Unbind, Misha for Vivod) — equity-bearing co-owner of the joint product; contributes labor in a non-engineering role.
3. **The joint Product Actor** (Studio Actor type; Unbind entity, Vivod entity) — the legal vehicle that owns the product IP. Pavel-as-cofounder + other cofounder are its co-owners.

Studio SRL is a **separate fourth Actor** that is engaged by the joint Product Actor as a vendor under a `client-engagement`-shaped sub-contract. This separation is non-negotiable per § 13.2 rule 1.

```
Joint Product (Vivod entity / Unbind entity)
├── owned by: Pavel-as-cofounder (50%) + other cofounder (50%)
├── engages: Studio SRL (vendor) ─→ separate client-engagement contract
│                                    Studio bills joint Product for dev time + AI markup
│                                    Studio's revenue here counts as Era-1 client revenue
└── delivers: product to end-market
   ├── Vivod: Israeli construction-management market
   └── Unbind: Russian-language breakup-recovery market
```

The **cofounder-engagement Contract itself** captures the equity / vesting / dual-interest dimension between the two human cofounders and the joint Product. The **Studio SRL → joint Product client-engagement Contract** captures the cash flow for engineering vendor services. The two contracts coexist; they are not substitutes.

---

## 3. Equity vesting language

Concrete vesting schedules ratify in the founders agreement (legal PDF) drafted by i-avocat (Andrei + Dmitry). This document fixes the **shape** of vesting terms the studio operates under as a default; deviations record in `<slug>.yaml § engagement.cofounder.vesting_overrides`.

### Default vesting (proposed pending kickoff per engagement)

- **Cliff:** 12 months. No equity vests if Pavel-as-cofounder departs the engagement within the first 12 months for cause attributable to Pavel.
- **Vest period:** 36 months following the cliff (total 48 months from engagement kickoff).
- **Acceleration on change of control:** double-trigger acceleration (sale + Pavel's role eliminated within 12 months) accelerates 100% of unvested equity. Single-trigger acceleration on sale alone is NOT default; reserve for case-by-case.
- **Repurchase rights:** joint Product Actor may repurchase unvested equity at original strike price if Pavel-as-cofounder departs before full vest. Vested equity is retained by Pavel-as-cofounder regardless.
- **Tag-along + drag-along rights:** standard reciprocal tag/drag clauses; threshold for drag is 75% of equity holders.

These defaults match prevailing Israeli + EU early-stage cofounder agreement norms; i-avocat (Andrei + Dmitry) finalizes the jurisdiction-specific text.

### Per-engagement deviations (recorded at kickoff)

- Vivod (Israeli market, Israeli-incorporated joint Product likely): legal layer follows Israeli company law; Misha + Pavel ratify per the i-avocat-drafted founders agreement.
- Unbind (jurisdiction TBD per [`unbind.yaml`](./projects/unbind.yaml) — likely Moldova or EU): Olya + Pavel ratify same.

---

## 4. Dual-interest discipline (§ 13.3 made operational)

The engagement charter § 13.3 names the structural dual interest: Pavel-as-cofounder of the joint Product has commercial interests that may diverge from Pavel-as-administrator-of-Studio (which Studio SRL bills the joint Product as a vendor). This document operationalizes the discipline.

### Required disclosures (non-negotiable)

1. **Studio rate changes** affecting the joint Product's vendor invoice are disclosed to the other cofounder *before* execution. Disclosure: written, in the joint Product's decisions registry; CC the founders agreement counsel (i-avocat).
2. **Specialist hires** onto the joint Product's engagement (Studio SRL routing a specialist into the engagement) are disclosed to the other cofounder *before* the specialist starts work, with the specialist's NDA + scope visible to the other cofounder.
3. **Scope changes** to Studio's deliverables that affect the joint Product's cost basis are disclosed *before* execution.
4. **Engagement renewal or termination** is a joint decision with the other cofounder; Pavel cannot unilaterally extend or end the Studio SRL vendor contract without the other cofounder's signature.

### Required exclusions

1. **Studio SRL invoices** to the joint Product are reviewed by the **non-Pavel cofounder** before payment, even when Pavel-as-cofounder is responsible for tech delivery on the receiving side. The non-Pavel cofounder retains a *veto* on the invoice (not just a review) if the invoice contradicts disclosed terms.
2. **Studio SRL decisions** affecting the joint Product's interests (e.g. taking on a competing engagement, hiring a specialist also engaged by a competitor of the joint Product) **exclude Pavel-as-cofounder** from the Studio's side of that decision. Studio's decision logs record the exclusion.

### Recorded disclosures

Every cofounder engagement carries a `disclosures.md` artifact in the joint Product's repository (path TBD per engagement). Each disclosure is a dated entry:

```
2026-MM-DD — [Type: studio-rate-change | specialist-hire | scope-change | renewal]
  Subject: <one line>
  Disclosed by: Pavel-as-cofounder
  Disclosed to: <other cofounder name>
  Acknowledgment: <other cofounder signature / Slack confirm / email confirm>
  Linked artifact: <Studio decisions.md ID / Linear issue / contract amendment ID>
```

---

## 5. Commit attribution discipline (§ 13.2 rule 1, non-negotiable)

Commits to the joint Product's repository carry one of two identities — never both, never blended:

| Identity | Author email | Bot? | Equity-bearing? |
|---|---|---|---|
| **Pavel-as-cofounder** | Pavel's personal GitHub email (e.g. `pavel.rapoport@gmail.com`) | No | YES — counts toward founders agreement vesting |
| **Studio SRL** | `rapoport-studio-bot` GitHub identity | Yes | NO — billed as vendor labor |

A single commit never claims both. Pull requests opened by `rapoport-studio-bot` carry a `Co-authored-by:` trailer naming Pavel-as-cofounder only when the PR is from a Pavel-cofounder-attributed working branch — and even then, Studio retains commit authorship for the bot's bot identity.

**Operational note.** Pavel switches Git identity locally per task:

```bash
git config user.email "pavel.rapoport@gmail.com"      # cofounder commits
git config user.email "hello@rapoport.studio"          # studio-administrator commits (rare; bots usually handle these)
```

For Forge runs, the bot identity is automatic per [`platform.md § Bot identities`](../current/platform.md). Forge-generated PRs always attribute to Studio SRL; only manual Pavel commits to the joint Product carry the cofounder identity.

### Pre-commit hook (recommended, not yet implemented)

A pre-commit hook on cofounder engagement repos checks `git config user.email` matches one of the two allowed identities and rejects commits with mismatched author. Implementation deferred until first attribution drift incident.

---

## 6. Cost ledger flow for cofounder engagements

Two ledgers coexist per cofounder engagement; reading them together produces the studio's complete economic view of the engagement.

### Ledger A — Studio SRL ↔ Joint Product (client-engagement shape)

Tracks: Studio's cash revenue from billing the joint Product as a vendor; Studio's cost of delivery (Anthropic / specialist / overhead).

- Revenue: Studio fees billed monthly / per-milestone to joint Product.
- Cost: Anthropic usage tagged to joint Product per [`design.md § D-ECN-3`](../archive/2026-05-13-establish-studio-economy/design.md); specialist subcontracts; overhead allocation.
- Margin: realized this quarter, same as any client-engagement.

### Ledger B — Pavel-as-cofounder equity in Joint Product

Tracks: Pavel's equity vesting in the joint Product; mark-to-market value (illustrative, not realized).

- Vesting events: monthly tick per the founders agreement schedule.
- Mark-to-market: based on most recent valuation event (priced round, revenue multiple, comparable, or founder-set placeholder). Recorded as an estimate in `<slug>.yaml § engagement.cofounder.equity_value_estimate`.
- Realized only at: liquidity event (acquisition / dividend / secondary sale / IPO). Until then, unrealized.

### Cross-engagement dashboard view

Per [`design.md § D-ECN-6`](../archive/2026-05-13-establish-studio-economy/design.md):

| Column | Cofounder engagement read |
|---|---|
| Revenue (realized, EUR) | Ledger A only (Studio fees) |
| Cost (Anthropic + specialist + overhead) | Ledger A only |
| Margin % | Ledger A only |
| Equity value (mark-to-market) | Ledger B only |
| Founders-agreement status | drafted / signed / not-applicable |

The strategic question "client work vs cofounder bets?" reads: total Ledger A revenue (Studio fees from cofounder engagements + client revenue) vs total Ledger B unrealized equity value.

---

## 7. IP carve-out for cofounder engagements

Per `rapoport-studio-engagement.md § 13.2 rule 5` default, with cofounder-specific clarifications:

| IP class | Owned by | Licensed to |
|---|---|---|
| Joint Product source code | Joint Product Actor (joint cofounders own equally) | n/a |
| Joint Product business model + brand | Joint Product Actor | n/a |
| OpenSpec capability specs authored against joint Product | Joint Product Actor | Studio SRL retains mutual license for methodology reuse |
| Forge engine + Studio CLI tools | Studio SRL | Joint Product Actor uses under engagement license; reverts on engagement off-ramp per § 11 of charter |
| Muse base prompts + specialist-network registry | Studio SRL | Joint Product Actor uses under engagement license |
| Custom prompts authored against joint Product domain | Joint Product Actor | Studio SRL retains mutual license |
| Pavel-attributed code in Pavel's personal GitHub identity | Joint Product Actor (work for hire under founders agreement) | Pavel-as-cofounder retains personal moral rights per author law of relevant jurisdiction |

IP carve-out **deviations** per engagement record in `<slug>.yaml § engagement.cofounder.ip_overrides`.

---

## 8. Founders agreement integration

Every cofounder engagement requires a founders agreement legal document **before** the cofounder-engagement Contract advances past `Draft` status. This document drafts the founders agreement's shape; the legal text is i-avocat's deliverable per the studio's standing legal-services relationship with Andrei + Dmitry.

### Required clauses (founders agreement minimum)

1. **Equity allocation + vesting** per § 3 above; per-engagement deviations explicit.
2. **Role definitions** for each cofounder, with the studio relationship explicit (which cofounder is also the Studio's vendor counterpart).
3. **Decision rights** for joint Product Actor: which decisions require unanimity, which require majority, which require veto.
4. **Dispute resolution** mechanism, jurisdiction, governing law.
5. **Disclosed dual interest** per § 4 (explicit, not implicit; documents Andrei's i-avocat firm as drafting party — see PF-3 in `establish-studio-economy/tasks.md`).
6. **IP assignment** + carve-outs per § 7.
7. **Exit clauses**: voluntary departure, involuntary departure (for cause), good leaver / bad leaver definitions, repurchase mechanics.
8. **Drag-along + tag-along** rights.
9. **ROFR** (right of first refusal) on equity transfers.

### Drafting flow

```
1. Cofounder engagement reaches kickoff stage.
2. Pavel + other cofounder confirm equity split + vesting at kickoff (D-SE8 row in <slug>.yaml).
3. Pavel briefs i-avocat (Andrei + Dmitry) on the agreed terms + the jurisdiction.
4. i-avocat drafts founders agreement legal PDF; turns around within agreed Discovery scope.
5. Pavel + other cofounder review; redlines via Andrei.
6. Both cofounders sign; signed PDF stored in joint Product Actor's secure storage + referenced in <slug>.yaml § engagement.cofounder.founders_agreement_ref.
7. cofounder-engagement Contract advances Draft → Proposed → Accepted (Accepted = founders agreement signed).
```

### Disclosure chain on Andrei's dual relationship (PF-3)

Because Andrei's firm (i-avocat) is **also a paying Studio client**, both cofounders are entitled to know that the founders agreement was drafted by Studio's legal counsel. The disclosure flows:

- Studio admin (Pavel) discloses to the non-Pavel cofounder **before** Andrei is engaged on the founders agreement.
- The non-Pavel cofounder has the right to engage independent counsel for review (cost typically borne by the joint Product).
- The disclosure + acknowledgment recorded in `<slug>.yaml § engagement.cofounder.legal_counsel_disclosure` and in the founders agreement preamble itself.

---

## 9. What this document does NOT cover

- **Client-engagement Contracts.** Use [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md) standalone for those.
- **Specialist subcontracts.** See [`specialist-network.md`](./specialist-network.md).
- **Per-engagement specifics** (vesting %, exit specifics, jurisdiction). Those live in `<slug>.yaml § engagement.cofounder` and ratify at kickoff.
- **Legal text.** This document is the operational shape; the founders agreement legal PDF is i-avocat's deliverable.

---

## 10. Cross-references

- [`rapoport-studio-engagement.md`](./rapoport-studio-engagement.md) — engagement charter (this doc is its companion).
- [`rapoport-studio-engagement.md § 13`](./rapoport-studio-engagement.md) — four-actor model (this doc operationalizes § 13 for cofounder engagements specifically).
- [`archive/2026-05-13-establish-studio-economy/design.md § D-ECN-1`](../archive/2026-05-13-establish-studio-economy/design.md) — Contract subtype enum (`cofounder-engagement` vs `client-engagement`).
- [`archive/2026-05-13-establish-studio-economy/proposal.md`](../archive/2026-05-13-establish-studio-economy/proposal.md) — strategic rationale for the cofounder vs client distinction.
- [`projects/vivod.yaml`](./projects/vivod.yaml) — first cofounder engagement to ratify D-SE8 (pre-kickoff late May / early June 2026).
- [`projects/unbind.yaml`](./projects/unbind.yaml) — second cofounder engagement to ratify D-SE8 (active engagement; retroactive ratification per PF-1).
- [`specialist-network.md`](./specialist-network.md) — specialist routing into engagements.
- [`risk-taxonomy.md`](./risk-taxonomy.md) — risk model that every cofounder Contract `## Risks` block references.
- [`archive/2026-05-13-amend-vivod-cofounder-economics/`](../archive/2026-05-13-amend-vivod-cofounder-economics/) — change folder appending Vivod D-SE8.
- [`archive/2026-05-13-amend-unbind-cofounder-economics/`](../archive/2026-05-13-amend-unbind-cofounder-economics/) — change folder appending Unbind D-SE8.
