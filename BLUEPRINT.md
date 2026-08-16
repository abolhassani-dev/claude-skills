# BLUEPRINT — Engineering Skills Pack

> وضعیت: **پیش‌نویس، منتظر تأیید**
> تا تأیید صریح: هیچ Skill، هیچ shared file، هیچ نصب، هیچ clone.

---

## Context

پروژهٔ قبلی (AI Engineering OS) با هدف ساده‌ای شروع شد و به orchestration، debate loop،
state machine و dashboard تبدیل شد. درسی که گرفتیم دو تا بود و هر دو در این سند سخت‌گیرانه اعمال شده‌اند:

1. **مشکل، نبودِ زیرساخت نبود. نبودِ تخصص بود.**
   یک agent با ده لایه هماهنگی، اگر نداند یک Senior Security Engineer واقعاً به چه چیزی نگاه می‌کند،
   فقط با سرعت بیشتری چیز بی‌ربط تولید می‌کند.

2. **هر چیزی که «شاید بعداً لازم شود» بسازیم، همان چیزی است که پروژه را می‌کشد.**

پس این پروژه دو ستون دارد و بس:

```
    ستون اول                          ستون دوم
    RESEARCH-FIRST                    MVP کوچک و بی‌رحمانه
    هیچ Skill از حافظهٔ مدل            سه Skill، بعد توقف
    ساخته نمی‌شود                      تا اثبات ارزش روی پروژهٔ واقعی
```

بخش ۱ این سند — Research-First — **مهم‌ترین بخش است**. بقیه جزئیات اجرایی است.

---

# بخش ۱ — Research-First Development

## ۱.۱ قانون بنیادین

> **هیچ Skill ای مستقیماً از حافظهٔ مدل تولید نمی‌شود.**

Claude دربارهٔ امنیت، دیتابیس و معماری «چیزهایی می‌داند». آن دانش، میانگینِ مبهمِ اینترنت است:
قدیمی، سطحی، و پر از توصیه‌هایی که ده سال پیش درست بودند.

یک Skill که از آن حافظه نوشته شود، یک چک‌لیست عمومی تولید می‌کند که همه‌جا صدق می‌کند
و هیچ‌جا مفید نیست. دقیقاً همان چیزی که نمی‌خواهیم.

پس قبل از هر Skill، **Domain را مطالعه می‌کنیم**، نه Skillهای مشابه را.

## ۱.۲ سؤال محوری هر Research

برای هر حوزه، تحقیق باید به این سؤال جواب دهد:

> **یک متخصص Senior واقعیِ همین حوزه، امروز، وقتی این کد را باز می‌کند،
> دقیقاً دنبال چه می‌گردد — و چرا؟**

نه «چه چیزهایی خوب است». نه «best practiceها چیست».
بلکه: **این آدم در عمل چه کار می‌کند، به چه ترتیبی، و چه چیزی را عمداً نادیده می‌گیرد.**

مثال از تفاوت این دو:

| حافظهٔ مدل می‌گوید | Research باید کشف کند |
|---|---|
| «SQL Injection را بررسی کن» | امروز اکثر ORMها این را حل کرده‌اند؛ یک Senior AppSec وقتش را روی مرزهای اعتماد، منطق مجوزدهی (authorization)، و SSRF می‌گذارد — نه روی رشته‌های SQL |
| «ایندکس بگذار» | یک Senior DB Engineer اول به الگوی نوشتن/خواندن و ایمنی migration نگاه می‌کند، چون ایندکس اشتباه بدتر از نبودِ ایندکس است |
| «تست بنویس» | تست‌های زیاد با پوشش بالا که هیچ رفتاری را واقعاً تأیید نمی‌کنند، رایج‌ترین شکست است |

این تفاوت، تفاوت بین یک Skill بی‌مصرف و یک Skill واقعی است.

> **توجه:** جدول بالا فقط برای نشان‌دادن *شکل* تفاوت است. ستون راست هنوز **فرضیه** است،
> نه یافته. خودِ همین‌ها هم باید در Research تأیید یا رد شوند — وگرنه دقیقاً همان
> اشتباهی را کرده‌ایم که این بخش می‌خواهد جلویش را بگیرد.

## ۱.۳ چرخهٔ Research برای هر Domain

