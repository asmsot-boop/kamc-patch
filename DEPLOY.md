# نشر 24-10 — patch-2026.08.24-10

> **تطبَّق فوق 24-9.** أربعة ملفات: `win_inventory.py` · `vault.py` · `app.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `60579c1025877b4c91597925`.

## 🔒 «Connection reset by peer» — والسبب خلط بين المشفّر وغيره

بعد ضبط المنفذ في الخزنة تغيّر العطب من `AccessDenied` إلى:
`ConnectionError: ('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer'))`

⚠️ **وهذه بصمة معروفة لا لغز**: نخاطب مستمع WinRM **المشفّر** بـHTTP، فيقطع الخادم الاتصال بلا رسالة مفهومة. والسبب أنّ التشفير كان **وسيطة منفصلة عن المنفذ**: من كتب `5986` ونسي `KAMC_WIN_SSL=1` في حقل «إضافي» وقع فيه.

**ما تغيّر:**
- **المنفذ يقرّر المخطّط حين لا يُصرَّح بالتشفير**: `5986` ⇒ **HTTPS**، وما عداه ⇒ HTTP. **والتصريح يبقى فوق الاستنتاج** (من كتب 5986 مع `KAMC_WIN_SSL=0` يُحترم اختياره).
- اختبار الخزنة يبني المخطّط بالمنطق نفسه، ويتجاوز التحقق من الشهادة الذاتية.
- **والرسالة تترجم العطب إلى سببه**: القطع ⇒ خلط تشفير · الرفض ⇒ تفويض · تعذّر الوصول ⇒ خدمة أو جدار.

**مقاس بالتشغيل** على أربع حالات: 5985 · 5986 · 5986 مع تصريح مخالف · منفذ غير قياسي.

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-10.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-10.tar.gz | cut -c1-24    # يجب 60579c1025877b4c91597925
./patch.sh patch-2026.08.24-10.tar.gz
```

## ⚠️ وقبل السحب: أيّ منفذ يستمع فعلا على الخادم؟

```powershell
winrm enumerate winrm/config/listener
```
إن ظهر `Transport = HTTP · Port = 5985` فأعِد المنفذ في الخزنة إلى **5985**، وسيعود العطب إلى `AccessDenied` — **وهو تقدّم لا تراجع**: يعني أن الاتصال صار سليما ولم يبقَ إلا **تفويض الحساب** (خطوات رقعة 24-8).
