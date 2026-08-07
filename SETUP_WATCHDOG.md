# 🐕 تركيبُ مؤقّت المِرصاد (مرّةً واحدة على خادم المدينة)

> `watchdog.sh` يُشحن مع الرقعة، لكنّ **مؤقّتَه لا يُنشأ تلقائيًّا**. هذا يركّبه
> مرّةً واحدةً كخدمةِ مستخدمٍ (`--user`، بلا sudo)، فيفحص المنصّةَ كلَّ دقيقة
> ويُنعشها لو تجمّدت. **مهمّةُ كلود جهاز العمل** عبر SSH إلى `kamc@10.11.72.145`.

## الخطوات (على الخادم، مستخدم kamc)
1. انسخِ الوحدتين لمجلّد خدمات المستخدم:
   ```
   mkdir -p ~/.config/systemd/user
   scp kamc-watchdog.service kamc-watchdog.timer kamc@10.11.72.145:~/.config/systemd/user/
   ```
2. فعّلها (على الخادم):
   ```
   ssh kamc@10.11.72.145 'systemctl --user daemon-reload && systemctl --user enable --now kamc-watchdog.timer'
   ```
3. **اضمن بقاءها بلا جلسة** (مهمّ لخادمٍ لا أحدَ مسجَّلٌ فيه دائمًا):
   ```
   ssh kamc@10.11.72.145 'loginctl enable-linger kamc' 2>/dev/null || true
   ```
   (قد تحتاج مسؤولَ النظام لو رفضها — الخدمةُ الأساسيّةُ kamc تعمل أصلًا فاللينجر غالبًا مُفعّل.)

## تحقّق
```
ssh kamc@10.11.72.145 'systemctl --user list-timers kamc-watchdog.timer --no-pager'
```
يجب أن يظهر المؤقّتُ بموعدِ تشغيلٍ قادمٍ (NEXT) خلال دقيقة. وبعد دقيقتين:
```
ssh kamc@10.11.72.145 'systemctl --user status kamc-watchdog.service --no-pager | tail -5; cat ~/kamc_ea/data/watchdog.log 2>/dev/null | tail'
```

## تراجع (لو لزم)
```
ssh kamc@10.11.72.145 'systemctl --user disable --now kamc-watchdog.timer'
```