```
  ۱  Understand the Domain
       حوزه دقیقاً چیست؟ مرزش کجاست؟ چه کسی این کار را انجام می‌دهد؟
                    ↓
  ۲  Study Official Standards
       استاندارد رسمی و به‌روزِ حوزه (و بررسی اینکه نسخهٔ جدیدتری هست یا نه)
                    ↓
  ۳  Study Authoritative Technical Sources
       مستندات رسمی ابزارها، نوشتهٔ نگهدارندگان، سازمان‌های فنی شناخته‌شده
                    ↓
  ۴  Study High-Quality Repositories / Existing Skills
       فقط به‌عنوان یکی از منابع — نه منبع اصلی
                    ↓
  ۵  Compare Different Approaches
       جاهایی که منابع معتبر با هم اختلاف دارند را پیدا کن و ثبت کن
                    ↓
  ۶  Identify Blind Spots and False Positives
       ⚠️ حیاتی‌ترین گام — پایین توضیح داده شده
                    ↓
  ۷  Extract Current Best Practices
       «current» یعنی امروز، نه آنچه پنج سال پیش درست بود
                    ↓
  ۸  Design Our Review Methodology
       روال بازرسی خودمان: با چه ترتیبی، با چه اولویتی، با چه شواهدی
                    ↓
  ۹  Build the Skill
                    ↓
  ۱۰ Evaluate
```

### چرا گام ۶ حیاتی‌ترین است

هر کسی می‌تواند چک‌لیست جمع کند. چیزی که یک Skill را از یک چک‌لیست جدا می‌کند این است که
**بداند کجا اشتباه می‌کند**.

Research باید صراحتاً ثبت کند:

- **False Positiveهای رایج** — چیزهایی که *شبیه* مشکل‌اند ولی نیستند.
  مثال از شکلِ چنین موردی: کدی که خطرناک به نظر می‌رسد ولی ورودی‌اش هرگز از کاربر نمی‌آید.
  Skill باید بداند قبل از گزارش، این را چک کند.
- **Blind Spotها** — چیزهایی که یک بازرس (انسان یا مدل) معمولاً از دستش می‌دهد،
  چون در کد دیده نمی‌شوند: تنظیمات محیط، ترتیب اجرا، رفتار زیر بار.
- **توصیه‌های منسوخ** — چیزهایی که هنوز در اینترنت هستند ولی دیگر درست نیستند.
  Skill باید صراحتاً بگوید این‌ها را گزارش نکند.

بدون این گام، Skill پرِ نویز می‌شود و بعد از دو بار استفاده کنارش می‌گذاری.

## ۱.۴ سلسله‌مراتب منابع

منابع به این ترتیب اعتبار دارند. منبع پایین‌تر هرگز منبع بالاتر را نقض نمی‌کند:

```
۱. Official standards                    ← بالاترین اعتبار
۲. Official documentation
۳. Recognized technical organizations
۴. Maintainer documentation / نوشتهٔ سازندگان
۵. High-quality engineering publications
۶. Academic / research material (جایی که مرتبط باشد)
۷. High-quality GitHub repositories
۸. Existing Claude / Agent Skills        ← پایین‌ترین اعتبار
```

**GitHub یکی از منابع است، نه منبع اصلی.**
Skillهای موجود در پایین‌ترین رده هستند — از آن‌ها *ساختار* و *ایده* می‌گیریم، نه *محتوای فنی*.

**بررسی تازگی اجباری است.** برای هر استاندارد یا منبع: آیا نسخهٔ جدیدتری منتشر شده؟
آیا چیزی جایگزینش شده؟ آیا این توصیه هنوز معتبر است؟ اگر تاریخ منبع را نمی‌دانیم، نباید استفاده شود.

## ۱.۵ ارزیابی Repository — Star معیار نیست

هر repository ای که به آن استناد می‌کنیم باید روی این ده محور بررسی شود:

| محور | چه چیزی را چک کنیم |
|---|---|
| latest meaningful commit | آخرین commit *واقعی* — نه ویرایش README |
| contributor activity | چند نفر؟ یا یک نفر که دو سال است غیبش زده؟ |
| issue / PR activity | آیا کسی جواب می‌دهد؟ issueها باز مانده‌اند؟ |
| reputation | نویسنده کیست؟ سابقه‌اش در این حوزه چیست؟ |
| documentation quality | توضیح داده که *چرا*، یا فقط *چه*؟ |
| license | آیا اصلاً اجازهٔ برداشت داریم؟ **قبل از هر کپی‌برداری** |
| real-world adoption | کسی واقعاً استفاده می‌کند یا فقط ستاره خورده؟ |
| technical depth | عمق دارد یا فهرست بی‌جان است؟ |
| **AI-generated / shallow؟** | متن پرطمطراق و توخالی، مثال‌های عمومی، بدون درد واقعی |
| recommendations still current؟ | توصیه‌هایش هنوز درست‌اند یا مربوط به دورهٔ دیگری‌اند؟ |

