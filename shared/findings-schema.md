# Findings Schema

> **قرارداد مشترک — ساختار خروجی همهٔ Reviewerها.**
> قبل از نوشتن فایل خروجی، `trust-model.md` را هم بخوان — آن سند بر این یکی اولویت دارد.

---

## یک فایل، یک منبع حقیقت

هر Reviewer **دقیقاً یک فایل** تولید می‌کند:

```
.audit/<skill-name>-findings.json
```

مثال: `.audit/security-findings.json`

`report.md` تولید **نمی‌شود**. دلیل: دو فایل با یک محتوا یعنی دو نسخه که از هم جدا می‌افتند.
به‌جای آن، بعد از نوشتن فایل، یک خلاصهٔ کوتاه **فارسی** در ترمینال نمایش بده (بخش پایانی این سند).

**زبان:** محتوای JSON انگلیسی است — چون بعداً به coding agent داده می‌شود و اصطلاحات فنی
در انگلیسی دقیق‌ترند. فقط خلاصهٔ ترمینال فارسی است.

---

## ساختار کامل

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

## فیلدهای سطح بالا

| فیلد | اجباری | توضیح |
|---|---|---|
| `skill` | ✅ | نام Skill تولیدکننده |
| `repo` | ✅ | نام repository |
| `repo_commit` | ✅ | commit کوتاهِ HEAD موقع بازرسی — بدون این، گزارش قابل بازتولید نیست |
| `generated_at` | ✅ | تاریخ `YYYY-MM-DD` |
| `project_map` | ✅ | وضعیت نقشهٔ پروژه (پایین) |
| `coverage` | ✅ | چه چیزی دیده شد و چه چیزی نه |
| `checked_and_clean` | ✅ | چه چیزی بررسی شد و سالم بود |
| `findings` | ✅ | آرایه — **می‌تواند خالی باشد** |

### `project_map`

نشان می‌دهد بازرسی با چه اطلاعاتی از پروژه انجام شده:

```json
{
  "used": true,
  "repo_commit": "a3f91c2",
  "staleness": "current"
}
```

`staleness` یکی از این سه:

| مقدار | یعنی |
|---|---|
| `current` | `repo_commit` نقشه با HEAD یکی است |
| `stale-minor` | فرق دارد، ولی فقط تغییرات معمولیِ داخل فایل‌ها — نقشه هنوز معتبر است |
| `stale-structural` | تغییر ساختاری رخ داده — **نقشه باید refresh شود** |

اگر `stale-structural` بود، به کاربر بگو `project-understanding` را دوباره اجرا کند
و بازرسی را متوقف کن. بازرسی با نقشهٔ غلط بدتر از بازرسی بدون نقشه است.

### نقشه در Phase 1 اجباری است

**اگر `.audit/project-map.md` وجود نداشت، بازرسی شروع نمی‌شود.**

```
project-map.md نیست؟
   ↓
متوقف شو. به کاربر بگو اول project-understanding اجرا شود.
هیچ فایل خروجی ننویس.
```

دلیل: در معماری Phase 1، `project-understanding` عمداً قبل از Reviewerها قرار گرفته.
یک Reviewer بدون نقشه نمی‌داند کدام ورودی از اینترنت می‌آید و کدام داخلی است،
کجا مرزهای اعتماد هستند، و کدام بخش‌ها اصلاً در مسیر اجرا قرار دارند.
نتیجه‌اش گزارشی است که همه‌چیز را `high` می‌کند — دقیقاً همان نویزی که این پروژه
برای حذفش ساخته شده.

**`"used": false` در Phase 1 معتبر نیست.** این حالت برای Reviewerهای آینده‌ای نگه داشته
شده که نقشه برایشان واقعاً اختیاری است. هیچ‌کدام از سه Skill فعلی چنین نیستند.

### `coverage`

**این بخش اختیاری نیست و خالی گذاشتنش تخلف است.**

گزارشی که مرز خودش را نگوید، خواننده را فریب می‌دهد — چون فرض می‌کند همه‌چیز دیده شده.

- `reviewed` — مسیرهایی که واقعاً بررسی شدند
- `file_count` — تعداد فایل‌های بررسی‌شده
- `not_reviewed` — چیزی که عمداً بررسی نشد، **با دلیل**
- `could_not_inspect` — چیزی که قابل بررسی نبود (`unknown` در trust model)، **با دلیل**

`reason` باید مشخص باشد. «مرتبط نبود» دلیل نیست.

### `checked_and_clean`

چیزهایی که با دقت بررسی شدند و مشکلی نداشتند.

این بخش نشان می‌دهد بازرسی جدی بوده. بدون آن، گزارشِ خالی و بازرسیِ انجام‌نشده
از بیرون یکسان به نظر می‌رسند.

**هر عضو یک شیء است، نه یک رشته — و باید ارجاع واقعی به فایل داشته باشد:**

```json
{
  "area": "Session cookie configuration",
  "file": "src/auth/session.ts",
  "lines": "18-31",
  "note": "httpOnly and secure flags are enabled"
}
```

| فیلد | اجباری | توضیح |
|---|---|---|
| `area` | ✅ | چه چیزی بررسی شد |
| `file` | ✅ | فایلی که **واقعاً** خواندی |
| `lines` | ✅ | خطوطی که تصمیمت بر آن استوار است |
| `note` | ✅ | چه دیدی که باعث شد بگویی سالم است |

**چرا اجباری:** ادعای «سالم است» هم یک ادعاست و همان بار اثبات را دارد که یک finding.
بدون این قاعده می‌شد نوشت «احراز هویت بررسی شد و مشکلی نداشت» بدون اینکه حتی یک فایل
باز شده باشد — و این دقیقاً همان چیزی است که `trust-model.md` ممنوع کرده،
فقط با علامت مخالف.

