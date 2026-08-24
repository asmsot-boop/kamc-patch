# نشر 24-7 — patch-2026.08.24-7

> **تطبَّق فوق 24-6.** ثلاثة ملفات: `sources.py` · `shell.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `bf76fc728adfd0b01711fae7`.

## 🧭 مدخل الجرد — سؤال «من وين أسوي الجرد؟»

كان الجواب رابطا يدويا (`/repo/win`)، وهذا **عيب تنقّل لا عيب منطق**:

- **تبويب «🪟 خوادم ويندوز»** في مصادر الربط كان يعرض نقطة الحالة **وجسمه فارغ** فيحيل إلى «⚡ معالجة المدخلات» — **وهي إحالة خاطئة هنا**: بقية المصادر تحضّر وتنتظر مراجعة، أما هذا **فيسحب ويطبّق بنفسه** فلا وسيط بينه وبين المستودع. الآن التبويب **يعرض شاشة الجرد كاملة** بأزرارها.
- **وبند مستقل في الرف** تحت «النظام» بجوار مصادر الربط: **🪟 جرد خوادم ويندوز**.

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-7.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-7.tar.gz | cut -c1-24    # يجب bf76fc728adfd0b01711fae7
./patch.sh patch-2026.08.24-7.tar.gz
```

## بعد التطبيق

الرف ← **النظام** ← **🪟 جرد خوادم ويندوز**. أو: **🔌 مصادر الربط** ← تبويب **🪟 خوادم ويندوز**.