آخرین دو مورد بیشترین کاربرد را دارند. مخازن زیادی هستند که ستاره دارند و محتوایشان
متنِ تولیدشدهٔ بی‌عمق است. اگر repository در این دو محور رد شد، **در Research به‌عنوان
«بررسی و رد شد» ثبت می‌شود** — این خودش یک یافتهٔ ارزشمند است.

## ۱.۶ Research Artifact — اجباری

قبل از نوشتن حتی یک خط از هر Skill، این فایل باید وجود داشته باشد:

```
research/<skill-name>.md
```

با این پانزده بخش. هیچ‌کدام اختیاری نیست:

```markdown
# Research — <skill-name>

## 1. Domain Definition
حوزه دقیقاً چیست؟ مرزش کجاست؟ چه چیزی جزو آن نیست؟

## 2. What a Senior Specialist Reviews
یک متخصص واقعی در عمل چه کار می‌کند، به چه ترتیبی، با چه اولویتی

## 3. Official Standards
استاندارد رسمی + نسخه + تاریخ + تأیید اینکه جدیدترین است

## 4. Authoritative Sources
مستندات رسمی، سازمان‌های فنی، نوشتهٔ نگهدارندگان

## 5. Important Review Dimensions
محورهای بازرسی — هر کدام با ارجاع به منبعی که از آن آمده

## 6. Common Failure Modes
در واقعیت این حوزه چطور خراب می‌شود

## 7. Common False Positives
چیزهایی که شبیه مشکل‌اند و نیستند + چطور تشخیصشان دهیم

## 8. Common Blind Spots
چیزهایی که معمولاً از دست می‌روند + چرا

## 9. Existing Approaches
روش‌های موجود برای بازرسی این حوزه

## 10. Relevant GitHub Repositories
هر کدام با نتیجهٔ ارزیابی ده‌محوریِ بخش ۱.۵ — شامل ردشده‌ها

## 11. Comparison
جاهایی که منابع با هم اختلاف دارند + تحلیل ما

## 12. What We Adopt
چه چیزی برمی‌داریم و از کدام منبع

## 13. What We Reject
چه چیزی را عمداً کنار می‌گذاریم و **چرا** — این بخش به اندازهٔ بخش ۱۲ مهم است

## 14. Final Skill Design Decisions
از تحقیق به طراحی: Skill چه ساختاری خواهد داشت و چرا

## 15. Sources
هر منبع با لینک، تاریخ دسترسی، و رده‌اش در سلسله‌مراتب ۱.۴
```

## ۱.۷ Research Quality Gate

Research ضعیف = Skill ساخته نمی‌شود. «ضعیف» یعنی هر کدام از این‌ها:

- ❌ هیچ منبع رده ۱–۳ (استاندارد یا مستندات رسمی) ندارد
- ❌ یک محور بازرسی وجود دارد که به هیچ منبعی وصل نیست (یعنی از حافظهٔ مدل آمده)
- ❌ بخش False Positives خالی یا سطحی است
- ❌ بخش «What We Reject» خالی است — یعنی هیچ قضاوتی نشده، فقط جمع‌آوری شده
- ❌ تازگی منابع بررسی نشده
- ❌ فقط از Skillها و مخازن GitHub تغذیه شده (رده ۷ و ۸)

**من (تو) Research را می‌خوانی و تأیید می‌کنی. تا آن تأیید، Skill نوشته نمی‌شود.**

## ۱.۸ روش انجام Research

- جستجو و خواندن از طریق وب — **بدون clone، بدون اجرا، بدون نصب**
- فایل‌های Skillهای دیگر به‌صورت خام از وب خوانده می‌شوند
- لایسنس هر منبع قبل از هرگونه برداشت متن بررسی می‌شود
- Research در همان جلسهٔ Claude Code انجام می‌شود — بدون ابزار جدید

---

# بخش ۲ — Product Vision

## ۲.۱ چه می‌سازیم

سه پوشهٔ Markdown که به Claude Code یاد می‌دهند مثل یک متخصص Senior رفتار کند
و خروجی ساختاریافته و مبتنی بر شواهد بدهد.

| Skill | نقش |
|---|---|
| `project-understanding` | نقشهٔ پروژه — پیش‌نیاز بقیه |
| `security-review` | بازرسی امنیتی، تولید findings |
| `remediation-planner` | ادغام و اولویت‌بندی findings |

همین. سه تا.

## ۲.۲ چه نمی‌سازیم

هیچ کدِ اجرایی‌ای که «کار را انجام دهد». Claude Code خودش موتور است.
ما فقط تخصص و قرارداد خروجی می‌نویسیم. (بخش ۹)

