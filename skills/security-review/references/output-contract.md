# Output and Presentation Contract — security-review

Read this before finalizing findings. It carries the exact document shape, the
severity rubric, the confidence caps, the validation gate, and the presentation rules
for anything the user reads.

It does not change any rule in `SKILL.md`; it is the detail behind them.

---

## 0. Hard gate

These are not defaults to adapt. If you cannot meet them, **stop and say why** — never
invent a fallback format.

- The artifact is exactly `.audit/security-findings.json`, or the same JSON in the
  conversation when the environment cannot write files
- Valid JSON, matching the shape in section 1
- **No `.audit/security-review.md`. No separate Markdown report.** Ever
- `confidence` is exactly one of `verified`, `inferred`, `unverified` — nothing else
- **No numeric confidence.** No `9/10`, no percentages, no scores, no invented
  thresholds such as "dropped everything below 8". Confidence is a category, not a
  number, and there is no cut-off to compute
- `id` is `SEC-###`
- Every schema field present, none renamed or added
- Every `snippet` an exact copy — **no `...`, no elision, no paraphrase**
- Every cited file and line re-opened and re-read before the item is written
- `coverage` and `checked_and_clean` valid per sections 2 and 5

A report that is well reasoned but off-contract has failed. The contract is what makes
findings comparable and mergeable across reviews.

---

## 1. The document

One JSON document. No separate Markdown report — two documents with one content
become two documents that disagree.

Write it to `.audit/security-findings.json` if the environment allows. If not,
present the same JSON in the conversation; its validity does not depend on being a
file.

The JSON content is English. Only the user summary is Persian.

```json
{
  "skill": "security-review",
  "repo": "<name>",
  "repo_commit": "<commit id, or \"unknown\">",
  "generated_at": "<YYYY-MM-DD>",
  "project_map": {
    "used": true,
    "repo_commit": "<the map's commit id, or \"unknown\">",
    "staleness": "current | stale-minor | stale-structural"
  },

  "coverage": {
    "reviewed": ["<paths actually read>"],
    "file_count": 0,
    "not_reviewed": [{ "path": "<path>", "reason": "<specific reason>" }],
    "could_not_inspect": [{ "item": "<what>", "reason": "<why>" }]
  },

  "checked_and_clean": [
    {
      "area": "<what you checked>",
      "file": "<file you actually read>",
      "lines": "<range your judgement rests on>",
      "note": "<what you saw that made it clean>"
    }
  ],

  "findings": [
    {
      "id": "SEC-001",
      "title": "<one specific sentence>",
      "severity": "critical | high | medium | low | info",
      "confidence": "verified | inferred | unverified",
      "status": "open",
      "effort": "S | M | L",
      "evidence": [
        { "file": "<path>", "lines": "<n or n-m>", "snippet": "<exact copy>" }
      ],
      "explanation": "<why this code, in this project, is a problem>",
      "impact": "<what happens if left, and to whom>",
      "verification": "<how to prove or disprove it>",
      "suggested_fix": "<smallest change that resolves it>",
      "refs": ["<classification id>"]
    }
  ]
}
```

## 2. Top-level fields

All are required. `findings` may be empty; `coverage` may not.

- `repo_commit` — the code version at review time. Without it the report cannot be
  reproduced. If no identifier is available, write `"unknown"` and say so in
  `coverage`. Never invent one.
- `project_map.used` is always `true` here. This review does not run without a map.
- `project_map.staleness` — `current`, `stale-minor` (ordinary in-file changes, the
  map still holds), or `stale-structural` (the map is wrong and the review stops).

### `coverage`

Not optional, and leaving it empty is a violation. A report that does not state its
own boundary misleads the reader, who assumes everything was seen.

- `reviewed` — paths actually read
- `file_count` — how many files
- `not_reviewed` — deliberately skipped, with a reason
- `could_not_inspect` — could not be examined at all (the `unknown` level), with a
  reason

Every `reason` must be specific. "Not relevant" is not a reason.

Because depth follows risk, `coverage` should also say where you read deeply and
where you only glanced — not just which files you opened.

### `checked_and_clean`

Areas examined carefully that turned out fine. Without this section, an empty report
and a review that never happened look identical from the outside.

All four fields are required — `area`, `file`, `lines`, `note` — and `file` and
`lines` are validated exactly like `evidence`.

"This is fine" is a claim, and carries the same burden of proof as a finding. Only
list an area you examined with real depth: seeing two routes does not let you write
"authorization checked, clean". If you looked at something but cannot point to a
specific file and line, it belongs in `coverage.reviewed` or nowhere.

## 3. Finding fields

Required: `id`, `title`, `severity`, `confidence`, `status`, `effort`, `evidence`,
`explanation`, `impact`, `verification`, `suggested_fix`.
Optional: `refs`.

- `id` — `SEC-<NNN>`, numbered from `001`, unique within the report. The prefix keeps
  the origin visible after reports are merged.
- `title` — one sentence, specific, no generalities.
- `status` — always `open` here. Later stages move it to `confirmed`, `rejected`,
  `deferred`, or `fixed`.
- `effort` — `S` local change · `M` several files · `L` needs redesign.
- `refs` — optional external classifications. Use current ones; categories move
  between revisions and citing a retired one is wrong.

### `evidence`

An array, at least one entry:

