
# 📧 Microsoft Exchange Server Setup & Configuration Guide

این مستند شامل مراحل نصب، پیش‌نیازها، اتصال به Active Directory، تنظیم DNS، SSL، و رفع مشکلات Microsoft Exchange Server است.

---

## 🛠 پیش‌نیازهای ویندوز

قبل از نصب Exchange، ویژگی‌های زیر باید نصب شوند:

- Server-Media-Foundation: پشتیبانی از رسانه‌ها و کدک‌ها
- RSAT-ADDS: ابزارهای مدیریت Active Directory
- RSAT-ADDS-Tools: ابزارهای اضافی برای مدیریت AD
- RSAT-AD-AdminCenter: مرکز مدیریت Active Directory
- RSAT-DNS-Server: مدیریت سرور DNS
- RSAT-DFS-Mgmt-Con: مدیریت DFS
- RSAT-File-Services: مدیریت خدمات فایل
- RSAT-Clustering: مدیریت خوشه‌ها
- Web-Server: نصب IIS
- Web-Mgmt-Tools: ابزارهای مدیریت IIS
- Web-Mgmt-Console: کنسول مدیریت IIS
- Web-Mgmt-Service: سرویس مدیریت IIS
- Web-Basic-Auth: احراز هویت پایه
- Web-Windows-Auth: احراز هویت ویندوز
- Web-Client-Auth: احراز هویت مشتری
- Web-Digest-Auth: احراز هویت هضم شده
- Web-Dyn-Compression: فشرده‌سازی دینامیک
- Web-Filtering: فیلتر درخواست‌ها
- Web-ISAPI-Ext: پشتیبانی ISAPI Extensions
- Web-ISAPI-Filter: فیلترهای ISAPI
- Web-Http-Logging: ثبت لاگ HTTP
- Web-Request-Monitor: نظارت بر درخواست‌ها
- Web-Http-Tracing: ردیابی HTTP
- Web-Stat-Compression: فشرده‌سازی استاتیک
- Web-Default-Doc: پشتیبانی اسناد پیش‌فرض
- Web-Dir-Browsing: مرور دایرکتوری
- Web-Http-Errors: مدیریت خطاهای HTTP
- Web-Http-Redirect: ریدایرکت HTTP
- Web-Http-Response: مدیریت پاسخ‌ها
- Web-Asp-Net45: پشتیبانی ASP.NET 4.5
- Web-Net-Ext45: پشتیبانی .NET Framework 4.5
- Web-Static-Content: محتوای استاتیک
- Web-Performance: بهینه‌سازی عملکرد
- Web-Security: امنیت IIS
- Web-Cert-Auth: احراز هویت با گواهی‌نامه
- Web-Mgmt-Compat: سازگاری با ابزارهای قدیمی
- Web-Lgcy-Mgmt-Console: کنسول مدیریت قدیمی
- NET-Framework-45-Core: نصب .NET Framework 4.5
- NET-Framework-45-ASPNET: نصب ASP.NET
- NET-WCF-HTTP-Activation45: فعال‌سازی WCF HTTP
- Windows-Identity-Foundation: بنیاد هویت ویندوز

نصب با پاورشل:
```powershell
Install-WindowsFeature Server-Media-Foundation, RSAT-ADDS, RSAT-ADDS-Tools, RSAT-AD-AdminCenter, RSAT-DNS-Server, RSAT-DFS-Mgmt-Con, RSAT-File-Services, RSAT-Clustering, Web-Server, Web-Mgmt-Tools, Web-Mgmt-Console, Web-Mgmt-Service, Web-Basic-Auth, Web-Windows-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dyn-Compression, Web-Filtering, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Http-Logging, Web-Request-Monitor, Web-Http-Tracing, Web-Stat-Compression, Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Http-Redirect, Web-Http-Response, Web-Asp-Net45, Web-Net-Ext45, Web-Static-Content, Web-Performance, Web-Security, Web-Cert-Auth, Web-Mgmt-Compat, Web-Lgcy-Mgmt-Console, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, Windows-Identity-Foundation
```

---

## 🌐 اتصال سرور به Active Directory

1. در **Server Manager → Local Server → Workgroup** به دامنه وصل شوید.
2. سیستم را ریستارت کنید.
3. با یوزر دامنه (مثلاً `asghar`) وارد شوید.

---

## 💽 نصب Exchange Server

1. پیش‌نیازها را نصب کنید.
2. Setup Exchange را اجرا کنید.
3. Mailbox Role را انتخاب کنید.
4. مسیر نصب را مشخص کنید.
5. نصب را تکمیل کنید.

---

## 📜 تنظیم DNS

| Record Type  | Name / Host                   | Points to                  |
|--------------|-------------------------------|----------------------------|
| A            | autodiscover.hesaba.net        | IP Exchange Server          |
| A            | email.hesaba.net               | IP Exchange Server          |
| A            | admin.email.hesaba.net         | IP Exchange Server          |

---

## 📦 فعال‌سازی Mailbox برای کاربران OU

```powershell
$OU = "OU=Users,DC=hesaba,DC=net"
Get-User -OrganizationalUnit $OU | Where-Object {$_.RecipientType -eq "User"} | Enable-Mailbox -Database "Mailbox Database 0450261961"
```

برای اضافه‌شدن خودکار کاربران جدید، می‌توان اسکریپت زمان‌بندی‌شده ایجاد کرد.

---

## 🛡 SSL و گواهی

### گواهی Self-Signed
```powershell
New-ExchangeCertificate -FriendlyName "HesabaCert" -DomainName email.hesaba.net,admin.email.hesaba.net,autodiscover.hesaba.net -Services IIS
Enable-ExchangeCertificate -Thumbprint <Thumbprint> -Services IIS
iisreset
```

بعداً می‌توان گواهی واقعی اضافه کرد.

---

## ⚙️ IIS Authentication

- **owa** و **ecp**:  
  - Windows Authentication → Enable  
  - Forms Authentication → Enable  

---

## 🚀 URLهای سفارشی

- Webmail: `https://email.hesaba.net`
- Admin Panel: `https://admin.email.hesaba.net`

برای حذف `/owa` و `/ecp` از **IIS ARR** یا **Nginx** استفاده کنید.

---

## 🧪 تست دسترسی

```powershell
Test-NetConnection email.hesaba.net -Port 443
```

---

## 🐞 رفع مشکلات

- Access Denied → گواهی در Trusted Root نصب شود + IIS Authentication بررسی شود
- Database Not Found → نام دیتابیس صحیح وارد شود
- Insufficient Access Rights → یوزر باید Organization Management داشته باشد

---

## 📚 منابع

- [Microsoft Exchange Docs](https://learn.microsoft.com/en-us/exchange/)
- [IIS ARR Docs](https://learn.microsoft.com/en-us/iis/extensions/planning-for-arr/)
