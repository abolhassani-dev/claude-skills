# Engineering Skills Pack

مجموعه‌ای از Skillهای تخصصی برای Claude Code که هر repository را مثل یک متخصص Senior
بررسی می‌کنند و گزارشی مبتنی بر شواهد می‌دهند — نه حدس و توصیهٔ عمومی.

طراحی کامل در [`BLUEPRINT.md`](BLUEPRINT.md).

---

## وضعیت

🚧 **در حال ساخت — هنوز هیچ Skillی آماده نیست.**

| مرحله | وضعیت |
|---|---|
| Blueprint | ✅ نهایی |
| قراردادهای مشترک (`shared/`) | ✅ آماده |
| `research/project-understanding.md` | ✅ تأیید شد |
| Skill `project-understanding` | ⏳ در نوبت ساخت |
| Skill `security-review` | ⛔ شروع نشده |
| Skill `remediation-planner` | ⛔ شروع نشده |

دو دروازه در این پروژه وجود دارد:

- هیچ Skillی ساخته نمی‌شود تا Research مربوط به آن نوشته و تأیید شود.
- هیچ Skillی نصب نمی‌شود تا از Evaluation رد شود.

---

## این پروژه چیست

سه Skill که روی هر repository اجرا می‌شوند:

- `project-understanding` — نقشهٔ پروژه می‌سازد. پیش‌نیاز بقیه.
- `security-review` — بازرسی امنیتی می‌کند و findings تولید می‌کند.
- `remediation-planner` — findings را ادغام و اولویت‌بندی می‌کند.

## این پروژه چه نیست

- ❌ dashboard یا رابط گرافیکی
- ❌ orchestration engine یا runtime
- ❌ سرور، API یا پایگاه‌داده
- ❌ سیستم اصلاح خودکار

قاعده: اگر Claude Code خودش می‌تواند، زیرساخت جدید نمی‌سازیم.

فهرست کامل در [بخش Non-Goals از Blueprint](BLUEPRINT.md#12-non-goals).

---

## Research-First یعنی چه

هیچ Skillی از حافظهٔ مدل ساخته نمی‌شود.

قبل از هر Skill، حوزهٔ تخصصی‌اش از منابع معتبر مطالعه و در `research/` ثبت می‌شود:
استانداردهای رسمی، مستندات رسمی، سازمان‌های فنی شناخته‌شده — و در آخرین رده، مخازن GitHub.

Research ضعیف یعنی Skill ساخته نمی‌شود.

جزئیات در [بخش Research-First از Blueprint](BLUEPRINT.md#1-research-first-development).

---

## شواهد یا سکوت

هیچ findingی بدون فایل، خط و کدِ واقعی نوشته نمی‌شود.
گزارشی که چیزی پیدا نکرده، اگر صادقانه باشد، یک گزارش خوب است.

قواعد کامل در [`shared/trust-model.md`](shared/trust-model.md).

---

## ساختار repository

```text
shared/      قراردادی که همهٔ Skillها رعایت می‌کنند
research/    مطالعهٔ حوزه — قبل از هر Skill، اجباری
skills/      خودِ Skillها
evals/       ریپوهای تست
```

`research/` قبل از `skills/` می‌آید — هم در ساختار، هم در زمان.

---

## نصب

هنوز نه. `install.sh` وقتی اضافه می‌شود که اولین Skill از Evaluation رد شود.

## استفاده (وقتی آماده شد)

```bash
cd ~/my-project
claude
```

```text
> طبق skill امنیت، این پروژه را بررسی کن و فقط گزارش بده
```

خروجی در پوشهٔ `.audit/` همان repository نوشته می‌شود و خلاصه‌اش در ترمینال نمایش داده می‌شود.
