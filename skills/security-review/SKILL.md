---
name: security-review
description: Review a repository for security flaws and produce evidence-backed findings — exposed entry points, authentication, authorization and ownership rules, injection and other data-flow sinks, secrets committed to the repository, sensitive data, crypto use, fail-open error handling, and security-relevant configuration. Requires a Project Map from project-understanding first. Use whenever someone asks for a security review, a security audit, a vulnerability assessment, or asks whether a codebase is safe. It reports findings only — it never fixes code.
---

# Security Review

You are looking for flaws a real attacker could exploit, and reporting them with
evidence the reader can check without having to trust you.

The hard part of this job is not finding things — it is **saying no** to the things
you found. Models doing exactly this task reliably show high recall and weaker
precision: you will generate more candidates than are real. Everything below exists
to make you reject the ones that are not.

## What you produce, and what you never do

You produce **one findings document** and a short summary for the user.

You do **not** fix, patch, refactor, or reorganize anything. You do not edit any
application file — the only file you may write is the review artifact. You do not
plan remediation beyond the one-line `suggested_fix` on each finding. You do not
write exploit code, test anything against a live system, or validate a discovered
credential.

If the user asked for a security review, they asked for a review.

## Scope

**IN SCOPE** — a flaw with all three of:

1. a plausible attacker
2. a traceable path from a real input
3. a consequence for confidentiality, integrity, or availability

**OUT OF SCOPE** — and it belongs to a different reviewer, so leave it out rather
than reframing it as a security concern:

- code quality, readability, naming, duplication, dead code — unless the defect
  leaks information or fails open
- schema design, indexing, migration safety, data integrity with no attacker
  involved
- slowness with no attacker dimension — but resource exhaustion an attacker can
  *control* is yours
- design and structure with no nameable attack — if you can name the attack, it is
  yours; if it is about cohesion and maintainability, it is not
- pipeline health, infrastructure access, and the deployment process — but
  configuration **committed to this repository** with a direct security consequence
  is yours
- confirming a vulnerability actually works at runtime — outside static review

## Before you start: the Project Map

This review runs **after** `project-understanding`. It needs that map to know which
inputs come from the internet and which are internal, where the trust boundaries
are, which frameworks and versions are in play, and which code is actually live.

The map may be a file in the repository or in this conversation — either is fine, as
long as it came from `project-understanding`.

```text
No map?
   → Stop. Produce no findings document.
   → Tell the user to run project-understanding first.
```

**Never invent a map to get past this gate.** A map you improvised is exactly the
missing context that makes every finding `high`.

If a map exists, check it against the current code. Ordinary in-file changes: use it,
and say so in `coverage`. Structural change — new components, new dependencies, moved
entry points, shifted trust boundaries — **stop** and ask for a refreshed map.
Reviewing against a wrong map is worse than reviewing without one; it points you
confidently in the wrong direction.

**The map is input, not evidence.** It tells you where to look; the repository tells
you what is true. Before recording any finding, open the real file and read the real
code. A claim supported only by a line in the map is not a finding.

## How to work

**Use your full reasoning and repository exploration capability.**

This skill defines minimum quality constraints, known failure modes, evidence
requirements, and blind spots. It does **not** define the complete set of valid
reasoning paths.

There is no required order and no checklist to walk. Follow what you find: an odd
permission check leads to the middleware, which leads to the route table, which leads
back to a handler you had already dismissed.

If you find an attack path none of the categories below anticipated — a logic flaw
specific to this project, a state transition that should not be reachable —
**investigate it**. Business logic flaws in particular do not come from patterns, and
they are where careful reading beats any scanner.

```text
Explore freely
Generate hypotheses freely
Verify aggressively
Report conservatively
```

**Depth follows risk.** Internet-facing code, anonymous endpoints, authentication and
authorization paths, privilege and payment changes, and anything touching secrets get
deep reading. Code nothing reaches gets a glance. A review that reads everything at
one depth has spent its attention in the wrong place — and `coverage` records not
just what you read but how deeply.

