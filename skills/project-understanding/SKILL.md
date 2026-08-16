---
name: project-understanding
description: Build a compact, evidence-backed Project Map of a repository — what it does, its stack and frameworks, entry points and how exposed they are, data stores, external services, authentication, roles, trust boundaries, and everything that could not be determined. Use this whenever someone asks you to review, audit, analyse, or explain a repository or codebase, and always before any specialist review such as security, architecture, or database, because those depend on this map. Also use it for questions like "what is this project?" or "explain this codebase to me".
---

# Project Understanding

You are building a **Project Map**: a description of what a repository actually is,
so that you — and any specialist reviewer that runs after you — can reason about it
without rediscovering it from scratch.

## What you are producing

A map, not a copy.

```text
Project Map = map of the repository
not a copy of the repository
```

You are **not** producing:

- a list of problems — you report no findings, no severities, no recommendations
- onboarding documentation for a new teammate
- a claim to *understand* the project

The last one matters. A program's intent cannot be recovered from its text alone.
You are building a map, and the map should never pretend to be more than that.

## How to work

**Use your full reasoning and repository exploration capability.**

This skill defines minimum quality constraints, evidence requirements, and known
failure modes. It does **not** define the complete set of valid reasoning paths.

There is no fixed order. Do not work through a checklist like
`README → manifest → routes → database`. Real comprehension is not linear: follow
what you find, form a hypothesis, chase it, abandon it when it does not hold, and
come back to it later if something else points that way.

If you discover a pattern, dependency, or relationship that matters for
understanding this project and is not listed below, **investigate it and record
it**. Do not ignore useful evidence merely because this skill did not anticipate it.

What is *not* free is the order of evidence: you cannot interpret imports before
you know what the manifest declares. Data dependencies are real even when
cognitive ones are not.

The discipline is:

```text
Explore freely
Generate hypotheses freely
Verify aggressively
Report conservatively
```

Forming a hypothesis is free. Turning it into a recorded claim is not — that
requires evidence, or an honest confidence level.

## Minimum coverage

These are the things the map should not silently omit. They are a **floor, not a
ceiling**. Add anything else that matters for this particular project.

**A — What the project is**

- What it does and for whom
- Languages, frameworks and **their versions**, per component, and which default
  protections those frameworks bring
- Components, deployable units, module boundaries, and which parts are actually live

**B — What crosses its edges**

- Entry points, and for each one how exposed it is:
  `public-anonymous` / `public-authed` / `internal` / `admin` / `unknown`
- Data stores and whether access goes through an ORM, raw queries, or both
- External services and outbound calls
- Sensitive data and where it lives

**C — Who can do what**

- The authentication mechanism in use
- Roles, and the intended authorization and ownership rules
- Trust boundaries — where data crosses from one level of trust to another

**D — What lies outside view**

- Controls handled outside the application: gateway, proxy, WAF, service mesh,
  rate limiting, TLS termination
- Coverage: what you inspected, what you skipped and why, what you could not inspect

Section D is not optional. A map that does not state its own boundary misleads its
reader, because the reader assumes everything was seen.

Before counting or concluding anything about languages and dependencies, exclude
vendored, generated, and documentation files. They distort every count.

## Evidence

Every claim that matters carries evidence, or it is marked `unknown`. There is no
third option.

```text
file      the real path
anchor    an exact copied snippet, or a stable symbol name
lines     optional — a shortcut, not the proof
```

**The anchor is how evidence is found. The line number is only a hint.** If the
line no longer matches, the evidence is not lost — search for the anchor. If the
anchor cannot be found either, that claim has gone stale and needs rechecking.

The anchor must be copied exactly from the file. Never paraphrase it.

Never invent a file, a line, or a snippet. Not as an example, not for
illustration, not approximately.

### Confidence

Use exactly these four levels — no others:

- `verified` — you read the code and the fact is visible in the code itself
- `inferred` — the code strongly indicates it, but the conclusion depends on
  something you did not see
- `unverified` — you suspect it; confirming it needs runtime access or information
  you do not have
- `unknown` — you could not inspect it at all

When torn between two levels, always choose the lower one. A `verified` claim that
was really `inferred` destroys trust in the whole map; the reverse is merely cautious.

### Documentation is a clue, not a fact

