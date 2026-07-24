---
name: review-capture-helper
description: Wraps gentle-ai review capture-result and finalize commands to survive the RDD 2.1.11 reviewer-output formatting bug. Sanitizes reviewer JSON, retries on raw fallbacks, and submits each lens result via the native facade. Use when you need to run a 4R review and `capture-result` fails with `decode reviewer result: invalid character ...`.
---

# review-capture-helper

Workaround skill for the gentle-ai 2.1.11 reviewer-output formatting bug. The native reviewer occasionally emits a markdown-fenced JSON block (for example ```` ```json ... ``` ````) plus a trailing summary instead of raw JSON. `capture-result` fails with `decode reviewer result: invalid character ... looking for beginning of value` and stores the undecodable output under `.git/gentle-ai/review-transactions/incidents/<lineage>/<order>-<lens>-<token>.raw`. This helper recovers that output, normalizes it to a valid reviewer JSON document, and re-submits it.

## When to Use

- `gentle-ai review capture-result` returns `decode reviewer result: invalid character ...`.
- The preserved `.raw` artifact under `.git/gentle-ai/review-transactions/incidents/<lineage>/` exists.
- You can read the file system and shell out to `gentle-ai review ...` commands.

Do NOT use this helper for:

- A reviewer result whose first character is `{` and last character is `}` — capture it directly.
- Any failure that is not a JSON-decode error from `capture-result`.
- Anything that requires editing repository files; this helper is read-only.

## Workflow

### Step 1 — Discover the preserved artifact by identity, not by recency

The captured-result CLI preserves failures under `<git-dir>/gentle-ai/review-transactions/incidents/<lineage>/<order>-<lens>-<token>.raw`. You MUST select the artifact by exact identity, never by recency. Two failed lenses could leave two `.raw` files in the same directory; selecting the latest one would attribute the wrong result to the wrong lens and corrupt the review.

```
$lineageDir = Join-Path (git rev-parse --git-dir) "gentle-ai/review-transactions/incidents/$Lineage"
$pattern = "$Order-$Lens-*.raw"
$raw = Get-ChildItem -LiteralPath $lineageDir -Filter $pattern -File | Select-Object -First 1
if (-not $raw) { throw "no preserved artifact for lineage=$Lineage lens=$Lens order=$Order" }
$rawPath = $raw.FullName
$rawText = Get-Content -LiteralPath $rawPath -Raw
```

If more than one file matches the pattern, fail closed: STOP and report the duplicate instead of picking one. Do NOT select by `LastWriteTime` or any other heuristic.

### Step 2 — Extract the embedded JSON object

If the file starts with a markdown fence, strip leading prose/fence and isolate the FIRST complete JSON object. Use a regex-based extractor that searches for the outer-most `{ ... }` braces that contain the `findings` and `evidence` keys:

```
$jsonText = Extract-EmbeddedJson -Input $rawText
```

The extractor MUST:

- Strip BOM characters (`\uFEFF`) at the start.
- Strip leading lines until the first `{` character.
- If the text begins with ```` ```json ````, drop the opening fence line.
- Walk braces from the first `{` to balance nested braces, strings, escapes, and template literals.
- Reject the input if no balanced JSON object is found.
- Optionally strip a single trailing markdown fence (```` ``` ````) before serializing.

### Step 3 — Validate and repair the JSON shape

```
$obj = $jsonText | ConvertFrom-Json
if (-not ($obj.PSObject.Properties.Name -contains 'findings' -and $obj.PSObject.Properties.Name -contains 'evidence')) {
    throw "recovered JSON missing required fields"
}
$obj.findings = @($obj.findings)
$obj.evidence = @($obj.evidence)
```

- Cast `findings` and `evidence` to arrays (single-element results must still be JSON arrays).
- Confirm every finding has only the allowed fields: `location`, `severity`, `claim`, `evidence_class`, `causal_disposition`, `proof_refs`. If any finding contains a non-allowed field, STOP and ask the user before stripping it (see Boundaries). NEVER silently remove fields.
- Confirm `severity` is one of `BLOCKER | CRITICAL | WARNING | SUGGESTION`.

### Step 4 — Write the recovered JSON to a temp file

```
$recoveredPath = Join-Path $env:TEMP "gentle-ai-<lineage>-<lens>-<order>.json"
$jsonText | Set-Content -LiteralPath $recoveredPath -NoNewline -Encoding UTF8
```

Encode with `ConvertTo-Json -Depth 16 -Compress` (PowerShell) or `JSON.stringify` (Node) so the first byte is `{` and the last byte is `}` with no BOM or trailing newline.

### Step 5 — Re-submit via the native facade

Read the lineage, revision, target, lens, and order from the last `gentle-ai review status` output and call capture-result with the recovered file:

```
$args = @(
  "review", "capture-result",
  "--cwd", (Get-Location).Path,
  "--lineage", "<lineage>",
  "--expected-revision", "<revision>",
  "--target", "<target>",
  "--lens", "<lens>",
  "--order", "<order>",
  "--input", $recoveredPath
)
& gentle-ai @args
```

On success the captured SHA256 path replaces the `.raw` incident. On failure, stop and report the new error; do not retry blindly.

### Step 6 — Verify with status

After each successful capture, re-run status. The `next_transition.collect.inputs` array should advance to the next lens (or to finalize after the last lens).

### Step 7 — Finalize

Once all lenses are captured, run:

```
gentle-ai review finalize --cwd <repo> --lineage <lineage> --captured-results
```

Only use `--captured-evidence` when the transition is `validating` and the evidence file has already been captured. Re-run status before each command.

## Sample Extractor (PowerShell)

This is the only extractor version validated for capture-result. Earlier drafts had two real bugs:

- `$c -eq '\\'` compared a single character to a two-character string and never matched, so `\"` inside strings closed the string early and the brace counter returned truncated JSON.
- `param([string]$Input)` shadowed PowerShell's automatic `$Input` pipeline variable.

```
function Extract-EmbeddedJson {
    param([string]$Text)
    $text = $Text.TrimStart([char]0xFEFF)
    $lines = $text -split "`n"
    $start = -1
    foreach ($line in $lines) {
        $idx = $line.IndexOf('{')
        if ($idx -ge 0) { $start = $idx; break }
    }
    if ($start -lt 0) { throw "no JSON object found" }
    $cursor = $start
    $depth = 0; $inString = $false; $escape = $false; $ticks = 0
    while ($cursor -lt $text.Length) {
        $c = $text[$cursor]
        if ($inString) {
            if ($escape) { $escape = $false }
            elseif ($c -eq [char]'\') { $escape = $true }
            elseif ($c -eq '"') { $inString = $false }
        } else {
            switch ($c) {
                '"' { $inString = $true }
                '`' { $ticks = ($ticks + 1) % 2 }
                '{' { if ($ticks -eq 0) { $depth++ } }
                '}' { if ($ticks -eq 0) { $depth--; if ($depth -eq 0) { return $text.Substring($start, $cursor - $start + 1) } } }
            }
        }
        $cursor++
    }
    throw "unbalanced JSON object"
}
```

## Boundaries

- This helper NEVER edits repository files, source code, or SDD artifacts.
- This helper NEVER modifies the preserved `.raw` incident.
- This helper NEVER invents findings or evidence; it only recovers what the reviewer already produced.
- If the preserved `.raw` artifact is missing or unreadable, STOP and report; do not fabricate input.
- If the reviewer result still contains a non-allowed field after recovery, STOP and ask the user before stripping it.
