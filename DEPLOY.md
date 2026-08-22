# نشر 22-1 — patch-2026.08.22-1

> **تجُبّ كل ما قبلها** (لقطة جذر كاملة · 171 ملفًّا · النسخة 2026.08.22-1) — تصلح من أي نسخة سابقة.
> بصمة الرقعة **داخل** التعمية: `ba661214977994f79e7f5d79` — قارنها بعد الفك قبل التطبيق.
> `shared_viz.py.enc` لم يتغير — **لا خطوة يدوية**.

## ما فيها — إصلاحات قواعد البيانات الثلاثة (تدقيق كودكس 22/8، فحصها سونيت مستقلًّا)

1. **🔗 إنفاذ الترابط بين الجداول (DB-01)**: قيود `REFERENCES` كانت معلنة في المخطط **بلا إنفاذ** — علاقة تشير لعنصر وهمي كانت تمرّ صامتة. الآن `PRAGMA foreign_keys=ON` في كل اتصال، فالكتابة اليتيمة تُرفض برسالة صريحة. **أُثبت بالتشغيل على الورشة**: إدراج علاقة لعنصر غير موجود يمرّ قبلها ويُرفض بعدها.
2. **🔎 فهرس تاريخ العنصر (DB-02)**: شاشات «تاريخ هذا العنصر» كانت تمسح سجل `history` كله عند كل فتحة. فهرس `ix_hist_ref (what, ref, id)` يجعلها قفزة مباشرة — **يبنيه الكود تلقائيًّا عند أول إقلاع، بلا مسّ للبيانات**.
3. **🧯 دمج ذري لسجل الأخطاء (DB-05)**: خطآن متطابقان في نفس اللحظة كانا يشقّان العدّاد صفّين (سباق فحص-ثم-إدراج). الإدراج صار `ON CONFLICT(sig) DO UPDATE` ذريًّا، والفهرس الفريد `ux_errors_sig` شرطُه — **يبنيه `_init` تلقائيًّا بعد دمج أي توائم قائمة في قاعدة المدينة** (المدينة قد تحمل توائم وقعت فعلًا — تُدمج بجمع العدّادات لا بالحذف الأعمى).

**الاختبارات: بوابة فحص الدخان مرّت كاملة على ورشة معزولة** (نسخة 19-1 + هذه الإصلاحات الثلاثة فقط — لا شيء آخر في هذه الرقعة).

⚠️ **حدّ يُذكر بصدق**: تفعيل إنفاذ الترابط يجري أول مرة على بيانات المدينة الحقيقية — قاعدة الورشة فُحصت (`foreign_key_check` = صفر مخالفات) لكن قاعدة المدينة قد تحمل إشارات يتيمة قديمة. **لا يكسر التفعيلُ الموجودَ** (القيد يُفحص عند الكتابة الجديدة فقط)، لكن لو ظهر بعد الرقعة `FOREIGN KEY constraint failed` في `/repo/errors` فهذا أثره — يُلتقط ويُصلح مصدره، ولا يحتاج تراجعًا.

## التطبيق (المعتاد)

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.22-1.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.22-1.tar.gz | cut -c1-24    # يجب ba661214977994f79e7f5d79
./patch.sh patch-2026.08.22-1.tar.gz
```
وللتراجع: `./patch.sh --rollback`

## بعد التطبيق — ثلاثة تحققات

1. **الترابط**: `python3 -c "import repo; c=repo.conn(); print(c.execute('PRAGMA foreign_keys').fetchone()[0])"` — يجب `1`. ثم `python3 -c "import repo; c=repo.conn(); print(len(c.execute('PRAGMA foreign_key_check').fetchall()), 'مخالفة قائمة')"` — **أبلغ الرقم كائنًا ما كان** (الصفر سلامة، وغيره جردة يتائم قديمة تُصلح لاحقًا بلا عجلة).
2. **الفهرسان**: `python3 -c "import sqlite3; c=sqlite3.connect('data/repo.db'); print([r[1] for r in c.execute('PRAGMA index_list(history)')], [r[1] for r in c.execute('PRAGMA index_list(errors)')])"` — يجب أن يظهر `ix_hist_ref` و`ux_errors_sig`.
3. **سجل الأخطاء**: `/repo/errors` يفتح طبيعيًّا، وأي خطأ جديد مكرر يرفع العدّاد بدل صفّ جديد.