## ۲.۳ تجربهٔ کاربر

```
cd ~/my-project
claude

> «طبق skill امنیت، این پروژه را بررسی کن و فقط گزارش بده»

  [Claude ابتدا project-map را می‌سازد یا می‌خواند، بعد بازرسی می‌کند]

  ✓ .audit/security-findings.json نوشته شد

  خلاصه:
  ۳ مورد verified پیدا شد — یکی high، دو تا medium.
  مهم‌ترین: مسیر فایل از ورودی کاربر بدون نرمال‌سازی — src/api/files.ts:42
  ۲ مورد هم نیاز به بررسی دستی دارند (unverified).
  ۶۸ فایل بررسی شد، پوشهٔ migrations قدیمی بررسی نشد.
```

خلاصهٔ فارسی در ترمینال نمایش داده می‌شود. دادهٔ ساختاریافته در فایل می‌ماند.

---

# بخش ۳ — Phase 1 Workflow

نسخهٔ اول **فقط** این است:

```
   ┌─────────────────────────┐
   │  PROJECT UNDERSTANDING  │  →  .audit/project-map.md
   └───────────┬─────────────┘
               ↓
   ┌─────────────────────────┐
   │     SECURITY REVIEW     │  →  .audit/security-findings.json
   └───────────┬─────────────┘
               ↓
   ┌─────────────────────────┐
   │   REMEDIATION PLANNER   │  →  .audit/remediation-plan.md
   └───────────┬─────────────┘
               ↓
   ┌─────────────────────────┐
   │  NORMAL CLAUDE CODE FIX │  یک finding، یک تست، یک commit
   └───────────┬─────────────┘   بدون Skill جدید
               ↓
   ┌─────────────────────────┐
   │       RE-REVIEW         │  همان security-review، روی تغییرات
   └───────────┬─────────────┘
               ↓
         ⛔  توقف کامل
```

**بعد از این، ساخت متوقف می‌شود.** روی یک پروژهٔ واقعی امتحان می‌کنیم:

- اگر مفید بود → Skill بعدی
- اگر نبود → Blueprint اصلاح می‌شود، نه اینکه ۹ Skill دیگر هم ساخته شود

`project-map.md` یک بار ساخته می‌شود و می‌ماند. security-review اول آن را می‌خواند
تا بداند کدام ورودی از اینترنت می‌آید و کدام داخلی است — بدون این، همه چیز `high` می‌شود.

---

# بخش ۴ — Skill Architecture

```
skills/
  project-understanding/
    SKILL.md
    references/

  security-review/
    SKILL.md
    references/

  remediation-planner/
    SKILL.md
    references/
```

**همین. فایل دیگری اضافه نمی‌شود مگر با دلیل عملی مشخص.**

- `SKILL.md` — زیر ۳۰۰ خط: نقش، IN/OUT SCOPE، روال بازرسی، قوانین شواهد، قرارداد خروجی
- `references/` — چک‌لیست تخصصی و جزئیات بلند که همیشه لازم نیستند

Claude محتوای `SKILL.md` را همیشه موقع فعال‌شدن می‌خواند، ولی `references/` را فقط وقتی لازم شود.
پس هر چه در SKILL.md است باید هر بار ارزش خوانده‌شدن داشته باشد.

## قراردادهای مشترک

سه فایل در `shared/` که **مستقیماً خوانده می‌شوند، نه کپی**:

```
shared/
  findings-schema.md
  trust-model.md
  severity-rubric.md
```

`install.sh` یک symlink ثابت می‌سازد:

```
~/.claude/skills-shared/  →  <repo>/shared/
```

و هر SKILL.md می‌گوید: «قبل از نوشتن خروجی، `~/.claude/skills-shared/findings-schema.md` را بخوان.»

یک نسخه، یک محل، بدون همگام‌سازی، بدون اسکریپت.
(بسته‌بندی خودکفا برای زمانی است که بخواهیم Skillها را مستقل توزیع کنیم — بخش Future.)

---

# بخش ۵ — Trust Model

**قلب پروژه.** اگر این کار کند، بقیه جزئیات است.

## چهار سطح اطمینان

| سطح | تعریف | سقف severity |
|---|---|---|
| **verified** | کد را خواندم؛ مشکل در خودِ همین کد دیده می‌شود | بدون محدودیت |
| **inferred** | کد قویاً نشان می‌دهد، ولی به config/داده/runtime وابسته است که ندیدم | `high` |
| **unverified** | ظن است؛ اثباتش به اجرا یا اطلاعاتی که ندارم نیاز دارد | `medium`، و در بخش جدا |
| **unknown** | نتوانستم بررسی کنم (باینری، تولیدشده، سرویس بیرونی) | finding نیست — در coverage ثبت می‌شود |

