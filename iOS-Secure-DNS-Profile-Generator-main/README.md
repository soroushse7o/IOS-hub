# 🛡️ iOS Secure DNS (DoH / DoT) Profile Generator

[![Live Demo](https://img.shields.io/badge/Launch-iOS_DNS_Generator-34C759?style=for-the-badge&logo=apple&logoColor=white)](https://soroushse7o.github.io/iOS-Secure-DNS-Profile-Generator/)


[English](#english) | [فارسی](#فارسی)

---

<div dir="rtl">

## فارسی

یک ابزار تحت وب ساده، امن و کاملاً سمت کاربر (Client-Side) برای تولید پروفایل‌های پیکربندی DNS رمزنگاری‌شده (`.mobileconfig`) سازگار با **Cloudflare Gateway** و سرورهای سفارشی **DoH / DoT** در دستگاه‌های iOS و iPadOS (نسخه ۱۴ به بالا) و macOS.

### ✨ قابلیت‌ها

* **تولید ساختار استاندارد Apple / Cloudflare Gateway:** دارای قوانین خودکار فعال‌سازی (`OnDemandRules`) برای اتصال دائم روی شبکه دیتا (Cellular) و وای‌فای (WiFi).
* **پشتیبانی از هر دو پروتکل:** امکان تولید هم‌زمان یا مجزای پی‌لودهای **DNS-over-HTTPS (DoH)** و **DNS-over-TLS (DoT)**.
* **۱۰۰٪ سمت کاربر و امن:** تمام پروسه ساخت فایل روی مرورگر انجام شده و هیچ‌گونه آدرس یا داده‌ای به سرور فرستاده نمی‌شود.
* **دانلود مستقیم در Safari:** استفاده از شناسه MIME استاندارد اپل (`application/x-apple-aspen-config`) برای اجرای مستقیم پنجره نصب در سافاری.
* **سازگار با Cloudflare Pages و GitHub Pages:** پیاده‌سازی سبک، تک‌صفحه‌ای و بدون نیاز به بیلد یا وابستگی جانبی.

### 📲 راهنمای استفاده و نصب

1. با مرورگر **Safari** وارد آدرس وب‌سایت شوید.
2. آدرس سرور DoH یا دامنه DoT خود (مثلاً آدرس دریافتی از Cloudflare Zero Trust / Gateway) را وارد کنید.
3. روی دکمه **دانلود و ساخت پروفایل** بزنید و پیام **Allow** را در سافاری تایید کنید.
4. وارد تنظیمات دستگاه (**Settings**) شوید.
5. از بالای صفحه گزینه **Profile Downloaded** را انتخاب کرده و روی **Install** بزنید.

> ### 🛡️ [ورود به ابزار آنلاین iOS Secure DNS Profile Generator](https://soroushse7o.github.io/iOS-Secure-DNS-Profile-Generator/)
> برای ساخت و دریافت مستقیم پروفایل‌های امن DoH / DoT روی iOS، روی لینک بالا کلیک کنید.


</div>

---

## English

A lightweight, private, and fully client-side web utility to generate Apple Mobile Configuration profiles (`.mobileconfig`) for Encrypted DNS (**DNS-over-HTTPS** & **DNS-over-TLS**) compatible with **Cloudflare Gateway**, NextDNS, or custom DoH/DoT endpoints on iOS 14+, iPadOS, and macOS.

### ✨ Features

* **Apple & Cloudflare Gateway Compliant:** Includes native `OnDemandRules` for seamless automatic encryption across both Wi-Fi and Cellular interfaces.
* **Multi-Protocol Support:** Generate dual-payload profiles containing both **DoH** and **DoT**, or isolate either protocol as needed.
* **100% Client-Side Privacy:** All XML/Plist generation takes place directly inside your browser via JavaScript. No telemetry or query endpoints are logged or transmitted.
* **Direct Safari Install Flow:** Served using Apple's standard MIME type (`application/x-apple-aspen-config`) to trigger the native profile download prompt immediately.
* **Zero-Config Deployment:** Ready for instant static hosting on **GitHub Pages** and **Cloudflare Pages**.

### 📲 Installation Steps on iOS

1. Open the deployed website using **Safari** on your iOS device.
2. Enter your custom DoH URL and/or DoT Hostname.
3. Tap **دانلود و ساخت پروفایل / Generate Profile** and confirm with **Allow**.
4. Open the iOS **Settings** app.
5. Tap **Profile Downloaded** at the top of Settings and proceed with **Install**.

---

### 📄 License

MIT License. Free to use, modify, and distribute.