## Two stages: candidates, then rejection

Keep these separate. Collapsing them is how over-reporting wins.

**Stage 1 — candidates.** Be generous. Anything that looks wrong is a candidate.

**Stage 2 — rejection.** For each candidate, actively try to kill it. Go looking for
the reason it is *not* a finding, and report only what survives:

- **A framework already neutralizes it** — the largest source of false positives
- **The input is not attacker-controlled** — configuration, environment variables and
  code constants are server-controlled; request parameters, headers, uploaded files
  and stored user-supplied values are not. Trace the source; do not assume it
- **The check exists elsewhere** — authorization commonly lives in middleware,
  decorators, route configuration or the framework layer, spread across files.
  IDOR-shaped claims are wrong more than half the time for exactly this reason
- **The code is unreachable** — dead, superseded, test, example, generated, vendored
- **The sink is not dangerous here** — ask what the value is *used for*; a weak hash
  as a cache key or file fingerprint is not a crypto failure
- **The behaviour is intentional**, and the code says so
- **The control is outside the repository**, where you cannot see it either way

Two rules govern this stage:

**Rejection is not silence.** If a candidate is real but you cannot fully prove it,
the answer is a **lower `confidence`**, not deletion. Dropping a genuine insight
because you were unsure is the opposite failure, and just as bad.

**A killed candidate stays killed.** If re-reading shows the code is not what you
thought, throw the finding away — do not repair it, retarget it, or convert it into a
different finding about the same file. If something else there is wrong, investigate
that independently.

## Minimum coverage

A floor of topics — not a checklist, not a ceiling. Depending on what the project is,
look at least at:

- **Edges** — entry points and how exposed each is; input validation; output handling
- **Identity** — authentication mechanism; session and token handling; roles
- **Access** — authorization and object-level checks; ownership rules; privilege
  escalation; broken object authorization (IDOR)
- **Injection and data flow** — SQL and NoSQL, command execution, path traversal and
  file handling, SSRF and outbound requests, XSS, and CSRF where the architecture
  makes it relevant
- **Data** — sensitive data and where it goes; secrets and credentials committed to
  the repository; crypto use; database and storage access rules
- **Logic and state** — business logic abuse; state transitions; fail-open behaviour;
  error handling and security logging
- **Configuration** — security-relevant configuration inside the repository;
  framework protections and versions; deployment controls the repository reveals

**If a category does not apply to this project, skip it.** Do not manufacture a
finding to fill it. Zero findings is a valid and complete result.

## Proof models differ by class

**Data-flow findings** — injection, XSS, command injection, path traversal, SSRF and
relatives. A dangerous function alone is not a finding. To reach `verified` you must
show a real chain:

```text
attacker-controlled source
   → path through the application
   → dangerous sink
   → no sanitizer, validator, or framework protection in between
```

If part of the chain cannot be confirmed statically — a value crossing into a queue,
a worker, another service — do not fake it and do not discard it. Lower `confidence`
and write in `verification` **exactly which link is unproven and why**. Failing to
prove the path is not proof that the path is safe.

**Everything else** — do not force that model onto other classes. Missing
authorization is the *absence* of a check, not the flow of a value. Business logic
flaws are a broken rule. Unsafe state transitions, crypto misuse and fail-open error
handling each have their own shape of proof. Reducing all of security review to
data-flow tracing is a domain error.

## Framework protections

Before reporting anything about injection, XSS or CSRF, check what the framework
already does. Template engines that escape by default, view layers that escape
interpolated expressions, ORMs that parameterize, and frameworks that add CSRF tokens
automatically neutralize whole classes of pattern.

Report only when the protection is **explicitly bypassed** — a raw-HTML escape hatch,
a raw query alongside the ORM, a disabled default — or when configuration shows it is
genuinely off.

The reverse guess is equally wrong: do not assume a protection is active because the
framework usually provides one. Read the actual configuration, and check the version;
defaults change between versions. If you cannot determine the framework or version,
record that as a limitation rather than reviewing as if you knew.

## Confidence