## Evidence اجباری

هیچ finding ای بدون این هفت مورد نوشته نمی‌شود:

```
actual file       مسیر واقعی و موجود
actual line/range خط یا بازهٔ واقعی
exact snippet     عیناً کپی‌شده از فایل — نه بازنویسی، نه خلاصه
explanation       چرا این، در همین کد، یک مشکل است
impact            اگر رها شود چه می‌شود و برای چه کسی
verification      دقیقاً چطور می‌شود اثبات یا ردش کرد
suggested fix     کمترین تغییر ممکن که حلش می‌کند
```

## قوانین ضدتوهم

1. قبل از نوشتن هر finding، خطوطی که به آن استناد می‌کنی را **دوباره بخوان**.
   اگر کد آن چیزی نیست که فکر می‌کردی، finding را **دور بینداز** — اصلاحش نکن.
2. **Zero findings کاملاً قابل قبول است.** هیچ تعداد هدفی وجود ندارد.
   گزارشِ صفر با coverage صادقانه، یک گزارش خوب است. پرکردن گزارش شکست است.
3. توصیهٔ عمومی بدون ارجاع به خط مشخص، finding نیست. ننویس.
4. اگر چیزی که ادعایت به آن وابسته است پیدا نشد، confidence را پایین بیاور
   و در `verification` بنویس دنبال چه می‌گشتی.
5. «بررسی نشد» را بنویس، نه سکوت. گزارشی که مرزش را نگوید قابل اعتماد نیست.
6. severity از `severity-rubric.md` می‌آید، نه از حس.

## خط قرمز مطلق

```
Fake file  |  Fake line  |  Fake code snippet   =   FAIL
```

شواهد ساختگی تحت **هیچ** شرایطی قابل قبول نیست. حتی یک مورد ⇒ Skill نصب نمی‌شود.

---

# بخش ۶ — Findings Schema

## یک منبع حقیقت

هر Reviewer **یک فایل** تولید می‌کند:

```
.audit/<skill-name>-findings.json
```

`report.md` تولید نمی‌شود. دلیل: دو فایل با یک محتوا یعنی دو نسخه که از هم جدا می‌افتند.
Claude بعد از بازرسی، خلاصهٔ فارسی را در ترمینال نمایش می‌دهد. اگر بعداً واقعاً لازم شد
گزارش را برای کسی بفرستی که Claude ندارد، آن وقت دلیلِ عملی داریم و اضافه‌اش می‌کنیم. (Future)

## ساختار

```json
{
  "skill": "security-review",
  "repo": "my-project",
  "commit": "a3f91c2",
  "date": "2026-08-16",

  "coverage": {
    "reviewed": ["src/api", "src/auth", "src/db"],
    "file_count": 68,
    "not_reviewed": [
      { "path": "migrations/legacy", "reason": "پیش از بازنویسی ۲۰۲۴، دیگر اجرا نمی‌شود" }
    ],
    "could_not_inspect": [
      { "item": "production config", "reason": "خارج از repository" }
    ]
  },

  "checked_and_clean": [
    "مدیریت نشست — کوکی‌ها httpOnly و secure هستند (src/auth/session.ts)"
  ],

  "findings": [
    {
      "id": "SEC-001",
      "title": "User-controlled path passed to fs.readFile without normalization",
      "severity": "high",
      "confidence": "verified",
      "status": "open",
      "evidence": [
        {
          "file": "src/api/files.ts",
          "lines": "42-47",
          "snippet": "const p = req.query.path;\nres.send(await fs.readFile(p));"
        }
      ],
      "explanation": "...",
      "impact": "...",
      "verification": "...",
      "suggested_fix": "...",
      "effort": "S",
      "refs": ["CWE-22"]
    }
  ]
}
```

**نکات:**
- `id` با پیشوند دامنه (`SEC-`, `ARCH-`, `DB-`) تا بعد از ادغام قابل ردیابی بماند
- `checked_and_clean` نشان می‌دهد بازرسی جدی بوده — نه اینکه چیزی پیدا نشد چون نگاه نشد
- `status`: `open` → بعداً `confirmed` / `rejected` / `deferred` / `fixed`
- بدنهٔ فنی انگلیسی (چون بعداً به coding agent داده می‌شود)، خلاصهٔ ترمینال فارسی

---

# بخش ۷ — Domain Ownership

هر Reviewer در SKILL.md دو بخش صریح دارد:

```markdown
## IN SCOPE
دقیقاً چه چیزی مسئولیت من است

## OUT OF SCOPE
چه چیزی را گزارش نمی‌کنم چون Skill دیگری مسئولش است
```

