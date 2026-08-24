# نشر 24-9 — patch-2026.08.24-9

> **تطبَّق فوق 24-8.** أربعة ملفات: `vault.py` · `win_inventory.py` · `nutanix_inventory.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `09bc3794bf6b31273b4d15bb`.

## 🔌 المنفذ من الخزنة — لا من افتراض في الكود

**عتب المالك:** «ليش تكتبه؟ أنا وضعته في الخزنة — اقرأ بيانات الاتصال في الخزنة واستعملها فقط».

⚠️ **والعطب كان أدق مما يبدو:** `env_for` تعيد **المضيف والحساب والكلمة** و**تُسقط المنفذ**. فالخزنة تعرض حقل المنفذ ويملؤه المالك، **ثم لا يصل القارئ فيستعمل افتراضا مكتوبا في الكود**. ومن أدخل `5986` ظنّ أنه يتصل مشفّرا **وهو يتصل بـ5985**.

**ما تغيّر:**
- `env_for` تعيد **المنفذ** لكل نطاق (`KAMC_VC_PORT` · `KAMC_NTX_PORT` · `KAMC_AD_PORT` · `KAMC_WIN_PORT`).
- **وحقل «إضافي» صار قناة وسائط**: يُكتب فيه `KAMC_WIN_SSL=1` أو `KAMC_WIN_TRANSPORT=kerberos` (مفصولة بفواصل أو أسطر) فتصل القارئ — **فما لا حقل له في الشاشة له طريق**.
- ويندوز ونيوتانكس يأخذان المنفذ **من الخزنة أولا**، والافتراض القياسي **آخر الملاذ** لا أوله.

**مقاس بالتشغيل:** `5985` · `5986` مع SSL · `47001` مع kerberos — **الثلاثة وصلت القارئ كما أُدخلت**.

ℹ️ **وقاعدة قائمة في الخزنة يحسن معرفتها:** الحقل الفارغ عند الحفظ يعني **«لا تغيّر»** لا «امسح» — فلا تفقد كلمة سر بحفظ نموذج فارغ.

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-9.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-9.tar.gz | cut -c1-24    # قارنه بالبصمة أعلاه
./patch.sh patch-2026.08.24-9.tar.gz
```
