<a href="https://command-convert-ish.vercel.app/">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=150&section=header&text=termux-to-ish&fontSize=30&animation=fadeIn" width="100%">
</a>

مبدل دستور ترموکس به iSH

**فارسی** · [English](README.en.md)

1. فایل `index.html` را در مرورگر باز کنید (آنلاین یا آفلاین؛ فقط فونت وزیرمتن از گوگل بارگذاری می‌شود و در نبود اینترنت فونت جایگزین استفاده می‌شود). زبان پیش‌فرض فارسی است و با دکمه گوشه صفحه (`English`) می‌توانید به انگلیسی بروید؛ انتخاب شما در مرورگر ذخیره می‌شود.
<a href="https://command-convert-ish.vercel.app/">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=22&pause=1000&color=22C55E&center=true&vCenter=true&width=780&lines=Convert+Termux+commands;to+iSH+seamlessly" alt="Typing SVG">
</a>
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
مجوز

iSH تحت **GPLv3** (با شروط تکمیلی `LICENSE.IOS`) منتشر شده است. این ابزار فقط به مستندات و رفتار iSH ارجاع می‌دهد و کدی از آن را کپی نمی‌کند.
