# نشر 17-1 — patch-2026.08.17-1

> **تجُبّ كل ما قبلها** (لقطة جذر كاملة · 170 ملفا · النسخة 2026.08.17-1) — سواء كانت المدينة على 16-9 أو 16-10.
> بصمة الرقعة داخل التعمية: `6db147dfd294dae8471a0a39` — قارنها بعد الفك قبل التطبيق.
> ⚠️ **هذه المرة خطوة يدوية إلزامية**: `shared_viz.py.enc` **تغيّر** (إصلاح treemap في العربية) — طبّقه بعد الرقعة (القسم الأخير).

## ما فيها — دفعة ليلة 16/8 (اعتماد شامل من المالك)

1. **🔒 سد ثغرة «خروج ثم تراجع»**: `Cache-Control: no-store` على `/repo|/api|/doc|/login` (الأصول تبقى immutable) + حارس `pageshow` يجبر إعادة التحميل عند استعادة الصفحة من ذاكرة المتصفح — زر الرجوع بعد الخروج لا يعرض لقطة الجلسة.
2. **🔐 صفحة القفل**: سطر مقتضب «انتهت الجلسة لعدم النشاط. [متابعة]» + **قصّ البريد من عنوان الصفحة** (كان بريد المستخدم يظهر في الرابط) + تنظيف سجل المتصفح (`history.replaceState`).
3. **🗂 هندسة القائمة**: وثيقة المراجعة صارت تبويبا تحت الوثيقة المرجعية (أُلغي بندها من أسفل الرف) · المناظير القياسية انتقلت إلى الامتثال · ترتيب المناظير: الخرائط، المنظر، المصفوفة، التوأم.
4. **✍️ نبرة اسمية مقتضبة** على كل أوصاف القائمة واللوحة والشاشات (18 ملف شاشة + بطاقات اللوحة صارت تُترجم كالرف).

> إن كانت المدينة على 16-9: يصل معها أيضا كل ما في 16-10 (المرحلة 2 للقارئ: زر «📤 صدّر للاستخراج» + بذرة 17/17 وثيقة · 577 مقترحا + اسم الوثيقة رابط الفتح). البذرة دمج يضيف ولا يمس المقبول — لا خطر تكرار.

## التطبيق (المعتاد)

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.17-1.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
./patch.sh patch-2026.08.17-1.tar.gz
```
(الكلمة الثابتة في `~/.config/kamc/deploy_pass` على جهازك — لا تحتاجها في الرسالة.)

## ⚠️ الخطوة اليدوية — shared/viz.py (إلزامية هذه المرة)

الرقعة لا تشحن `shared/` (خارج جذر المنصة). الإصلاح: treemap كان يرث RTL فتخرج العناوين من بلاطاتها وتُقص.

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/shared_viz.py.enc -o shared_viz.py.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/shared_viz.sha256 -o shared_viz.sha256
sha256sum -c <(echo "$(cat shared_viz.sha256)  shared_viz.py.enc")   # يجب OK
SHARED=$(.venv/bin/python3 -c "import env; print(env.SHARED)")
cp "$SHARED/viz.py" "$SHARED/viz.py.bak-20260817"    # نسخة قبل الاستبدال
openssl enc -d -aes-256-cbc -pbkdf2 -in shared_viz.py.enc -out "$SHARED/viz.py" -pass file:$HOME/.config/kamc/deploy_pass
systemctl --user restart kamc
```

## بعد التطبيق — أربعة تحققات

1. **الخروج ثم زر الرجوع**: سجّل خروجا ثم اضغط رجوع — يجب ألا تظهر لقطة الجلسة (يعاد التوجيه للدخول).
2. **صفحة القفل**: انتظر انتهاء الجلسة — سطر «انتهت الجلسة لعدم النشاط» والرابط في شريط العنوان **بلا بريد**.
3. **القائمة**: وثيقة المراجعة تبويب داخل الوثيقة المرجعية · المناظير القياسية تحت الامتثال.
4. **treemap** (اللوحة/الخرائط): عناوين البلاطات داخل بلاطاتها لا خارجها.

بعد النجاح أبلغ المالك سطرا واحدا: **«✅ نُشرت 17-1 · postcheck مرّ · shared/viz طُبّق»**. المالك سيراجع شاشات المصطلحات بنفسه بعدها (لقطات شاشة) — لا شيء مطلوبا منك فيها.

للتراجع: `./patch.sh --rollback` (و`viz.py.bak-20260817` لإرجاع shared).
