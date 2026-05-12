# Convention adapter — worked example

> **Audience.** Engineers adding a new engagement to Forge. Pair with
> `openspec/current/forge/entities.md § Convention adapters` and
> `openspec/archive/2026-05-08-add-forge-convention-adapter/design.md § 1`.

A **convention adapter** lets Forge run against a non-studio OpenSpec corpus without leaking per-project conditionals into the build agent. Each engagement ships exactly one TS file under `packages/forge/src/conventions/<slug>.ts` plus a one-line registration in the built-in map.

This template walks through what a minimal adapter looks like, what hooks are mandatory vs optional, and where the seam lives in the rest of Forge.

---

## When to write a new adapter

Write one when the engagement's OpenSpec corpus diverges from the studio's prose-style minimal convention in any of these ways:

- New capability specs need YAML frontmatter (the studio default is none).
- The corpus has cross-cutting registries (`decisions.md`, `dependencies.md`, etc.) that need rows appended at decision-finalization time.
- The corpus uses **diff-at-archive** — change folders never edit `current/` directly; edits are described in `tasks.md` and applied at archive time.
- Pre-commit linting goes beyond what the studio adapter enforces (slug regex, no `specs/` subdir, table-line length).
- Capability decision IDs follow a different scheme (e.g. `D-SE-3` rather than free-form prose).

If none of the above apply, the engagement uses the studio adapter — no work needed.

---

## File shape

`packages/forge/src/conventions/<engagement-slug>.ts`:

```ts
import type {
  AdapterContext,
  AdapterViolation,
  DecisionInput,
  OpenSpecConventionAdapter,
} from './adapter.js';

const PROMPT_AUGMENTATION = [
  '<engagement> uses the prose-style minimal convention plus <list of extensions>.',
  // One paragraph. Anchor the additions; never restate the base studio rules
  // (those are covered by the un-augmented prompt).
].join(' ');

export const <slug>Adapter: OpenSpecConventionAdapter = {
  slug: '<engagement-slug>',
  name: '<Engagement name + one-line summary of the convention>',

  // ── Required hooks ─────────────────────────────────────

  capabilityFrontmatter(capability: string, ctx: AdapterContext): string | null {
    // Return a YAML block prepended to new capability specs.
    // Return null when the convention has no frontmatter.
    const today = new Date().toISOString().slice(0, 10);
    return [
      '---',
      `slug: ${capability}`,
      `owner: ${ctx.project.botAccount ?? '<default-owner>'}`,
      `last_updated: ${today}`,
      // ... other fields per the engagement's `unbind.yaml`-equivalent profile
      '---',
      '',
      '',
    ].join('\n');
  },

  allowsDirectCurrentEdits(): boolean {
    // Studio: true (current/ edits live alongside the change folder).
    // Diff-at-archive corpora: false — the build agent must not write to current/
    // from inside a change folder.
    return false;
  },

  promptAugmentation(): string {
    return PROMPT_AUGMENTATION;
  },

  // ── Optional hooks ─────────────────────────────────────

  async mirrorDecision(input: DecisionInput, ctx: AdapterContext): Promise<void> {
    // Append a row to the engagement's decisions.md registry.
    // Use ctx.fs.exists / readFile / appendFile — never node:fs directly.
    // ID auto-assignment goes through nextDecisionId (below).
  },

  async nextDecisionId(
    capability: string,
    ctx: AdapterContext,
  ): Promise<string | null> {
    // Read decisions.md, find the highest-numbered ID for the capability prefix,
    // return the next one. Return null when the adapter does not assign IDs.
    return null;
  },

  async validate(
    changePath: string,
    ctx: AdapterContext,
  ): Promise<AdapterViolation[]> {
    // Pre-commit lint. Return an array of violations (empty = pass).
    // Error-severity violations abort the build before the migrations checkpoint.
    // Warning-severity violations surface as a Linear comment but do not block.
    return [];
  },

  async archive(changePath: string, ctx: AdapterContext): Promise<void> {
    // Parse tasks.md diff-at-archive section, apply ops to current/ + registries
    // via ctx.fs, then rename the change folder to archive/<YYYY-MM-DD>-<slug>/.
    // The Phase 3C grammar parser at packages/forge/src/conventions/diff-grammar.ts
    // handles the parsing; this hook wires it to the engagement-specific naming
    // and post-conditions.
  },
};
```

---

## Registration

`packages/forge/src/conventions/index.ts`:

```ts
import { studioAdapter } from './studio.js';
import { unbindAdapter } from './unbind.js';
import { <slug>Adapter } from './<engagement-slug>.js';

const BUILT_IN: Record<string, OpenSpecConventionAdapter> = {
  studio: studioAdapter,
  unbind: unbindAdapter,
  '<engagement-slug>': <slug>Adapter,
};
```

Then declare the project in `forge.config.mjs`:

```js
projects: {
  <slug>: {
    issuePrefix: '<TEAM>',
    repo: '<owner>/<repo>',
    worktreeRoot: join(homedir(), 'Documents/GitHub/<repo>'),
    conventionAdapter: '<engagement-slug>',
    openspecRoot: 'openspec',
    botAccount: '<bot-username>',
  },
}
```

`forge --project=<slug> ...` now routes through the new adapter for every command.

---

## Tests

Add unit tests at `packages/forge/src/__tests__/conventions/<slug>.test.ts`. Use the in-memory filesystem helper:

```ts
import { makeContext } from './in-memory-fs.js';
import { <slug>Adapter } from '../../conventions/<slug>.js';

describe('<slug> adapter', () => {
  it('emits frontmatter with derived defaults', () => {
    const { ctx } = makeContext({ project: { botAccount: '<bot>' } });
    const fm = <slug>Adapter.capabilityFrontmatter!('<capability>', ctx);
    expect(fm).toContain('slug: <capability>');
  });

  it('appends a decision row to the registry', async () => {
    const { ctx, fs } = makeContext();
    await <slug>Adapter.mirrorDecision!({ /* ... */ }, ctx);
    expect(fs.dump()['openspec/decisions.md']).toContain('<expected row>');
  });
});
```

The Phase 2 conventions tests (`packages/forge/src/__tests__/conventions/{studio,unbind,registry}.test.ts`) are the canonical reference for shape and coverage.

---

## What you do **not** need to wire

- Prompt augmentation in the build subprocess — Forge's build orchestrator already appends `adapter.promptAugmentation()` to the system prompt (RAP-86 Phase 3A).
- Pre-migrations validation — already wired (RAP-86 Phase 3B).
- Adapter resolution in `cli.ts` — every command handler calls `forge.resolveProject(--project)` and routes through the resolved adapter.
- `forge spec --fix` integration — the adapter's `capabilityFrontmatter` is automatically prepended to generated `proposal.md` (RAP-86 Phase 4).

---

## Cross-references

- `openspec/current/forge/entities.md § Convention adapters` — runtime contract + integration points.
- `openspec/archive/2026-05-08-add-forge-convention-adapter/design.md` — full interface, eight locked decisions (§ 9), diff-at-archive grammar (§ 7).
- `openspec/_methodology/projects/unbind.yaml` — example of an engagement profile that drives an adapter's frontmatter / registry shape.
- `packages/forge/src/conventions/unbind.ts` — the most feature-complete reference adapter today.
