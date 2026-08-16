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
    "Session handling — cookies are httpOnly and secure (src/auth/session.ts:18-31)"
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

اگر نقشه‌ای وجود نداشت: `{"used": false}` و در `coverage` ذکر کن که بازرسی
بدون نقشه انجام شده و این چه محدودیتی ایجاد کرده.

### `coverage`

**این بخش اختیاری نیست و خالی گذاشتنش تخلف است.**

گزارشی که مرز خودش را نگوید، خواننده را فریب می‌دهد — چون فرض می‌کند همه‌چیز دیده شده.

- `reviewed` — مسیرهایی که واقعاً بررسی شدند
- `file_count` — تعداد فایل‌های بررسی‌شده
- `not_reviewed` — چیزی که عمداً بررسی نشد، **با دلیل**
- `could_not_inspect` — چیزی که قابل بررسی نبود (`unknown` در trust model)، **با دلیل**

`reason` باید مشخص باشد. «مرتبط نبود» دلیل نیست.

### `checked_and_clean`

چیزهایی که با دقت بررسی شدند و مشکلی نداشتند — ترجیحاً با ارجاع فایل.

این بخش نشان می‌دهد بازرسی جدی بوده. بدون آن، گزارشِ خالی و بازرسیِ انجام‌نشده
از بیرون یکسان به نظر می‌رسند.

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

1. هر `file` واقعاً وجود دارد
2. هر `lines` با محتوای واقعی فایل می‌خواند
3. هر `snippet` عیناً در آن خطوط پیدا می‌شود
4. `severity` از سقفِ `confidence` رد نشده (`severity-rubric.md`)
5. `id` ها یکتا هستند و پیشوند درست دارند
6. `coverage` پر است
7. JSON معتبر است

**اگر مورد ۱، ۲ یا ۳ رد شود، آن finding حذف می‌شود — نه اینکه اصلاح شود.**

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

  بررسی و سالم بود: مدیریت نشست، اعتبارسنجی ورودی در مسیرهای عمومی،
                    مدیریت اسرار
  پوشش: ۶۸ فایل
  بررسی نشد: تنظیمات production (خارج از repository)
```

خلاصه، فایل را جایگزین نمی‌کند — فقط به کاربر می‌گوید بعدش کجا نگاه کند.
