<a href="https://command-convert-ish.vercel.app/">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=150&section=header&text=termux-to-ish&fontSize=30&animation=fadeIn" width="100%">
</a>

مبدل دستور ترموکس به iSH

**فارسی** · [English](README.en.md)

<a href="https://command-convert-ish.vercel.app/">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=22&pause=1000&color=22C55E&center=true&vCenter=true&width=780&lines=Convert+Termux+commands;to+iSH+seamlessly" alt="Typing SVG">
</a>
یک صفحه HTML تک‌فایلی و فارسی (راست‌به‌چپ) که دستورهای مخصوص **ترموکس** (Termux در اندروید) را به دستورهای سازگار با **iSH** (پوسته لینوکس روی آیفون) تبدیل می‌کند. همه‌چیز در مرورگر و بدون سرور اجرا می‌شود؛ هیچ داده‌ای ارسال نمی‌شود.

---

## ۱. iSH چیست و چرا دستورها یکسان نیستند؟

بر اساس سورس رسمی iSH (فایل `README.md` و کدهای `app/` و `kernel/`):

- iSH با **شبیه‌سازی پردازنده x86 در سطح کاربر** و **ترجمه فراخوانی‌های سیستمی لینوکس** یک پوسته لینوکس روی iOS اجرا می‌کند. یعنی پردازنده شبیه‌سازی‌شده **۳۲بیتی (i386/i686)** است، نه arm64 و نه amd64.
- سیستم‌فایل پیش‌فرض **Alpine Linux** است. مخزن‌های apk در سورس (`app/gen_apk_repositories.py` و `app/CurrentRoot.h`) روی **Alpine v3.19** و معماری **x86** تنظیم شده‌اند و از آینه‌ی `apk.ish.app` به‌صورت یک نسخه ثابت (snapshot) خوانده می‌شوند.
- فرمان اجرای پیش‌فرض برنامه `/bin/login -f root` است (`app/UserPreferences.m`)؛ پس شما **root** هستید و `sudo` لازم نیست. متغیر `TERM` برابر `xterm-256color` است.
- پوسته‌ی پیش‌فرض Alpine، `ash` (از busybox) است، نه `bash`.

ترموکس برعکس، روی اندروید با پردازنده ARM، مدیر بسته `pkg` (مبتنی بر `apt`) و مسیرهای `/data/data/com.termux/files/...` کار می‌کند. به همین دلیل بسیاری از دستورها بدون تبدیل در iSH اجرا نمی‌شوند.

## ۲. این ابزار چه کارهایی انجام می‌دهد؟

### ۲.۱ تبدیل مدیر بسته

| ترموکس | iSH |
|---|---|
| `pkg install` / `apt install` / `apt-get install` | `apk add` |
| `pkg remove` / `uninstall` / `purge` | `apk del` |
| `pkg update` | `apk update` |
| `pkg upgrade` / `full-upgrade` | `apk upgrade` |
| `pkg search` | `apk search` |
| `pkg list-installed` | `apk info` |
| `pkg show` | `apk info -a` |
| `pkg files` | `apk info -L` |
| `pkg reinstall` | `apk fix --reinstall` |
| `pkg clean` | `apk cache clean` |
| `pkg autoremove` | حذف می‌شود (apk نیازی ندارد) |

گزینه‌هایی مانند `-y`، `--yes` و `--no-install-recommends` حذف می‌شوند چون `apk` پرسش تأیید ندارد.

### ۲.۲ نگاشت نام بسته

نام بسته‌ها در ترموکس و آلپاین گاهی فرق دارد. نمونه‌ها:

| ترموکس | آلپاین |
|---|---|
| `python` | `python3` |
| `python-pip` | `py3-pip` |
| `nodejs` / `nodejs-lts` | `nodejs npm` |
| `golang` | `go` |
| `rust` | `rust cargo` |
| `build-essential` | `build-base` |
| `openssl-tool` | `openssl` |
| `libffi`، `libxml2`، `zlib` … | `libffi-dev`، `libxml2-dev`، `zlib-dev` … |
| `dnsutils` | `bind-tools` |
| `netcat` | `netcat-openbsd` |
| `pkg-config` | `pkgconf` |
| `python-numpy` | `py3-numpy` |

فهرست کامل در متغیر `PKG` داخل کد است. **بسته‌ای که در جدول نباشد با همان نام می‌ماند** و ابزار هشدار می‌دهد تا با `apk search نام` آن را بررسی کنید.