در Phase 1 با یک Reviewer، همپوشانی کم است — ولی از همین اول رعایت می‌شود،
چون هدفش جلوگیری از finding تکراری بین Skillهاست و بعداً اضافه‌کردنش سخت می‌شود.

**هشدار برای آینده:** در لیست اولیهٔ ۱۲ Skill، سه مورد مرز مشخصی ندارند و اگر همان‌طور
ساخته شوند گزارش‌ها به‌شدت تکراری می‌شوند و کار planner را خراب می‌کنند:

| Skill | مشکل |
|---|---|
| `backend-review` | عمدتاً = architecture + security + database روی یک لایه |
| `frontend-review` | همپوشانی سنگین با code-quality و performance |
| `ui-ux-review` | بدون اجرای برنامه بیشترش حدس است — بالاترین ریسک توهم |

تصمیم دربارهٔ این‌ها بعد از اثبات Phase 1 گرفته می‌شود، نه الان.

---

# بخش ۸ — Remediation Workflow

## ورودی planner

همهٔ `*-findings.json` های موجود در `.audit/` + `project-map.md`.
(در Phase 1 فقط یکی هست، ولی طراحی از همان اول چندتایی است.)

## چهار کار planner

1. **ادغام تکراری‌ها** — دو finding با فایل و خطوط هم‌پوشان و علت یکسان یکی می‌شوند؛
   `id`های اصلی در `merged_from` حفظ می‌شوند
2. **علامت‌زدن تناقض‌ها** — planner **خودش تصمیم نمی‌گیرد**؛ هر دو دیدگاه را نشان می‌دهد، انتخاب با توست
3. **جداکردن findingهای ضعیف** — هر چیزی با `confidence = unverified` یا evidence ناقص
   به «Needs Verification» می‌رود و وارد صف اصلاح **نمی‌شود**
4. **اولویت‌بندی** — بر اساس `severity × confidence × (۱/effort)`، با یک قاعدهٔ سخت:
   **هیچ چیزی که verified نیست بالای صف نمی‌رود**، هر چقدر severity بالا باشد

## خروجی

`remediation-plan.md` — یک صف مرتب. هر آیتم یک واحد کار مستقل با فایل‌های درگیر،
اصلاح پیشنهادی، و راه اثبات درستی.

## مرحلهٔ اصلاح

**Claude Code معمولی، بدون Skill جدید.** یک قاعده: یک finding → یک تست → یک commit.
پیام commit شامل `id` است تا ردیابی ممکن بماند.

---

# بخش ۹ — Evaluation

## دو ریپو، نه بیشتر

| ریپو | هدف | چه چیزی می‌سنجد |
|---|---|---|
| **A — intentionally flawed** | پروژهٔ کوچکی با مشکلات مشخصِ کاشته‌شده | آیا مشکلات واقعی را پیدا می‌کند |
| **B — one real project** | یکی از پروژه‌های خودت | usefulness، false positiveها، و مقایسه با Claude بدون Skill |

Clean repository و مجموعهٔ benchmark کامل فعلاً لازم نیست. (Future)

## دروازهٔ مطلق

```
Fake file  |  Fake line  |  Fake code snippet   =   FAIL
```

یک مورد کافی است. Skill نصب نمی‌شود، بازنویسی می‌شود.

## روش سنجش — بدون هیچ زیرساختی

یک **جلسهٔ تازهٔ Claude** که Skill را ندیده، `findings.json` را می‌گیرد و برای هر finding
سه چیز را چک می‌کند:

1. آیا این فایل و این خطوط وجود دارند؟
2. آیا کد واقعی همان چیزی است که snippet می‌گوید؟
3. آیا ادعا از همین کد قابل نتیجه‌گیری است؟

خروجی: `PASS` / `FAIL` / `NEEDS-HUMAN`. مورد سوم را خودت نگاه می‌کنی.

## مقایسهٔ پایه

همان پرامپت **بدون Skill** هم اجرا می‌شود.
اگر Skill از Claude خالی بهتر نبود، ارزش نصب ندارد — هر چقدر هم خوب نوشته شده باشد.

---

# بخش ۱۰ — Repository Structure

```
claude-skills/
├── BLUEPRINT.md              ← همین سند
├── README.md                 ← نصب و استفاده در ۱۰ خط
├── install.sh
│
├── shared/                   ← مستقیماً خوانده می‌شود
│   ├── findings-schema.md
│   ├── trust-model.md
│   └── severity-rubric.md
│
├── research/                 ← قبل از هر Skill، اجباری
│   ├── project-understanding.md
│   ├── security-review.md
│   └── remediation-planner.md
│
├── skills/
│   ├── project-understanding/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── security-review/
│   │   ├── SKILL.md
│   │   └── references/
│   └── remediation-planner/
│       ├── SKILL.md
│       └── references/
│
└── evals/
    └── fixtures/             ← ریپوی A
```

