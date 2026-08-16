# Engineering Skills Pack

مجموعه‌ای از Skillهای تخصصی برای Claude Code که هر repository را مثل یک متخصص Senior
بررسی می‌کنند و گزارش **مبتنی بر شواهد** می‌دهند — نه حدس و توصیهٔ عمومی.

طراحی کامل: [`BLUEPRINT.md`](BLUEPRINT.md)

---

## وضعیت

🚧 **در حال ساخت — هنوز هیچ Skill ای آماده نیست.**

| گام | وضعیت |
|---|---|
| Blueprint | ✅ نهایی |
| قراردادهای مشترک (`shared/`) | ✅ آماده |
| `project-understanding` | ⏳ منتظر Research |
| `security-review` | ⛔ |
| `remediation-planner` | ⛔ |

هیچ Skill ای ساخته نمی‌شود تا Research مربوطه‌اش نوشته و تأیید شود.
هیچ Skill ای نصب نمی‌شود تا از Evaluation رد شود.

---

## نصب

هنوز نه. `install.sh` وقتی اضافه می‌شود که اولین Skill آماده باشد.

## استفاده (وقتی آماده شد)

```bash
cd ~/my-project
claude
```

```
> طبق skill امنیت، این پروژه را بررسی کن و فقط گزارش بده
```

خروجی در `.audit/` همان repository نوشته می‌شود و خلاصه‌اش در ترمینال نمایش داده می‌شود.

---

## ساختار

```
shared/      قراردادی که همهٔ Skillها رعایت می‌کنند
research/    مطالعهٔ حوزه — قبل از هر Skill، اجباری
skills/      خودِ Skillها
evals/       ریپوهای تست
```

## دو اصل

**۱. Research-First** — هیچ Skill ای از حافظهٔ مدل ساخته نمی‌شود.
قبل از هر Skill، حوزه‌اش از منابع معتبر مطالعه و در `research/` ثبت می‌شود.

**۲. شواهد یا سکوت** — هیچ finding ای بدون فایل، خط و کدِ واقعی نوشته نمی‌شود.
گزارشِ خالی، اگر صادقانه باشد، یک گزارش خوب است.

جزئیات: [`shared/trust-model.md`](shared/trust-model.md)
