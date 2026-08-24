# نشر 24-8 — patch-2026.08.24-8

> **تطبَّق فوق 24-7.** ثلاثة ملفات: `vault.py` · `app.py` · `BUILD.json`.
> بصمة الرقعة **داخل** التعمية: `373dce961dc055c28b1f029e`.

## 🩺 أول سحب حقيقي ردّ AccessDenied — والعطب في الصلاحية لا في الشبكة

نصّ الردّ: `WSManFaultError: Access is denied · wsmanfault_code 5 · HTTP 500`.
**ومعناه دقيق:** المنفذ 5985 مفتوح، والخادم استقبل الطلب وردّ عليه، **ثم رفض الحساب**. أي أن الشبكة والخدمة سليمتان، والمفقود **تفويض الحساب بالإدارة عن بعد**.

**ما تغيّر في الرقعة:**
- ⚠️ **اختبار الخزنة كان يفحص المنفذ وحده فيقول «سليم» ويكذب.** صار اختبار نطاق `win` يجري **نداء PowerShell حقيقيا** (`$env:COMPUTERNAME`) ويعيد **نوع العطب وعلاجه** — ولا يعيد مخرَجا ولا اسما ولا مسارا.
- شاشة الجرد صارت **تترجم AccessDenied إلى الخطوة المطلوبة** بدل عرض نصّ WSMan الخام.

## 🔧 العلاج — على الخادم الهدف (KAMCWEBEXT01) بصلاحية مسؤول

```powershell
# ① الخدمة والمنفذ (تأكيد)
winrm quickconfig
Get-Service WinRM

# ② عضوية الإدارة عن بعد — وهي الأرجح سببا
Add-LocalGroupMember -Group "Remote Management Users" -Member "KAMC\<الحساب>"

# ③ صلاحية على إعداد جلسة PowerShell
Set-PSSessionConfiguration -Name Microsoft.PowerShell -ShowSecurityDescriptorUI
#   امنح الحساب: Read + Execute(Invoke)

# ④ صلاحية قراءة WMI على root\cimv2
#   wmimgmt.msc ← خصائص ← Security ← Root\CIMV2 ← Advanced ← Add
#   امنح: Enable Account + Remote Enable

# ⑤ لو الحساب محليّ لا نطاقيّ (تقييد UAC للحسابات المحلية)
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name LocalAccountTokenFilterPolicy -Value 1 -PropertyType DWord -Force
```

ثم من المنصّة: **`/repo/vault` ← نطاق «خوادم ويندوز» ← اختبار** — يجب أن يقول «ارتباط ناجح · وقراءة PowerShell تعمل». وبعدها **`/repo/win` ← «اسحب الآن وطبّق»**.

## التطبيق

```bash
cd ~/kamc_ea
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.tar.gz.enc -o latest.tar.gz.enc
curl -sL https://raw.githubusercontent.com/asmsot-boop/kamc-patch/main/latest.sha256 -o latest.sha256
sha256sum -c <(echo "$(cat latest.sha256)  latest.tar.gz.enc")   # يجب OK
openssl enc -d -aes-256-cbc -pbkdf2 -in latest.tar.gz.enc -out patch-2026.08.24-8.tar.gz -pass file:$HOME/.config/kamc/deploy_pass
sha256sum patch-2026.08.24-8.tar.gz | cut -c1-24    # يجب 373dce961dc055c28b1f029e
./patch.sh patch-2026.08.24-8.tar.gz
```

⚠️ **الرقعة تشخّص ولا تفتح صلاحية** — الخطوات أعلاه تُنفَّذ على الخادم الهدف بيد مسؤول ويندوز، **والمنصّة لا تملك ولا يجوز أن تملك** صلاحية تعديل تفويضات الخوادم.
