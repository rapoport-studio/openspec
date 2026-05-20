# Orchestra risk register

> Twelve risks tracked for the agent orchestra MVP, ranked by probability × impact. Each row carries the originating source taxonomy and the MVP mitigation. R1–R4 are must-fix before v1.0 GA; R5–R8 are addressed by existing spec design; R9–R12 are monitored, not blockers.
>
> **Source.** This register is a verbatim transcription of `openspec/_methodology/research/orchestra-risk-metrics-learning-2026-05.md § 1.5 Consolidated risk register для Rapoport MVP — Top 12`. Any wording delta is intentional and called out in the commit message that introduced or amended the delta.
>
> **Consumers.** Maestro spec (`openspec/current/agents/maestro.md § Risk matrix` cites these R-IDs); Maestro runtime defenses (orchestra_journal, decision-on-ambiguity surface, cost circuit breaker); the four EU AI Act log-retention / human-oversight / incident-response principles applied to Rapoport's non-high-risk MVP per `orchestra-risk-metrics-learning § 4.1`.
>
> **Review cadence.** Quarterly. R12 (EU AI Act classification drift) explicitly anchors the quarterly review trigger; any new client engagement also re-runs the R12 classification check.

## Top 12 — ranked by probability × impact

| # | Risk | Source | Probability | Impact | MVP mitigation |
|---|---|---|---|---|---|
| R1 | **Routing decision undefined** (Maestro не знает куда отправить) | MAST Spec | High | Med | Decision-on-ambiguity surface to Pavel; default safe = ambiguity |
| R2 | **Handoff schema drift** (Muse expects X, Forge gives Y) | MAST Spec | Med | High | JSON schema validation на каждом event_type, contract test в `__tests__` |
| R3 | **Cost runaway loop** (Maestro reroutes infinitely) | OWASP ASI08 | Med | High | Per-session cap $X (Δ5), per-issue cap, hard stop с journal entry |
| R4 | **Indirect prompt injection** через Linear issue body | OWASP ASI01 | Med | Med | Strip-and-quote (Δ6); issue body wrapped в `<untrusted_user_content>` |
| R5 | **Journal write race / inconsistent seq** | MAST Coord | Low | High | `seq monotonic per work_item_key`, content_hash на read |
| R6 | **Agent_status stale** (Muse crashed, не обновил) | MAST Verif | Med | Med | Stale-detect heuristic — если no journal entry от агента > N min, surface drift |
| R7 | **Forge worktree lock contention** (RAP-405 known issue) | Infra | Low | Low | `forge cancel <other-key>` mechanism уже есть, document как часть Maestro playbook |
| R8 | **Decision laundering** (Pavel думает что Maestro решила, на самом деле Muse в prompt'e подсказала) | Maestro spec | Med | Med | `originating_agent_id` + content_hash в каждом journal entry (уже в spec'е) |
| R9 | **Cost guard miss** (Anthropic dashboard latency, Pavel узнаёт пост-фактум) | Infra | High | Low | Langfuse cost tracking real-time + alert threshold (Δ1) |
| R10 | **Tool misuse** (Forge получает не тот repo, пишет в wrong worktree) | OWASP ASI02 | Low | High | `--project=<slug>` mandatory на multi-project; sandbox=docker default |
| R11 | **Knowledge fragmentation** (один lesson написан в Forge, другой in Maestro, никто не видит вместе) | Learning | Med | Low | `_methodology/lessons/` aggregator script (post-MVP) |
| R12 | **EU AI Act classification drift** (Rapoport случайно становится «high-risk» через client engagement) | Compliance | Low | High | Risk register quarterly review, classification re-check на каждый new client mode |

R1, R2, R3, R4 — **must-fix перед v1.0 GA**. R5–R8 — addressed by existing spec'ом design. R9–R12 — monitor, не блокеры.

## Source taxonomies

- **MAST** (Multi-Agent System Failure Taxonomy) — Cemri et al. *Why Do Multi-Agent LLM Systems Fail?* (arXiv:2503.13657, NeurIPS 2025 Datasets and Benchmarks Track; 1,642 traces analysed). Subcategories cited: `MAST Spec` (specification undefinedness), `MAST Coord` (coordination failure), `MAST Verif` (verification failure).
- **OWASP ASI** — OWASP Top 10 for Agentic Applications 2026. `ASI01` = indirect prompt injection; `ASI02` = tool misuse; `ASI08` = cost / resource runaway.
- **Maestro spec** — `openspec/current/agents/maestro.md § Risk matrix` (decision laundering, bottleneck cascade, refusal failure). This register expands on the Maestro-spec risk matrix with the MAST + OWASP-ASI rows.
- **Infra** — Rapoport-specific operational hazards observed in the forge / studio runtime.
- **Learning** — knowledge-management hazard surfaced by the lessons-learned pattern.
- **Compliance** — EU AI Act regulatory hazard; non-high-risk MVP today but watch for client-engagement drift.

## Cross-references

- Maestro runtime substrate that consumes this register:
  - `orchestra_journal` table (RAP-185, migration `supabase/migrations/20260510030411_add_orchestra_journal.sql`) — `originating_agent_id` + `content_hash` + `(session_id, sequence)` UNIQUE defend R5, R8.
  - `agent_status` table (RAP-186, migration `supabase/migrations/20260510000001_create_agent_status.sql`) — `last_heartbeat_at` drives R6 stale-detect.
  - `packages/muse/src/modes/maestro/route.ts:57` — `route()` validator's 4 invariants (invalid_target / agent_claimed / stale_snapshot / agent_idle_required) defend R1, R5.
  - `MAESTRO_SESSION_CAP_USD` env override (P3 of `add-maestro-runtime-mvp`) — defends R3 cost runaway.
- Deferred defenses (tracked but not landed on v1.0):
  - Langfuse self-host (Δ1) — `add-langfuse-self-host` change. Real-time cost-guard for R9.
  - Strip-and-quote on Linear issue body (Δ6) — `add-maestro-linear-webhook` or `add-maestro-prompt-injection-guard`. Defends R4.
  - Cross-issue lessons aggregator (Δ3) — `add-maestro-lessons-writeback`. Defends R11.