`research/` قبل از `skills/` می‌آید — هم در ساختار، هم در زمان.

---

# بخش ۱۱ — Installation

```
<repo>/skills/security-review/   →  ~/.claude/skills/security-review/
<repo>/shared/                   →  ~/.claude/skills-shared/
```

`install.sh` این symlinkها را می‌سازد. بعد از آن هر `git pull` بلافاصله همه‌جا اعمال می‌شود.
`~/.claude/skills/` یعنی global — در هر repository ای در دسترس است.

Skillی که هنوز از Evaluation رد نشده، symlink نمی‌شود.

خروجی‌ها در `.audit/` داخل همان repository نوشته می‌شوند و در `.gitignore` قرار می‌گیرند —
مسیرهای فایل نسبی و درست می‌مانند و چیزی تصادفاً commit نمی‌شود.

---

# بخش ۱۲ — Non-Goals

نمی‌سازیم:

- ❌ dashboard
- ❌ orchestration engine
- ❌ runtime
- ❌ agent debate
- ❌ state machine
- ❌ server
- ❌ API
- ❌ database
- ❌ MCP server
- ❌ autonomous fixing system

**قاعده:** اگر Claude Code خودش می‌تواند، زیرساخت جدید نمی‌سازیم.

---

# بخش ۱۳ — Roadmap

| گام | کار | دروازه |
|---|---|---|
| ۰ | تأیید همین Blueprint | ✅ تو |
| ۱ | اسکلت مخزن + سه فایل `shared/` | ✅ تو |
| ۲ | `research/project-understanding.md` | ✅ تو (Research Gate) |
| ۳ | ساخت Skill اول | |
| ۴ | Evaluation روی ریپوهای A و B | ✅ دروازهٔ مطلق |
| ۵ | نصب | ✅ تو |
| ۶ | همان چرخه برای `security-review` | ✅ تو |
| ۷ | همان چرخه برای `remediation-planner` | ✅ تو |
| ۸ | **توقف** — تست کامل روی پروژهٔ واقعی | 🎯 |

بین گام ۷ و هر کار بعدی، **توقف اجباری**.

---

# بخش ۱۴ — Definition of Done

## برای یک Research

- [ ] هر پانزده بخش پر شده
- [ ] حداقل یک منبع رده ۱–۳
- [ ] هر محور بازرسی به یک منبع وصل است
- [ ] False Positives و Blind Spots واقعاً پر شده‌اند
- [ ] «What We Reject» خالی نیست
- [ ] تازگی منابع بررسی شده
- [ ] **تو تأیید کرده‌ای**

## برای یک Skill

- [ ] Research تأییدشده وجود دارد
- [ ] `SKILL.md` زیر ۳۰۰ خط، با IN/OUT SCOPE
- [ ] خروجی دقیقاً مطابق `findings-schema.md`
- [ ] روی ریپو A مشکلات کاشته‌شده را پیدا می‌کند
- [ ] روی ریپو B مفید است و false positive کم دارد
- [ ] **صفر شواهد ساختگی**
- [ ] بهتر از Claude بدون Skill بودنش نشان داده شده
- [ ] **تو تأیید کرده‌ای** → نصب

## برای Phase 1

- [ ] هر سه Skill قبول و نصب شده
- [ ] یک چرخهٔ کامل واقعی: map → review → plan → fix → re-review
- [ ] حداقل یک اصلاح واقعیِ commit شده که از این مسیر آمده
- [ ] `README.md` طوری که شش ماه بعد بدون یادآوری قابل استفاده باشد
- [ ] هیچ dashboard، orchestration یا سروری ساخته نشده ✅

---

# بخش ۱۵ — Future Enhancements

هیچ‌کدام تا اثبات Phase 1 ساخته نمی‌شوند:

**نسخه‌بندی رسمی** — فایل VERSION، CHANGELOG هر Skill، تگ انتشار، semantic versioning،
تاریخچهٔ نسخه‌بندی‌شدهٔ evalها. *فعلاً git history کافی است.*

**بسته‌بندی خودکفا** — کپی قراردادهای مشترک داخل هر Skill (`sync-shared.sh`).
*فقط اگر بخواهیم Skillها را مستقل توزیع کنیم.*

**`report.md`** — گزارش Markdown کنار JSON.
*فقط اگر لازم شد برای کسی بفرستی که Claude ندارد.*