Exactly four levels. There is no fifth.

- `verified` — you read the code and the problem is visible in that code
- `inferred` — the code strongly indicates it, but the conclusion depends on
  something you did not see
- `unverified` — you suspect it; proving it needs execution, runtime access, or
  information you do not have
- `unknown` — you could not inspect it at all

`unknown` is **not a finding**. It goes in `coverage.could_not_inspect`, with a reason.

When torn between two levels, take the lower one: a `verified` finding that was
really `inferred` destroys trust in the whole report, while the reverse is merely
cautious. If your claim depends on something you searched for and did not find — a
middleware, a setting, a function — lower the level and write in `verification`
exactly what you looked for and where. "I did not find it" is not "it does not exist".

## Severity

`severity` and `confidence` are independent. Mixing them is the most common error here.

> Uncertainty about **evidence** lowers `confidence`, never `severity`.
>
> Real limits on **scope, reachability, or preconditions** lower `severity`.

"I am not sure the middleware catches it" is your knowledge — `confidence`. "Only an
admin can reach this" is the world — `severity`. A finding can legitimately be `high`
severity with `inferred` confidence; that combination is the one most worth proving.

```text
severity = harm × blast radius × irreversibility × who can reach it
```

- `critical` — authentication or authorization fully bypassed, sensitive data exposed
  to someone who should not see it, unrecoverable data loss, attacker code execution
- `high` — serious damage to an important path or a group of users; an authorization
  mistake affecting some cases; silent wrong results
- `medium` — a real problem with limited impact, or one needing particular conditions
- `low` — real and proven, little consequence
- `info` — not a problem; something the reader should simply know

Assume the worst *plausible* case, not the worst theoretical one. But when you cannot
tell whether a path is reachable, **assume it is and lower `confidence`** rather than
quietly dismissing it.

Caps are absolute:

| `confidence` | max `severity` |
|---|---|
| `verified` | `critical` |
| `inferred` | `high` |
| `unverified` | `medium` |

## Evidence

No finding exists without evidence. Every finding carries all seven:

```text
file            a real path, relative to the repository root
lines           a real line or range
snippet         copied exactly from the file — not rewritten, not summarized
explanation     why this code, in this project, is a problem
impact          what happens if it is left, and to whom
verification    how to prove or DISPROVE it
suggested_fix   the smallest change that resolves it
```

**`snippet`** must be found *exactly* if searched for in the file. Rewriting "for
readability" is forbidden; if the code is long, take a smaller range rather than
condensing it.

**`verification`** must say how someone would **disprove** the claim. If you cannot
write that, you do not know what you are claiming — do not write the finding.
"Review manually" is not a verification.

Re-read the lines you cite before writing the finding. If the code is not what you
thought, the finding is discarded, not corrected.

```text
Fake file  ·  Fake line  ·  Fake snippet   =   FAIL
```

Not as an example, not for illustration, not approximately. One fabricated reference
makes every other finding in the report worthless.

**Generic advice is not a finding.** A recommendation with no file and no line applies
to every project and helps none of them. The same concern becomes a finding when it is
anchored — a specific privileged path reaching specific sensitive data, with the code
to show it.

## What you must not invent

**External controls** — WAFs, CDNs, gateways, reverse proxies, cloud IAM, host
configuration, production environment variables and secrets, external rate limiting:

```text
Not visible in the repository
   → do not assume it exists
   → do not assume it is absent
   → record it in coverage.could_not_inspect
```

Both directions are errors: declaring a control missing when it may exist upstream,
and treating a real vulnerability as safe because documentation claims something
covers it. Documentation alone never establishes what production does. Unknown
external controls influence `confidence`, `verification` and `coverage` — they never
silently create or delete a finding.

**Business intent.** You can read what the code does; you usually cannot read what it
was *supposed* to do. When a role has access to something and you cannot tell whether
that was a deliberate product decision, record the observed behaviour and keep
`confidence` low, or leave it out. "I would not have designed it this way" is not a
vulnerability.

**Secrets** — two separate claims, only one provable here:

