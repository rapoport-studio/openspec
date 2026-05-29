# Convention leak log — research frontmatter ambiguity ledger

> **Purpose.** Replace the endogenous "convention churn count" gate in
> [`add-inquiry-entity` design.md §6](../../archive/2026-05-29-add-inquiry-entity/design.md)
> with a **use-induced** signal: every time someone (human, agent, Forge) has
> to read prose to answer a question that the structured `> **Field:** value`
> frontmatter convention was *supposed* to answer, log one line here.
>
> The signal is asymmetric on purpose: 0 leaks may mean either (a) convention
> is leak-proof, or (b) nobody actually queried it. ≥3 leaks by 2026-07-10
> trips the Phase 2 vesting gate — convention needs work before `Inquiry`
> entity layers on top.

## Ledger format

One line per leak. Append-only. Newest at top.

```
YYYY-MM-DD | <querier> | <query intent> | <field that should have answered> | <what prose-reading produced> | <fix-or-defer>
```

Querier: `human:pavel` / `agent:claude` / `forge:<role>` / etc.

## Entries

<!-- 2026-05-28 — log opened; no entries yet -->
