# Output Contract — project-understanding

Read this before finalizing the Project Map. It holds the document shape, the claim
format, the language rule, and the presentation rules for the map and the summary.

It changes no rule in `SKILL.md` — the evidence, confidence, unknown and compression
rules all live there and are unaffected by anything here.

---

## 1. The document

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

Keep the skeleton, but never treat a heading as a quota. A section that needs one line
gets one line, and a section that genuinely does not apply to this project says so
briefly rather than being filled with empty text.

`repo_version` is `unknown` when no identifier is available. Never fabricate one.

## 2. The shape of a claim

```markdown
- **<the claim>** · `<confidence>`
  <file> · `<anchor>`
  <optional note, or where you looked if unknown>
```

The note on the third line is optional, and for most claims it is left out. Add it
only when it would change the reader's conclusion — an exception, a boundary, or where
you looked for something you could not find.

One line of evidence is usually enough:

```markdown
- **Protected pages check the session before rendering** · `verified`
  `<guard component>` · `<the session lookup and redirect>`
```

An `unknown` claim takes the same shape, with the note recording where you searched in
place of an anchor:

```markdown
- **Production rate limiting** · `unknown`
  Looked in `<middleware directory>` and the route configuration; found no
  application-level control. It may exist at gateway level, which is outside
  this repository.
```

A grouped claim keeps its exception separate rather than folding it in:

```markdown
- **12 admin routes, all behind the same role check** · `verified`
  `<routes file>` · `<the shared check>`
- **Exception: one of them has no role check** · `verified`
  `<handler file>` · `<what runs instead>`
```

## 3. Language

Write the map in Persian, keeping file paths, code, framework names and the four
confidence values in their original form.

Technical terms whose translation costs precision stay in English —  authentication,
authorization, middleware, repository, entry point. Avoid unstructured mixing of the
two languages: either a term is a technical token and stays English in backticks where
appropriate, or it is prose and is Persian.

## 4. Storage

The map is logically one Markdown document. Write it to `.audit/project-map.md` if the
environment allows it. If it does not, present the same structure in the conversation —
its validity does not depend on being a file.

Do not produce a second copy in another format. One map, one place.

## 5. Presentation

The map and the summary are read by a person, so they must be scannable. These are the
runtime subset of the project's shared presentation contract.

- Short paragraphs, two to four lines, one idea each. A wall of text is a defect even
  when every claim in it is correct
- Bullets for sets of items, with at most one level of nesting
- Backticks for every file path, symbol, route, field and literal value
- Bold only where it genuinely matters; when everything is bold, nothing is
- Code blocks only for code, a small structural sketch, or a snippet — never as a
  decorative box around prose
- Tables only where a comparison is genuinely clearer as a grid
- Meaningful whitespace between independent sections
- At most three heading levels, and no heading for two sentences

## 6. The user summary

After the map, a short plain-language summary. A few lines. The user should not need to
understand the map's internal structure to read it.

```text
پروژه را بررسی کردم.

- Backend: <framework> · Frontend: <framework> · Database: <engine>
- <n> نقش کاربری و <n> سرویس خارجی پیدا شد
- <n> مسیر API، که <n> تای آن‌ها بدون احراز هویت در دسترس‌اند
- چند مورد مربوط به production قابل تأیید نبود

Project Map آماده است.
```

If something important could not be determined, say that here too — briefly, in one
line. It is the single most useful thing the user can know before asking for a
specialist review.
