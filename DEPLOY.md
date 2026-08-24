# نشر 24-11 — patch-2026.08.24-11

> **تطبَّق فوق 24-10.** ثلاثة ملفات: `app.py` · `vault.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `cc254d7a6be886f4ad02b74a`.

## 🔐 «شهادة موقّعة ذاتيا» — آخر عقبة قبل الجرد

`SSLCertVerificationError: certificate verify failed: self-signed certificate`

**والمعنى تقدّم لا تراجع:** المستمع مشفّر فعلا على 5986، **والاتصال يصل**، ولم يبق إلا أن شهادة الخادم موقّعة ذاتيا فرفضها التحقّق.

**والوسيطة موجودة في القارئ أصلا** (`KAMC_WIN_INSECURE`) ولا يعرفها من يقرأ نص الخطأ. فصارت الرسالة **تقول العلاج حرفا**.

## ✅ الحل — من الخزنة بلا رقعة ولا كود

`/repo/vault` ← «خوادم ويندوز» ← حقل **«إضافي»** أضف السطر:

```
KAMC_WIN_INSECURE=1
```

⚠️ **والتحقق يبقى مفعّلا افتراضا** — تخطّيه **قرار معلن لا سلوك صامت**، ويصحّ داخل شبكة المدينة على عنوان معروف. والأنظف على المدى الطويل: شهادة صادرة عن مرجع المدينة على الخادم.

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-11.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-11.tar.gz | cut -c1-24    # يجب cc254d7a6be886f4ad02b74a
./patch.sh patch-2026.08.24-11.tar.gz
```

ℹ️ **ويمكن تجربة الحل قبل تطبيق الرقعة** — الوسيطة تعمل منذ 24-9، والرقعة تحسّن الرسالة فقط.
