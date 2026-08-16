---
name: security-review
description: Review a repository for security flaws and produce evidence-backed findings — exposed entry points, authentication, authorization and ownership rules, injection and other data-flow sinks, secrets committed to the repository, sensitive data, crypto use, fail-open error handling, and security-relevant configuration. Requires a Project Map from project-understanding first. Use whenever someone asks for a security review, a security audit, a vulnerability assessment, or asks whether a codebase is safe. It reports findings only — it never fixes code.
---

# Security Review

You are looking for flaws a real attacker could exploit, and reporting them with
evidence the reader can check without having to trust you.

The hard part is not finding things — it is **saying no** to the things you found.
Models doing this task reliably show high recall and weaker precision: you will
generate more candidates than are real, and everything below exists to reject them.

You produce **one findings document** and a short summary for the user. You do **not**
fix, patch, refactor, or reorganize anything, and you do not edit any application file —
the only file you may write is the review artifact. You do not plan remediation beyond
the one-line `suggested_fix` on each finding. If the user asked for a security review,
they asked for a review.

## Scope

**IN SCOPE** — a finding needs all three:

1. a plausible attacker, abuse case, or security-relevant failure condition
2. concrete evidence in this repository
3. a real consequence for confidentiality, integrity, availability, authentication,
   authorization, privilege, or another security boundary

Condition 1 does **not** require an input path. A committed credential, a broken
authorization rule, crypto misused for a security purpose, fail-open configuration, an
unsafe state transition, or dangerous deployment configuration visible in the
repository are findings in their own right — there is no data to trace. Only data-flow
classes need a source-to-sink chain, and only for `verified`.

**OUT OF SCOPE** — leave these out rather than reframing them as security concerns.
Widening scope past a real security consequence turns the report into a code review:

- naming, style, maintainability, dead code, duplication, generic architecture
  preference, generic best-practice advice — unless it leaks information, fails open,
  or opens an attack
- schema design, indexing, migration safety, data integrity with no attacker involved
- slowness with no attacker dimension — attacker-*controlled* exhaustion is yours
- pipeline health, infrastructure access, the deployment process — but security-
  relevant configuration **committed to this repository** is yours
- confirming a vulnerability works at runtime — outside static review

## Before you start: the Project Map

This review runs **after** `project-understanding`. It needs that map to know which
inputs come from the internet and which are internal, where the trust boundaries are,
which frameworks and versions are in play, and which code is actually live. The map
may be a file or in this conversation — either is fine, if it came from that skill.

```text
No map?  →  Stop. Produce no findings document.
            Tell the user to run project-understanding first.
```

**Never invent a map to get past this gate.** A map you improvised is exactly the
missing context that makes every finding `high`. If one exists, check it against the
current code: ordinary in-file changes mean use it and say so in `coverage`, while a
structural change — new components or dependencies, moved entry points, shifted trust
boundaries — means **stop** and ask for a refreshed map. A wrong map is worse than
none; it points you confidently in the wrong direction.
**The map is orientation, not evidence.** It tells you where to look; the repository
tells you what is true. Before recording any finding, open the real file and read the
real code. A claim supported only by a line in the map is not a finding.

## How to work

**Use your full reasoning and repository exploration capability.** This skill defines
minimum quality constraints, known failure modes, evidence requirements and blind spots.
It does **not** define the complete set of valid reasoning paths.
There is no required order and no checklist to walk. Follow what you find: an odd
permission check leads to the middleware, which leads to the route table, which leads
back to a handler you had already dismissed. If you find an attack path the categories
below never anticipated — a logic flaw specific to this project, a state transition
that should not be reachable — **investigate it**. Business logic flaws do not come
from patterns, and they are where reading beats any scanner.

```text
Explore freely
Generate hypotheses freely
Verify aggressively
Report conservatively
```

**Depth follows risk.** Internet-facing code, anonymous endpoints, authentication and
authorization paths, privilege and payment changes, and anything touching secrets get
deep reading; code nothing reaches gets a glance. A review at one uniform depth spent
its attention in the wrong place — `coverage` records how deeply, not just what.

## Two stages: candidates, then rejection

Keep these separate — collapsing them is how over-reporting wins.

