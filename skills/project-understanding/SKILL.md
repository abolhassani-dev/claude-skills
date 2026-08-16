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
- a repository report — a document that walks through everything the repository
  contains
- onboarding documentation for a new teammate
- a claim to *understand* the project

The last one matters. A program's intent cannot be recovered from its text alone.
You are building a map, and the map should never pretend to be more than that.

## The inclusion test

Compactness is not a style preference. Irrelevant context measurably degrades
reasoning, so a bloated map recreates the problem it exists to solve.

Before you write any item into the map, ask one question:

```text
If I removed this, could the next reviewer misunderstand the project
or reach a different conclusion?
```

`yes` → keep it. `no` → drop it, or fold it into a group.

Apply this to **every** item, including the ones the coverage floor below asks for.
Covering a topic can be a single line.

**There is no numeric limit** — not on words, lines, or tokens. Size follows the
repository's complexity, so a small repository should naturally produce a short map.
The goal is not the shortest possible document; it is maximum useful context with
minimum unnecessary detail. A human should be able to understand the shape of the
project from it in a few minutes.

### Compress repetition, preserve exceptions

Group similar things instead of listing them one by one — one claim for the group,
then the exception on its own. The reference shows the shape.

**Never compress away exceptions.** If one item in a group behaves differently, it
gets its own entry, in full. The exception is usually the most valuable line in the
map: *every data store is access-controlled except one, which is publicly readable*
is worth more than the stores that behave identically.

Never compress away important relationships, sensitive data paths, trust
boundaries, or structure either.

### Detail on demand

The map carries what a reviewer needs to **orient** — not everything a reviewer
might one day want to know.

```text
Global map   → compact
Reviewer     → opens the relevant file when deeper detail is needed
```

So do not pre-explain implementation detail. Spend words on a detail only when it
changes the architecture, explains important system behaviour, is an exception, is
a trust boundary or a sensitive flow, or would change the next reviewer's
conclusion.

For a large repository the same rule applies at a larger scale: a compact global
map, plus targeted detail only where it is actually needed. If the repository is
too large to inspect fully, say so in coverage — honestly and specifically.

### Code quality is not part of the map

Unused imports, dead helpers, naming, formatting, and minor duplication do not
belong here. They are work for later reviewers. Mention such a thing only when it
genuinely changes how the architecture or the system's behaviour is understood.

## How to work

**Use your full reasoning and repository exploration capability.**

This skill defines minimum quality constraints, evidence requirements, and known
failure modes. It does **not** define the complete set of valid reasoning paths.

There is no fixed order. Do not work through a checklist like
`README → manifest → routes → database`. Real comprehension is not linear: follow
what you find, form a hypothesis, chase it, abandon it when it does not hold, and
come back to it later if something else points that way.

If you discover a pattern, dependency, or relationship that matters for
understanding this project and is not listed below, **investigate it** — and record
it if it passes the inclusion test. Do not ignore useful evidence merely because
this skill did not anticipate it. Investigating widely and reporting narrowly is
the intended shape: what you learn tells you where to look next even when it never
reaches the map.

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

It is a floor of **topics, not of detail**. Each topic must still earn its length
through the inclusion test — for a small project, most of these are one or two
lines each, and some are a single `unknown`.

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

**Evidence gets shorter, never dropped.** The evidence requirement is absolute; a
paragraph explaining each piece of evidence is not. File plus anchor *is* the
evidence, and one line is usually enough — the reference shows the exact shape.

Add a note under a claim only when the note itself would change the reader's
conclusion — an exception, a boundary, or where you looked for something you could
not find. Never trade evidence for brevity: the way to shorten the map is to carry
fewer claims, not to carry claims with weaker support.

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

An `unknown` claim carries the same shape as any other, with the note recording where
you searched instead of an anchor. The reference shows it.

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

## Output

The Project Map is a single Markdown document, written to `.audit/project-map.md` when
the environment allows and presented in the conversation when it does not — its validity
does not depend on being a file.

**Before finalizing the map, read `references/output-contract.md`.** It holds the
document header, the section skeleton, the shape every claim takes, the language rule,
and the presentation rules for both the map and the summary. Do not produce the map from
memory or from a layout you invented.

Two things worth knowing in advance: `repo_version` is `unknown` when no identifier is
available and is **never fabricated**, and the map is written in Persian, keeping file
paths, code, framework names and the four confidence values in their original form.

Keep the map scannable — short paragraphs, bullets for sets with at most one level of
nesting, backticks for paths and symbols, restrained emphasis, and code blocks only for
code. A wall of text is a defect even when every claim in it is correct.

## Tell the user, simply

After the map, give the user a short plain-language summary in the shape the reference
defines. A few lines. They should not need to understand the map's internal structure.

If something important could not be determined, say that here too — briefly, in one
line. It is the single most useful thing the user can know before asking for a
specialist review.