A claim whose only evidence is documentation — `README`, a `docs/` folder, a
comment — can never be `verified`. At most `unverified`, and the text must say
that documentation is its source.

Documented architecture and implemented architecture drift apart. That gap is
normal, not exceptional.

When code and documentation disagree, record both separately: what the code shows,
and what the documentation claims. Do not silently pick one.

## Unknowns

`unknown` is a valid and often correct outcome. It is not a failure.

When something cannot be determined:

- do not guess
- do not declare it absent
- say where you looked
- say what would be needed to confirm it

```markdown
- **Production rate limiting** · `unknown`
  Looked in `src/middleware/` and the route configuration; found no
  application-level control. It may exist at gateway level, which is outside
  this repository.
```

"I did not find it" and "it does not exist" are different statements. Never let
the first become the second.

This cuts both ways for controls outside the repository. Not seeing a WAF does not
mean there is none; a `README` claiming there is one does not make it real. The
honest answer is `unknown`, recorded as such.

Some things are structurally invisible to code reading and will often stay
`unknown`: the *intended* authorization and ownership rules, production
configuration, deployment-level controls, and whether a given entry point is
genuinely reachable from the internet. Record what the code shows, record what you
inferred, and record what is simply missing — as three separate things.

## Keeping the map compact

The map is compact by default. This is not a style preference: irrelevant context
measurably degrades reasoning, so a bloated map recreates the problem it exists to
solve.

**Compress repetition.** Group similar things instead of listing them one by one.

```markdown
### Transfer endpoints
- **18 routes, all requiring authentication** · `verified`
  `transfers/urls.py` · `router.register(r'transfers', TransferViewSet)`
- **Sensitive operations in this group:** create · transfer ownership · cancel
```

**Never compress away:**

- **exceptions** — if one of those 18 routes behaves differently, it gets its own
  entry, in full. The exception is usually the most valuable line in the map.
- important relationships, sensitive paths, trust boundaries, structure

**The test:** if removing an item would lead the next reviewer to a *different*
conclusion, keep it. If removing it only makes the document shorter, drop it.

For a large repository: a compact global map, plus targeted detail only where it
is actually needed. If the repository is too large to inspect fully, say so in
coverage — honestly and specifically.

## Output

A single Markdown document with a short header:

```markdown
---
repo: <name>
repo_version: <commit or version identifier, or `unknown`>
generated_at: <YYYY-MM-DD>
---

# Project Map — <name>

## A. پروژه چیست
### A1 — هویت و هدف
### A2 — پشته و فریم‌ورک‌ها
### A3 — ساختار و اجزا

## B. چه چیزی وارد و خارج می‌شود
### B1 — نقاط ورود
### B2 — ذخیره‌سازی داده
### B3 — سرویس‌های بیرونی
### B4 — داده‌های حساس

## C. چه کسی چه کاری می‌تواند بکند
### C1 — احراز هویت
### C2 — نقش‌ها و مجوزدهی
### C3 — مرزهای اعتماد

## D. چه چیزی بیرون از دید است
### D1 — محیط و کنترل‌های بیرونی
### D2 — پوشش و محدودیت‌ها
```

Each claim takes this shape:

```markdown
- **<the claim>** · `<confidence>`
  <file> · `<anchor>`
  <optional note, or where you looked if unknown>
```

If a section genuinely does not apply to this project, say so briefly rather than
filling it with empty text.

**Language:** write the map in Persian, keeping file paths, code, framework names,
and the four confidence values in their original form.

**If `repo_version` is not available, write `unknown`.** Never fabricate an
identifier.

**Storage:** the map is logically one Markdown document. Write it to
`.audit/project-map.md` if the environment allows it. If it does not, present the
same structure in the conversation — its validity does not depend on being a file.

## Tell the user, simply

After the map, give the user a short plain-language summary. A few lines. They
should not need to understand the map's internal structure.

```text
پروژه را بررسی کردم.

- Backend: Django · Frontend: React · Database: PostgreSQL
- ۳ نقش کاربری و ۲ سرویس خارجی پیدا شد
- ۲۳ مسیر API، که ۴ تای آن‌ها بدون احراز هویت در دسترس‌اند
- چند مورد مربوط به production قابل تأیید نبود

Project Map آماده است.
```

If something important could not be determined, say that here too — briefly, in
one line. It is the single most useful thing the user can know before asking for a
specialist review.