بسته‌های مخصوص ترموکس (مثل `termux-api`، `termux-tools`، `root-repo`، `x11-repo`، `proot-distro`، `tsu`) حذف می‌شوند و هشدار نمایش داده می‌شود (متغیر `DROP`).

### ۲.۳ تبدیل `bash <(curl …)`

جایگزینی فرایند (`<( )`) در `ash` کار نمی‌کند. ابزار این الگو را به «دانلود فایل و اجرای آن» تبدیل می‌کند:

```sh
# ورودی
bash <(curl -fsSL https://example.com/install.sh)
# خروجی
curl -fsSL https://example.com/install.sh -o install.sh && chmod +x install.sh && bash install.sh
```

- مفسر اصلی حفظ می‌شود: `bash`، `sh` و `zsh` (برای zsh هشدار نصب داده می‌شود).
- برای `source <(curl …)` و `. <(curl …)` فایل با `. ./file` در همان پوسته بارگذاری می‌شود تا تعریف توابع و متغیرها از بین نرود.
- آرگومان‌های بعد از دستور (مثل `--install`) و هدرهای `-H`/`-A` حفظ می‌شوند.
- آدرس‌هایی که کاراکتر ویژه دارند (`&`، `?` …) خودکار داخل کوتیشن قرار می‌گیرند.
- برای `curl … | bash` نیازی به تبدیل نیست، اما `bash` باید نصب باشد (بخش ۲.۷).

### ۲.۴ مسیرها و متغیرها

| ترموکس | iSH |
|---|---|
| `/data/data/com.termux/files/usr` و `$PREFIX` | `/usr` |
| `/data/data/com.termux/files/home` | `/root` |
| `$TMPDIR` | `/tmp` |
| `~/storage/shared`، `/sdcard`، `/storage/emulated/0` | `/mnt/files` (پس از mount) |

### ۲.۵ دستورهای `termux-*` که معادل دارند

سورس iSH دو دستگاه ویژه می‌سازد (`app/AppDelegate.m`، `fs/devices.h`):

| ترموکس | iSH | توضیح |
|---|---|---|
| `termux-clipboard-get` | `cat /dev/clipboard` | خواندن کلیپ‌بورد iOS |
| `termux-clipboard-set متن` | `printf %s متن > /dev/clipboard` | نوشتن در کلیپ‌بورد |
| `… \| termux-clipboard-set` | `… > /dev/clipboard` | |
| `termux-location` | `cat /dev/location` | خروجی به شکل `+عرض,+طول` (نه JSON)؛ مجوز موقعیت لازم است |
| `termux-wake-lock` | `cat /dev/location > /dev/null &` | روش شناخته‌شده برای زنده نگه داشتن برنامه در پس‌زمینه |
| `termux-wake-unlock` | `pkill -f 'cat /dev/location'` | |
| `termux-setup-storage` | `mkdir -p /mnt/files && mount -t ios . /mnt/files` | دسترسی به فایل‌های آیفون |
| `termux-info` | `cat /proc/ish/version` | |

نکته‌ها از روی کد:

- کلیپ‌بورد: اگر محتوای کلیپ‌بورد وسط خواندن عوض شود، خواندن با خطا قطع می‌شود؛ حداکثر اندازه بافر ۸ مگابایت است (`app/PasteboardDevice.m`).
- موقعیت: iSH در سورس `allowsBackgroundLocationUpdates` را فعال کرده است؛ به همین دلیل خواندن `/dev/location` برنامه را در پس‌زمینه بیدار نگه می‌دارد، ولی باتری مصرف می‌کند.

`termux-fix-shebang` و `termux-reload-settings` حذف می‌شوند چون در iSH لازم نیستند.

### ۲.۶ دستورهایی که قابل تبدیل نیستند (فقط هشدار)

این موارد بدون تغییر می‌مانند و هشدار می‌گیرند: بقیه‌ی `termux-*` (مثل `termux-notification`، `termux-vibrate`، `termux-battery-status`، `termux-open-url`، `termux-sms-send`)، همچنین `proot`، `proot-distro`، `tsu`، `sv`، `am`، `pm`، `getprop`، `dumpsys`، `logcat` و `settings`. دلیل: این‌ها API یا ابزار اندروید هستند و در iSH معادلی ندارند.

### ۲.۷ گزینه‌ها