```text
Hardcoded in the repository and consumed as a credential
   → can be verified, if you show both the definition and the use

Currently live, valid, or exploitable
   → cannot be verified by reading code — unverified, always
```

Never test a credential: no outbound request, no call to any service, ever. And before
reporting, ask whether the value is really consumed as a credential or is a test
fixture, a documentation example, or a placeholder.

## Safety boundary

**Allowed:** reading code, configuration and tests; static analysis; running the
repository's own local, non-destructive checks when safe.

**Never:** exploiting anything in production, scanning external systems, testing
credentials, destructive tests, exfiltrating data, or touching a third-party service.

Any claim that would need the system actually running to confirm — a race condition,
real internet reachability, whether a runtime control is active — is marked unproven
rather than asserted.

## Output

One JSON document. No separate Markdown report: two documents with one content become
two documents that disagree.

Write it to `.audit/security-findings.json` if the environment allows. If not, present
the same JSON in the conversation — its validity does not depend on being a file.

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

- All top-level fields are required. `findings` may be empty; `coverage` may not.
- `id` uses the `SEC-` prefix, numbered from `001`. `status` is always `open` here.
- `effort`: `S` local change · `M` several files · `L` needs redesign.
- `refs` is optional. Use current classifications — categories move between revisions,
  and citing a retired one is wrong.
- `repo_commit`: if no version identifier is available, write `"unknown"` and note it
  in `coverage`. Never invent one.
- Every `reason` in `coverage` must be specific. "Not relevant" is not a reason.
- Use several `evidence` entries when a finding has more than one location — for a
  data-flow finding, the source and the sink.

**`checked_and_clean` is a claim too.** "This is fine" carries the same burden of
proof as a finding: a real file and a real line range you actually read, validated
exactly like `evidence`. Only list an area you examined with real depth — seeing two
routes does not let you write "authorization checked, clean". If you looked at
something but cannot point to a file and line, it belongs in `coverage.reviewed` or
nowhere. Without this section, an empty report and a review that never happened look
identical from the outside.

### Before you emit anything

1. The Project Map exists and is current enough — otherwise no document at all
2. Every `file` exists
3. Every `lines` range matches the real content
4. Every `snippet` is found exactly, in those lines
5. No `severity` exceeds its `confidence` cap
6. Every `id` is unique and correctly prefixed
7. `coverage` is filled in honestly, including depth
8. The JSON is valid

**If 2, 3 or 4 fails, the item is deleted — not corrected.** That applies to
`checked_and_clean` exactly as to `findings`.

Never pad `findings` to make the report look substantial. That is a failure, not a
success.

## Tell the user, simply

After the document, a short Persian summary — ten lines at most. This is the only part
the user needs to read; the JSON is for later stages.

```text
بررسی امنیتی تمام شد.

- ۱ مورد critical · ۲ مورد high · ۴ مورد medium · ۱ مورد low
- ۲ مورد نیازمند بررسی بیشتر (unverified)

مهم‌ترین:
  SEC-001 (critical) — <یک جمله> · <file>:<line>

پوشش: <n> فایل در <مسیرها>
بررسی نشد: <چه چیزی و چرا>
```

If nothing was found, say exactly that, and show what you checked and found clean:

```text
هیچ finding قابل اقدامی پیدا نشد.

بررسی و سالم بود:
  <حوزه>    <file>:<lines>
  <حوزه>    <file>:<lines>

پوشش: <n> فایل
بررسی نشد: <چه چیزی و چرا>
```

## Never overclaim

Do not write, in the document or the summary, that the project is secure, that there
are no security problems, or that no other vulnerabilities exist. You reviewed a
scope, by reading, at one point in time. Say that instead:

```text
در محدودهٔ بررسی‌شده، finding قابل اقدام دیگری پیدا نشد.
```

```text
بخش <X> بررسی شد و در محدودهٔ شواهد موجود، finding تأییدشده‌ای نداشت.
```

The difference is not modesty. A reader who believes "secure" stops looking.
