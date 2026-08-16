# Findings Schema

> **قرارداد مشترک**
>
> ساختار خروجی همهٔ Reviewerها.
> قبل از تولید سند خروجی، `trust-model.md` را هم بخوان — آن سند بر این یکی اولویت دارد.

---

## 1. یک سند، یک منبع حقیقت

هر Reviewer دقیقاً **یک سند JSON** تولید می‌کند.

این قرارداد **مستقل از محل نگهداری** است. سند همان سند است، چه در فایل نوشته شود،
چه در گفت‌وگو بماند.

اگر نوشتن در repository ممکن بود، محل طبیعی این است:

```text
.audit/<skill-name>-findings.json
```

برای نمونه: `.audit/security-findings.json`

اگر ممکن نبود، سند در همان گفت‌وگو نگه داشته می‌شود تا مراحل بعد از آن استفاده کنند.
این وابسته به `CR-2` در `BLUEPRINT.md` است.

گزارش Markdown جداگانه تولید **نمی‌شود**.
دلیل: دو سند با یک محتوا یعنی دو نسخه که از هم جدا می‌افتند.

به‌جای آن، همیشه یک خلاصهٔ کوتاه فارسی برای کاربر ارائه می‌شود (بخش 6).

**زبان:** محتوای JSON انگلیسی است — چون ساختاریافته و ماشین‌خوان است و اصطلاحات فنی
در انگلیسی دقیق‌ترند. فقط خلاصهٔ کاربر فارسی است.

---

## 2. ساختار کامل

```json
{
  "skill": "security-review",
  "repo": "my-project",
  "repo_commit": "a3f91c2",
  "generated_at": "2026-08-16",
  "project_map": {
    "used": true,
    "repo_commit": "a3f91c2",
    "staleness": "current"
  },

  "coverage": {
    "reviewed": ["src/api", "src/auth", "src/db"],
    "file_count": 68,
    "not_reviewed": [
      {
        "path": "migrations/legacy",
        "reason": "Superseded by the 2024 rewrite; no longer executed."
      }
    ],
    "could_not_inspect": [
      {
        "item": "production configuration",
        "reason": "Lives outside the repository."
      }
    ]
  },

  "checked_and_clean": [
    {
      "area": "Session cookie configuration",
      "file": "src/auth/session.ts",
      "lines": "18-31",
      "note": "httpOnly and secure flags are enabled"
    }
  ],

  "findings": [
    {
      "id": "SEC-001",
      "title": "User-controlled path passed to fs.readFile without normalization",
      "severity": "high",
      "confidence": "verified",
      "status": "open",
      "effort": "S",

      "evidence": [
        {
          "file": "src/api/files.ts",
          "lines": "42-47",
          "snippet": "const p = req.query.path;\nres.send(await fs.readFile(p));"
        }
      ],

      "explanation": "The `path` query parameter reaches fs.readFile with no normalization or root check. The route is registered on the public router in src/api/index.ts:12, so the value is attacker-controlled.",
      "impact": "Any unauthenticated caller can read arbitrary files the service process can access, including configuration and key material.",
      "verification": "Send GET /api/files?path=../../package.json and check whether the file contents are returned instead of an error.",
      "suggested_fix": "Resolve the path against a fixed root directory and reject anything that escapes it, before the read.",

      "refs": ["CWE-22"]
    }
  ]
}
```

---

## 3. فیلدهای سطح بالا

همهٔ این فیلدها اجباری هستند.

- `skill` — نام Skill تولیدکننده
- `repo` — نام repository
- `repo_commit` — شناسهٔ نسخهٔ کد موقع بازرسی. بدون این، گزارش قابل بازتولید نیست.
  اگر شناسه در دسترس نبود، مقدار `"unknown"` بگذار و در `coverage` ذکر کن —
  هرگز چیزی از خودت نساز. وابسته به `CR-3` در `BLUEPRINT.md`.
- `generated_at` — تاریخ به شکل `YYYY-MM-DD`
- `project_map` — وضعیت نقشهٔ پروژه (بخش 3.1)
- `coverage` — چه چیزی دیده شد و چه چیزی نه (بخش 3.2)
- `checked_and_clean` — چه چیزی بررسی شد و سالم بود (بخش 3.3)
- `findings` — آرایه. می‌تواند خالی باشد.

### 3.1 فیلد `project_map`

نشان می‌دهد بازرسی با چه اطلاعاتی از پروژه انجام شده:

```json
{
  "used": true,
  "repo_commit": "a3f91c2",
  "staleness": "current"
}
```

مقدار `staleness` یکی از این سه است:

