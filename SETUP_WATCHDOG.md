# 🐕 تركيبُ مؤقّت المِرصاد (مرّةً واحدة على خادم المدينة)

> 🔒 **عنوانُ الخادم واسمُ مستخدمِه لا يُكتبان هنا** — هذا مستودعٌ عامّ.
> صدّرهما في جلستك قبل الأوامر أدناه:
> `export KAMC_HOST=<user>@<ip>`


> `watchdog.sh` يُشحن مع الرقعة، لكنّ **مؤقّتَه لا يُنشأ تلقائيًّا**. هذا يركّبه
> مرّةً واحدةً كخدمةِ مستخدمٍ (`--user`، بلا sudo)، فيفحص المنصّةَ كلَّ دقيقة
> ويُنعشها لو تجمّدت. **مهمّةُ كلود جهاز العمل** عبر SSH إلى `$KAMC_HOST`.

## الخطوات (على الخادم، مستخدم kamc)
1. انسخِ الوحدتين لمجلّد خدمات المستخدم:
   ```
   mkdir -p ~/.config/systemd/user
   scp kamc-watchdog.service kamc-watchdog.timer $KAMC_HOST:~/.config/systemd/user/
   ```
2. فعّلها (على الخادم):
   ```
   ssh $KAMC_HOST 'systemctl --user daemon-reload && systemctl --user enable --now kamc-watchdog.timer'
   ```
3. **اضمن بقاءها بلا جلسة** (مهمّ لخادمٍ لا أحدَ مسجَّلٌ فيه دائمًا):
   ```
   ssh $KAMC_HOST 'loginctl enable-linger kamc' 2>/dev/null || true
   ```
   (قد تحتاج مسؤولَ النظام لو رفضها — الخدمةُ الأساسيّةُ kamc تعمل أصلًا فاللينجر غالبًا مُفعّل.)

## تحقّق
```
ssh $KAMC_HOST 'systemctl --user list-timers kamc-watchdog.timer --no-pager'
```
يجب أن يظهر المؤقّتُ بموعدِ تشغيلٍ قادمٍ (NEXT) خلال دقيقة. وبعد دقيقتين:
```
ssh $KAMC_HOST 'systemctl --user status kamc-watchdog.service --no-pager | tail -5; cat ~/kamc_ea/data/watchdog.log 2>/dev/null | tail'
```

## تراجع (لو لزم)
```
ssh $KAMC_HOST 'systemctl --user disable --now kamc-watchdog.timer'
```