- **اضافه کردن نصب پیش‌نیازها:** در ابتدای خروجی `apk update && apk add curl bash …` می‌گذارد. ابزارهای دیگر (مثل `git`، `python3`، `py3-pip`، `nodejs`، `openssl`، `jq`) فقط وقتی اضافه می‌شوند که واقعاً در جایگاه دستور استفاده شده باشند و خودِ دستور آن‌ها را نصب نکرده باشد.
- **`--break-system-packages` برای pip:** آلپاین ۳٫۱۹ نصب سراسری با pip را می‌بندد (خطای `externally-managed-environment`). با این گزینه، پرچم به `pip install` اضافه می‌شود. راه امن‌تر: venv یا نصب بسته با `apk add py3-نام`.

### ۲.۸ هشدارهای معماری

اگر در دستور نام‌هایی مثل `arm64`، `aarch64`، `armv7`، `amd64` یا `x86_64` دیده شود، هشدار می‌دهد که iSH فقط باینری ۳۲بیتی x86 (`386`/`i386`/`i686`) اجرا می‌کند.

### ۲.۹ رفتار هوشمند روی دستورهای پیچیده

- زنجیره‌ها (`&&`، `||`، `;`) تکه‌تکه تبدیل می‌شوند؛ عملگرهای داخل کوتیشن، پرانتز و کامنت دست‌نخورده می‌مانند.
- خط‌های ادامه‌دار با `\` به هم وصل می‌شوند.
- متن داخل heredoc (`<<EOF … EOF`) تبدیل **نمی‌شود**، فقط مسیرها و متغیرهای ترموکس در آن اصلاح می‌شوند.
- خطوط کامنت و خالی دست‌نخورده می‌مانند.

## ۳. راهنمای استفاده

1. فایل `index.html` را در مرورگر باز کنید (آنلاین یا آفلاین؛ فقط فونت وزیرمتن از گوگل بارگذاری می‌شود و در نبود اینترنت فونت جایگزین استفاده می‌شود). زبان پیش‌فرض فارسی است و با دکمه گوشه صفحه (`English`) می‌توانید به انگلیسی بروید؛ انتخاب شما در مرورگر ذخیره می‌شود.
2. دستور ترموکس را در کادر بالا بچسبانید؛ خروجی هم‌زمان ساخته می‌شود.
3. هشدارها (زرد) را بخوانید؛ بخش‌هایی که تبدیل نشده‌اند آنجا توضیح داده می‌شوند.
4. «کپی دستور» را بزنید و در iSH بچسبانید.
5. دکمه «نمونه» چند مثال آماده را یکی‌یکی نشان می‌دهد.

### نمونه‌ها

```sh
# ورودی
pkg update && pkg upgrade -y && pkg install -y python git openssh nodejs-lts
# خروجی
apk update && apk upgrade && apk add python3 git openssh nodejs npm
```

```sh
# ورودی
termux-setup-storage && cp ~/storage/shared/Download/a.txt ~/ && echo "ok" | termux-clipboard-set
# خروجی
mkdir -p /mnt/files && mount -t ios . /mnt/files && cp /mnt/files/Download/a.txt ~/ && echo "ok" > /dev/clipboard
```

## ۴. دسترسی به فایل‌های آیفون در iSH

با `mount -t ios . /mnt/files` (پس از `mkdir -p /mnt/files`) یک پنجره انتخاب پوشه در iOS باز می‌شود و پوشه‌ی انتخابی در `/mnt/files` نمایش داده می‌شود. در سورس (`app/iOSFS.m`) دو نوع سیستم‌فایل ثبت شده است: `ios` و `ios-unsafe`. نوع دوم به‌جای عملیات امن‌سازی‌شده‌ی iSH، عملیات خام فایل‌سیستم را به‌کار می‌برد؛ مگر در شرایط خاص همان `ios` را استفاده کنید.

## ۵. محدودیت‌ها و نکات مهم

- **مخزن‌های ثابت:** فایل `/etc/apk/repositories` را خود iSH مدیریت می‌کند و اگر پوشه `/ish` وجود داشته باشد هنگام اجرا بازنویسی می‌شود (متن کامنت داخل خود فایل همین را می‌گوید). تغییر دستی آدرس مخزن پایدار نیست. بسته‌ها هم نسخه‌ی snapshot هستند، پس `apk upgrade` معمولاً بسته‌ی جدیدی نمی‌آورد.
- **شبیه‌سازی کند است:** برنامه‌های سنگین (کامپایل، Node.js، Rust، Go) ممکن است کند باشند یا مشکل داشته باشند؛ این ابزار درباره‌ی پایداری آن‌ها تضمینی نمی‌دهد.
- **باینری ۳۲بیتی:** هر فایل دانلودی باید برای x86 ۳۲بیتی ساخته شده باشد.
- **فراخوانی‌های سیستمی ناقص:** بعضی فراخوانی‌ها (مثل `inotify`) در سورس فقط «stub» هستند؛ برنامه‌هایی که به آن‌ها وابسته‌اند ممکن است درست کار نکنند.
- **بدون API اندروید:** هیچ معادلی برای نوتیفیکیشن، پیامک، دوربین، لرزش و … وجود ندارد.
- **اجرای خودکار در شروع:** پوسته‌ی پیش‌فرض ورود `ash` است، پس `~/.bashrc` خوانده نمی‌شود مگر `bash` را دستی اجرا کنید. برای `ash` از `~/.profile` استفاده کنید. 
- **اعتبار تبدیل:** ابزار بر پایه‌ی قاعده (regex) کار می‌کند؛ قبل از اجرای دستورهای حساس (حذف فایل، دانلود و اجرای اسکریپت) خروجی را مرور کنید.

## ۶. ساختار کد

همه‌چیز در یک فایل است:

| بخش | کار |
|---|---|
| `MSG` / `UI` | متن‌های فارسی و انگلیسی پیام‌ها و رابط کاربری |
| `PKG` / `DROP` | جدول نگاشت و فهرست بسته‌های حذفی |
| `splitOps` | شکستن خط به بخش‌ها بر اساس `&&`، `\|\|`، `;` با رعایت کوتیشن و پرانتز |
| `convPkg` | تبدیل `pkg`/`apt` به `apk` |
| `convTermux` | دستورهای `termux-*` |
| `convFetch` | تبدیل `bash <(curl …)` |
| `convSeg` | ترکیب قواعد بالا برای هر بخش (حذف `sudo`، pip، هشدار دستورهای اندروید) |
| `convertAll` | مسیرها، خط‌به‌خط، heredoc، پیش‌نیازها؛ خروجی `{out, warns, infos}` (هشدارها کلید پیام + آرگومان‌اند) |
| `paint` / `run` / `setLang` | نمایش به زبان انتخابی و اتصال به رابط کاربری |

### افزودن بسته جدید

یک خط به `PKG` اضافه کنید:

```js
'نام-در-ترموکس':['نام-در-آلپاین'],
```

درستی نام را با `apk search` در خود iSH بررسی کنید.

### افزودن پیام یا زبان

هشدارها و توضیحات با کلید (`k0`، `k1`، …) و جای‌گذار `{0}`، `{1}` ذخیره می‌شوند. برای زبان جدید، یک ورودی به `MSG` و `UI` و یک شاخه در `setLang` اضافه کنید.

### تست

منطق تبدیل بدون DOM تست‌پذیر است. نمونه‌ی تست با Node: تابع `convertAll(متن, {pre:false, brk:false})` را فراخوانی کنید و `out`، `warns`، `infos` را بررسی کنید.

## ۷. منابع (بر اساس سورس iSH)

| موضوع | فایل در `ish-master` |
|---|---|
| معرفی و ساخت پروژه | `README.md` |
| مخزن‌های apk و نسخه‌ی Alpine | `app/gen_apk_repositories.py`، `app/CurrentRoot.h`، `app/CurrentRoot.m` |
| دستگاه‌های `/dev/clipboard` و `/dev/location` | `app/AppDelegate.m`، `app/PasteboardDevice.m`، `app/LocationDevice.m`، `fs/devices.h` |
| mount پوشه‌های iOS | `app/iOSFS.m` |
| فرمان ورود و `TERM` | `app/UserPreferences.m`، `app/AppDelegate.m` |
| `/proc/ish` | `fs/proc/ish.c` |
| فراخوانی‌های سیستمی | `kernel/calls.c` |

## ۸. مجوز

iSH تحت **GPLv3** (با شروط تکمیلی `LICENSE.IOS`) منتشر شده است. این ابزار فقط به مستندات و رفتار iSH ارجاع می‌دهد و کدی از آن را کپی نمی‌کند.