- `current` — شناسهٔ نسخهٔ نقشه با وضعیت فعلی پروژه یکی است
- `stale-minor` — فرق دارد، ولی فقط تغییرات معمولیِ داخل فایل‌ها؛ نقشه هنوز معتبر است
- `stale-structural` — تغییر ساختاری رخ داده و نقشه باید refresh شود

اگر `stale-structural` بود، به کاربر بگو `project-understanding` را دوباره اجرا کند
و بازرسی را متوقف کن.

بازرسی با نقشهٔ غلط بدتر از بازرسی بدون نقشه است.

#### نقشه در Phase 1 اجباری است

اگر نقشهٔ پروژه در دسترس نبود، بازرسی شروع نمی‌شود:

```text
نقشهٔ پروژه نیست؟
   ↓
متوقف شو. اول پروژه باید فهمیده شود.
هیچ سند خروجی تولید نکن.
```

دلیل: در معماری Phase 1، `project-understanding` عمداً قبل از Reviewerها قرار گرفته.

یک Reviewer بدون نقشه نمی‌داند کدام ورودی از اینترنت می‌آید و کدام داخلی است،
کجا مرزهای اعتماد هستند، و کدام بخش‌ها اصلاً در مسیر اجرا قرار دارند.
نتیجه‌اش گزارشی است که همه‌چیز را `high` می‌کند — دقیقاً همان نویزی که این پروژه
برای حذفش ساخته شده.

> **مقدار `"used": false` در Phase 1 معتبر نیست.**
>
> این حالت برای Reviewerهای آینده‌ای نگه داشته شده که نقشه برایشان واقعاً اختیاری است.
> هیچ‌کدام از سه Skill فعلی چنین نیستند.

### 3.2 فیلد `coverage`

این بخش اختیاری نیست و خالی گذاشتنش تخلف است.

گزارشی که مرز خودش را نگوید، خواننده را فریب می‌دهد — چون فرض می‌کند همه‌چیز دیده شده.

- `reviewed` — مسیرهایی که واقعاً بررسی شدند
- `file_count` — تعداد فایل‌های بررسی‌شده
- `not_reviewed` — چیزی که عمداً بررسی نشد، با دلیل
- `could_not_inspect` — چیزی که قابل بررسی نبود (سطح `unknown` در trust model)، با دلیل

مقدار `reason` باید مشخص باشد. «مرتبط نبود» دلیل نیست.

### 3.3 فیلد `checked_and_clean`

چیزهایی که با دقت بررسی شدند و مشکلی نداشتند.

این بخش نشان می‌دهد بازرسی جدی بوده. بدون آن، گزارشِ خالی و بازرسیِ انجام‌نشده
از بیرون یکسان به نظر می‌رسند.

هر عضو یک شیء است، نه یک رشته — و باید ارجاع واقعی به فایل داشته باشد:

```json
{
  "area": "Session cookie configuration",
  "file": "src/auth/session.ts",
  "lines": "18-31",
  "note": "httpOnly and secure flags are enabled"
}
```

هر چهار فیلد اجباری‌اند:

- `area` — چه چیزی بررسی شد
- `file` — فایلی که واقعاً خواندی
- `lines` — خطوطی که تصمیمت بر آن استوار است
- `note` — چه دیدی که باعث شد بگویی سالم است

**چرا اجباری:** ادعای «سالم است» هم یک ادعاست و همان بار اثبات را دارد که یک finding.

بدون این قاعده می‌شد نوشت «احراز هویت بررسی شد و مشکلی نداشت» بدون اینکه حتی یک فایل
باز شده باشد — و این دقیقاً همان چیزی است که `trust-model.md` ممنوع کرده، فقط با علامت مخالف.

فیلدهای `file` و `lines` اینجا دقیقاً مثل `evidence` اعتبارسنجی می‌شوند (بخش 5).

اگر حوزه‌ای را بررسی کردی ولی نمی‌توانی به فایل و خط مشخصی اشاره کنی،
آن حوزه در `checked_and_clean` نمی‌آید — یا در `coverage.reviewed` است، یا هیچ‌جا.

---

## 4. فیلدهای هر finding

### فیلدهای اجباری

- `id` — به شکل `<PREFIX>-<NNN>`
- `title` — یک جمله، مشخص، بدون کلی‌گویی
- `severity` — یکی از `critical` / `high` / `medium` / `low` / `info`
- `confidence` — یکی از `verified` / `inferred` / `unverified`
- `status` — در ابتدا `open`
- `effort` — یکی از `S` / `M` / `L`
- `evidence` — آرایه، حداقل یک عضو
- `explanation` — چرا این کد، در این پروژه، مشکل است
- `impact` — اگر رها شود چه می‌شود و برای چه کسی
- `verification` — چطور اثبات یا رد شود
- `suggested_fix` — کمترین تغییر ممکن