```json
{ "file": "<path>", "lines": "<n or n-m>", "snippet": "<exact copy>" }
```

- `file` — relative to the repository root; must actually exist
- `lines` — `"42"` or `"42-47"`
- `snippet` — copied exactly; searching the file for it must find it

Use several entries when a finding has more than one location. For a data-flow
finding, that means the source and the sink.

## 4. Severity

```text
severity = harm × blast radius × irreversibility × who can reach it
```

| axis | question |
|---|---|
| harm | in a realistic scenario, what is the worst that happens? |
| blast radius | how many people, how much data, how much of the system? |
| irreversibility | can it be repaired afterwards, or is the damage permanent? |
| reachability | who can get to this — anonymous internet, or admin only? |

Permanent damage (data loss, disclosure) always outranks temporary damage (an error,
slowness), even at smaller scale.

**`critical`** — the system or its data is fundamentally exposed; act tonight.
Authentication or authorization fully bypassed · sensitive data exposed to someone
who should not see it · unrecoverable data loss · attacker code execution · total
outage for all users.

**`high`** — serious damage to an important part of the system or a group of users;
ahead of routine work. Disclosure or tampering on a limited path · an authorization
mistake affecting some cases · data corruption under plausible conditions · silent
failure where the system looks fine but the result is wrong.

**`medium`** — a real problem with limited impact, or one that needs particular
conditions. Should be fixed, but not this week.

**`low`** — real and proven, little consequence. If it is never fixed, nothing
much happens.

**`info`** — not a problem. An observation the reader should know.

### The final test

> If this happened in production right now, what would happen and who would notice?

Nobody → `low` or `info`. Users → at least `medium`. Everyone, immediately →
`critical`. If you cannot answer, you do not understand it well enough to report it.

### Caps by confidence

Absolute, and not negotiable:

| `confidence` | max `severity` |
|---|---|
| `verified` | `critical` |
| `inferred` | `high` |
| `unverified` | `medium` |

An unproven suspicion never becomes `critical`, however frightening its consequence.
If it really is `critical`, go and prove it, then write it.

### Common severity mistakes

- **Raising severity so people take it seriously** — this devalues the whole report.
  Use the real severity plus a convincing `impact`.
- **Lowering severity because you are unsure** — that is `confidence`, not `severity`.
- **`critical` for something that only runs in development** — that ignores blast
  radius. It is `low` or `info`.
- **One severity for a group of similar items** — each has its own reach; judge them
  separately.
- **High severity for "bad code" with no concrete consequence** — severity is about
  outcome, not beauty. `low`, or not a finding at all.

## 5. Validation gate

Before emitting anything:

1. The Project Map exists and is current enough — otherwise produce no document
2. Every `file` exists
3. Every `lines` range matches the real content
4. Every `snippet` is found exactly, in those lines
5. No `severity` exceeds its `confidence` cap
6. Every `id` is unique and correctly prefixed
7. `coverage` is filled in honestly, including depth
8. The JSON is valid

**If 2, 3 or 4 fails, the item is deleted — not corrected.** That applies to
`checked_and_clean` exactly as it does to `findings`. A wrong reference inside a
"this is clean" claim is as serious as one inside a finding.

### An empty array is valid

```json
{ "findings": [] }
```

A report with zero findings, honest `coverage`, and a well-filled
`checked_and_clean` is a complete and good report. Never pad `findings` to make the
report look substantial — that is a failure, not a success.

## 6. The user summary

After the document, a short Persian summary. It does not replace the document — the
artifact holds the detail, the summary exists so the user can decide quickly. Do not
restate the report.

Use this shape:

```markdown
## نتیجه

۴ Finding پیدا شد:

- High: 1
- Medium: 3

### مهم‌ترین مورد

`SEC-001` — <یک جملهٔ کوتاه>

Severity: High
Confidence: Inferred
Location: `<file>:<line>`

یک یا دو جملهٔ توضیح.

### پوشش

بررسی شد:
- Authentication
- Authorization
- Business logic

بررسی نشد:
- Production firewall
- Real environment configuration
```

When nothing was found, keep the same shape and say so plainly under `## نتیجه`, then
list what you checked and found clean with its file and lines.

Never write that the project is secure or that no vulnerabilities exist. The honest
forms are *در محدودهٔ بررسی‌شده، finding قابل اقدام دیگری پیدا نشد* and *بخش `<X>`
بررسی شد و در محدودهٔ شواهد موجود، finding تأییدشده‌ای نداشت*.

## 7. Presentation rules

These apply to the summary and to anything else the user reads. They are the runtime
subset of the project's shared presentation contract.

- Persian prose; keep technical terms in English where translating costs precision —
  authentication, authorization, middleware, finding, repository, Project Map
- File paths, symbols, routes, fields and literal values always in backticks
- Short paragraphs, two to four lines, one idea each. No wall of text
- Bullets for sets of items; at most one level of nesting
- Bold only for what is genuinely important
- Code blocks only for code, JSON, commands, schema or a small diagram — never as a
  decorative box around prose
- Tables only when a comparison is genuinely clearer as a grid; no long prose in cells
- At most three heading levels, and no heading for two sentences
- Blank lines between independent sections

Inside the JSON, the same discipline applies to the text fields: `title` is one short
sentence, and `explanation`, `impact`, `verification` and `suggested_fix` say what is
needed and stop. No essays in JSON fields.
