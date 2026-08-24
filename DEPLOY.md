# نشر 24-12 — patch-2026.08.24-12

> **تطبَّق فوق 24-11.** ثلاثة ملفات: `vault.py` · `app.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `32a430cd1ef781fb0b213582`.

## 🧩 حقل «إضافي» — كان في القاعدة ولا مدخل له في الشاشة

**بلاغ المالك:** «ما وجدت مكان الحقل الإضافي». ⚠️ **وهو محق: الحقل لم يكن في الشاشة أصلا.**

العمود `extra` موجود في جدول الخزنة، و`put()` يقبله، و`env_for` تقرؤه منذ 24-9 — **والنموذج لا يعرضه، والمعالج يمرر فراغا مكانه دائما**.

**ما تغيّر:**
- **حقل «إضافي — وسائط بصيغة مفتاح=قيمة»** ظاهر في نموذج الخزنة، بمثال في مكانه: `KAMC_WIN_INSECURE=1, KAMC_WIN_TRANSPORT=ntlm`.
- والمعالج صار **يمرر قيمته** بدل الفراغ.

## بعد التطبيق — الحل الذي انتظرناه

`/repo/vault` ← «خوادم ويندوز» ← اضغط ✏️ على البيانة ← في **«إضافي»** اكتب:

```
KAMC_WIN_INSECURE=1
```

← احفظ ← ثم `/repo/win` ← **«اسحب الآن وطبّق»**.

**مقاس بالتشغيل:** الحقل يظهر · يُحفظ من الشاشة · ويصل القارئ (`KAMC_WIN_INSECURE=1` في مخرَج `env_for`).

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-12.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-12.tar.gz | cut -c1-24    # يجب 32a430cd1ef781fb0b213582
./patch.sh patch-2026.08.24-12.tar.gz
```