### فیلدهای اختیاری

- `refs` — ارجاعات بیرونی، برای نمونه `["CWE-22"]`
- `merged_from` — فقط planner این را می‌گذارد

### 4.1 پیشوند دامنه در `id`

| Skill | پیشوند |
|---|---|
| `security-review` | `SEC-` |
| `architecture-review` | `ARCH-` |
| `database-review` | `DB-` |
| `code-quality-review` | `CQ-` |
| `performance-review` | `PERF-` |
| `qa-testing-review` | `QA-` |
| `devops-review` | `OPS-` |
| `accessibility-review` | `A11Y-` |

شماره‌گذاری از `001` در هر گزارش شروع می‌شود.

پیشوند باعث می‌شود بعد از ادغام چند گزارش، منشأ هر مورد معلوم بماند.

### 4.2 آرایهٔ `evidence`

هر عضو این شکل را دارد:

```json
{
  "file": "src/api/files.ts",
  "lines": "42-47",
  "snippet": "const p = req.query.path;\nres.send(await fs.readFile(p));"
}
```

- `file` — مسیر نسبت به ریشهٔ repository؛ باید واقعاً وجود داشته باشد
- `lines` — به شکل `"42"` یا `"42-47"`
- `snippet` — عیناً از فایل؛ اگر جستجویش کنی باید دقیقاً پیدا شود

اگر یک finding چند نقطه دارد (برای نمونه مبدأ و مقصد یک مسیر داده)، چند عضو بگذار.

### 4.3 چرخهٔ `status`

Reviewer همیشه `open` می‌گذارد. بقیه را مراحل بعدی تغییر می‌دهند:

```text
open  ──→  confirmed  ──→  fixed
  │
  ├──────→  rejected     (بررسی شد، مشکل نبود)
  └──────→  deferred     (واقعی است، فعلاً حل نمی‌شود)
```

### 4.4 مقدار `effort`

تخمین کار اصلاح:

- `S` — تغییر موضعی
- `M` — چند فایل
- `L` — نیاز به بازطراحی

---

## 5. قواعد اعتبارسنجی

قبل از تولید سند، این موارد را چک کن.

**گام صفر:** نقشهٔ پروژه وجود دارد. اگر نه، اصلاً سندی تولید نکن (بخش 3.1).

در `evidence` و `checked_and_clean` هر دو:

1. هر `file` واقعاً وجود دارد
2. هر `lines` با محتوای واقعی فایل می‌خواند

فقط در `evidence`:

3. هر `snippet` عیناً در آن خطوط پیدا می‌شود

بقیه:

4. مقدار `severity` از سقفِ `confidence` رد نشده (`severity-rubric.md`)
5. مقادیر `id` یکتا هستند و پیشوند درست دارند
6. فیلد `coverage` پر است
7. خروجی JSON معتبر است

> **اگر مورد 1، 2 یا 3 رد شود، آن مورد حذف می‌شود — نه اینکه اصلاح شود.**
>
> این برای `checked_and_clean` هم صدق می‌کند: ارجاعِ نادرست در ادعای «سالم است»
> همان‌قدر جدی است که در یک finding.

### آرایهٔ خالی معتبر است

```json
{ "findings": [] }
```

گزارشی با صفر finding و `coverage` صادقانه و `checked_and_clean` پر،
یک گزارش کامل و خوب است.

آرایهٔ `findings` را برای پرکردن گزارش پر نکن. این شکست است، نه موفقیت.

---

## 6. خلاصه برای کاربر

بعد از تولید سند، یک خلاصهٔ کوتاه فارسی به کاربر بده. نه بیشتر از ده خط.

این تنها چیزی است که کاربر لازم است بخواند. سند JSON برای مراحل بعدی است، نه برای او.

```text
۳ مورد verified — یکی high، دو تا medium
  ۲ مورد unverified که نیاز به بررسی دستی دارند

  مهم‌ترین:
    SEC-001 (high) — مسیر فایل از ورودی کاربر بدون نرمال‌سازی
                     src/api/files.ts:42

  پوشش: ۶۸ فایل در src/api، src/auth، src/db
  بررسی نشد: migrations/legacy (منسوخ)
```

اگر چیزی پیدا نشد، همان را صریح بگو:

```text
هیچ findingی پیدا نشد.

  بررسی و سالم بود:
    مدیریت نشست            src/auth/session.ts:18-31
    اعتبارسنجی ورودی        src/api/middleware/validate.ts:7-44
    مدیریت اسرار            src/config/env.ts:1-22

  پوشش: ۶۸ فایل
  بررسی نشد: تنظیمات production (خارج از repository)
```

خلاصه، سند را جایگزین نمی‌کند — ولی برای کاربر کافی است.
