# Realtime Conformance Fixture

_Apply-phase deliverable for [`openspec/archive/2026-05-11-add-frontend-runtime-standards/design.md § E`](../../archive/2026-05-11-add-frontend-runtime-standards/design.md)._

_Referenced by [`openspec/current/frontend-runtime.md § Adapter conformance`](../../current/frontend-runtime.md)._

## Purpose

This document specifies the shared Supabase fixture used by both realtime adapter conformance test suites:

- **XState** — `packages/state/src/__tests__/realtime-conformance.test.ts` (RAP-279)
- **TanStack Query** — ships in the engagement repo (Vivod-side issue tracked)

Both suites run against the same fixture project and seed data so that any drift between adapter behaviours surfaces as an asymmetric test result rather than a symmetric pass.

---

## Fixture project

**Project**: Studio dev/test Supabase project  
**Project ID**: `nifagnmgwoqkplegsicy`  
**Region**: `eu-central-2`  
**Role**: Existing development and test database. Do not provision a new project — reuse this one.

---

## Seed SQL

Apply once, idempotent on re-run:

```sql
-- Create the conformance test table
CREATE TABLE IF NOT EXISTS public.test_realtime (
  id         uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  key        text        NOT NULL,
  value      jsonb,
  org_id     uuid        NOT NULL,
  created_at timestamptz DEFAULT now()
);

-- Enable row-level security
ALTER TABLE public.test_realtime ENABLE ROW LEVEL SECURITY;

-- SELECT policy: only rows whose org_id matches the caller's JWT claim
CREATE POLICY "test_realtime_select_own_org"
  ON public.test_realtime
  FOR SELECT
  USING (org_id::text = (auth.jwt() ->> 'org_id'));

-- Add to realtime publication so Postgres Changes events are broadcast
ALTER PUBLICATION supabase_realtime ADD TABLE public.test_realtime;
```

The `org_id` column uses text comparison (`org_id::text = (auth.jwt() ->> 'org_id')`) to match the JWT claim without requiring a uuid cast — consistent with `frontend-runtime.md § Authority delegation to RLS`.

---

## Teardown

After each test run, truncate (do **not** drop — the seed is persistent):

```sql
TRUNCATE public.test_realtime;
```

---

## Six conformance assertions

Each assertion maps to one row in [`frontend-runtime.md § Adapter-binding matrix`](../../current/frontend-runtime.md). Both adapters MUST pass all six. Assertions are labelled **A-1** through **A-6** to match issue RAP-274 and future cross-references.

### A-1 — Event received within visibility bound

**Obligation**: Invalidate or patch the local cache on each event.

**Assertion**: After subscribing to the `test_realtime` channel, the adapter delivers at least one event of each change type (`INSERT`, `UPDATE`, `DELETE`) to the consumer within the p95 visibility bound.

**Expected event sequence**:
1. Subscribe; wait for channel `SUBSCRIBED` acknowledgement.
2. Test inserts a row → consumer receives `{ eventType: 'INSERT', new: { ... } }` within **2 000 ms**.
3. Test updates the row → consumer receives `{ eventType: 'UPDATE', new: { ... } }` within **2 000 ms**.
4. Test deletes the row → consumer receives `{ eventType: 'DELETE', old: { ... } }` within **2 000 ms**.

**Timing budget**: 2 000 ms per event (aligns with `p95 ≤ 2 s` for `broadcast_changes` stated in `frontend-runtime.md § Layer 2`).

---

### A-2 — CONNECTED emitted before first event

**Obligation**: Emit `CONNECTED` / `DISCONNECTED` to the consumer.

**Assertion**: The adapter emits a `CONNECTED` signal (XState event, callback, or observable value — adapter-specific surface) before delivering the first data event to the consumer.

**Expected event sequence**:
1. Subscribe.
2. `CONNECTED` signal arrives.
3. Test inserts a row.
4. `INSERT` event arrives.

`CONNECTED` MUST precede the first data event. If the harness receives a data event before `CONNECTED`, the assertion fails.

