# نشر تصحيحي 16-5 — patch-2026.08.16-5

> **تجُبّ كل ما قبلها** (لقطة جذر كاملة · 169 ملفا · النسخة 2026.08.16-5) — المدينة على 16-4 المطبقة بنجاح.
> بصمة الرقعة داخل التعمية: `3c24aa7821f81d399855a8b1` — قارنها بعد الفك قبل التطبيق.
> `shared_viz.py.enc` لم يتغير — طبق في 16-4، لا خطوة يدوية هذه المرة.

## ما فيها — تصحيحا ملاحظتي فحصكم على 16-4

1. **⭐ أيقونة التبويب تفتح بلا ارتباط**: وسيط الصلاحيات صار يستثني ما ليس تحت `/repo` و`/api` — فالمتصفح يجلب النجمة قبل الدخول ومن التبويبات الجديدة (كانت 403 «بلا صلاحية»).
2. **👁 المنظر (`/repo/vista`) يعود مفتوحا للمطلع الممنوح** — هذا كان الانحدار الحقيقي الوحيد من فرض 16-4. **أما المحرر/مجموعات الفجوات/الصحة فمنع المطلع منها سلوك قديم قائم قبل 16-4** (بوابة `_ADMIN_PATHS`) — لم يمس، وفتحها سياسة يقررها المالك.

## التطبيق (المعتاد)

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.16-5.tar.gz -pass pass:<الكلمة المعتادة>
./patch.sh patch-2026.08.16-5.tar.gz
```

## بعد التطبيق — تحققان فقط

1. من متصفح **غير مرتبط** (تصفح خفي): `https://ea.kamc.med.sa/assets/favicon.png` → 200 والنجمة تظهر في تبويب جديد.
2. بهوية مطلع ممنوح: `/repo/vista` → 200 (كانت 403).

للتراجع: `./patch.sh --rollback`.
