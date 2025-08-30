# 📘 پیش‌نیازهای Exchange Server 2019 و SE

این README شامل پیش‌نیازهای سیستم برای نصب **Exchange Server 2019** و **Subscription Edition (SE)** است، شامل نقش‌های 📦 Mailbox و Edge Transport و همچنین ابزارهای مدیریت روی کلاینت‌ها.

---

## ⚠️ نکات مهم قبل از نصب

- 🏢 **Active Directory**: مطمئن شوید که AD شما با Exchange 2019 و SE سازگار است.
- 💻 **سیستم‌عامل پشتیبانی‌شده**: فقط از نسخه‌های پشتیبانی‌شده Windows Server استفاده کنید.
- 🔄 **به‌روزرسانی‌های ویندوز**: آخرین آپدیت‌ها باید نصب شده باشند.
- 🛠️ **سرویس Remote Registry**: روی `Automatic` تنظیم شود و غیرفعال نباشد.

---

## 📋 پیش‌نیازهای Windows Server برای Exchange Server

### 🗂️ آماده‌سازی Active Directory

**نرم‌افزارهای موردنیاز:**
- ✔️ نسخه پشتیبانی‌شده .NET Framework
- ✔️ Visual C++ Redistributable برای Visual Studio 2012

**نصب ابزارهای RSAT:**
```powershell
Install-WindowsFeature RSAT-ADDS
```

---

### 🖥️ ابزارهای مدیریت Exchange

**نرم‌افزارهای موردنیاز:**
- ✔️ نسخه پشتیبانی‌شده .NET Framework
- ✔️ Visual C++ Redistributable برای Visual Studio 2012

**نصب ویژگی‌های ویندوز:**
- برای سیستم‌عامل سرور:
```powershell
Install-WindowsFeature -Name Web-Mgmt-Console, Web-Metabase
```
- برای سیستم‌عامل کلاینت:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole
```

---

## ⚡ دستورات آماده‌سازی Active Directory

⚠️ **نکات مهم:**
- این دستورات باید با کاربری اجرا شوند که دسترسی‌های **Domain Admin** و **Schema Admin** داشته باشد.
- قبل از اجرا، از Active Directory بکاپ بگیرید.
- این مراحل باید **قبل از نصب Exchange Server** انجام شوند.

🔹 **دستورات موردنیاز:**

1. 📌 آماده‌سازی Schema (یک‌بار در هر Forest):
   ```bash
   Setup.exe /PrepareSchema /IAcceptExchangeServerLicenseTerms_DiagnosticDataOFF
   ```

2. 📌 آماده‌سازی Active Directory و تنظیم نام سازمان:
   ```bash
   Setup.exe /PrepareAD /OrganizationName:TrendParand /IAcceptExchangeServerLicenseTerms_DiagnosticDataOFF
   ```

3. 📌 آماده‌سازی Domain جاری:
   ```bash
   Setup.exe /PrepareDomain /IAcceptExchangeServerLicenseTerms_DiagnosticDataOFF
   ```

📌 **یادداشت‌ها:**
- این دستورات باید در پوشه‌ای اجرا شوند که شامل فایل `Setup.exe` است.
- بعد از اجرای موفقیت‌آمیز، می‌توانید نصب اصلی Exchange Server را ادامه دهید.
- در محیط‌های چنددامینی، دستور `/PrepareDomain` باید برای هر Domain جداگانه اجرا شود.

✅ پس از این مراحل، محیط Active Directory آماده نصب Exchange Server خواهد بود.