**`file` و `lines` اینجا دقیقاً مثل `evidence` اعتبارسنجی می‌شوند** (بخش «قواعد اعتبارسنجی»):
فایل باید وجود داشته باشد و خطوط باید واقعی باشند.

اگر حوزه‌ای را بررسی کردی ولی نمی‌توانی به فایل و خط مشخصی اشاره کنی،
آن حوزه در `checked_and_clean` نمی‌آید — یا در `coverage.reviewed` است، یا هیچ‌جا.

---

## فیلدهای هر Finding

| فیلد | اجباری | مقادیر |
|---|---|---|
| `id` | ✅ | `<PREFIX>-<NNN>` |
| `title` | ✅ | یک جمله، مشخص، بدون کلی‌گویی |
| `severity` | ✅ | `critical` \| `high` \| `medium` \| `low` \| `info` |
| `confidence` | ✅ | `verified` \| `inferred` \| `unverified` |
| `status` | ✅ | `open` در ابتدا |
| `effort` | ✅ | `S` \| `M` \| `L` |
| `evidence` | ✅ | آرایه، **حداقل یک مورد** |
| `explanation` | ✅ | چرا این کد، در این پروژه، مشکل است |
| `impact` | ✅ | اگر رها شود چه می‌شود و برای چه کسی |
| `verification` | ✅ | چطور اثبات یا **رد** شود |
| `suggested_fix` | ✅ | کمترین تغییر ممکن |
| `refs` | ❌ | ارجاعات بیرونی، مثل `["CWE-22"]` |
| `merged_from` | ❌ | فقط planner می‌گذارد |

### `id` — پیشوند دامنه

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

شماره‌گذاری از `001` در هر گزارش. پیشوند باعث می‌شود بعد از ادغام چند گزارش،
منشأ هر مورد معلوم بماند.

### `evidence`

هر عضو آرایه:

```json
{
  "file": "src/api/files.ts",
  "lines": "42-47",
  "snippet": "const p = req.query.path;\nres.send(await fs.readFile(p));"
}
```

- `file` — مسیر نسبت به ریشهٔ repository، **باید واقعاً وجود داشته باشد**
- `lines` — `"42"` یا `"42-47"`
- `snippet` — **عیناً از فایل.** اگر جستجویش کنی باید دقیقاً پیدا شود

اگر یک finding چند نقطه دارد (مثلاً مبدأ و مقصد یک مسیر داده)، چند عضو بگذار.

### `status`

Reviewer همیشه `open` می‌گذارد. بقیه را مراحل بعدی تغییر می‌دهند:

```
open  ──→  confirmed  ──→  fixed
  │
  ├──────→  rejected     (بررسی شد، مشکل نبود)
  └──────→  deferred     (واقعی است، فعلاً حل نمی‌شود)
```

### `effort`

تخمین کار اصلاح: `S` تغییر موضعی · `M` چند فایل · `L` نیاز به بازطراحی.

---

## قواعد اعتبارسنجی

قبل از نوشتن فایل، اینها را چک کن:

**۰. `project-map.md` وجود دارد** — اگر نه، اصلاً فایلی ننویس (بالاتر)

**در `evidence` و `checked_and_clean` هر دو:**
1. هر `file` واقعاً وجود دارد
2. هر `lines` با محتوای واقعی فایل می‌خواند

**فقط در `evidence`:**
3. هر `snippet` عیناً در آن خطوط پیدا می‌شود

**بقیه:**
4. `severity` از سقفِ `confidence` رد نشده (`severity-rubric.md`)
5. `id` ها یکتا هستند و پیشوند درست دارند
6. `coverage` پر است
7. JSON معتبر است

**اگر مورد ۱، ۲ یا ۳ رد شود، آن مورد حذف می‌شود — نه اینکه اصلاح شود.**
این برای `checked_and_clean` هم صدق می‌کند: ارجاعِ نادرست در ادعای «سالم است»
همان‌قدر جدی است که در یک finding.

---

## آرایهٔ خالی معتبر است

```json
{ "findings": [] }
```

گزارشی با صفر finding و `coverage` صادقانه و `checked_and_clean` پر، یک گزارش کامل و خوب است.

`findings` را برای پرکردن گزارش پر نکن. این شکست است، نه موفقیت.

---

## خلاصهٔ ترمینال

بعد از نوشتن فایل، یک خلاصهٔ کوتاه **فارسی** نمایش بده. نه بیشتر از ده خط.

```
✓ .audit/security-findings.json نوشته شد

  ۳ مورد verified — یکی high، دو تا medium
  ۲ مورد unverified که نیاز به بررسی دستی دارند

  مهم‌ترین:
    SEC-001 (high) — مسیر فایل از ورودی کاربر بدون نرمال‌سازی
                     src/api/files.ts:42

  پوشش: ۶۸ فایل در src/api، src/auth، src/db
  بررسی نشد: migrations/legacy (منسوخ)
```

اگر چیزی پیدا نشد، همان را صریح بگو:

```
✓ .audit/security-findings.json نوشته شد

  هیچ finding ای پیدا نشد.

  بررسی و سالم بود:
    مدیریت نشست            src/auth/session.ts:18-31
    اعتبارسنجی ورودی        src/api/middleware/validate.ts:7-44
    مدیریت اسرار            src/config/env.ts:1-22

  پوشش: ۶۸ فایل
  بررسی نشد: تنظیمات production (خارج از repository)
```

خلاصه، فایل را جایگزین نمی‌کند — فقط به کاربر می‌گوید بعدش کجا نگاه کند.