**Evaluation کامل** — ریپوی تمیز، benchmark suite، سنجه‌های عددی، رهگیری رگرسیون.

**Skillهای بیشتر** — database, architecture, code-quality, performance, QA, DevOps,
backend, frontend, accessibility, UX. *به ترتیبی که بعد از Phase 1 تصمیم می‌گیریم.*

**`remediation-engineer`** — فقط اگر ثابت شود Claude Code معمولی حلقهٔ
«یک finding، یک تست، یک commit» را رعایت نمی‌کند.

**یکپارچگی با CI** — فقط بعد از پایدارشدن Skillها.

---

# خلاصهٔ تغییرات نسبت به نسخهٔ قبل

## What Was Removed From MVP

| حذف‌شده | چرا |
|---|---|
| ۹ Skill از ۱۲ | ارزش هیچ‌کدام قبل از اثبات سه‌تای اول معلوم نیست |
| `report.md` | داده تکراری بدون دلیل عملی — یک منبع حقیقت کافی است |
| `templates/` در هر Skill | شِما در `shared/` است؛ قالب جدا لازم نیست |
| `evals/` داخل هر Skill | در `evals/` ریشه جمع می‌شود |
| `VERSION` و `CHANGELOG` هر Skill | git history برای مرحلهٔ آزمایشی کافی است |
| `scripts/sync-shared.sh` | `shared/` مستقیماً خوانده می‌شود؛ کپی لازم نیست |
| `references/_shared/` در هر Skill | همان دلیل |
| ریپوی تمیز (C) در Evaluation | دو ریپو برای MVP کافی است |
| شش سنجهٔ عددی Evaluation | جای آن: یک دروازهٔ مطلق + قضاوت انسانی |
| `remediation-engineer` | Claude Code معمولی این کار را می‌کند |

## What Was Moved To Future

نسخه‌بندی رسمی · بسته‌بندی خودکفا · `report.md` · Evaluation کامل و benchmark ·
۹ Skill باقی‌مانده · `remediation-engineer` · یکپارچگی CI
(جزئیات: بخش ۱۵)

## What Phase 1 Contains

```
۳ Skill        project-understanding, security-review, remediation-planner
۳ Research     یکی برای هر Skill — قبل از ساخت، با دروازهٔ تأیید
۳ قرارداد      findings-schema, trust-model, severity-rubric
۱ اسکریپت      install.sh (فقط symlink)
۲ ریپوی تست    A: مشکل‌دارِ عمدی · B: پروژهٔ واقعی
۱ خروجی        findings.json + خلاصهٔ فارسی در ترمینال
```

بعد: **توقف** و تست روی پروژهٔ واقعی.

## Research-First Development Rules

```
۱.  هیچ Skill ای از حافظهٔ مدل ساخته نمی‌شود.

۲.  اول Domain مطالعه می‌شود، نه Skillهای مشابه.
    سؤال محوری: یک متخصص Senior واقعی، امروز، دقیقاً دنبال چه می‌گردد و چرا؟

۳.  research/<skill-name>.md قبل از Skill وجود دارد — هر پانزده بخش.

۴.  سلسله‌مراتب منابع رعایت می‌شود.
    GitHub رده ۷ است. Skillهای موجود رده ۸.
    استانداردها و مستندات رسمی رده ۱ تا ۳.

۵.  Star معیار نیست. ده‌محور بخش ۱.۵ برای هر repository بررسی می‌شود —
    به‌ویژه «آیا AI-generated و توخالی است؟» و «آیا هنوز معتبر است؟»

۶.  تازگی همیشه بررسی می‌شود: آیا استاندارد یا منبع جدیدتری وجود دارد؟

۷.  False Positives و Blind Spots بخش اجباری‌اند — نه اختیاری.
    چیزی که Skill را از یک چک‌لیست جدا می‌کند این است که بداند کجا اشتباه می‌کند.

۸.  «What We Reject» باید پر باشد. تحقیقی که چیزی را رد نکرده، تحقیق نکرده —
    فقط جمع‌آوری کرده.

۹.  هر محور بازرسی به یک منبع مشخص وصل است.
    محورِ بدون منبع = از حافظهٔ مدل آمده = حذف می‌شود.

۱۰. Research ضعیف ⇒ Skill ساخته نمی‌شود. بدون استثنا.
```

---

## منتظر تأیید

هیچ Skill، هیچ shared file، هیچ نصب، هیچ clone انجام نشده.
گام بعدی پس از تأیید: **اسکلت مخزن + سه فایل `shared/`** — هنوز هیچ Skill ای.