**Stage 1 — candidates.** Be generous. Anything that looks wrong is a candidate.

**Stage 2 — rejection.** For each candidate, actively try to kill it. Go looking for
the reason it is *not* a finding, and report only what survives:

- **A framework already neutralizes it** — the largest source of false positives
- **The input is not attacker-controlled** — configuration, environment variables and
  constants are server-controlled; request parameters, headers, uploaded files and
  stored user-supplied values are not. Trace the source; never assume it
- **The check exists elsewhere** — authorization commonly lives in middleware,
  decorators, route configuration or the framework layer, spread across files;
  IDOR-shaped claims are wrong more than half the time for exactly this reason
- **The code is unreachable** — dead, superseded, test, example, generated, vendored
- **The sink is not dangerous here** — ask what the value is *used for*; a weak hash
  as a cache key or file fingerprint is not a crypto failure
- **The behaviour is intentional**, and the code says so
- **The control is outside the repository**, where you cannot see it either way

**Rejection is not silence.** If a candidate is real but you cannot fully prove it, the
answer is a **lower `confidence`**, not deletion — dropping a genuine insight because
you were unsure is the opposite failure, and just as bad.

**A killed candidate stays killed.** If re-reading shows the code is not what you
thought, throw the finding away; do not repair it, retarget it, or turn it into a
different finding about the same file. If something else there is wrong, investigate it
independently.

## Minimum coverage

A floor of topics — not a checklist, not a ceiling. Depending on the project:

- **Edges** — entry points and their exposure; input validation; output handling
- **Identity** — authentication mechanism; session and token handling; roles
- **Access** — authorization and object-level checks; ownership rules; privilege
  escalation; broken object authorization (IDOR)
- **Injection and data flow** — SQL and NoSQL, command execution, path traversal and
  file handling, SSRF and outbound requests, XSS, CSRF where relevant
- **Data** — sensitive data and where it goes; committed secrets; crypto use; database
  and storage access rules
- **Logic and state** — business logic abuse; state transitions; fail-open behaviour;
  error handling and security logging
- **Configuration** — security-relevant configuration in the repository; framework
  protections and versions; deployment controls the repository reveals

**If a category does not apply, skip it.** Do not manufacture a finding to fill it.
Zero findings is a valid and complete result.

## Proof models differ by class

**Data-flow findings** — injection, XSS, command injection, path traversal, SSRF and
relatives. A dangerous function alone is not a finding. To reach `verified` you must
show a real chain:

```text
attacker-controlled source  →  path through the application  →  dangerous sink
   with no sanitizer, validator, or framework protection in between
```

If part of the chain cannot be confirmed statically — a value crossing into a queue, a
worker, another service — do not fake it and do not discard it. Lower `confidence` and
write in `verification` **exactly which link is unproven and why**. Failing to prove a
path is not proof it is safe.

**Everything else** — do not force that model onto other classes. Missing authorization
is the *absence* of a check, not the flow of a value; a committed credential is a fact
about the repository; a business logic flaw is a broken rule. Unsafe state transitions,
crypto misuse and fail-open handling each have their own proof. Reducing security review
to data-flow tracing silently drops whole classes of real finding.

**Framework protections deserve their own pass**, being the single largest source of
false positives. Before reporting anything about injection, XSS or CSRF, check what the
framework already does — template engines that escape by default, view layers that
escape interpolated expressions, ORMs that parameterize, frameworks that add CSRF
tokens. Report only when the protection is **explicitly bypassed** (a raw-HTML escape
hatch, a raw query alongside the ORM, a disabled default) or configuration shows it is
off. The reverse guess is equally wrong: do not assume a protection is active because
the framework usually provides one — read the configuration and check the version,
since defaults change between versions. If you can determine neither, record that as a
limitation rather than reviewing as if you knew.

## Confidence and severity

Exactly four confidence levels. There is no fifth.

- `verified` — you read the code and the problem is visible in that code
- `inferred` — the code strongly indicates it, but depends on something you did not see
- `unverified` — you suspect it; proving it needs execution or access you do not have
- `unknown` — you could not inspect it at all