**Timing budget**: `CONNECTED` must arrive within **5 000 ms** of subscribe (standard Supabase WebSocket handshake budget).

---

### A-3 — DISCONNECTED emitted on channel error

**Obligation**: Emit `CONNECTED` / `DISCONNECTED` to the consumer.

**Assertion**: On a simulated `CHANNEL_ERROR`, the adapter emits a `DISCONNECTED` signal to the consumer within 3 000 ms.

**Expected event sequence**:
1. Subscribe; wait for `CONNECTED`.
2. Force `CHANNEL_ERROR` by calling `.unsubscribe()` or simulating a network drop at the Supabase client level.
3. `DISCONNECTED` signal arrives within **3 000 ms** of the error.

**Timing budget**: 3 000 ms after the error is induced.

---

### A-4 — Reconnect-refetch before resuming live patches

**Obligation**: Refetch authoritative state on reconnect, **before** resuming live patches.

**Assertion**: After a forced disconnect, the adapter fetches authoritative state from the server before applying any subsequent live events. Events received during the reconnect window are discarded; post-refetch state is canonical.

**Expected event sequence**:
1. Subscribe; wait for `CONNECTED`.
2. Insert row A. Verify consumer sees it.
3. Force disconnect (simulate `CHANNEL_ERROR`).
4. While disconnected, insert row B (missed event).
5. Adapter reconnects.
6. Adapter issues a fresh query — result MUST include both row A and row B.
7. No phantom `INSERT B` event is applied on top of the refetched state (i.e., row B does not appear twice).

**Timing budget**: Refetch MUST complete within **5 000 ms** of reconnection.

---

### A-5 — Visibility-change refetch

**Obligation**: Refetch on `visibilitychange → 'visible'`.

**Assertion**: When `document.visibilitychange` fires with `document.visibilityState === 'visible'`, the adapter refetches authoritative state via the same path as reconnect (A-4).

**Expected event sequence**:
1. Subscribe; wait for `CONNECTED`.
2. Insert row A. Verify consumer sees it.
3. Simulate tab hidden: dispatch `visibilitychange` with `visibilityState = 'hidden'`.
4. Insert row B (missed during hidden period).
5. Simulate tab visible: dispatch `visibilitychange` with `visibilityState = 'visible'`.
6. Adapter issues a fresh query — result MUST include both row A and row B.

**Timing budget**: Refetch MUST begin within **500 ms** of the `visibilitychange` event; MUST complete within **5 000 ms**.

**Note**: `supabaseRealtimeBridge` (XState adapter) marks A-5 as a follow-up (see `state.md`). Test expected to fail until RAP-279 implements the `VISIBILITY.VISIBLE` path.

---

### A-6 — Burst coalescing

**Obligation**: Coalesce burst events within ~250 ms.

**Assertion**: When 10 events for the same key arrive within 100 ms, the consumer receives at most 2 renders (state-machine sends / hook re-renders).

**Expected event sequence**:
1. Subscribe; wait for `CONNECTED`.
2. Rapidly insert 10 rows in ≤ 100 ms (fire-and-forget, no per-insert await).
3. Wait 500 ms.
4. Assert total consumer render count ≤ 2.

**Timing budget**: All 10 inserts within 100 ms; render count measured over the subsequent 500 ms window.

**Rationale**: The ~250 ms coalescing window collapses 10 events arriving within 100 ms into a single batch. The ≤ 2 budget allows for one render at the end of the batch plus one optional final reconciliation pass.

---

## References

- Spec: [`openspec/current/frontend-runtime.md § Adapter conformance`](../../current/frontend-runtime.md)
- Design (§ E): [`openspec/archive/2026-05-11-add-frontend-runtime-standards/design.md`](../../archive/2026-05-11-add-frontend-runtime-standards/design.md)
- XState adapter: `packages/state/src/actors/supabase-realtime-bridge.ts`
- XState conformance test (RAP-279): `packages/state/src/__tests__/realtime-conformance.test.ts`
- TanStack Query adapter: `Vivod#267`
