---
name: review-ambiguous-start
description: Clarifies that `gentle-ai review status` returning `applicability: ambiguous` / `action: select_lineage` does NOT mean `gentle-ai review start` is blocked. Prevents orchestrators from stalling, selecting an unrelated lineage, or bypassing the pre-commit/pre-push lineage guard when the real fix is a plain `review start`. Use when a commit/push is blocked by a missing approved receipt and `review status` reports ambiguous applicability.
---

# review-ambiguous-start

Diagnostic-only skill for a recurring orchestrator mistake: treating `review status`'s
`ambiguous` applicability as proof that the whole native review lifecycle is stuck,
when in practice `review start` is a **different operation** that is very likely to
succeed cleanly.

## Why this happens

`gentle-ai review status --next-transition` reports on lineages that **already
exist** in `.git/gentle-ai/review-transactions/`. For content that was never
reviewed before, none of the existing lineages match, so status has nothing
useful to resolve — it returns `applicability: "ambiguous"`, `action:
"select_lineage"`, and dumps every candidate lineage in the store (can be 50+).

That response looks like a hard block. It is not. `gentle-ai review start`
creates a **brand-new lineage bound to the current content's exact
`target_identity`** — it does not need to disambiguate against unrelated
history at all. Confirmed empirically: a repo with 58 unrelated candidate
lineages (none matching) still got `action: "created"` from a single plain
`review start` call, no flags, no selection.

## When to Use

- A commit/push/PR is blocked because a guard requires an "approved receipt"
  that doesn't exist yet.
- `gentle-ai review status --next-transition` (or the plain lifecycle status
  call) reports `applicability: "ambiguous"` and `action: "select_lineage"`
  (or equivalent phrasing) with a list of candidate lineages.
- The orchestrator's own reasoning concludes "review start is also blocked"
  **without having actually run `review start`**.

Do NOT use this for:

- A `capture-result` JSON-decode failure — that's `review-capture-helper`,
  a different bug class entirely.
- A case where `review start` genuinely returns `ambiguous` too (see Escalation
  below) — that is a real controller bug, not this pattern.

## Workflow

### Step 1 — Read the target identity from status

```
gentle-ai review status --cwd <repo> --contract gentle-ai.review-integration/v1 --next-transition
```

Note `target_identity` (`sha256:...`) and, if present, the candidate tree hash.

### Step 2 — Verify none of the candidates actually match (read-only, optional but recommended)

```
grep -r "<target_identity>" "$(git rev-parse --git-dir)/gentle-ai/review-transactions/"
```

If this returns nothing, no existing lineage owns this content — selecting any
candidate would misattribute fresh content to an unrelated review history.
Never select a candidate on this path.

### Step 3 — Try `review start` directly, no flags

```
gentle-ai review start --cwd <repo>
```

- `action: "created"` → a new lineage now exists (`lineage_id`, `selected_lenses`,
  `risk_level`, `correction_budget`). Proceed with the normal 4R/lens flow on
  this lineage. The guard's "approved receipt" requirement becomes reachable
  through this lineage, not through the ambiguous status response.
- Any other error → capture it verbatim and move to Escalation below.

### Step 4 — Re-run status

Once a lineage exists, `review status --next-transition` resolves to a
concrete native transition (e.g., the first lens, order 0) instead of
`select_lineage`. Continue the lifecycle from there.

## Boundaries

- NEVER select a candidate lineage from the `ambiguous` response as a way to
  "unblock" a commit — that binds unrelated content to someone else's review
  history and corrupts provenance.
- NEVER bypass the pre-commit/pre-push lineage guard by committing/pushing
  from a shell or tool outside the guarded environment as a shortcut. That
  does not resolve the ambiguous state — it just produces a commit with no
  real receipt, which the next gate (`pre-push`/`pre-PR`) will also reject.
- This skill is read-only diagnosis plus one native `review start` call. It
  never edits repository files and never fabricates a receipt.

## Escalation

If `gentle-ai review start` **itself** returns `ambiguous` / demands
`select_lineage` (not just `status`), that is a genuine controller bug, not
this pattern. Stop, do not select anything, do not bypass the guard, and
report it upstream with the full JSON from both `status` and `start`. See
Gentleman-Programming/gentle-ai#1789 for the sibling regression this pattern
was discovered alongside.