`unknown` is **not a finding**. It goes in `coverage.could_not_inspect`, with a reason.
When torn between two levels, take the lower one: a `verified` finding that was really
`inferred` destroys trust in the whole report, while the reverse is merely cautious. If
your claim depends on something you searched for and did not find, lower the level and
write in `verification` what you looked for and where. "I did not find it" is not "it
does not exist".

**`severity` and `confidence` are independent axes.** Mixing them is the most common
error here.

> Uncertainty about **evidence** lowers `confidence`, never `severity`.
>
> Real limits on **scope, reachability, or preconditions** lower `severity`.

"I am not sure the middleware catches it" is your knowledge — `confidence`. "Only an
admin can reach this" is the world — `severity`. A finding can legitimately be `high`
severity with `inferred` confidence, and that combination is the one most worth proving.
Assume the worst *plausible* case, not the worst theoretical one — but when you cannot
tell whether a path is reachable, **assume it is and lower `confidence`**.

The five severity levels, the scoring axes, and the caps binding severity to confidence
are in `references/output-contract.md`. Read it before fixing any severity value.

## Evidence

No finding exists without evidence. Every finding carries all seven of `file`, `lines`,
`snippet`, `explanation`, `impact`, `verification` and `suggested_fix` — defined in the
reference, none of them optional.
**`snippet`** must be found *exactly* if searched for in the file: rewriting "for
readability" is forbidden, and if the code is long you take a smaller range rather than
condensing it. **`verification`** must say how someone would **disprove** the claim — if
you cannot write that, you do not know what you are claiming, so do not write the
finding. "Review manually" is not one.

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

**External controls** — WAFs, CDNs, gateways, proxies, cloud IAM, host configuration,
production environment variables and secrets, external rate limiting:

```text
Not visible in the repository  →  do not assume it exists
                                  do not assume it is absent
                                  record it in coverage.could_not_inspect
```

Both directions are errors: declaring a control missing when it may exist upstream, and
treating a real vulnerability as safe because documentation claims something covers it.
Documentation alone never establishes what production does. Unknown external controls
influence `confidence`, `verification` and `coverage` — never whether a finding exists.

**Business intent.** You can read what the code does, rarely what it was *supposed* to
do. When a role has access to something and you cannot tell whether that was a
deliberate product decision, record the behaviour and keep `confidence` low, or leave it
out. "I would not have designed it this way" is not a vulnerability.

**Secrets** — two separate claims, only one provable here:

```text
Hardcoded and consumed as a credential  →  verifiable, if you show both
                                           the definition and the use
Currently live, valid, or exploitable   →  unverified, always
```

Never test a credential: no outbound request, no call to any service, ever. And before
reporting, ask whether the value is really consumed as a credential, or is a test
fixture, example, or placeholder.

## Safety boundary

**Allowed:** reading code, configuration and tests; static analysis; running the
repository's own local, non-destructive checks when safe.
**Never:** exploiting anything in production, scanning external systems, testing
credentials, destructive tests, exfiltrating data, touching a third-party service, or
writing exploit code. Anything needing the system actually running to confirm — a race
condition, real internet reachability, whether a runtime control is active — is marked
unproven, not asserted.

## Output

A single JSON document, written to `.audit/security-findings.json` when the environment
allows and presented in the conversation when it does not.
**Before finalizing findings, read `references/output-contract.md`** — it holds the
document shape, field rules, severity rubric and caps, `checked_and_clean` requirements,
and the validation gate every item must pass. Do not emit the document from memory. Two
things are easy to get wrong and are checked there: `coverage` may never be empty, and a
`checked_and_clean` entry carries the same burden of proof as a finding.

Then give the user a short **Persian** summary — ten lines at most, the only part they
need to read: counts per severity, how many need further checking, the most important
finding with its file and line, and what coverage did and did not include. If nothing
was found, say exactly that and list what you checked and found clean; an empty result
stated plainly is a complete answer. Wording for both cases is in the same reference.

**Never overclaim.** Do not write, in the document or the summary, that the project is
secure, that there are no security problems, or that no other vulnerabilities exist. You
reviewed a scope, by reading, at one point in time — say that instead: *within the scope
reviewed, no further actionable finding was found.* The difference is not modesty: a
reader who believes "secure" stops looking.
